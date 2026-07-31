---
title: "Cleanup"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

### Cleanup

Clean up resources on the **AWS Console** to avoid unexpected costs.

#### List of resources to delete (on Console):
1. **EC2 Instance**: Go to **EC2 Console** -> **Terminate instance**.
2. **S3 Buckets**: Go to **S3 Console** -> Delete `aws-rag-bucket-vanh1234` and `rag-vectors-vanh1234`.
3. **API Gateway**: Go to **API Gateway Console** -> Delete API.
4. **Amplify App**: Go to **Amplify Console** -> Delete App.
5. **IAM Roles**: Go to **IAM Console** -> Delete Role `rag-ec2-runtime-role`.

**Rule**: "Clean up as you go" - delete immediately after the demo to avoid billing.