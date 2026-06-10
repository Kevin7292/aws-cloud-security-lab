# AWS Cloud Security Lab

Hands-on AWS security labs built to demonstrate practical cloud security skills for cloud security, GRC, and security engineering roles. Each lab runs in a live AWS environment, documented with screenshots, and mapped to SAA-C03 exam domains.

**Background:** Security+ | CySA+ | Former SOC Analyst & Security Engineer | Targeting AWS SAA-C03

---

## Lab Index

| Lab | Topic | Key Concepts | Status |
|-----|-------|-------------|--------|
| [Lab 01](./lab-01-two-tier-vpc/) | Two-Tier VPC with S3 Gateway Endpoint | VPC, Subnets, Security Groups, NACLs, IAM Roles, VPC Endpoints | Complete |
| Lab 02 | IAM Hardening | Users, Groups, Roles, Least Privilege, MFA | Coming |
| Lab 03 | S3 Bucket Security | Bucket Policies, Block Public Access, Versioning, Encryption | Coming |
| Lab 04 | CloudTrail + Monitoring | Logging, Alerting, Audit Trails | Coming |
| Lab 05 | GuardDuty / Security Hub | Threat Detection, Security Posture | Coming |

---

## What This Demonstrates

Each lab is built from scratch in a live AWS environment, tested, documented, and cleaned up. Focus is on security architecture decisions and the reasoning behind them.

- Network segmentation (public/private subnets)
- Least privilege access (IAM roles, Security Groups scoped to specific sources)
- Defense in depth (layered controls: Security Groups + NACLs)
- Private connectivity (VPC Endpoints)
- Audit logging and monitoring (CloudTrail)
- Secure resource configuration and cleanup

---

## Lab 01 Highlight

Built a production-pattern two-tier AWS network with a bastion host, isolated private subnet, IAM instance role, and VPC Gateway Endpoint for S3.

- aws s3 ls succeeds from a private EC2 with zero internet access — via VPC Gateway Endpoint
- curl google.com times out — internet isolation confirmed

**[View Lab 01](./lab-01-two-tier-vpc/)**

---

*Active study journey. All resources terminated after each lab to stay within AWS Free Tier.*
