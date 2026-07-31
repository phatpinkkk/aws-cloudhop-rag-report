---
title: "Amazon API Gateway"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

The FastAPI backend on EC2 is available over HTTP, while the frontend delivered by AWS Amplify runs over HTTPS. A browser cannot safely call the HTTP backend directly from the HTTPS frontend, so **Amazon API Gateway** is used as the public HTTPS entry point for the application.

API Gateway receives requests from the browser and forwards them to the FastAPI service on EC2. It also provides the browser-facing CORS configuration required for the Amplify frontend to call the backend successfully.

<!--
Continue this section with:
- creation of the HTTP API in API Gateway;
- EC2 backend integration;
- the three routes: GET /health, POST /warmup, POST /query;
- CORS configuration for the Amplify frontend origin;
- deployment/auto-deploy of the API;
- testing the HTTPS /health route and a POST request through API Gateway.
Keep the Mixed Content explanation short because the reason is already established above.
-->

### API Gateway & CORS

API Gateway acts as a secure HTTPS layer, connecting the frontend (Amplify) to the backend (EC2).

#### 1. Console Steps: Create API Gateway
1. Access the **API Gateway Console** -> **Create API** -> **HTTP API**.
2. **Add integration**: Select **HTTP** and enter the EC2 backend URL (e.g., `http://54.251.81.140:8000`).
3. Define the routes:
   - `GET /health`
   - `POST /query`
   - `POST /warmup`
4. Deploy the API and note the **Invoke URL** (e.g., `https://b6asncvgs6.execute-api.ap-southeast-1.amazonaws.com`).

#### 2. Console Steps: Configure CORS
To allow the frontend to call the API, you must configure CORS:
1. In the API Gateway Console, select the **CORS** tab.
2. **Allow-Origin**: `https://<your-amplify-app-url>`
3. **Allow-Methods**: `GET`, `POST`, `OPTIONS`
4. **Allow-Headers**: `content-type`
5. Click **Save** and **Deploy** the API stage again.

#### 3. CLI Steps: Test API Gateway
Test via PowerShell:

```powershell
# Test health
curl.exe "https://<api-gateway-url>/health"

# Test query (using query.json file)
'{"question":"Were Scott Derrickson and Ed Wood of the same nationality?"}' | Set-Content -Encoding utf8 query.json

curl.exe -s -X POST "https://<api-gateway-url>/query" `
  -H "Content-Type: application/json" `
  --data-binary "@query.json"