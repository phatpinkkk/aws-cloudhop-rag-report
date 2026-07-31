---
title: "Amazon S3 Vectors"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

CloudHop RAG combines BM25 lexical retrieval with dense semantic retrieval. The dense side is deployed using **Amazon S3 Vectors**, which stores the BGE-M3 embeddings prepared in Chapter 5.4 and allows the backend to query them at runtime.

Each child chunk is represented by a **1,024-dimensional BGE-M3 vector**. This chapter creates the vector bucket and index, ingests the prepared vectors, and verifies that semantic retrieval works before the EC2 backend is deployed.

---

## 1. Vector Storage Design

Amazon S3 and Amazon S3 Vectors serve different purposes in the project.

| Service | Stored data | Role |
| --- | --- | --- |
| **Amazon S3** | Documents, BM25 artifacts, manifests | Persistent artifact storage |
| **Amazon S3 Vectors** | BGE-M3 embeddings | Dense similarity retrieval |

The vector resources use the same Region as the rest of the deployment:

`ap-southeast-1`

The final vector configuration is:

| Setting | Value |
| --- | --- |
| Vector bucket | `rag-vectors-vanh1234` |
| Index | `hotpotqa-val500-bge-m3-v002` |
| Embedding model | `BAAI/bge-m3` |
| Dimension | `1024` |
| Distance metric | `cosine` |
| Data type | `float32` |

These values must remain consistent with the offline build because the query embeddings produced by the backend must use the same vector space as the stored embeddings.

---

## 2. Create the Vector Bucket

In the AWS Management Console, open **Amazon S3 → Vector buckets** and create a new vector bucket in `ap-southeast-1`.

The project uses:

`rag-vectors-vanh1234`

When reproducing the workshop, choose your own vector bucket name if this name is already in use.

![Creating the vector bucket](/images/5-Workshop/5.6-S3-Vectors/create-vector-bucket.png)

The vector bucket acts as the container for one or more vector indexes. The actual embedding dimension and distance metric are configured at the index level.

---

## 3. Create the Vector Index

Inside the vector bucket, create an index with the same identifier used by the final artifact build:

`hotpotqa-val500-bge-m3-v002`

Configure:

- **Data type:** `float32`
- **Dimension:** `1024`
- **Distance metric:** `cosine`

![Creating the vector index](/images/5-Workshop/5.6-S3-Vectors/create-index.png)

The index name is intentionally versioned. This keeps the deployed vector index aligned with the corresponding v002 artifacts stored in Amazon S3.

The dimension must also match BGE-M3. If the index is created with a different dimension, the prepared vectors cannot be ingested correctly.

---

## 4. Ingest the Prepared Vectors

Chapter 5.4 generated the dense vectors and grouped them into import batches. Those batches are now written to the S3 Vectors index.

The project uses the generated ingestion script from the offline artifact folder:

```bash
python ingest_s3vectors.py --region ap-southeast-1
```

The final build contains **8,279 child vectors**, so the number of vectors successfully ingested should match the validated child-vector count from Chapter 5.4.

![Ingesting vectors into Amazon S3 Vectors](/images/5-Workshop/5.6-S3-Vectors/ingest-output.png)

Each vector uses the corresponding `child_id` as its key. This allows a returned vector result to be mapped back to the child document stored in Amazon S3 and then expanded to its parent document during retrieval.

---

## 5. Verify the Index

After ingestion, verify that the index exists and contains vectors.

A simple AWS CLI check is:

```bash
aws s3vectors list-vectors --vector-bucket-name rag-vectors-vanh1234 --index-name hotpotqa-val500-bge-m3-v002 --max-results 5 --region ap-southeast-1
```

The returned keys should correspond to child document identifiers from the v002 artifact build.

The more useful test is to run a real semantic retrieval query. The project includes a retrieval check that encodes a question with BGE-M3, queries the S3 Vectors index, and maps the returned vector keys back to the corresponding documents.

![Dense retrieval working against the vector index](/images/5-Workshop/5.6-S3-Vectors/retrieval-check.png)

For example, the multi-hop question:

*"Were Scott Derrickson and Ed Wood of the same nationality?"*

should return documents related to the entities in the question. This confirms that the embedding model, stored vectors, index configuration, and document mapping are consistent.

---

## 6. How the Backend Uses S3 Vectors

At runtime, the FastAPI backend follows a simple dense-retrieval flow:

**Question → BGE-M3 query embedding → Amazon S3 Vectors → matching child IDs → child documents → parent documents**

The vector index stores the embeddings and their keys, while the document text remains in the processed artifacts stored in ordinary Amazon S3.

This separation avoids duplicating the full document text inside the vector store. Amazon S3 Vectors is responsible for similarity search, while the backend uses the returned child IDs to recover the corresponding text and larger parent context.

The dense results are then combined with BM25 results as part of the hybrid retrieval pipeline described in Chapter 5.3.

---

## 7. Backend Permissions

The EC2 backend only needs runtime permissions for querying the vector index. It does not need to create indexes or modify stored vectors during normal operation.

The IAM role configured in Chapter 5.7 therefore provides access for the backend to query the required S3 Vectors resources, while vector creation and ingestion remain deployment-time operations.

This keeps the serving application separate from index management.

---

## 8. Result

At the end of this chapter, both retrieval stores are ready:

- **Amazon S3** contains the documents, BM25 artifacts, and manifests.
- **Amazon S3 Vectors** contains the 8,279 BGE-M3 child vectors used for semantic retrieval.

The dense index has also been checked with a real retrieval query before the application backend is introduced.

Chapter 5.7 deploys the FastAPI backend on Amazon EC2 and connects it to both retrieval sources.
