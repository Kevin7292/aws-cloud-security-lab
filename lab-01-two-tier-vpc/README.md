# Lab 01 — AWS Cloud Security Portfolio | SAA-C03 Study Journey

## Part 1: Create the VPC

Created a new VPC named `lab01-vpc` with the CIDR block `10.0.0.0/16`.

This gives the lab a dedicated network to build in and keeps the resources separated from anything else in the AWS account.

![Create VPC](./screenshots/Part1_CreateVPC.png)

---

## Part 2: Create Subnets

Created two subnets inside the VPC:

| Subnet                 |    CIDR Block | Purpose              |
| ---------------------- | ------------: | -------------------- |
| `lab01-public-subnet`  | `10.0.1.0/24` | Bastion host         |
| `lab01-private-subnet` | `10.0.2.0/24` | Private EC2 instance |

At this point, neither subnet has internet access.

![Create Subnets](./screenshots/Part2_CreateSubnets.png)

---

## Part 3: Attach an Internet Gateway

Created an Internet Gateway named `lab01-igw` and attached it to `lab01-vpc`.

Without the Internet Gateway, the VPC has no path to the internet. Even a public subnet needs an IGW and a route table entry before it can actually reach the internet.

![Attach IGW](./screenshots/Part3_CreateAttachIGW.png)

---

## Part 4: Configure Route Tables

Created a public route table named `lab01-public-rt`.

Added this route:

| Destination | Target      |
| ----------- | ----------- |
| `0.0.0.0/0` | `lab01-igw` |

Then associated the public route table with `lab01-public-subnet`.

The private subnet was left without an internet route on purpose. It only has local VPC routing, so it cannot reach the internet directly.

![Route Tables](./screenshots/Part4_RouteTables.png)

---

## Part 5: Create Security Groups

Created two Security Groups.

### Bastion Security Group

`lab01-bastion-sg`

Allowed SSH on port `22` from **my IP only**.

This is better than allowing SSH from `0.0.0.0/0` because only my current public IP can reach the bastion.

![Bastion Security Group](./screenshots/Part5_SecurityGroupBastion.png)

### Private EC2 Security Group

`lab01-private-sg`

Allowed SSH only from `lab01-bastion-sg`.

This means the private EC2 cannot be reached directly from the internet. The only allowed SSH path is through the bastion host.

![Private Security Group](./screenshots/Part5_SecurityGroupEC2.png)

---

## Part 6: Launch Bastion Host

Launched `lab01-bastion` using:

* Amazon Linux 2023
* `t3.micro`
* Public subnet
* Public IP enabled
* `lab01-bastion-sg` attached

The bastion host is the controlled entry point into the private subnet.

![Bastion Running](./screenshots/Part7_BastionRunning.png)

---

## Part 7: Launch Private EC2

Launched `lab01-private` using:

* Amazon Linux 2023
* `t3.micro`
* Private subnet
* No public IP
* `lab01-private-sg` attached

This instance has no public IP and no internet route, so it cannot be reached directly from outside the VPC.

![Private EC2 Running](./screenshots/Part8_PrivateEC2Running.png)

---

## Part 8: Configure Network ACL

Created `lab01-private-nacl` and associated it with the private subnet.

NACLs are stateless, which means return traffic is not automatically allowed. Both inbound and outbound rules have to be configured.

### Inbound Rules

| Rule | Type        |       Port | Source        | Action |
| ---: | ----------- | ---------: | ------------- | ------ |
|  100 | SSH         |         22 | `10.0.1.0/24` | Allow  |
|  110 | Custom TCP  | 1024-65535 | `0.0.0.0/0`   | Allow  |
|  200 | All traffic |        All | `0.0.0.0/0`   | Deny   |

### Outbound Rules

| Rule | Type        |       Port | Destination | Action |
| ---: | ----------- | ---------: | ----------- | ------ |
|   90 | HTTPS       |        443 | `0.0.0.0/0` | Allow  |
|  100 | Custom TCP  | 1024-65535 | `0.0.0.0/0` | Allow  |
|  200 | All traffic |        All | `0.0.0.0/0` | Deny   |

![NACL Inbound](./screenshots/Part9_NACLInbound.png)

![NACL Outbound](./screenshots/Part9_NACLOutbound.png)

---

## Part 9: Create S3 Gateway Endpoint

Created a VPC Gateway Endpoint named `lab01-s3-endpoint` for:

`com.amazonaws.us-east-2.s3`

Associated the endpoint with the private route table.

AWS automatically added a route for S3 traffic through the endpoint. This lets the private EC2 reach S3 without using an Internet Gateway or NAT Gateway.

![S3 Endpoint Created](./screenshots/Part10_S3EndpointCreated.png)

![Private Route Table S3 Route](./screenshots/Part10_Private_RT_S3_Route.png)

---

## Part 10: Attach IAM Role to Private EC2

Created an IAM role named `lab01-ec2-s3-role` with `AmazonS3ReadOnlyAccess`.

Attached the role to `lab01-private`.

EC2 instances do not get AWS permissions by default. The IAM role is the safer way to give the instance access to S3 without putting static credentials on the server.

---

## Part 11: Testing

Connected to the bastion host first, then connected from the bastion to the private EC2.

```bash
eval $(ssh-agent -s)
ssh-add /path/to/lab01-keypair.pem
ssh -A ec2-user@<BASTION-PUBLIC-IP>
# From the bastion:
ssh ec2-user@10.0.2.x
```

The `-A` flag enables SSH agent forwarding. This lets the bastion use my local SSH agent without copying the private key onto the bastion.

![Bastion SSH Connected](./screenshots/Part11_SSH_BastionConnected.png)

![Private EC2 SSH Connected](./screenshots/Part11_SSH_Private_Connected.png)

From inside the private EC2, I ran two tests:

```bash
aws s3 ls --region us-east-2
curl https://google.com --max-time 5
```

### Results

![S3 Success and Internet Blocked](./screenshots/lab01-11-s3-endpoint-success.png)

`aws s3 ls` returned the S3 bucket. This confirmed S3 was reachable through the VPC Gateway Endpoint.

`curl https://google.com --max-time 5` timed out after 5 seconds. This confirmed the private EC2 did not have general internet access.

---

## Part 12: Cleanup

Deleted the resources in dependency order to avoid charges and keep the AWS account clean.

Cleanup steps:

1. Terminated both EC2 instances
2. Deleted the S3 bucket
3. Deleted the VPC Gateway Endpoint
4. Disassociated and deleted the custom NACL
5. Detached the IAM role
6. Deleted the IAM role
7. Deleted custom Security Groups
8. Detached and deleted the Internet Gateway
9. Deleted both subnets
10. Deleted the custom route table
11. Deleted the VPC

---

# Troubleshooting Log

## Issue 1: SSH Permission Denied

### Symptom

I received this error when trying to connect to the bastion:

```bash
Permission denied (publickey)
```

### Root Cause

The SSH agent was not running, and the key was not loaded before connecting.

### Fix

```bash
eval $(ssh-agent -s)
ssh-add /path/to/lab01-keypair.pem
ssh -A ec2-user@<BASTION-IP>
```

### Lesson

The SSH key needs to be loaded into the local agent before using agent forwarding. The `-A` flag does not help if the key was never added to the agent in the first place.

---

## Issue 2: `aws s3 ls` Hanging

### Symptom

The command hung after the IAM role was attached and the S3 endpoint route was confirmed.

```bash
aws s3 ls --region us-east-2
```

### What I Checked

* IAM role was attached
* S3 endpoint route existed in the route table
* Private EC2 had no public IP
* Security Group rules looked correct
* NACL rules were missing required traffic

### Root Cause

The private NACL was blocking the traffic.

NACLs are stateless, so I had to allow both sides of the connection:

* Outbound HTTPS on port `443`
* Inbound ephemeral ports `1024-65535` for return traffic

### Fix

Added these rules:

* Outbound Rule 90: Allow HTTPS `443`
* Inbound Rule 110: Allow TCP `1024-65535`

After adding those rules, `aws s3 ls` returned results immediately.

### Lesson

Security Groups are stateful. NACLs are stateless.

That means Security Groups automatically allow return traffic, but NACLs do not. If the return path is not allowed, the connection can hang even when the route table and IAM role are correct.

This is an important AWS networking concept and a good SAA-C03 exam topic.

---

# Key Concepts Demonstrated

* Public and private subnet design
* Internet Gateway routing
* Bastion host access pattern
* Security Group source restrictions
* Security Group references
* Network ACL stateless behavior
* S3 Gateway Endpoint access from a private subnet
* IAM role access for EC2
* Troubleshooting route, identity, and network control issues
* Secure cleanup process
