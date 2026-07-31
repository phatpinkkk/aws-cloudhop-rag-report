---
title: "System Architecture"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

CloudHop RAG is deployed as a web-based RAG application in the **Asia Pacific (Singapore) Region (`ap-southeast-1`)**. The architecture separates the frontend, API layer, RAG backend, and retrieval storage so that each part of the system has a clear responsibility while remaining connected through a single request flow.

A user interacts with the frontend hosted on **AWS Amplify**. Requests are sent through **Amazon API Gateway** to the FastAPI backend running on **Amazon EC2**. The backend performs lexical retrieval using BM25 artifacts stored in **Amazon S3** and dense retrieval through **Amazon S3 Vectors**, then processes the retrieved evidence before sending the final context to the Groq LLM API.

## Architecture Diagram

![AWS CloudHop RAG architecture](/images/5-Workshop/5.3-Architecture/architecture.png)

<!--
Continue this section with:
- the role of each main AWS component: Amplify, API Gateway, EC2, S3, S3 Vectors, IAM, and Systems Manager;
- the complete request flow from browser to answer;
- the distinction between S3 storing BM25/processed artifacts and EC2 actually running BM25 retrieval;
- the EC2 placement inside the VPC/public subnet and use of an Elastic IP where relevant;
- short explanations of the main design decisions, especially why API Gateway is used between the HTTPS frontend and EC2 backend.
Keep this section focused on architecture and design. Do not include deployment commands.
-->

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