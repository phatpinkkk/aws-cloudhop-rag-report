---
title: "Operations and Troubleshooting"
date: 2026-07-31
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

After deployment, most day-to-day operation of CloudHop RAG centers on the EC2 backend. The application runs as the `aws-rag-api` systemd service, while **AWS Systems Manager Session Manager** provides access to the instance when maintenance or troubleshooting is required.

The backend exposes `/health` and `/warmup` endpoints for checking application status and initializing the retrieval pipeline. Service output can also be inspected through the systemd journal when the backend fails to start or a request does not behave as expected.

---

## 1. Start the System

If the EC2 instance has been stopped, start it again from the **EC2 Console** and wait until the instance status checks pass.

Then connect through **AWS Systems Manager Session Manager**. The project does not rely on normal SSH access for routine administration.

---

## 2. Start or Restart the Backend

After connecting to the EC2 instance, restart the FastAPI service:

`sudo systemctl restart aws-rag-api`

Check its status:

`sudo systemctl status aws-rag-api`

Before sending normal user queries, warm up the retrieval pipeline:

`curl -X POST http://127.0.0.1:8000/warmup`

Warm-up initializes the main retrieval components so that the first normal request does not also perform the startup work.

---

## 3. Verify the Backend

Check the health endpoint:

`curl http://127.0.0.1:8000/health`

If the service does not start correctly or a request fails, inspect the recent service logs:

`sudo journalctl -u aws-rag-api -n 100 --no-pager`

For live logs:

`sudo journalctl -u aws-rag-api -f`

These checks help determine whether a problem comes from the backend itself or from a later layer such as API Gateway or the frontend.

---

## 4. Troubleshooting

| Problem | Possible cause | Resolution |
| --- | --- | --- |
| **Mixed Content** | The Amplify frontend is calling the EC2 HTTP address directly | Confirm that `VITE_API_BASE_URL` uses the API Gateway HTTPS URL, then redeploy the frontend if the value was changed |
| **CORS Error** | The frontend origin is not allowed by API Gateway or the backend | Check the API Gateway CORS configuration and the backend CORS settings |
| **503 / Service Unavailable** | The backend is unavailable, API Gateway cannot reach EC2, or the request takes too long | Check `aws-rag-api`, test the EC2 endpoint directly, and warm up the backend before retrying |
| **Backend does not start** | Application or configuration error | Check the service status and inspect the logs with `journalctl` |
| **First request is slow** | Retrieval components have not been initialized yet | Call `/warmup` before sending normal queries |
| **Cannot access EC2 through Session Manager** | Instance, IAM role, or Systems Manager connection issue | Confirm that the instance is running and that `AmazonSSMManagedInstanceCore` is attached to the EC2 role |

---

## 5. Operational Workflow

The normal operating sequence is straightforward:

1. Start the EC2 instance.
2. Connect through Session Manager if maintenance is required.
3. Confirm that `aws-rag-api` is running.
4. Call `/warmup`.
5. Check `/health`.
6. Use the deployed frontend or test through API Gateway.

This keeps routine operation focused on the EC2 backend while the remaining AWS components continue running as managed services.