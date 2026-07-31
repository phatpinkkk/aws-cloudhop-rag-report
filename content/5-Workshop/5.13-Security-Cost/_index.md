---
title: "Security and Cost Considerations"
date: 2026-07-31
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

CloudHop RAG was deployed as a practical project environment rather than a production-scale service. Even so, security and cost were considered throughout the architecture.

The main security choices focus on avoiding long-lived AWS credentials, keeping storage private, limiting backend permissions, and using managed AWS access mechanisms. On the cost side, the main recurring expense comes from the EC2 backend, while most other services are storage- or request-based.

---

## 1. Security Considerations

### IAM-based AWS access

The EC2 backend uses the IAM role `rag-ec2-runtime-role` instead of storing AWS access keys in the application.

The role provides the permissions required to:

- read retrieval artifacts from Amazon S3;
- query Amazon S3 Vectors;
- connect to the instance through AWS Systems Manager Session Manager.

The backend does not need permission to modify the stored retrieval artifacts during normal operation.

### Private storage

The project data stored in Amazon S3 is not publicly accessible. The backend accesses the bucket through IAM permissions rather than public URLs.

Amazon S3 Vectors is also accessed through the EC2 instance role. The frontend never communicates directly with either storage service.

### Administrative access

Routine administration uses **AWS Systems Manager Session Manager** rather than SSH. This avoids keeping an SSH key in the project workflow and removes the need to expose port `22`.

### HTTPS and CORS

The user-facing application is served over HTTPS through AWS Amplify, while Amazon API Gateway provides the HTTPS endpoint for backend requests.

CORS is configured to allow the expected frontend origin rather than using unrestricted browser access.

### External API credential

The Groq API key is stored in the EC2 `.env.prod` file and excluded from Git.

This is acceptable for the current project deployment, but a stronger production design would move external API credentials to a managed secret-storage service.

---

## 2. Current Limitations

The deployment intentionally remains simple, which leaves several limitations.

| Limitation | Current design | Possible production improvement |
| --- | --- | --- |
| Backend network exposure | EC2 port `8000` is reachable for API Gateway integration | Place the backend in a private network and use a private integration |
| User authentication | No end-user authentication | Add an authentication or authorization layer |
| Backend capacity | Single EC2 instance | Use multiple instances or a scalable compute layer |
| Availability | Single backend instance | Deploy across multiple availability zones |
| Groq API key | Stored in `.env.prod` | Move the secret to managed secret storage |
| Monitoring | Service status and logs checked through `systemd` and `journalctl` | Add centralized monitoring and alerting |

These limitations are acceptable for an internship-scale demonstration but would need to be addressed before using the system as a production service.

---

## 3. Main Cost Drivers

The AWS services have different billing characteristics.

| Service | Main cost source |
| --- | --- |
| **Amazon EC2** | Instance running time |
| **EBS** | Storage attached to the EC2 instance |
| **Public IPv4 / Elastic IP** | Public IPv4 usage |
| **Amazon S3** | Artifact storage and requests |
| **Amazon S3 Vectors** | Vector storage and retrieval requests |
| **Amazon API Gateway** | API requests |
| **AWS Amplify Hosting** | Build, hosting, and data transfer |

The external **Groq API** also has usage-based cost, but it is not part of the AWS bill.

For this project, EC2 is the main resource that should be actively managed because it continues consuming compute cost while running even when nobody is using the application.

---

## 4. Cost Control

Several simple practices help keep the deployment inexpensive:

1. **Stop the EC2 instance when the application is not being used.**
2. **Remove obsolete artifact and vector-index versions** once they are no longer needed.
3. **Keep the application resources in the same AWS Region** to avoid unnecessary cross-region transfer.
4. **Avoid deploying unnecessary infrastructure** for a small demonstration environment.
5. **Delete all project resources after the workshop is complete.**

The project also keeps computationally expensive corpus embedding outside the online AWS backend. The BGE-M3 embeddings are prepared offline once and then reused by the deployed application.

---

## 5. Summary

The deployment uses IAM roles instead of embedded AWS credentials, keeps the retrieval stores private, uses Session Manager for EC2 administration, and exposes the frontend through managed HTTPS services.

At the same time, the architecture remains intentionally lightweight. The main limitations are the publicly reachable EC2 backend, lack of user authentication, single-instance deployment, and file-based storage of the external API key.

From a cost perspective, EC2 is the main recurring resource to manage. Stopping unused compute and removing obsolete deployment resources are the most effective controls for this project.

Chapter 5.14 removes the AWS resources created during the workshop to prevent unnecessary charges after the project is complete.