# 🌍 GlobalBook Multi-Region E-commerce Platform

<div align="center">

**Enterprise-Grade Multi-Region 3-Tier Architecture with Disaster Recovery**

[![AWS](https://img.shields.io/badge/AWS-CloudFormation-orange?style=for-the-badge&logo=amazon-aws)](https://aws.amazon.com/)
[![Multi-Region](https://img.shields.io/badge/Multi--Region-Active-success?style=for-the-badge)](https://aws.amazon.com/)
[![High Availability](https://img.shields.io/badge/HA-99.99%25-brightgreen?style=for-the-badge)](https://aws.amazon.com/)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Features](#-features)
- [Quick Start](#-quick-start)
- [Deployment Guide](#-deployment-guide)
- [Project Structure](#-project-structure)
- [Cost Estimation](#-cost-estimation)
- [Troubleshooting](#-troubleshooting)
- [Documentation](#-documentation)

---

## 🎯 Overview

GlobalBook is a production-ready, multi-region e-commerce platform built on AWS using Infrastructure as Code (CloudFormation). The architecture provides high availability, automatic failover, and disaster recovery capabilities across two AWS regions.

### Key Highlights

- ✅ **Multi-Region Deployment**: Primary (us-east-1) + Spare (us-west-2)
- ✅ **3-Tier Architecture**: Web → App → Database with proper isolation
- ✅ **Auto Scaling**: Automatic scaling based on CPU utilization
- ✅ **Disaster Recovery**: Automatic failover via Route 53 health checks
- ✅ **Global CDN**: CloudFront for worldwide content delivery
- ✅ **Cost Optimized**: Regional NAT Gateway, single-AZ spare region
- ✅ **Monitoring**: CloudWatch alarms with SNS notifications

---

## 🏗️ Architecture

### Global Architecture Overview

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                    GLOBALBOOK MULTI-REGION ARCHITECTURE                        ║
╚═══════════════════════════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────────────────────────┐
│                              INTERNET USERS                                  │
│                         (Global End Users)                                   │
└────────────────────────────────────┬──────────────────────────────────────────┘
                                     │
                                     │ HTTPS/HTTP Requests
                                     │ www.globalbook.com
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          ☁️  CLOUDFRONT CDN                                  │
│                    ┌──────────────────────────────┐                          │
│                    │  Distribution Domain:        │                          │
│                    │  d1234567890.cloudfront.net  │                          │
│                    │                              │                          │
│                    │  Origin Group (Failover):     │                          │
│                    │  • Primary: Primary ALB      │                          │
│                    │  • Secondary: Spare NLB      │                          │
│                    │                              │                          │
│                    │  Features:                   │                          │
│                    │  ✓ SSL/TLS Termination       │                          │
│                    │  ✓ DDoS Protection           │                          │
│                    │  ✓ Edge Caching              │                          │
│                    │  ✓ Auto Failover (5xx)       │                          │
│                    └──────────────────────────────┘                          │
└────────────────────────────────────┬──────────────────────────────────────────┘
                                     │
                                     │ Forward to Origin
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            🌐 ROUTE 53 DNS                                   │
│                    ┌──────────────────────────────┐                          │
│                    │  Hosted Zone:                │                          │
│                    │  globalbook.com              │                          │
│                    │                              │                          │
│                    │  Record: www.globalbook.com  │                          │
│                    │  ┌────────────────────────┐ │                          │
│                    │  │ PRIMARY (Failover)     │ │                          │
│                    │  │ • Alias: Primary ALB   │ │                          │
│                    │  │ • Health Check: ✓      │ │                          │
│                    │  │ • Status: ✅ Active    │ │                          │
│                    │  └────────────────────────┘ │                          │
│                    │  ┌────────────────────────┐ │                          │
│                    │  │ SECONDARY (Failover)   │ │                          │
│                    │  │ • Alias: Spare NLB     │ │                          │
│                    │  │ • Status: ⏸️ Standby   │ │                          │
│                    │  └────────────────────────┘ │                          │
│                    └──────────────────────────────┘                          │
└────────────────────────────────────┬──────────────────────────────────────────┘
                                     │
                  ┌──────────────────┴──────────────────┐
                  │                                     │
                  ▼                                     ▼
    ╔═══════════════════════════════════════════════════════════════════╗
    ║                        PRIMARY REGION                              ║
    ║                        us-east-1                                  ║
    ║                  Status: ✅ ACTIVE                                 ║
    ╠═══════════════════════════════════════════════════════════════════╣
    ║                                                                   ║
    ║  VPC: 10.0.0.0/20 | 2 Availability Zones                         ║
    ║                                                                   ║
    ║  ┌───────────────────────────────────────────────────────────┐ ║
    ║  │  Availability Zone A (us-east-1a)                         │ ║
    ║  │  • Public Subnet: 10.0.0.0/24 (Web Tier)                  │ ║
    ║  │  • Private Subnet: 10.0.10.0/24 (App Tier)                │ ║
    ║  │  • Private Subnet: 10.0.11.0/24 (DB Tier)                 │ ║
    ║  └───────────────────────────────────────────────────────────┘ ║
    ║                                                                   ║
    ║  ┌───────────────────────────────────────────────────────────┐ ║
    ║  │  Availability Zone B (us-east-1b)                         │ ║
    ║  │  • Public Subnet: 10.0.1.0/24 (Web Tier)                   │ ║
    ║  │  • Private Subnet: 10.0.12.0/24 (App Tier)                 │ ║
    ║  │  • Private Subnet: 10.0.13.0/24 (DB Tier)                 │ ║
    ║  └───────────────────────────────────────────────────────────┘ ║
    ║                                                                   ║
    ║  Components:                                                     ║
    ║  • 🌐 Internet-facing ALB (Web Tier)                           ║
    ║  • 🔒 Internal ALB (App Tier)                                  ║
    ║  • 📊 Auto Scaling Groups (Web & App)                          ║
    ║  • 🗄️  RDS MariaDB (Single-AZ)                                 ║
    ║  • 🔄 Regional NAT Gateway                                     ║
    ║  • 📈 CloudWatch Alarms + SNS                                  ║
    ║                                                                   ║
    ╚═══════════════════════════════════════════════════════════════════╝
                  │
                  │
    ╔═══════════════════════════════════════════════════════════════════╗
    ║                        SPARE REGION                               ║
    ║                        us-west-2                                 ║
    ║                  Status: ⏸️ STANDBY                              ║
    ╠═══════════════════════════════════════════════════════════════════╣
    ║                                                                   ║
    ║  VPC: 10.1.0.0/20 | 1 Availability Zone                           ║
    ║                                                                   ║
    ║  ┌───────────────────────────────────────────────────────────┐ ║
    ║  │  Availability Zone A (us-west-2a) - Single AZ            │ ║
    ║  │  • Public Subnet: 10.1.0.0/24 (Web Tier)                 │ ║
    ║  │  • Private Subnet: 10.1.10.0/24 (App Tier)                │ ║
    ║  │  • Private Subnet: 10.1.11.0/24 (DB Tier)                  │ ║
    ║  │  • Private Subnet: 10.1.12.0/24 (DB Tier - 2nd AZ)        │ ║
    ║  └───────────────────────────────────────────────────────────┘ ║
    ║                                                                   ║
    ║  Components:                                                     ║
    ║  • ⚡ Internet-facing NLB (Web Tier)                            ║
    ║  • 📊 Auto Scaling Groups (Web & App)                          ║
    ║  • 🗄️  RDS MariaDB (Single-AZ)                                 ║
    ║  • 🔄 Regional NAT Gateway                                     ║
    ║  • 📈 CloudWatch Alarms + SNS                                  ║
    ║                                                                   ║
    ╚═══════════════════════════════════════════════════════════════════╝

═══════════════════════════════════════════════════════════════════════════════
                            TRAFFIC FLOW LEGEND
═══════════════════════════════════════════════════════════════════════════════
  ✅ Normal Flow:     Internet → CloudFront → Route 53 → Primary ALB → Web ASG → App ALB → App ASG → RDS
  ⚠️  Failover Flow:   Internet → CloudFront → Route 53 → Spare NLB → Web ASG → App ASG → RDS
  🔄 Health Check:    Route 53 → Primary ALB (every 30s)
═══════════════════════════════════════════════════════════════════════════════
```

### 3-Tier Architecture Detail

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          PRIMARY REGION (us-east-1)                          │
│                             3-Tier Architecture                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        WEB TIER (Public)                              │   │
│  │  ┌──────────────────┐         ┌──────────────────┐                │   │
│  │  │  Public Subnet A │         │  Public Subnet B │                │   │
│  │  │  10.0.0.0/24     │         │  10.0.1.0/24     │                │   │
│  │  │                  │         │                  │                │   │
│  │  │  ┌────────────┐  │         │  ┌────────────┐  │                │   │
│  │  │  │ Web EC2    │  │         │  │ Web EC2    │  │                │   │
│  │  │  │ (ASG)      │  │         │  │ (ASG)      │  │                │   │
│  │  │  └────────────┘  │         │  └────────────┘  │                │   │
│  │  └────────┬──────────┘         └────────┬──────────┘                │   │
│  │           │                            │                           │   │
│  │           └────────────┬───────────────┘                           │   │
│  │                        │ Port 80                                    │   │
│  │                        ▼                                            │   │
│  │              ┌──────────────────┐                                  │   │
│  │              │  Internet ALB    │                                  │   │
│  │              │  (L7 Load Bal.)  │                                  │   │
│  │              └──────────────────┘                                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      APP TIER (Private)                              │   │
│  │  ┌──────────────────┐         ┌──────────────────┐                  │   │
│  │  │ Private Subnet A1│         │ Private Subnet B1│                 │   │
│  │  │  10.0.10.0/24    │         │  10.0.12.0/24    │                 │   │
│  │  │                  │         │                  │                 │   │
│  │  │  ┌────────────┐  │         │  ┌────────────┐  │                 │   │
│  │  │  │ App EC2    │  │         │  │ App EC2    │  │                 │   │
│  │  │  │ (ASG)      │  │         │  │ (ASG)      │  │                 │   │
│  │  │  └────────────┘  │         │  └────────────┘  │                 │   │
│  │  └────────┬──────────┘         └────────┬──────────┘                 │   │
│  │           │                            │                            │   │
│  │           └────────────┬───────────────┘                            │   │
│  │                        │ Port 80                                     │   │
│  │                        ▼                                             │   │
│  │              ┌──────────────────┐                                   │   │
│  │              │  Internal ALB    │                                   │   │
│  │              │  (L7 Load Bal.)  │                                   │   │
│  │              └──────────────────┘                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      DB TIER (Private)                               │   │
│  │  ┌──────────────────┐         ┌──────────────────┐                 │   │
│  │  │ Private Subnet A2│         │ Private Subnet B2│                 │   │
│  │  │  10.0.11.0/24    │         │  10.0.13.0/24    │                 │   │
│  │  │                  │         │                  │                 │   │
│  │  │  ┌────────────┐  │         │  ┌────────────┐  │                 │   │
│  │  │  │ RDS       │  │         │  │ (Standby) │  │                 │   │
│  │  │  │ MariaDB   │  │         │  │            │  │                 │   │
│  │  │  │ Single-AZ │  │         │  │            │  │                 │   │
│  │  │  └────────────┘  │         │  └────────────┘  │                 │   │
│  │  └──────────────────┘         └──────────────────┘                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## ✨ Features

### High Availability
- **Multi-AZ Deployment**: Primary region spans 2 availability zones
- **Auto Scaling**: Automatic scaling based on CPU utilization (50% threshold)
- **Load Balancing**: Application Load Balancer (L7) for Web tier, Internal ALB for App tier
- **Health Checks**: ELB health checks with grace period (300s)

### Disaster Recovery
- **Multi-Region Setup**: Primary (us-east-1) + Spare (us-west-2)
- **Automatic Failover**: Route 53 health checks with 30s interval, 3 failure threshold
- **CloudFront Origin Group**: Automatic failover on 5xx errors (500, 502, 503, 504)
- **DNS-Based Routing**: Seamless failover without user intervention

### Security
- **Network Isolation**: 3-tier architecture with public/private subnets
- **Security Groups**: Least privilege access (tier-to-tier only)
- **Private Database**: RDS in private subnets, no public access
- **Regional NAT**: Cost-effective outbound internet access

### Monitoring & Alerts
- **CloudWatch Alarms**: CPU utilization monitoring (>70% threshold)
- **SNS Notifications**: Email alerts for critical events
- **Health Checks**: Route 53 health checks on primary ALB
- **Auto Scaling Metrics**: Target tracking on CPU utilization

### Cost Optimization
- **Regional NAT Gateway**: Single NAT for all AZs (vs. per-AZ)
- **Single-AZ Spare Region**: Cost-effective DR setup
- **CloudFront Price Class**: Configurable edge location pricing
- **Auto Scaling**: Scale down during low traffic

---

## 🚀 Quick Start

### Prerequisites

- AWS Account with appropriate permissions
- AWS CLI configured (optional, for CLI deployment)
- Domain name (optional, for Route 53 setup)
- Email address for SNS notifications

### Deployment Order

```bash
1️⃣  Deploy Primary Region (us-east-1)
   └─> cloudformation-template-with-dependencies.yaml

2️⃣  Deploy Spare Region (us-west-2)
   └─> cloudformation-spare-region.yaml

3️⃣  Deploy Global Services (us-east-1)
   └─> cloudformation-route53-cloudfront.yaml
```

---

## 📖 Deployment Guide

### Step 1: Deploy Primary Region

**Region:** `us-east-1`

**Template:** `cloudformation-template-with-dependencies.yaml`

**Required Parameters:**
- `DBUsername`: Database master username
- `DBPassword`: Database master password (min 8 characters)
- `EmailEndpoint`: Email for SNS notifications
- `AmiId`: Amazon Linux 2 AMI ID for us-east-1 (default provided)

**Key Outputs to Save:**
- `ALBDNS`: Primary ALB DNS name (use this value for `PrimaryALBDNSName` in Route 53 stack)
- `ALBHostedZoneId`: Primary ALB Hosted Zone ID (use this value for `PrimaryALBHostedZoneId` in Route 53 stack)

**Example:**
```bash
aws cloudformation create-stack \
  --stack-name globalbook-primary \
  --template-body file://cloudformation-template-with-dependencies.yaml \
  --parameters \
    ParameterKey=DBUsername,ParameterValue=admin \
    ParameterKey=DBPassword,ParameterValue=SecurePass123! \
    ParameterKey=EmailEndpoint,ParameterValue=your-email@example.com \
  --region us-east-1
```

### Step 2: Deploy Spare Region

**Region:** `us-west-2`

**Template:** `cloudformation-spare-region.yaml`

**Required Parameters:**
- `DBUsername`: Database master username (same as primary)
- `DBPassword`: Database master password (same as primary)
- `EmailEndpoint`: Email for SNS notifications
- `AmiId`: **Amazon Linux 2 AMI ID for us-west-2** ⚠️ (must be us-west-2 AMI!)

**How to Get us-west-2 AMI ID:**
```bash
# AWS CLI
aws ec2 describe-images \
  --region us-west-2 \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text

# Or via Console: EC2 > Launch Instance > Amazon Linux 2 AMI > Copy AMI ID
```

**Key Outputs to Save:**
- `NLBDNS`: Spare NLB DNS name (use this value for `SpareNLBDNSName` in Route 53 stack)
- `NLBHostedZoneId`: Spare NLB Hosted Zone ID (use this value for `SpareNLBHostedZoneId` in Route 53 stack)

**Example:**
```bash
aws cloudformation create-stack \
  --stack-name globalbook-spare \
  --template-body file://cloudformation-spare-region.yaml \
  --parameters \
    ParameterKey=DBUsername,ParameterValue=admin \
    ParameterKey=DBPassword,ParameterValue=SecurePass123! \
    ParameterKey=EmailEndpoint,ParameterValue=your-email@example.com \
    ParameterKey=AmiId,ParameterValue=ami-XXXXXXXXX \
  --region us-west-2
```

### Step 3: Deploy Global Services

**Region:** `us-east-1` (Route 53 and CloudFront are global)

**Template:** `cloudformation-route53-cloudfront.yaml`

**Required Parameters:**
- `DomainName`: Your domain name (e.g., `globalbook.com`)
- `PrimaryALBDNSName`: From Step 1 outputs
- `PrimaryALBHostedZoneId`: From Step 1 outputs
- `SpareNLBDNSName`: From Step 2 outputs
- `SpareNLBHostedZoneId`: From Step 2 outputs

**Optional Parameters:**
- `HealthCheckPath`: Health check endpoint (default: `/health`)
- `HealthCheckProtocol`: HTTP or HTTPS (default: HTTPS)
- `HealthCheckPort`: Health check port (default: 443)
- `CloudFrontPriceClass`: Edge location pricing (default: `PriceClass_100`)

**Example:**
```bash
aws cloudformation create-stack \
  --stack-name globalbook-r53 \
  --template-body file://cloudformation-route53-cloudfront.yaml \
  --parameters \
    ParameterKey=DomainName,ParameterValue=globalbook.com \
    ParameterKey=PrimaryALBDNSName,ParameterValue=primary-alb-123.us-east-1.elb.amazonaws.com \
    ParameterKey=PrimaryALBHostedZoneId,ParameterValue=Z35SXDOTRQ7X7K \
    ParameterKey=SpareNLBDNSName,ParameterValue=spare-nlb-123.elb.us-west-2.amazonaws.com \
    ParameterKey=SpareNLBHostedZoneId,ParameterValue=Z2IFOLF7N12345 \
  --region us-east-1
```

**After Deployment:**
1. Check SNS email and confirm subscription
2. Update your domain registrar with Route 53 name servers (from stack outputs)
3. Wait for DNS propagation (typically 5-10 minutes)

---

## 📁 Project Structure

```
nti_aws_project/
│
├── 📄 README.md                              # This file - Project overview
│
├── ☁️  CloudFormation Templates
│   ├── cloudformation-template-with-dependencies.yaml  # Primary region stack
│   ├── cloudformation-spare-region.yaml                # Spare region stack
│   └── cloudformation-route53-cloudfront.yaml          # Global services stack
│
└── 📚 Documentation
    ├── 01_Scope_of_Work.md                  # Project scope and requirements
    ├── 02_Architecture_Diagram.md           # Detailed architecture diagrams
    ├── 03_Technical_Details.md              # Technical design decisions
    ├── 04_Cost_Estimation.md                # Cost breakdown and optimization
    ├── 05_Size_Estimation.md                # Capacity and sizing guide
    └── 06_AWS_GUI_Steps.md                  # Manual AWS Console setup guide
```

---

## 💰 Cost Estimation

### Monthly Cost Breakdown (Approximate)

| Component | Primary Region | Spare Region | Global | Monthly Cost |
|-----------|---------------|--------------|--------|--------------|
| **EC2 Instances** | 4x t3.micro | 2x t3.micro | - | ~$30 |
| **RDS MariaDB** | db.t3.micro | db.t3.micro | - | ~$30 |
| **Load Balancers** | ALB (2x) | NLB (1x) | - | ~$25 |
| **NAT Gateway** | Regional NAT | Regional NAT | - | ~$65 |
| **Route 53** | - | - | Hosted Zone | ~$0.50 |
| **CloudFront** | - | - | Distribution | ~$5-50* |
| **Data Transfer** | - | - | - | Variable |
| **CloudWatch** | Alarms | Alarms | - | ~$1 |
| **SNS** | Topic | Topic | - | ~$0.10 |
| **Total (Base)** | | | | **~$156/month** |

\* CloudFront costs depend on traffic volume

**Cost Optimization Tips:**
- Use Regional NAT Gateway (vs. per-AZ) → Saves ~$32/month
- Single-AZ spare region → Saves ~$30/month
- Auto Scaling → Reduces costs during low traffic
- CloudFront caching → Reduces origin requests

📊 **Detailed cost analysis:** See [04_Cost_Estimation.md](04_Cost_Estimation.md)

---

## 🔧 Troubleshooting

### Common Issues

#### 1. Stack Creation Fails - AMI ID Not Found
**Error:** `The image id '[ami-xxx]' does not exist`

**Solution:**
- Ensure AMI ID matches the deployment region
- Get correct AMI ID for the region:
  ```bash
  aws ec2 describe-images --region <region> \
    --owners amazon \
    --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
    --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId'
  ```

#### 2. RDS Subnet Group Error
**Error:** `The DB subnet group doesn't meet Availability Zone (AZ) coverage requirement`

**Solution:**
- Spare region template includes 2 DB subnets (different AZs) to meet RDS requirement
- Ensure both `DbSubnet` and `DbSubnet2` are created successfully

#### 3. Instances Unhealthy in Target Group
**Error:** Instances show as unhealthy in ALB/NLB target group

**Solution:**
- Check security groups allow traffic from load balancer
- Verify user data script completed (check `/var/log/cloud-init-output.log`)
- Ensure health check path matches (`/health` endpoint exists)
- Check Apache is running: `systemctl status httpd`

#### 4. Route 53 Health Check Failing
**Error:** Primary region health check shows as unhealthy

**Solution:**
- Verify ALB is accessible and returns 200 OK
- Check health check path is correct (`/health` or `/`)
- Ensure security groups allow Route 53 health check IPs
- Verify ALB listener is configured correctly

#### 5. CloudFront Distribution Validation Error
**Error:** `OriginGroups: expected type: JSONObject, found: JSONArray`

**Solution:**
- Template has been fixed - ensure you're using the latest version
- `OriginGroups` must be an object with `Quantity` and `Items` properties

#### 6. SNS Email Not Received
**Solution:**
- Check SNS subscription email and click confirmation link
- Verify email address is correct in stack parameters
- Check spam/junk folder

### Getting Help

1. **Check CloudFormation Events**: Review stack events for detailed error messages
2. **Check CloudWatch Logs**: Review instance logs for application errors
3. **Verify Security Groups**: Ensure rules allow necessary traffic
4. **Review Documentation**: See detailed guides in documentation files

---

## 📚 Documentation

### Detailed Documentation Files

| File | Description |
|------|-------------|
| [01_Scope_of_Work.md](01_Scope_of_Work.md) | Project scope, requirements, and deliverables |
| [02_Architecture_Diagram.md](02_Architecture_Diagram.md) | Comprehensive architecture diagrams and traffic flows |
| [03_Technical_Details.md](03_Technical_Details.md) | Technical design decisions and rationale |
| [04_Cost_Estimation.md](04_Cost_Estimation.md) | Detailed cost breakdown and optimization strategies |
| [05_Size_Estimation.md](05_Size_Estimation.md) | Capacity planning and sizing guidelines |
| [06_AWS_GUI_Steps.md](06_AWS_GUI_Steps.md) | Step-by-step manual setup via AWS Console |

---

## 🎯 Architecture Components

### Primary Region (us-east-1)

- **VPC**: `10.0.0.0/20` (4,096 IPs)
- **Availability Zones**: 2 (us-east-1a, us-east-1b)
- **Subnets**: 6 total
  - 2 Public (Web tier)
  - 2 Private (App tier)
  - 2 Private (DB tier)
- **Load Balancers**: 
  - Internet-facing ALB (Web tier)
  - Internal ALB (App tier)
- **Auto Scaling**: 
  - Web ASG: 2-4 instances (t3.micro)
  - App ASG: 2-4 instances (t3.micro)
- **Database**: RDS MariaDB (db.t3.micro, single-AZ)
- **Networking**: Regional NAT Gateway, Internet Gateway

### Spare Region (us-west-2)

- **VPC**: `10.1.0.0/20` (4,096 IPs)
- **Availability Zones**: 1 (us-west-2a) + 1 additional for DB subnet
- **Subnets**: 4 total
  - 1 Public (Web tier)
  - 1 Private (App tier)
  - 2 Private (DB tier - 2 AZs for RDS requirement)
- **Load Balancer**: Internet-facing NLB (Web tier)
- **Auto Scaling**: 
  - Web ASG: 1-3 instances (t3.micro)
  - App ASG: 1-3 instances (t3.micro)
- **Database**: RDS MariaDB (db.t3.micro, single-AZ)
- **Networking**: Regional NAT Gateway, Internet Gateway

### Global Services

- **Route 53**: 
  - Hosted zone for domain
  - Health check on primary ALB
  - Failover routing policy
- **CloudFront**: 
  - Origin group with failover
  - Primary: Primary ALB (HTTPS)
  - Secondary: Spare NLB (HTTP)
  - Automatic failover on 5xx errors

---

## 🔐 Security Best Practices

- ✅ **Network Isolation**: 3-tier architecture with public/private subnets
- ✅ **Security Groups**: Least privilege access (tier-to-tier only)
- ✅ **Private Database**: RDS in private subnets, no public access
- ✅ **Encryption**: CloudFront HTTPS, ALB HTTPS listener
- ✅ **Monitoring**: CloudWatch alarms for security events
- ✅ **IAM**: Use least privilege IAM roles (if extending)

---

## 📈 Scaling & Performance

### Auto Scaling Configuration

- **Metric**: CPU Utilization
- **Target**: 50% CPU
- **Scale Out**: When CPU > 50% for 5 minutes
- **Scale In**: When CPU < 50% for 15 minutes
- **Cooldown**: 300 seconds

### Capacity Planning

- **Web Tier**: 2-4 instances (handles ~1,000-2,000 req/sec)
- **App Tier**: 2-4 instances (handles ~500-1,000 req/sec)
- **Database**: db.t3.micro (suitable for small-medium workloads)

📊 **Detailed sizing guide:** See [05_Size_Estimation.md](05_Size_Estimation.md)

---

## 🚨 Disaster Recovery

### Failover Scenarios

1. **Primary Region Failure**
   - Route 53 health check fails (3 consecutive failures = 90s)
   - Route 53 automatically routes to Spare NLB
   - CloudFront detects 5xx errors and fails over to Spare NLB
   - Total failover time: < 3 minutes

2. **Primary ALB Failure**
   - CloudFront origin group detects 5xx errors
   - Automatically fails over to Spare NLB
   - Route 53 health check fails and routes to Spare NLB
   - Total failover time: < 2 minutes

3. **Database Failure**
   - Manual failover required (restore from backup)
   - Or configure Multi-AZ RDS (additional cost)

### Recovery Procedures

- **Primary Region Recovery**: Fix issues, health check passes, Route 53 routes back
- **Data Sync**: Manual database replication or use RDS Multi-AZ
- **Testing**: Regularly test failover scenarios

---

## 📞 Support & Contact

For issues, questions, or contributions:

1. Review the troubleshooting section above
2. Check detailed documentation files
3. Review CloudFormation stack events
4. Check AWS CloudWatch logs

---

## 📝 License

This project is provided as-is for educational and demonstration purposes.

---

## 🙏 Acknowledgments

- AWS CloudFormation
- Amazon Route 53
- Amazon CloudFront
- AWS Auto Scaling
- Amazon RDS

---

<div align="center">

**Built with ❤️ using AWS CloudFormation**

[⬆ Back to Top](#-globalbook-multi-region-e-commerce-platform)

</div>
