---
title: Introduction
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Architecture Overview
The system is divided into two distinct pipelines to optimize cost and performance:
1. **Offline Ingestion Pipeline:** Runs once to chunk documents, generate embeddings (BAAI/bge-m3), build the BM25 index, and upload artifacts to Amazon S3 and Amazon S3 Vectors.
2. **Online Query Pipeline:** A FastAPI application deployed on EC2 that handles incoming user queries, performs hybrid retrieval, and calls the Groq LLM API.

#### AWS Services Used & Justification
*   **Amazon EC2:** Hosts the FastAPI backend application efficiently.
*   **Amazon API Gateway:** Acts as an HTTPS proxy in front of the EC2 instance to prevent "Mixed Content" errors from the HTTPS Amplify frontend.
*   **Amazon S3 & S3 Vectors:** Cost-effective, serverless storage for processed documents, manifests, and dense vector embeddings.
*   **AWS Systems Manager (SSM) & Secrets Manager:** Centralized configuration management and secure storage of the Groq API key, adhering to security best practices (no hard-coded credentials).

![overview](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)