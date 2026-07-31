---
title: "Offline Artifact Build"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Before CloudHop RAG can answer questions, the HotpotQA data must be converted into retrieval-ready artifacts. This work is performed offline so that document preparation, BM25 indexing, and embedding generation do not need to happen during a user request.

The final build uses **500 questions from the HotpotQA Distractor validation split**. The data is prepared into parent documents and child chunks, indexed for both lexical and dense retrieval, validated, and then packaged for the AWS deployment described in the following chapters.

---

## 1. Build Environment

The offline build was prepared in **Google Colab**. A GPU runtime was used mainly for generating BGE-M3 embeddings, which is the most computationally demanding part of preprocessing.

This work is intentionally kept separate from the EC2 backend. The serving instance only needs to load and use the prepared artifacts instead of rebuilding them.

The main build notebook is:

`backend/notebooks/build_s3_offline_artifacts.ipynb`

---

## 2. Dataset and Final Build

CloudHop RAG uses the **HotpotQA Distractor** setting because each question already includes a limited set of candidate documents together with annotated supporting evidence. This makes it suitable for building and evaluating the multi-hop retrieval pipeline.

The final artifact version is **v002**.

| Item | Final build |
| --- | ---: |
| Evaluation questions | 500 |
| Parent documents | 4,937 |
| Passages processed | 4,963 |
| Child vectors | 8,279 |
| Embedding model | `BAAI/bge-m3` |
| Embedding dimension | 1,024 |
| Missing supporting titles | 0 |
| Title collisions | 0 |

An earlier artifact build was found to have incomplete supporting-document coverage. The final **v002** build was therefore rebuilt from HotpotQA Distractor and validated before it was used for evaluation or deployment.

---

## 3. Parent and Child Documents

The retrieval pipeline uses a **parent-child document structure**.

Each source article is kept as a **parent document**, while smaller **child chunks** are created for retrieval. The smaller chunks make matching more precise, while the parent document preserves the broader context that can later be passed to the language model.

The build uses:

- child chunk size: **500 characters**
- child chunk overlap: **100 characters**

The relationship is:

**Source article → parent document → child chunks**

Each child chunk keeps a reference to its parent so that a retrieved result can be mapped back to the larger document before answer generation.

---

## 4. Building BM25 and Dense Embeddings

The same child chunks are prepared for two complementary retrieval methods.

### BM25

A BM25 index is built over the child chunks for lexical retrieval. This is useful for questions containing exact names, entities, numbers, or distinctive terms.

The BM25 artifacts are saved locally and later uploaded to **Amazon S3**, where the EC2 backend can load them at startup.

### BGE-M3 Embeddings

The child chunks are also encoded with **`BAAI/bge-m3`** to produce 1,024-dimensional dense vectors for semantic retrieval.

These vectors are prepared for **Amazon S3 Vectors**, which will store and search them during online queries.

![Embedding the child documents](/images/5-Workshop/5.4-Offline-Artifact-Build/embedding-progress.png)

Using both BM25 and dense embeddings gives the backend two different ways to retrieve evidence from the same corpus. Their results are combined later in the online RAG pipeline.

---

## 5. Artifact Validation

Before any files are uploaded to AWS, the build is validated to make sure the retrieval benchmark and deployment artifacts are internally consistent.

The final validation checks include:

- all 500 evaluation questions are present;
- every required supporting title is available in the corpus;
- parent and child document mappings are valid;
- embedding dimension is 1,024;
- the expected number of vectors is present;
- no title collisions are detected;
- all required artifact files exist.

The final **v002** build passed these checks with **zero missing supporting titles**.

![Final artifact validation](/images/5-Workshop/5.4-Offline-Artifact-Build/validation-checklist.png)

This validation step is important because retrieval metrics are only meaningful when the evidence required to answer each question is actually present in the corpus.

---

## 6. Final Artifacts

After the build and validation steps are complete, the project has a set of versioned artifacts ready for AWS.

| Artifact | Purpose | Next destination |
| --- | --- | --- |
| Corpus and evaluation files | Source data and benchmark reference | Amazon S3 |
| Parent documents | Larger context for retrieved evidence | Amazon S3 |
| Child documents | Retrieval units and parent mapping | Amazon S3 |
| BM25 artifacts | Lexical retrieval | Amazon S3 |
| Index manifest | Records artifact version and configuration | Amazon S3 |
| BGE-M3 vectors | Dense semantic retrieval | Amazon S3 Vectors |

The final identifiers used by the deployment are:

| Item | Value |
| --- | --- |
| Processed ID | `hotpotqa-val500-v002` |
| Vector index | `hotpotqa-val500-bge-m3-v002` |
| Embedding model | `BAAI/bge-m3` |

At this point, the retrieval artifacts are complete but have not yet been deployed. **Chapter 5.5** uploads the persistent files to Amazon S3, while **Chapter 5.6** creates and populates the Amazon S3 Vectors index.
