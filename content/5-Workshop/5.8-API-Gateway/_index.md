---
title: "Amazon API Gateway"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

The FastAPI backend deployed in Chapter 5.7 is available through HTTP on the EC2 instance. The frontend, however, is served over HTTPS by AWS Amplify. **Amazon API Gateway** is therefore used as the public HTTPS entry point between the browser and the backend.

API Gateway forwards requests to the FastAPI service and provides the browser-facing CORS configuration required by the frontend.

---

## 1. Create the HTTP API

In the AWS Management Console, open:

**Amazon API Gateway → Create API → HTTP API**

Create an HTTP API for the CloudHop RAG backend and use the EC2 Elastic IP from Chapter 5.7 as the integration target.

The project exposes three backend endpoints:

| Route | Backend endpoint | Purpose |
| --- | --- | --- |
| `GET /health` | `http://<elastic-ip>:8000/health` | Check whether the backend is running |
| `POST /warmup` | `http://<elastic-ip>:8000/warmup` | Initialize the retrieval pipeline |
| `POST /query` | `http://<elastic-ip>:8000/query` | Submit a question and receive an answer |

Using the Elastic IP keeps the integration target stable even if the EC2 instance is stopped and started.

---

## 2. Configure the Routes

Create the three routes above and connect each one to the matching FastAPI endpoint.

The route and backend path should stay consistent. For example:

**`POST /query` → `http://<elastic-ip>:8000/query`**

This makes it easy to verify each part of the system independently. If the direct EC2 endpoint works but the API Gateway route does not, the problem is limited to the gateway configuration rather than the RAG backend itself.

---

## 3. Configure CORS

The Amplify frontend and API Gateway use different origins, so the browser requires Cross-Origin Resource Sharing (CORS) to be configured.

For the API Gateway HTTP API, allow the deployed Amplify origin and the methods used by the application.

| Setting | Value |
| --- | --- |
| Allowed origin | `https://<your-amplify-app>.amplifyapp.com` |
| Allowed methods | `GET`, `POST`, `OPTIONS` |
| Allowed headers | `content-type` |

The FastAPI backend also allows the expected frontend origin through its runtime configuration.

Using the specific Amplify origin instead of a wildcard keeps browser access limited to the intended frontend.

---

## 4. Use the HTTPS Invoke URL

After the API is created, API Gateway provides an HTTPS Invoke URL in the form:

`https://<api-id>.execute-api.ap-southeast-1.amazonaws.com`

This becomes the public API base URL used by the frontend in Chapter 5.9.

The browser request path is now:

**AWS Amplify → HTTPS → Amazon API Gateway → HTTP → Amazon EC2**

The browser never needs to call the EC2 address directly.

---

## 5. Test the API

Test the health endpoint first because it does not require the full RAG pipeline:

`curl.exe "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/health"`

A successful response confirms that API Gateway can reach the EC2 backend.

Then test the query route with the same HotpotQA question used in the previous chapters:

```powershell
curl.exe -X POST "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/query" -H "Content-Type: application/json" --data "{\"question\":\"Were Scott Derrickson and Ed Wood of the same nationality?\"}"
```

A successful response should include the generated answer and supporting sources.

Testing the backend directly in Chapter 5.7 and then testing it again through API Gateway makes deployment problems much easier to locate.

---

## 6. Result

Amazon API Gateway now provides the HTTPS endpoint used by the CloudHop RAG frontend.

At this stage:

- the FastAPI backend is running on Amazon EC2;
- API Gateway forwards `/health`, `/warmup`, and `/query` requests to the backend;
- CORS allows requests from the Amplify frontend;
- the query endpoint can be reached through HTTPS.

Chapter 5.9 deploys the React frontend with AWS Amplify and configures it to use this API Gateway Invoke URL.
