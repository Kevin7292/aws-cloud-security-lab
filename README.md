# AWS Cloud Security Lab

Hands-on AWS security labs built to demonstrate practical cloud security skills for a career in cloud security, GRC, and security engineering. Each lab is based on real AWS services, documented with screenshots, and connected to SAA-C03 exam domains.

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

Each lab is built from scratch in a live AWS environment (not simulated), tested, documented, and cleaned up. The focus is on security architecture decisions and the reasoning behind them — not just following steps.

**Security concepts covered across all labs:**
- Network segmentation (public/private subnets)
- Least privilege access (IAM roles, Security Groups scoped to specific sources)
- Defense in depth (layered controls: Security Groups + NACLs)
- Private connectivity (VPC Endpoints — no internet exposure for internal AWS traffic)
- Audit logging and monitoring (CloudTrail)
- Secure resource configuration and cleanup

---

## Lab 01 Highlight — Two-Tier VPC

Built a production-pattern two-tier AWS network with a bastion host, isolated private subnet, IAM instance role, and VPC Gateway Endpoint for S3. Validated with live tests:

- ✅ `aws s3 ls` succeeds from a private EC2 with **zero internet access** — via VPC Gateway Endpoint
- ❌ `curl google.com` times out — internet isolation confirmed

> The private EC2 can reach S3 privately through AWS's internal network without an internet gateway, NAT gateway, or any public exposure. This is the architecture pattern used in enterprise environments to prevent data exfiltration.

**[→ View Lab 01](./lab-01-two-tier-vpc/)**

---

## SAA-C03 Exam Alignment

| Domain | Weight | Labs Covering It |
|--------|--------|-----------------|
| Domain 1: Secure Architectures | 30% | Lab 01, Lab 02 (KMS, IAM) |
| Domain 2: Resilient Architectures | 26% | Lab 03 (SQS DLQ, async processing) |
| Domain 3: High-Performing Architectures | 24% | Lab 01 (VPC Endpoints), Lab 03 (Lambda, SQS) |
| Domain 4: Cost-Optimized Architectures | 20% | All labs (free tier, endpoint vs NAT) |

---

## Lab 02 Highlight — S3 Encryption with KMS

Built an S3 encryption lab using a customer-managed KMS key with a restrictive key policy. Validated with live tests:

- ✅ Authorized IAM role can upload and download encrypted objects
- ❌ Restricted IAM user with S3 read access gets `kms:Decrypt` denied — cannot download

> S3 permissions and KMS key policies are independent access control layers. Having S3 read access is not enough when objects are encrypted with a customer-managed key. Both layers must explicitly allow the identity.

**[→ View Lab 02](./lab-02-s3-encryption-KMS/)**

---
*Active study journey — labs added as completed. All resources terminated after each lab to stay within AWS Free Tier.*
