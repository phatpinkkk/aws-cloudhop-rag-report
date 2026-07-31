# AWS RAG Project

A multi-hop Retrieval-Augmented Generation (RAG) question-answering system over
a HotpotQA corpus, built with a hybrid BM25 + dense-vector retriever, LLM-driven
query decomposition and adaptive hop planning, a cross-encoder reranker, and
Groq for final answer generation. The backend runs on Amazon EC2 behind API
Gateway, retrieval artifacts and vectors are served from Amazon S3 / Amazon S3
Vectors, and the frontend is a React app hosted on AWS Amplify.

## 1. Architecture

### 1.1 Two pipelines: offline ingestion vs. online query

The system is deliberately split so that all expensive work (chunking,
embedding, index building) happens **offline, once**, and the online service
only ever *loads* pre-built artifacts and answers queries.

```text
OFFLINE (run once per corpus/index version, from a notebook/script, not a request)
  corpus.jsonl (HotpotQA subset)
    -> chunk into parent docs (whole article) + child docs (sub-chunks)
    -> embed child docs with BAAI/bge-m3 (or bge-small-en-v1.5 for CPU-only demo)
    -> build BM25 index (bm25s) over child docs
    -> write index_manifest.json (records embedding model, chunk sizes, paths)
    -> upload everything to S3 (processed docs, BM25 index, manifest)
    -> push embeddings into Amazon S3 Vectors (PutVectors)

ONLINE (FastAPI, one process, answers every request)
  startup:
    -> download/load parent_docs.jsonl, child_docs.jsonl, BM25 index, manifest
    -> connect to Amazon S3 Vectors (or local ChromaDB) for dense retrieval
    -> never re-chunks, re-embeds, or rebuilds any index

  per request:
    question
      -> [optional] Groq: decompose into sub-questions (multi-hop)
      -> hybrid retrieval per sub-question: BM25 + S3 Vectors, merged by
         Reciprocal Rank Fusion (LangChain EnsembleRetriever)
      -> small-to-big: map winning child chunks back to their parent article
      -> adaptive hop planning: Groq looks at retrieved evidence and decides
         "sufficient" or proposes the next search query (up to N hops)
      -> cross-encoder reranker scores parent candidates against the
         question + sub-questions + hop queries (optional, off by default in
         prod for latency)
      -> build context from the top-N reranked docs
      -> Groq LLM generates a short-form answer from that context
      -> return {answer, sources, timings, token usage}
```

### 1.2 Deployed AWS topology (`full_structure` branch)

```text
Browser
  -> HTTPS: AWS Amplify Hosting (React/Vite frontend)
       https://full-structure.d13msmcco5ag7z.amplifyapp.com
  -> HTTPS: Amazon API Gateway (HTTP API, HTTPS proxy)
       https://b6asncvgs6.execute-api.ap-southeast-1.amazonaws.com
       routes: GET /health, POST /warmup, POST /query
  -> HTTP:  Amazon EC2 (Ubuntu), FastAPI under systemd, port 8000
       http://<ec2-public-ip>:8000
       -> Amazon S3 (processed docs + BM25 index + manifest)
       -> Amazon S3 Vectors (dense vector search, PutVectors/QueryVectors)
       -> AWS Systems Manager Parameter Store (non-secret runtime config)
       -> AWS Secrets Manager (GROQ_API_KEY)
       -> Groq API (query decomposition, hop planning, answer generation)
```

Why API Gateway sits in front of EC2: Amplify serves the frontend over HTTPS,
and browsers block an HTTPS page from calling a plain HTTP API ("Mixed
Content"). API Gateway terminates HTTPS and proxies to the EC2 backend over
HTTP, and also owns CORS for the browser-facing routes.

Current deployed resource names (see `docs/STEP_5`..`STEP_9` for the full
history of how these were created/changed):

```text
Region                 : ap-southeast-1
EC2 public API         : http://54.254.25.249:8000
API Gateway (HTTPS)    : https://b6asncvgs6.execute-api.ap-southeast-1.amazonaws.com
Amplify frontend       : https://full-structure.d13msmcco5ag7z.amplifyapp.com
Regular S3 bucket      : aws-rag-bucket-vanh1234
S3 Vectors bucket      : rag-vectors-vanh1234
S3 Vectors index       : hotpotqa-val100-bge-m3-v001
Processed id           : hotpotqa-val100-v001
Embedding model        : BAAI/bge-m3
SSM parameter prefix   : /prod/aws-rag/*
Secrets Manager secret : /prod/aws-rag/groq-api-key
```

These values can change over time (index rebuilt, EC2 IP changed, etc.) -
treat `docs/STEP_9_CENTRALIZED_CONFIG.md` and the live SSM parameters as the
source of truth, this README as a snapshot for reporting purposes.

## 2. Repository structure

```text
aws-rag-project/
  backend/
    advanced_rag/     Core RAG package (retrieval, rerank, generation, config)
    app/               FastAPI app (main.py) + AWS runtime config loader
    scripts/           Offline jobs: build_corpus.py, build_offline_artifacts.py, ...
    tools/             Local CLI: query.py, index_run.py, load_data.py
    evals/             eval_hotpotqa.py, eval_full.py, eval_stability.py
    data/              Small checked-in eval data (eval.jsonl, eval_demo.jsonl)
    artifacts/         Local generated indexes/vector stores (gitignored)
    notebooks/         Reference notebooks (corpus building, offline build)
    requirements/      Extra dependency sets (processing.txt, offline.txt)
  frontend/
    src/               React app (App.jsx) - single-page question/answer UI
  docs/
    STEP_1..STEP_9_*.md         Chronological deployment log (see below)
    AWS_RAG_DEPLOYMENT_TASKS.md Full target-architecture task list
    CHANGES_LOG.md              Retrieval/ranking tuning history
    TEST_QUESTIONS*.md          Sample HotpotQA questions (+ expected answers)
```

`docs/STEP_*.md` is the authoritative, chronological record of how the AWS
deployment was actually built (S3 -> offline artifacts -> online pipeline ->
S3 Vectors -> FastAPI -> EC2 -> Amplify/API Gateway -> warmup/fast-mode ->
centralized SSM/Secrets config). Read them in order for the full deployment
narrative; this README summarizes the end state.

## 3. RAG pipeline in detail

Core package: `backend/advanced_rag/`

| Module | Responsibility |
| --- | --- |
| `config.py` | All tunables (chunk sizes, top-k, weights, Groq models), env-overridable |
| `artifacts.py` | Loads/downloads offline artifacts, builds retrievers, no rebuilding |
| `retrieval.py` | Hybrid BM25 + vector retrieval (Reciprocal Rank Fusion), small-to-big expansion |
| `bm25s_retriever.py` | BM25 index build/load (bm25s library) |
| `s3vectors_retriever.py` | Dense retrieval via Amazon S3 Vectors QueryVectors |
| `query_optimizer.py` | Groq-based multi-hop query decomposition |
| `hop_planner.py` | Groq-based adaptive hop planning (what to search next) |
| `rerank.py` | Cross-encoder reranking of candidate parent docs |
| `generation.py` | Groq-based short-form answer generation |
| `groq_utils.py` | Retry/backoff + token-usage tracking for Groq calls |
| `pipeline.py` | `AdvancedRAGPipeline` - orchestrates the steps above per query |

Key design decisions (see inline comments in `config.py` and
`docs/CHANGES_LOG.md` for the empirical reasoning behind each):

- **Hybrid retrieval weights differ by hop.** Hop 1 (usually a "who/what is X"
  named-entity lookup from decomposition) weights BM25 higher
  (`HYBRID_WEIGHTS_HOP1`); later hops (usually a relational/descriptive
  rewrite from the hop planner) weight the vector retriever higher
  (`HYBRID_WEIGHTS_LATER_HOPS`).
- **Per-hop candidate cap (`HOP_CANDIDATE_CAP`)**, not one global cap after
  merging - prevents a noisy hop from consuming the entire rerank budget
  before a later, correct hop gets to contribute.
- **Adaptive hop planning replaced a regex-based bridge-entity heuristic**
  that had structural blind spots (single-word names, disambiguating
  suffixes like "(film)"). The hop planner asks Groq directly what to search
  next based on evidence retrieved so far.
- **Answers are short-form** (single word/name/number) to match HotpotQA's
  answer format for EM/F1 scoring; comparison questions are forced through an
  explicit "Reasoning:" line before the final "Answer:" line so the model
  computes the comparison instead of guessing from mention order.
- **`RAG_FAST_MODE`** (see `docs/STEP_8_WARMUP_FAST_MODE.md`) trades some
  retrieval quality for latency in the deployed demo: skips query
  decomposition, reduces top-k/hop counts, keeps the pipeline within API
  Gateway's ~30s timeout.

## 4. Local development

### 4.1 Backend

```bash
cd backend
pip install -r requirements.txt
cp ../.env.example ../.env   # then fill in GROQ_API_KEY
```

Run a query against a local/S3 artifact bundle:

```bash
python tools/query.py \
  --artifact-layout s3vectors \
  --index-id hotpotqa-val100-bge-m3-v001 \
  --s3-bucket aws-rag-bucket-vanh1234 \
  --s3-processed-id hotpotqa-val100-v001 \
  --s3-vector-bucket rag-vectors-vanh1234 \
  --download-artifacts \
  --device cpu \
  --skip-reranker \
  "Were Scott Derrickson and Ed Wood of the same nationality?"
```

Run the FastAPI service locally:

```bash
cd backend
uvicorn app.main:app --reload --port 8000
curl http://127.0.0.1:8000/health
curl -X POST http://127.0.0.1:8000/warmup
curl -X POST http://127.0.0.1:8000/query \
  -H "Content-Type: application/json" \
  -d '{"question":"Were Scott Derrickson and Ed Wood of the same nationality?"}'
```

Extra dependency sets (only needed for their respective jobs):

```bash
pip install -r requirements/processing.txt  # backend/scripts/build_corpus.py
pip install -r requirements/offline.txt     # backend/scripts/build_offline_artifacts.py
```

### 4.2 Frontend

```bash
cd frontend
npm install
echo "VITE_API_BASE_URL=http://127.0.0.1:8000" > .env.local
npm run dev
```

For production builds, `VITE_API_BASE_URL` must point at an HTTPS endpoint
(the API Gateway URL), not the raw EC2 HTTP address - see the Mixed Content
note in section 1.2.

## 5. Configuration reference

### 5.1 Local development (`.env` at repo root)

```env
GROQ_API_KEY=your-groq-api-key-here
```

### 5.2 Production (EC2, `backend/.env.prod`)

Bootstrap-only values; everything else is loaded at process startup from SSM
Parameter Store / Secrets Manager via `backend/app/aws_runtime_config.py`
(uses the EC2 instance's IAM role - no hard-coded AWS keys):

```env
APP_ENV=prod
AWS_REGION=ap-southeast-1
CONFIG_PREFIX=/prod/aws-rag
GROQ_SECRET_NAME=/prod/aws-rag/groq-api-key
AWS_CONFIG_OVERRIDE_ENV=true
```

`GET /health` reports whether this loaded correctly:

```json
{
  "status": "ok",
  "pipeline_loaded": false,
  "aws_runtime_config": {
    "enabled": true,
    "ssm_parameters_loaded": 18,
    "groq_secret_loaded": true
  }
}
```

### 5.3 Runtime config keys (SSM path `/prod/aws-rag/*`, see `PARAMETER_ENV_NAMES`)

| Key | Purpose | Demo value |
| --- | --- | --- |
| `rag-artifact-layout` | `s3vectors` or `chroma` | `s3vectors` |
| `rag-index-id` / `s3-vector-index` | Which built index to serve | `hotpotqa-val100-bge-m3-v001` |
| `s3-artifact-bucket` / `s3-vector-bucket` | Regular / S3 Vectors buckets | `aws-rag-bucket-vanh1234` / `rag-vectors-vanh1234` |
| `s3-processed-id` | Processed-docs prefix | `hotpotqa-val100-v001` |
| `rag-auto-download-artifacts` | Download artifacts from S3 on boot | `true` |
| `rag-fast-mode` | Skip decomposition, shrink top-k/hops | `true` |
| `bm25-top-k` / `vector-top-k` | Per-retriever candidate count | `15` / `15` |
| `rerank-top-n` | Docs kept after reranking | `5` |
| `hop-candidate-cap` / `max-adaptive-hops` / `hop-evidence-top-n` | Hop planning limits | `15` / `1` / `3` |
| `rag-device` | `cpu` or `cuda` | `cpu` |
| `rag-use-reranker` | Enable cross-encoder reranker | `false` (disabled for demo latency) |
| `rag-warmup-question` | Question used by `/warmup` | see STEP_8 |
| `cors-allow-origins` | Comma-separated allowed origins | localhost + Amplify URL |

## 6. Offline artifact build workflow

```bash
cd backend

# 1. Build a corpus (HotpotQA subset)
python scripts/build_corpus.py --output-dir data/corpus_demo_100

# 2. Build parent/child docs + BM25 + vectors, upload to S3 (+ S3 Vectors)
python scripts/build_offline_artifacts.py \
  --corpus-path data/corpus_demo_100/corpus.jsonl \
  --index-id <new-index-id> \
  --output-dir artifacts/<new-index-id> \
  --vector-backend s3vectors \
  --s3-vector-bucket rag-vectors-vanh1234 \
  --s3-vector-index <new-index-id> \
  --s3-bucket aws-rag-bucket-vanh1234 \
  --push-to-s3vectors \
  --force-rebuild

# 3. Verify the online loader can read it back
python scripts/check_online_artifacts.py \
  --artifact-layout s3vectors --index-id <new-index-id> \
  --s3-bucket aws-rag-bucket-vanh1234 --s3-processed-id <new-index-id> \
  --s3-vector-bucket rag-vectors-vanh1234 --download-artifacts

# 4. Point production at the new index (no code/EC2 image change needed)
aws ssm put-parameter --region ap-southeast-1 --name "/prod/aws-rag/rag-index-id" --type String --value "<new-index-id>" --overwrite
aws ssm put-parameter --region ap-southeast-1 --name "/prod/aws-rag/s3-processed-id" --type String --value "<new-index-id>" --overwrite
aws ssm put-parameter --region ap-southeast-1 --name "/prod/aws-rag/s3-vector-index" --type String --value "<new-index-id>" --overwrite
# then on EC2: sudo systemctl restart aws-rag-api
```

## 7. API reference

`GET /health` - liveness + config status (see 5.2).

`POST /warmup` - loads the pipeline and performs one vector retrieval, does
not call Groq generation. Used by systemd right after service restart so the
first real user query isn't paying the model/artifact load cost.

```json
{
  "status": "ok",
  "pipeline_loaded": true,
  "vector_backend": "s3vectors",
  "elapsed_seconds": 4.2,
  "warmup_child_hits": 12
}
```

`POST /query` - answers a question.

Request:

```json
{ "question": "Were Scott Derrickson and Ed Wood of the same nationality?", "top_n": 5 }
```

Response:

```json
{
  "question": "...",
  "answer": "yes",
  "context": "...",
  "sources": [
    { "title": "...", "score": 0.91, "content": "...", "metadata": {} }
  ],
  "timings": { "decompose": 0.4, "retrieve": 1.1, "rerank": 0.0, "total": 1.5, "api_total": 1.6 },
  "num_candidates": 12,
  "token_usage_total": { "calls": 2, "prompt_tokens": 500, "completion_tokens": 40, "total_tokens": 540 }
}
```

## 8. Evaluation

```bash
cd backend
python evals/eval_hotpotqa.py     # full eval set, exact-match/F1 vs eval.jsonl
python evals/eval_full.py 100     # first 100 questions
python evals/eval_stability.py    # repeated-run stability check
```

Sample questions with known-good answers: `docs/TEST_QUESTIONS_WITH_ANSWERS.md`.

## 9. Known limitations / next steps

From `docs/STEP_6` and `STEP_7`:

- Backend runs on CPU only; first query after a restart is slower (mitigated
  by `/warmup`, not eliminated).
- No authentication on the public API.
- `/query` latency is close to API Gateway's ~30s timeout under the full
  (non-fast-mode) pipeline; `RAG_FAST_MODE` is the current mitigation.
- EC2 is reached directly by API Gateway over plain HTTP on port 8000; there
  is no ALB/Nginx+TLS in front of EC2 itself yet.
- Reranker is disabled in production for latency; this trades some answer quality for speed.
- Current corpus is a 100-row HotpotQA validation slice, not the full corpus.
