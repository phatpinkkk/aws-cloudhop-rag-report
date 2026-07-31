---
title: "Architecture Overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Architecture Overview

This project implements an end-to-end RAG (Retrieval-Augmented Generation) system on AWS. The architecture follows a serverless and containerized approach to ensure scalability and ease of deployment.

#### Architecture Diagram

![Architecture Diagram](/images/5-Workshop/5.1-Architecture-Overview/architecture.png)

#### Flow
1. **User Request**: User sends a query through the frontend application.
2. **Frontend**: A React-based application hosted on AWS Amplify.
3. **API Gateway**: Serves as the HTTPS entry point, handling CORS and routing requests to the backend.
4. **Backend (EC2)**: A FastAPI service running on EC2 processes the query.
5. **Retrieval**: The backend performs dense retrieval using S3 Vectors and BM25 index stored in S3.
6. **Generation**: The retrieved context is passed to the Groq API to generate the final answer.

<!-- [Insert architectural diagram image here] -->