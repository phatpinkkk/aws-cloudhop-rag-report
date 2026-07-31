---
title: "Amazon S3 Storage"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

The retrieval artifacts created in the previous step need to remain available independently of the EC2 instance that runs the backend. CloudHop RAG therefore uses **Amazon S3** as the persistent store for the processed corpus, BM25 index, document mappings, and index manifests required by the online RAG pipeline.

Keeping these files in S3 allows the backend to download and load the required artifact version when it starts, without rebuilding the corpus or retrieval indexes on EC2.

<!--
Continue this section with:
- creation of the project S3 bucket in ap-southeast-1;
- the main S3 layout for corpora, processed documents, BM25 indexes, manifests, and S3 Vectors import files;
- the upload command or upload procedure;
- verification that the expected files exist in S3;
- how EC2 later reads these artifacts through its IAM role.
Keep the explanation specific to how CloudHop RAG uses S3. Do not explain generic S3 concepts.
-->

### S3 Storage Setup

S3 acts as the central storage for corpus data, processed artifacts, and the BM25 index.

#### 1. Console Steps: Create Buckets
Access the **S3 Console** to create buckets in the `ap-southeast-1` region:
- Bucket 1: `aws-rag-bucket-vanh1234` (for storing corpus/artifacts)
- Bucket 2: `rag-vectors-vanh1234` (for storing vector embeddings)

#### 2. CLI Steps: Prepare Data and Upload
Ensure you have the directory structure prepared on your local machine:

```text
./rag/
  corpora/ ...
  processed/ ...
  indexes/ ...
```

Use the AWS CLI to sync data from local to S3:

```bash
# Upload corpus & indexes
aws s3 sync "./rag" "s3://aws-rag-bucket-vanh1234/rag" --region ap-southeast-1

# Verify uploaded data
aws s3 ls s3://aws-rag-bucket-vanh1234/rag/ --recursive --region ap-southeast-1