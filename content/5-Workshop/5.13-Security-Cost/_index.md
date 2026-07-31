---
title: "Security and Cost Considerations"
date: 2026-07-31
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

CloudHop RAG was deployed as a practical project environment, but security and cost still influenced several implementation choices. AWS permissions are provided to the EC2 backend through an IAM role rather than embedding AWS credentials in the application, and Systems Manager Session Manager is used to manage the instance.

API Gateway provides the HTTPS-facing API used by the Amplify frontend, while CORS limits browser access to the configured frontend origin. At the same time, the main recurring cost comes from keeping the EC2 instance running, together with storage, vector search, API requests, and frontend hosting.

<!--
Continue this section with two concise parts:
Security:
- IAM role for EC2 and least-privilege access;
- Session Manager access;
- HTTPS through API Gateway;
- CORS restriction;
- handling of application/API credentials without exposing secret values;
- brief remaining limitations such as the public API and lack of end-user authentication if still true.

Cost:
- main cost sources: EC2, S3, S3 Vectors, API Gateway, Amplify, and external Groq usage where relevant;
- practical controls such as stopping EC2 when unused, avoiding duplicate indexes/artifacts, and deleting resources after the workshop.
Keep this section focused on the actual deployment. Do not invent exact cost figures.
-->