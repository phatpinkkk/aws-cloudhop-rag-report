---
title: "S3 Vectors Setup"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### S3 Vectors Setup

S3 Vectors stores vector embeddings used for dense retrieval features.

#### 1. Console Steps: Configure Index
Access the **S3 Vectors Console** (or corresponding service) to create a new index with the following parameters:
- **Dimension**: 1024
- **Distance metric**: cosine
- **Bucket**: `rag-vectors-vanh1234`
- **Index name**: `hotpotqa-val500-bge-m3-v001`

#### 2. CLI Steps: Ingest Vectors
Use the `ingest_s3vectors.py` script (located in the project) to push data from the JSON files created in step 5.2 to the S3 Vectors bucket:

```bash
# Navigate to the script directory
cd path/to/your/scripts

# Run the ingest script
python ingest_s3vectors.py --region ap-southeast-1
```

#### 3. CLI Steps: Verification
Use the AWS CLI to list vectors and ensure the data is present:

```bash
aws s3vectors list-vectors \
  --region ap-southeast-1 \
  --vector-bucket-name rag-vectors-vanh1234 \
  --index-name hotpotqa-val500-bge-m3-v001 \
  --max-items 5