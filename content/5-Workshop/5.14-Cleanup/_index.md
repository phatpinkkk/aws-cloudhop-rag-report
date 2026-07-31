---
title: "Cleanup"
date: 2026-07-31
weight: 14
chapter: false
pre: " <b> 5.14. </b> "
---

After the workshop is complete, the AWS resources created for CloudHop RAG should be removed when they are no longer needed. Cleaning up the deployment avoids leaving unused compute, storage, networking, and application resources active in the AWS account.

Resources should be removed in an order that avoids leaving dependent components behind.

<!--
Continue this section with a short cleanup sequence covering:
- AWS Amplify application;
- API Gateway API;
- EC2 instance and Elastic IP;
- S3 Vectors index and vector bucket;
- regular S3 artifacts and bucket;
- project-specific IAM roles/policies or other configuration resources that are no longer required.
Finish with a simple verification step in the AWS Console.
Keep this section short and action-oriented.
-->

### Cleanup

Clean up resources on the **AWS Console** to avoid unexpected costs.

#### List of resources to delete (on Console):
1. **EC2 Instance**: Go to **EC2 Console** -> **Terminate instance**.
2. **S3 Buckets**: Go to **S3 Console** -> Delete `aws-rag-bucket-vanh1234` and `rag-vectors-vanh1234`.
3. **API Gateway**: Go to **API Gateway Console** -> Delete API.
4. **Amplify App**: Go to **Amplify Console** -> Delete App.
5. **IAM Roles**: Go to **IAM Console** -> Delete Role `rag-ec2-runtime-role`.

**Rule**: "Clean up as you go" - delete immediately after the demo to avoid billing.