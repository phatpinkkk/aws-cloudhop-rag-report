---
title: "Amazon S3 Storage"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

The artifacts prepared in Chapter 5.4 must remain available independently of the EC2 instance that runs the backend. CloudHop RAG therefore uses **Amazon S3** to store the processed documents, BM25 artifacts, and index metadata required by the online retrieval pipeline.

This chapter creates the project S3 bucket, uploads the prepared artifact tree, and verifies that the files are available before the backend is deployed.

---

## 1. Role of Amazon S3

Amazon S3 is used for the persistent, non-vector artifacts of the RAG system.

| Stored in Amazon S3 | Purpose |
| --- | --- |
| Parent and child documents | Retrieval and context construction |
| BM25 artifacts | Lexical retrieval |
| Index manifest | Artifact version and configuration |
| Corpus and evaluation files | Reproducibility and later evaluation |

Dense embeddings are handled separately by **Amazon S3 Vectors** in Chapter 5.6.

Keeping these artifacts outside EC2 means the backend can be replaced or restarted without losing the retrieval data.

---

## 2. Create the Artifact Bucket

The project uses the **Asia Pacific (Singapore) Region (`ap-southeast-1`)**, so the S3 bucket is created in the same Region as the rest of the deployment.

In the AWS Management Console, open **Amazon S3 → Create bucket** and configure:

| Setting | Value |
| --- | --- |
| Bucket name | A globally unique name |
| AWS Region | `ap-southeast-1` |
| Object Ownership | ACLs disabled |
| Block Public Access | Keep all public access blocked |
| Default encryption | SSE-S3 |

The project deployment uses:

`aws-rag-bucket-vanh1234`

When reproducing the workshop, choose your own globally unique bucket name.

![Creating the artifact bucket](/images/5-Workshop/5.5-S3-Storage/create-bucket.png)

The bucket remains private because the frontend never reads these files directly. Access is provided to the backend later through its IAM role.

---

## 3. Artifact Layout

The files produced in Chapter 5.4 are uploaded under the `rag/` prefix.

The important structure is:

**rag/**  
→ **corpora/** – source and evaluation files  
→ **processed/** – parent documents, child documents, and mappings  
→ **indexes/** – BM25 artifacts, manifest, and vector-import files

For the final v002 build, the main identifiers are:

| Item | Value |
| --- | --- |
| Processed ID | `hotpotqa-val500-v002` |
| Index ID | `hotpotqa-val500-bge-m3-v002` |
| Artifact prefix | `rag` |

Keeping the artifact and index identifiers versioned allows a new build to be uploaded without replacing the previous one.

---

## 4. Upload the Artifacts

The folder created in Chapter 5.4 already mirrors the expected S3 structure, so it can be uploaded recursively.

A typical AWS CLI upload is:

```bash
aws s3 sync "s3_manual_upload/hotpotqa-val500-bge-m3-v002/rag" "s3://aws-rag-bucket-vanh1234/rag" --region ap-southeast-1
```

If a different bucket name is used, replace `aws-rag-bucket-vanh1234` with that name.

![Uploading the artifact tree to Amazon S3](/images/5-Workshop/5.5-S3-Storage/upload-output.png)

The upload should include the corpus files, processed parent/child documents, BM25 artifacts, manifest, and S3 Vectors import files prepared earlier.

---

## 5. Verify the Upload

After the upload finishes, verify the files either in the S3 console or with the AWS CLI.

```bash
aws s3 ls s3://aws-rag-bucket-vanh1234/rag/ --recursive --region ap-southeast-1
```

At minimum, the following artifact groups should be present:

- `rag/corpora/...`
- `rag/processed/hotpotqa-val500-v002/...`
- `rag/indexes/hotpotqa-val500-bge-m3-v002/bm25/...`
- `rag/indexes/hotpotqa-val500-bge-m3-v002/manifests/...`
- `rag/indexes/hotpotqa-val500-bge-m3-v002/s3vectors-import/...`

![Uploaded RAG artifacts in Amazon S3](/images/5-Workshop/5.5-S3-Storage/s3-console-tree.png)

The most important runtime files are the processed parent/child documents, BM25 artifacts, and index manifest. The vector-import files are kept in S3 so they can be used to populate Amazon S3 Vectors in the next chapter.

---

## 6. Backend Access

The S3 bucket remains private. The FastAPI backend does not use embedded AWS access keys; instead, the EC2 instance receives access through the IAM role configured in Chapter 5.7.

The backend requires read access to the project artifact prefix so that it can:

- load the index manifest;
- download the processed parent and child documents;
- load the BM25 index when the service starts.

The serving application does not need to modify these files during normal query processing.

Detailed IAM configuration is covered later with the EC2 deployment and security sections rather than repeated here.

---

## 7. Result

At the end of this chapter, the non-vector RAG artifacts are stored in a private Amazon S3 bucket in `ap-southeast-1` and are ready to be consumed by the backend.

The storage responsibilities are now separated clearly:

**Amazon S3** stores documents, BM25 artifacts, and metadata.  
**Amazon S3 Vectors** stores and searches the dense BGE-M3 vectors.

Chapter 5.6 creates the vector bucket and index and loads the vectors prepared in Chapter 5.4.
