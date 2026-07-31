---
title: "Operational Guide"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### Operational Guide

Daily system operation guide.

#### 1. Console Steps: Start System
After stopping the EC2 instance, when you need to use it again:
1. **Start EC2**: Start the instance in the **EC2 Console**.
2. **SSM Session**: Connect via **Session Manager** (no SSH key required).

#### 2. CLI Steps: Start Backend
After entering the EC2 terminal:
```bash
sudo systemctl restart aws-rag-api
# Warmup (required to load the model)
curl -X POST http://127.0.0.1:8000/warmup
```

#### 3. CLI Steps: Verification
```bash
curl http://127.0.0.1:8000/health
```

#### 4. Troubleshooting

| Error | Cause | Resolution |
| :--- | :--- | :--- |
| **Mixed Content** | Frontend HTTPS calling backend HTTP | Check `VITE_API_BASE_URL` in Amplify Console |
| **CORS Error** | API Gateway blocking request | Check CORS tab configuration in API Gateway Console |
| **503 Service** | Backend is down | Run `sudo systemctl status aws-rag-api` to check logs |
| **SSH Timeout** | Security Group blocking | Use Session Manager in Console instead of SSH |