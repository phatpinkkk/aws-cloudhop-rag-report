---
title: "Amazon EC2 Backend Deployment"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

The FastAPI backend is the main execution layer of CloudHop RAG. It receives questions from the API layer, loads the prepared retrieval artifacts, performs BM25 and dense retrieval, coordinates the multi-hop pipeline, constructs the final context, and sends the selected evidence to Groq for answer generation.

The backend is deployed on **Amazon EC2** so that the Python application, BM25 index, and embedding model can remain available between requests. This chapter deploys the backend and connects it to the Amazon S3 and Amazon S3 Vectors resources prepared in Chapters 5.5 and 5.6.

---

## 1. Create the EC2 Runtime Role

Before launching the instance, create an IAM role named:

`rag-ec2-runtime-role`

The role is attached to the EC2 instance and provides the backend with the AWS permissions it needs without storing AWS access keys in the application.

The runtime role needs permission to:

- read the CloudHop RAG artifacts from **Amazon S3**;
- query the required **Amazon S3 Vectors** index;
- use **AWS Systems Manager Session Manager** for instance administration.

The managed policy **`AmazonSSMManagedInstanceCore`** is attached so that the instance can be accessed through Session Manager.

The application does not need permission to modify the S3 artifacts or vector index during normal query processing.

---

## 2. Launch the EC2 Instance

Create an Ubuntu EC2 instance in **`ap-southeast-1`** and attach the `rag-ec2-runtime-role`.

The main configuration is:

| Setting | Project configuration |
| --- | --- |
| Operating system | Ubuntu Server LTS |
| Region | `ap-southeast-1` |
| IAM role | `rag-ec2-runtime-role` |
| Backend port | `8000` |
| Administration | AWS Systems Manager Session Manager |
| Runtime device | CPU |

The instance must have enough memory for the Python environment, BGE-M3 query encoder, BM25 index, and downloaded artifacts.

An **Elastic IP** is associated with the instance so that the backend address remains stable when the instance is stopped and started. This address is later used by Amazon API Gateway.

![EC2 backend instance](/images/5-Workshop/5.7-EC2-Backend/ec2-instance.png)

The workshop deployment exposes port `8000` for the API Gateway integration. This is a practical project configuration rather than a production-grade network design; the remaining security limitations are discussed in Chapter 5.13.

---

## 3. Connect with Systems Manager

Administrative access uses **AWS Systems Manager Session Manager** instead of normal SSH.

From the AWS Console:

**EC2 → Instances → Select the backend instance → Connect → Session Manager**

This avoids keeping an SSH key in the project workflow and does not require port `22` to be opened.

![Connecting through Session Manager](/images/5-Workshop/5.7-EC2-Backend/session-manager.png)

After connecting, switch to the Ubuntu user if necessary and continue the installation from the instance terminal.

---

## 4. Install the Backend

Clone the project and create the Python environment on EC2.

```bash
cd ~
git clone https://github.com/vietanh1802/aws-rag-project.git
cd ~/aws-rag-project/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

The backend does not rebuild the corpus or regenerate the full vector index. Those artifacts were already prepared offline and stored in AWS.

At runtime, the application downloads the required S3 artifacts and queries S3 Vectors when processing questions.

---

## 5. Configure the Production Environment

Runtime settings are stored in:

`/home/ubuntu/aws-rag-project/backend/.env.prod`

The configuration identifies the AWS resources and selects the lighter production-oriented retrieval settings used by the deployed application.

```env
AWS_REGION=ap-southeast-1
GROQ_API_KEY=<your-groq-key>

S3_ARTIFACT_BUCKET=aws-rag-bucket-vanh1234
S3_VECTOR_BUCKET=rag-vectors-vanh1234
RAG_INDEX_ID=hotpotqa-val500-bge-m3-v002
S3_PROCESSED_ID=hotpotqa-val500-v002
S3_VECTOR_INDEX=hotpotqa-val500-bge-m3-v002

RAG_DEVICE=cpu
RAG_FAST_MODE=true
RAG_USE_RERANKER=false
BM25_TOP_K=15
VECTOR_TOP_K=15
HOP_CANDIDATE_CAP=15
MAX_ADAPTIVE_HOPS=1
HOP_EVIDENCE_TOP_N=3
```

The main distinction from the quality-oriented evaluation setup is that the deployed configuration reduces retrieval depth and disables the cross-encoder reranker to lower CPU and latency requirements.

The Groq API key is kept in `.env.prod` and the file is excluded from Git. AWS credentials are not stored in this file because the AWS SDK obtains temporary credentials from the EC2 instance role.

![Production environment configuration](/images/5-Workshop/5.7-EC2-Backend/env-prod.png)

When capturing this configuration for the report, the Groq API key must be removed or hidden.

---

## 6. Run FastAPI with systemd

The backend is managed as a `systemd` service so that it can start automatically with the instance and restart if the process fails.

The service launches Uvicorn on port `8000` and loads `.env.prod` as its runtime configuration.

After creating the service file, enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable aws-rag-api
sudo systemctl restart aws-rag-api
sudo systemctl status aws-rag-api
```

Application logs can be inspected with:

`sudo journalctl -u aws-rag-api -f`

The backend also provides a `/warmup` endpoint. It can be called after startup so that the retrieval components are initialized before normal user requests arrive.

---

## 7. Verify the Backend

First verify that the service is running locally on the EC2 instance:

`curl http://127.0.0.1:8000/health`

Then initialize the retrieval pipeline:

`curl -X POST http://127.0.0.1:8000/warmup`

Finally, send a test question to the query endpoint:

```bash
curl -X POST http://127.0.0.1:8000/query -H "Content-Type: application/json" -d '{"question":"Were Scott Derrickson and Ed Wood of the same nationality?"}'
```

A successful response confirms that the backend can:

- load the processed documents and BM25 artifacts from Amazon S3;
- query Amazon S3 Vectors;
- execute the RAG pipeline;
- call Groq for answer generation.

At this stage, the backend is working on EC2 over HTTP. The next chapter places **Amazon API Gateway** in front of it to provide the HTTPS endpoint used by the frontend.

---

## 8. Result

The CloudHop RAG backend is now running as a persistent FastAPI service on Amazon EC2.

The deployment has:

- an EC2 instance with a stable Elastic IP;
- IAM-based access to Amazon S3 and Amazon S3 Vectors;
- Session Manager for administration;
- production configuration in `.env.prod`;
- a `systemd` service for persistent execution;
- working health, warm-up, and query endpoints.

Chapter 5.8 exposes this backend through Amazon API Gateway so that the AWS Amplify frontend can call it over HTTPS.
