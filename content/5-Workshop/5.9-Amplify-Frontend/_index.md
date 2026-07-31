---
title: "AWS Amplify Frontend Deployment"
date: 2026-07-31
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

The final user-facing component of CloudHop RAG is a **React/Vite** web application deployed with **AWS Amplify**. The interface allows users to submit a question, receive the generated answer, and inspect the supporting sources returned by the RAG backend.

The frontend communicates with the backend through the HTTPS endpoint created in Amazon API Gateway. This keeps the browser-facing application separate from the EC2 service while providing a single public interface for users.

<!--
Continue this section with:
- connecting the frontend source repository to AWS Amplify;
- the frontend build configuration and Vite output directory;
- setting VITE_API_BASE_URL to the API Gateway HTTPS endpoint;
- deploying the Amplify application;
- opening the deployed URL and verifying that the interface loads;
- submitting one test question to confirm frontend-to-API connectivity.
Keep frontend implementation details brief; the focus is deployment and integration.
-->

### Amplify Frontend

Deploy the React interface to AWS Amplify.

#### 1. Console Steps: Configure Build Settings
In the **Amplify Console**, connect your GitHub repository and configure:
- **Build settings**:
  ```yaml
  version: 1
  frontend:
    phases:
      preBuild:
        commands:
          - npm install
      build:
        commands:
          - npm run build
    artifacts:
      baseDirectory: dist
      files:
        - '**/*'
    cache:
      paths:
        - node_modules/**/*
  ```

#### 2. Console Steps: Environment Variables
This is the most critical step for the frontend to connect to the correct API:
1. Go to **App settings** -> **Environment variables**.
2. Add variable:
   - Key: `VITE_API_BASE_URL`
   - Value: `https://<api-gateway-url>` (from step 5.6)

> **Note**: Do not use the EC2 HTTP URL as it will cause a **Mixed Content** error (the browser will block HTTP calls from HTTPS).

#### 3. Console Steps: Deployment
Click **Redeploy this version** to allow Amplify to automatically build and update the frontend URL.