---
title: "Amazon S3 Vectors"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

BM25 is effective when the wording of a question overlaps strongly with the supporting document, but multi-hop questions may also require evidence expressed in different terms. CloudHop RAG therefore combines lexical retrieval with dense semantic retrieval using **Amazon S3 Vectors**.

Each child chunk produced during the offline build is encoded with **BGE-M3** into a 1,024-dimensional vector. These vectors are stored in an S3 Vectors index and queried by the EC2 backend at runtime using the embedding of the incoming question or retrieval query.

<!--
Continue this section with:
- creation of the S3 vector bucket and vector index;
- final index settings: BGE-M3, dimension 1024, cosine distance;
- ingestion of the prepared child vectors;
- the final v002 index identifier;
- verification that the expected number of vectors is present;
- a short explanation of the runtime flow: query embedding → QueryVectors → child results → parent-document mapping.
Do not repeat the full hybrid retrieval algorithm from earlier sections.
-->

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