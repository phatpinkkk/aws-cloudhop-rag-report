---
title: "Offline Artifact Build"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Before the RAG backend can answer questions, the source data must be converted into retrieval-ready artifacts. This work is performed offline so that document processing, chunking, embedding generation, and index construction do not need to happen during a user request.

CloudHop RAG uses the **HotpotQA Distractor** dataset as the benchmark source for this process. HotpotQA is designed for multi-hop question answering, where the information needed for an answer may be distributed across several supporting documents. Its annotated questions, answers, contexts, and supporting facts make it suitable for building and evaluating the retrieval pipeline used in this project.

The final artifact build uses **500 questions from the validation split**. Their contexts are normalized into the project corpus format before the parent documents, child chunks, BM25 index, and BGE-M3 embeddings are generated.

<!--
Continue this section with:
- the input HotpotQA fields used by the project;
- normalization into corpus.jsonl and eval.jsonl;
- parent-document and child-chunk creation;
- BM25 index construction;
- BGE-M3 embedding generation for the child chunks;
- generation of the S3 Vectors import files and index manifest;
- final validated v002 artifact summary: 4,937 parent documents, 4,963 passages processed, 8,279 child vectors, 1,024-dimensional embeddings, 0 missing supporting titles, and 0 title collisions;
- a short output-folder/file overview.
Focus on what is built and why. Do not turn this into a line-by-line notebook walkthrough.
-->

### Offline Artifact Build

This step prepares the data and creates the necessary index files.

#### 1. CLI Steps: Notebook Environment
Run the `build_s3_offline_artifacts.ipynb` notebook located in the `backend/notebooks/` directory. Ensure you have the necessary libraries installed:

```bash
pip install jupyter pandas numpy scikit-learn
```

#### 2. Notebook UI Steps: Dataset Configuration
Open the Jupyter Notebook, locate the data configuration section, and adjust the dataset size as needed (e.g., 500 samples for the demo):

```python
# Configuration in the notebook
DATASET_SIZE = 500
```

#### 3. Notebook UI Steps: Execution
Execute all cells in the notebook. Once completed, the necessary files (`corpus.jsonl`, `index_manifest.json`, etc.) will be generated in the `output/` directory.