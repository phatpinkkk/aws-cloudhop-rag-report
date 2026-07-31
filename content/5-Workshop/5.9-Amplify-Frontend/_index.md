---
title: "Amplify Frontend"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

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