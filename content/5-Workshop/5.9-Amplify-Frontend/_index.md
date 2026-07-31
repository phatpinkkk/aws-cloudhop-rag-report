---
title: "AWS Amplify Frontend Deployment"
date: 2026-07-31
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

The final user-facing component of CloudHop RAG is a **React/Vite** web application deployed with **AWS Amplify Hosting**. The interface allows users to submit a question, receive the generated answer, and inspect the supporting sources returned by the RAG backend.

The frontend communicates with the backend through the **Amazon API Gateway HTTPS endpoint** created in Chapter 5.8. This completes the browser-facing path of the application.

---

## 1. Connect the Frontend Repository

In the AWS Management Console, open:

**AWS Amplify → Create new app → Deploy from Git**

Connect the project repository and select the branch used for deployment.

Because the repository contains both backend and frontend code, Amplify must build from the frontend directory.

| Setting | Value |
| --- | --- |
| Application | React/Vite frontend |
| App root | `frontend` |
| Build command | `npm run build` |
| Output directory | `dist` |

Amplify installs the frontend dependencies, runs the Vite production build, and publishes the generated static files.

---

## 2. Configure the API Endpoint

The frontend needs the public API Gateway URL so that it knows where to send user questions.

In **Amplify → App settings → Environment variables**, add:

| Variable | Value |
| --- | --- |
| `VITE_API_BASE_URL` | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com` |

This must be the **API Gateway HTTPS URL**, not the EC2 HTTP address.

The request path is therefore:

**Browser → AWS Amplify → Amazon API Gateway → Amazon EC2**

Using the API Gateway URL avoids direct browser access to the EC2 backend and keeps the frontend request path on HTTPS.

Because Vite includes environment variables during the build process, the application must be redeployed after `VITE_API_BASE_URL` is added or changed.

---

## 3. Deploy with AWS Amplify

After the repository and environment variable are configured, start the Amplify deployment.

Amplify performs the frontend build and publishes the application to an HTTPS `amplifyapp.com` address. Future pushes to the connected branch can trigger new deployments automatically.

Once the deployment succeeds, open the generated Amplify URL in a browser.

---

## 4. Verify the Complete Application

Submit a multi-hop question such as:

*"Were Scott Derrickson and Ed Wood of the same nationality?"*

A successful request passes through the complete deployed system:

**Browser → Amplify → API Gateway → EC2 FastAPI → Amazon S3 + Amazon S3 Vectors → Groq → Answer**

The interface should display the generated answer together with the supporting sources returned by the backend.

![Deployed CloudHop RAG application](/images/5-Workshop/5.9-Amplify-Frontend/deployed-app.png)

This browser test is the first visible confirmation that all major components of the AWS deployment are connected successfully.

---

## 5. Result

The CloudHop RAG application is now available through a hosted HTTPS web interface.

At this stage:

- AWS Amplify hosts the React/Vite frontend;
- the frontend sends requests to Amazon API Gateway;
- API Gateway forwards them to the FastAPI backend on EC2;
- the backend retrieves evidence from Amazon S3 and Amazon S3 Vectors;
- Groq generates the final answer;
- the frontend displays the answer and supporting sources.

The deployment steps are now complete. Chapter 5.10 validates the full system systematically, while Chapter 5.11 evaluates retrieval quality, answer quality, and latency.
