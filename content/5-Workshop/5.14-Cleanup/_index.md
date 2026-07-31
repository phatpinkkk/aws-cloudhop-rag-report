---
title: "Cleanup"
date: 2026-07-31
weight: 14
chapter: false
pre: " <b> 5.14. </b> "
---

After the workshop is complete, the AWS resources created for CloudHop RAG should be removed when they are no longer needed. Cleanup prevents unused compute, storage, networking, and application resources from remaining active in the AWS account.

If the system may still be demonstrated later, the EC2 instance can be stopped temporarily instead of deleting the whole deployment.

---

## 1. Cleanup Order

The resources should be removed in an order that avoids leaving dependent components behind.

| Order | Resource | Action |
| ---: | --- | --- |
| 1 | **AWS Amplify app** | Delete the deployed frontend |
| 2 | **Amazon API Gateway API** | Delete the HTTP API and its routes |
| 3 | **Amazon EC2 instance** | Terminate the backend instance |
| 4 | **Elastic IP** | Release the address after the instance is terminated |
| 5 | **Amazon S3 Vectors index** | Delete the deployed vector index |
| 6 | **Amazon S3 Vectors bucket** | Delete the vector bucket after its indexes are removed |
| 7 | **Amazon S3 artifact bucket** | Empty the bucket and then delete it |
| 8 | **IAM role** | Delete `rag-ec2-runtime-role` after the EC2 instance is removed |

The Elastic IP should be released explicitly after the EC2 instance is terminated so that an unused public address is not left allocated.

---

## 2. Remove the Storage Resources

The vector index must be deleted before its Amazon S3 Vectors bucket can be removed.

For the normal S3 artifact bucket, delete all project objects before deleting the bucket itself.

If bucket versioning is enabled, remember that previous object versions and delete markers may also need to be removed before AWS allows the bucket to be deleted.

The final project identifiers used during cleanup are:

| Resource | Project value |
| --- | --- |
| Artifact bucket | `aws-rag-bucket-vanh1234` |
| Vector bucket | `rag-vectors-vanh1234` |
| Vector index | `hotpotqa-val500-bge-m3-v002` |
| EC2 role | `rag-ec2-runtime-role` |

When reproducing the workshop with different names, delete the resources created for that deployment instead.

---

## 3. Pausing Instead of Deleting

If CloudHop RAG may still be used for demonstrations, the deployment can be paused rather than removed completely.

The most useful action is to **stop the EC2 instance**, since the backend does not need to remain running when nobody is using the application.

The remaining storage and configuration can be kept temporarily so that the application can be started again without rebuilding the retrieval artifacts.

When the EC2 instance is started again:

1. Wait for the instance status checks to pass.
2. Confirm that `aws-rag-api` is running.
3. Call `/warmup`.
4. Check `/health`.
5. Test the application through API Gateway or the Amplify frontend.

If the project will no longer be used, full deletion is preferable.

---

## 4. Verify the Cleanup

After removing the project resources, check the AWS Console to confirm that nothing important was left behind.

| AWS area | Expected result |
| --- | --- |
| EC2 Instances | No running or stopped CloudHop RAG instance |
| Elastic IP addresses | No project Elastic IP remains allocated |
| EBS Volumes | No unused project volume remains |
| Amazon S3 | Project artifact bucket removed |
| Amazon S3 Vectors | Project vector index and vector bucket removed |
| API Gateway | CloudHop RAG API removed |
| AWS Amplify | Frontend application removed |
| IAM Roles | `rag-ec2-runtime-role` removed |

Checking the Elastic IP and EBS volume separately is useful because these resources can remain after the EC2 instance itself has been removed.

---

## 5. What Should Be Kept

Deleting the AWS deployment does not mean deleting the project itself.

The following should be kept:

- the project source repository;
- the offline artifact build notebook;
- a local copy of the validated v002 artifacts if they may be reused;
- the internship report and workshop documentation.

These files are enough to reproduce the deployment later by following Chapters 5.4–5.9 again.

---

Once these resources have been removed or intentionally paused, the CloudHop RAG workshop is complete and the AWS account no longer needs to maintain the active deployment.