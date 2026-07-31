---
title: "Testing and Validation"
date: 2026-07-31
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

After the AWS components are connected, the deployment is validated from the EC2 backend through to the browser. This section focuses on **functional validation**: whether the components can communicate correctly and complete an end-to-end request. Retrieval and answer quality are evaluated separately in Chapter 5.11.

## 1. Backend and Retrieval Validation

The FastAPI backend runs on Amazon EC2 as the `aws-rag-api` systemd service.

Its status can be checked with:

`sudo systemctl status aws-rag-api`

The `/health` endpoint confirms that the application is running:

`curl http://127.0.0.1:8000/health`

The retrieval pipeline can then be initialized through:

`curl -X POST http://127.0.0.1:8000/warmup`

A successful warm-up confirms that the backend can initialize the retrieval components required by the deployed pipeline.

## 2. API Gateway Validation

Once the backend works directly on EC2, the same application is tested through Amazon API Gateway.

The deployed API exposes:

- `GET /health`
- `POST /warmup`
- `POST /query`

The `/health` route verifies that an HTTPS request can reach the EC2 backend through API Gateway. CORS is also checked to confirm that requests from the AWS Amplify frontend are accepted.

A real question is then submitted through `/query`. A successful response confirms that the backend can perform retrieval, generate an answer, and return the supporting sources through the public API.

## 3. Browser Validation

The final functional check is performed from the deployed AWS Amplify application.

A successful browser request follows the complete path:

**Browser → AWS Amplify → Amazon API Gateway → Amazon EC2 → Amazon S3 + Amazon S3 Vectors → Groq → Answer + supporting sources**

This confirms that the frontend, API layer, backend, retrieval storage, and external LLM are connected correctly.

## 4. Validation Results

| Validation test | Expected result | Status |
| --- | --- | --- |
| EC2 backend | FastAPI service is running | Pass |
| Health endpoint | Successful response | Pass |
| Pipeline warm-up | Retrieval pipeline initializes successfully | Pass |
| Amazon S3 | Required retrieval artifacts are accessible | Pass |
| Amazon S3 Vectors | Dense retrieval returns results | Pass |
| API Gateway | HTTPS requests reach the backend | Pass |
| CORS | Amplify origin is accepted | Pass |
| Query endpoint | Answer and supporting sources are returned | Pass |
| Amplify frontend | Browser displays the completed response | Pass |

The validation confirms that CloudHop RAG operates as one connected application and that a user question can pass through the complete deployed system and return an answer with supporting evidence.