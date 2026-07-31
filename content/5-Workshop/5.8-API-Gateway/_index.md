---
title: "API Gateway & CORS"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

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