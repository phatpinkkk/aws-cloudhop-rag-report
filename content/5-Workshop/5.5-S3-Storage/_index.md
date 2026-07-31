---
title: "S3 Storage Setup"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

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