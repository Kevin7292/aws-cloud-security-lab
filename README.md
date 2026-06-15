# AWS Cloud Security Lab

Hands-on AWS security labs built while studying for AWS SAA-C03. Each lab is built from scratch in a live AWS environment, tested, documented with screenshots, and cleaned up when done.

**Background:** Security+ | CySA+ | Former SOC Analyst & Security Engineer | Targeting AWS SAA-C03

---

## Lab Index

| Lab | Topic | Key Concepts | Status |
|-----|-------|-------------|--------|
| [Lab 01](./lab-01-two-tier-vpc/) | Two-Tier VPC with S3 Gateway Endpoint | VPC, Subnets, Security Groups, NACLs, IAM Roles, VPC Endpoints | ✅ Complete |
| [Lab 02](./lab-02-s3-encryption-KMS/) | S3 Encryption with KMS | SSE-KMS, KMS Key Policies, IAM Roles, S3 Versioning, Lifecycle Rules | ✅ Complete |
| Lab 03 | SQS + Lambda Pipeline | SQS, Lambda, DLQ, Async Processing, Serverless | 🔜 Coming |

*More labs in progress — added as completed.*

---

## What This Demonstrates

Each lab focuses on the reasoning behind security decisions, not just the steps. The goal is to understand why things are built a certain way, not just how to click through the console.

Security concepts covered across all labs:
- Network segmentation (public/private subnets)
- Least privilege access (IAM roles, Security Groups scoped to specific sources)
- Defense in depth (layered controls: Security Groups + NACLs)
- Private connectivity (VPC Endpoints, no internet exposure for internal AWS traffic)
- Audit logging and monitoring (CloudTrail)
- Secure resource configuration and cleanup

---

## Lab 01 Highlight — Two-Tier VPC

Built a two-tier VPC with a bastion host, private subnet, IAM instance role, and S3 Gateway Endpoint. Validated with live tests:

- ✅ `aws s3 ls` works from a private EC2 with no internet access, routed through the VPC Gateway Endpoint
- ❌ `curl google.com` times out, confirming internet isolation

The private EC2 can reach S3 without an internet gateway or NAT gateway. The traffic stays inside AWS's network. No public exposure.

**[View Lab 01](./lab-01-two-tier-vpc/)**

---

## Lab 02 Highlight — S3 Encryption with KMS

Built an S3 encryption lab using a customer-managed KMS key with a key policy scoped to one IAM role. Validated with live tests:

- ✅ The authorized IAM role can upload and download encrypted objects
- ❌ A separate IAM user with S3 read access gets `kms:Decrypt` denied and cannot download

S3 permissions and KMS key policies are separate. Having S3 read access does not help if you are not in the key policy.

**[View Lab 02](./lab-02-s3-encryption-KMS/)**

---

## SAA-C03 Exam Alignment

| Domain | Weight | Labs Covering It |
|--------|--------|-----------------|
| Domain 1: Secure Architectures | 30% | Lab 01, Lab 02 (KMS, IAM) |
| Domain 2: Resilient Architectures | 26% | Lab 03 (SQS DLQ, async processing) |
| Domain 3: High-Performing Architectures | 24% | Lab 01 (VPC Endpoints), Lab 03 (Lambda, SQS) |
| Domain 4: Cost-Optimized Architectures | 20% | All labs (free tier, endpoint vs NAT) |

---

*Active study journey. Labs added as completed. All resources deleted after each lab to stay within AWS Free Tier.*
