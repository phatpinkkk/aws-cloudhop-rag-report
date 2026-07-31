---
title: "System Architecture"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

CloudHop RAG is deployed as a web-based multi-hop RAG application in the **Asia Pacific (Singapore) Region (`ap-southeast-1`)**. The system separates the frontend, API layer, backend, and retrieval storage so that each component has a clear responsibility while still working together in one end-to-end flow.

The frontend is hosted on **AWS Amplify**. User requests are sent through **Amazon API Gateway** to a FastAPI backend running on **Amazon EC2**. The backend performs lexical retrieval using BM25 artifacts stored in **Amazon S3** and dense retrieval through **Amazon S3 Vectors**, then combines the retrieved evidence and sends the selected context to the external **Groq API** for answer generation.

This chapter focuses only on the overall design. The following chapters explain how each part is prepared, deployed, and tested.

---

## 1. Architecture Overview

<img src="/images/2-Proposal/AWS-RAG.drawio.png" alt="Architecture diagram" width="700">

At a high level, the request flow is:

**User → AWS Amplify → Amazon API Gateway → Amazon EC2 → Amazon S3 / Amazon S3 Vectors → Groq API → Answer**

The architecture has four main parts:

1. **Frontend** – a React/Vite application hosted on AWS Amplify.
2. **API layer** – Amazon API Gateway provides the HTTPS endpoint used by the browser and forwards requests to the backend.
3. **Application layer** – FastAPI runs on Amazon EC2 and coordinates retrieval, evidence processing, and LLM calls.
4. **Retrieval storage** – Amazon S3 stores the processed corpus and BM25 artifacts, while Amazon S3 Vectors stores the dense vectors used for semantic retrieval.

The **Groq API** sits outside AWS and is used for language-model operations such as query decomposition, multi-hop planning, and final answer generation.

---

## 2. Offline and Online Processing

A key design decision is to separate **offline artifact preparation** from **online query processing**.

| | Offline pipeline | Online pipeline |
| --- | --- | --- |
| Runs | When a new corpus or index version is prepared | On every user request |
| Main work | Prepare documents, build BM25 artifacts, generate embeddings, validate and upload artifacts | Retrieve evidence, process the question, call the LLM, return an answer |
| Environment | Notebook or preparation script | FastAPI backend on EC2 |

The offline stage performs the expensive preparation work once. The online service then reuses those artifacts instead of rebuilding them for every request.

This separation keeps the backend focused on retrieval and generation, while also allowing a new artifact version to be prepared and validated before it is used by the deployed application.

---

## 3. Offline Artifact Flow

![Offline artifact pipeline](/images/5-Workshop/5.3-Architecture/offline-pipeline.png)

Before deployment, the HotpotQA corpus is converted into retrieval-ready artifacts.

The project uses a **parent-child document structure**. Parent documents preserve the larger article context, while child chunks are smaller pieces of text used for retrieval.

The child chunks are indexed in two complementary ways:

- **BM25** supports lexical retrieval and is useful for exact names, entities, and terms.
- **BGE-M3 embeddings** support semantic retrieval when the question and relevant evidence use different wording.

The processed documents, BM25 files, and index metadata are stored in **Amazon S3**. Dense vectors are stored in **Amazon S3 Vectors**.

This gives the backend two retrieval methods while keeping data preparation separate from the live application. The full artifact construction and validation process is covered in **Chapter 5.4**.

---

## 4. Online Query Flow

![Online query flow](/images/5-Workshop/5.3-Architecture/online-query-flow.png)

When a user submits a question, the backend uses the prepared artifacts to retrieve evidence and generate an answer.

The main flow is:

**Question → optional decomposition → BM25 + dense retrieval → candidate combination → parent-document expansion → adaptive multi-hop retrieval → optional reranking → context construction → answer generation**

The main stages are:

1. **Query decomposition** *(optional)* – a complex question can be divided into smaller sub-questions.
2. **Hybrid retrieval** – BM25 and S3 Vectors retrieve evidence using lexical and semantic matching.
3. **Candidate combination** – results from both retrieval methods are merged into a shared candidate set.
4. **Parent-document expansion** – retrieved child chunks are mapped back to their parent documents so that the system can use broader context.
5. **Adaptive multi-hop retrieval** – if the available evidence is not sufficient, the system can perform another retrieval step using information discovered earlier.
6. **Reranking** *(optional)* – candidate evidence can be rescored before final context selection.
7. **Answer generation** – the selected evidence is sent to Groq to generate a short factual answer.
8. **Response** – the answer is returned together with the supporting sources.

The deployed runtime uses a lighter configuration than the quality-oriented evaluation setup because it must also consider available compute resources and response time. The measured quality and latency trade-offs are discussed separately in **Chapter 5.11**.

---

## 5. AWS Services and Responsibilities

Each AWS service has a specific role in the deployed system.

| Service | Role in CloudHop RAG |
| --- | --- |
| **AWS Amplify Hosting** | Hosts the React/Vite frontend |
| **Amazon API Gateway** | Provides the HTTPS API used by the frontend |
| **Amazon EC2** | Runs the FastAPI backend and coordinates the RAG pipeline |
| **Amazon S3** | Stores processed documents, BM25 artifacts, and manifests |
| **Amazon S3 Vectors** | Stores dense vectors and performs semantic retrieval |
| **AWS IAM** | Controls the backend's access to AWS resources |
| **AWS Systems Manager** | Provides Session Manager access for EC2 administration |

**Groq** is the external LLM provider used by the RAG pipeline.

The most important separation is that **Amazon S3 and S3 Vectors store retrieval data, while Amazon EC2 executes the RAG logic**. BM25 is not executed inside S3; its artifacts are stored there and loaded by the backend. Dense vector search, on the other hand, is performed through Amazon S3 Vectors.

The later workshop chapters follow this architecture directly:

- **5.4** prepares the offline artifacts.
- **5.5** uploads persistent artifacts to Amazon S3.
- **5.6** creates and populates the Amazon S3 Vectors index.
- **5.7** deploys the FastAPI backend on Amazon EC2.
- **5.8** exposes the backend through Amazon API Gateway.
- **5.9** deploys the frontend with AWS Amplify.

With the architecture established, the next step is to prepare and validate the retrieval artifacts before uploading them to AWS.
