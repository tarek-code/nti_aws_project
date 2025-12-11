# Scope of Work (SoW)

## GlobalBook Inc. - Multi-Region 3-Tier E-commerce Architecture

---

## 📋 Project Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    GLOBALBOOK E-COMMERCE PLATFORM                │
│                    Multi-Region Disaster Recovery                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌─────────────────────────────────────────┐
        │         GLOBAL COMPONENTS                │
        │  • Route 53 (DNS Failover)              │
        │  • CloudFront (CDN + Origin Failover)    │
        └─────────────────────────────────────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
    ┌──────────────────┐         ┌──────────────────┐
    │  PRIMARY REGION  │         │   SPARE REGION   │
    │   (us-east-1)   │         │   (us-west-2)    │
    │                 │         │                  │
    │  • 2 AZs        │         │  • 1 AZ          │
    │  • ALB (Web)    │         │  • NLB (Web)     │
    │  • ALB (App)    │         │  • App ASG       │
    │  • RDS MariaDB  │         │  • RDS MariaDB   │
    └──────────────────┘         └──────────────────┘
```

---

## 🎯 Project Deliverables

### 1. Infrastructure Components

```
┌──────────────────────────────────────────────────────────────┐
│                    PRIMARY REGION (us-east-1)                 │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  VPC: 10.0.0.0/20                                            │
│  ├── 2 Availability Zones                                    │
│  ├── 6 Subnets (2 Public + 4 Private)                       │
│  ├── Internet Gateway                                        │
│  ├── Regional NAT Gateway                                    │
│  │                                                           │
│  Web Tier (Public):                                          │
│  ├── Application Load Balancer (Internet-facing)            │
│  └── Auto Scaling Group (Web Instances)                     │
│                                                               │
│  App Tier (Private):                                         │
│  ├── Internal Application Load Balancer                     │
│  └── Auto Scaling Group (App Instances)                     │
│                                                               │
│  DB Tier (Private):                                          │
│  └── RDS MariaDB (Single-AZ)                                 │
│                                                               │
│  Monitoring:                                                 │
│  ├── CloudWatch Alarms                                       │
│  └── SNS Email Notifications                                 │
│                                                               │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    SPARE REGION (us-west-2)                   │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  VPC: 10.1.0.0/20                                            │
│  ├── 1 Availability Zone                                    │
│  ├── 3 Subnets (1 Public + 2 Private)                      │
│  ├── Internet Gateway                                        │
│  ├── Regional NAT Gateway                                    │
│  │                                                           │
│  Web Tier (Public):                                          │
│  ├── Network Load Balancer (Internet-facing)                │
│  └── Auto Scaling Group (Web Instances)                     │
│                                                               │
│  App Tier (Private):                                         │
│  └── Auto Scaling Group (App Instances)                     │
│                                                               │
│  DB Tier (Private):                                          │
│  └── RDS MariaDB (Single-AZ)                                │
│                                                               │
│  Monitoring:                                                 │
│  ├── CloudWatch Alarms                                       │
│  └── SNS Email Notifications                                 │
│                                                               │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    GLOBAL COMPONENTS                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Route 53 (DNS Service):                                      │
│  ├── Hosted Zone (Public DNS for domain)                    │
│  ├── Health Check                                            │
│  │   ├── Monitors Primary ALB DNS name                       │
│  │   ├── Protocol: HTTPS (port 443)                         │
│  │   ├── Path: / (health check endpoint)                    │
│  │   ├── Interval: 30 seconds                               │
│  │   └── Failure Threshold: 3 consecutive failures          │
│  ├── Primary Record Set                                      │
│  │   ├── Type: A (Alias)                                    │
│  │   ├── Name: www.globalbook.com                           │
│  │   ├── Routing Policy: Failover (PRIMARY)                │
│  │   ├── Alias Target: Primary ALB                          │
│  │   └── Health Check: Enabled (attached)                   │
│  └── Secondary Record Set                                    │
│      ├── Type: A (Alias)                                    │
│      ├── Name: www.globalbook.com                           │
│      ├── Routing Policy: Failover (SECONDARY)               │
│      ├── Alias Target: Spare NLB                            │
│      └── Health Check: Disabled (secondary doesn't need)    │
│                                                               │
│  CloudFront (CDN):                                            │
│  ├── Distribution                                             │
│  │   ├── Price Class: North America & Europe                │
│  │   ├── SSL Certificate: Default CloudFront                │
│  │   └── Viewer Protocol: Redirect HTTP to HTTPS           │
│  ├── Origin Group (Failover)                                 │
│  │   ├── Primary Origin: Primary ALB                        │
│  │   │   ├── Protocol: HTTPS Only                          │
│  │   │   └── Port: 443                                     │
│  │   ├── Secondary Origin: Spare NLB                       │
│  │   │   ├── Protocol: HTTP Only                           │
│  │   │   └── Port: 80                                      │
│  │   └── Failover Criteria: 500, 502, 503, 504 errors      │
│  ├── Cache Behavior                                          │
│  │   ├── Allowed Methods: GET, HEAD, OPTIONS               │
│  │   ├── Cached Methods: GET, HEAD                         │
│  │   ├── Compress: Enabled                                  │
│  │   ├── Query String: Forward all                         │
│  │   └── Cookies: None                                      │
│  └── Features                                                │
│      ├── Global Edge Locations                              │
│      ├── Content Caching                                    │
│      ├── DDoS Protection                                    │
│      └── Origin Shield                                      │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 2. Infrastructure as Code (CloudFormation)

```
📁 CloudFormation Templates:
│
├── 📄 cloudformation-template-with-dependencies.yaml
│   └── Primary Region Stack (VPC, ALBs, ASGs, RDS, Monitoring)
│
├── 📄 cloudformation-spare-region.yaml
│   └── Spare Region Stack (VPC, NLB, ASGs, RDS, Monitoring)
│
└── 📄 cloudformation-route53-cloudfront.yaml
    └── Global Stack (Route 53 + CloudFront)
```

### 3. Documentation Deliverables

```
📚 Documentation Package:
│
├── 📄 01_Scope_of_Work.md (This File)
│   └── Project scope and deliverables overview
│
├── 📄 02_Architecture_Diagram.md
│   └── High-level architecture and traffic flow diagrams
│
├── 📄 03_Technical_Details.md
│   └── Technical rationale for HA, DR, and scalability
│
├── 📄 04_Cost_Estimation.md
│   └── Cost breakdown and optimization tips
│
├── 📄 05_Size_Estimation.md
│   └── IP allocation, instance sizing, bandwidth planning
│
├── 📄 06_AWS_GUI_Steps.md
│   └── Step-by-step AWS Console manual deployment guide
│
└── 📄 README.md
    └── Project overview and quick start guide
```

---

## 🔄 Traffic Flow Diagram

```
Normal Operation (Primary Region Active):
┌─────────┐
│  User   │
└────┬────┘
     │ HTTPS Request
     ▼
┌─────────────────┐
│   CloudFront    │ (CDN + Origin Failover)
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│   Route 53      │ (DNS Resolution)
└────┬────────────┘
     │ www.globalbook.com
     ▼
┌─────────────────┐
│  Primary ALB    │ (Internet-facing, us-east-1)
└────┬────────────┘
     │ Port 80
     ▼
┌─────────────────┐
│   Web ASG       │ (Public Subnets, 2 AZs)
│  EC2 Instances  │
└────┬────────────┘
     │ Port 80
     ▼
┌─────────────────┐
│  App ALB        │ (Internal, Private Subnets)
└────┬────────────┘
     │ Port 80
     ▼
┌─────────────────┐
│   App ASG       │ (Private Subnets, 2 AZs)
│  EC2 Instances  │
└────┬────────────┘
     │ Port 3306
     ▼
┌─────────────────┐
│   RDS MariaDB   │ (Private Subnets, Single-AZ)
└─────────────────┘


Failover Scenario (Primary Region Down):
┌─────────┐
│  User   │
└────┬────┘
     │ HTTPS Request
     ▼
┌─────────────────┐
│   CloudFront    │
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│   Route 53      │ (Health Check Failed → Failover)
└────┬────────────┘
     │ www.globalbook.com
     ▼
┌─────────────────┐
│   Spare NLB     │ (Internet-facing, us-west-2)
└────┬────────────┘
     │ Port 80
     ▼
┌─────────────────┐
│   Web ASG       │ (Public Subnet, 1 AZ)
│  EC2 Instances  │
└────┬────────────┘
     │ Port 80
     ▼
┌─────────────────┐
│   App ASG       │ (Private Subnet, 1 AZ)
│  EC2 Instances  │
└────┬────────────┘
     │ Port 3306
     ▼
┌─────────────────┐
│   RDS MariaDB   │ (Private Subnet, Single-AZ)
└─────────────────┘
```

---

## ✅ Key Requirements

### Network Infrastructure

- ✅ Multi-region VPC architecture (Primary + Spare)
- ✅ Multi-AZ deployment in primary region
- ✅ Single-AZ cost-optimized spare region
- ✅ Regional NAT Gateway (no EIP required)
- ✅ Proper security group isolation (3-tier architecture)

### Load Balancing

- ✅ Application Load Balancer (Primary region - Web & App tiers)
- ✅ Network Load Balancer (Spare region - Web tier)
- ✅ Health checks and target group configuration
- ✅ Auto Scaling Group integration

### Database

- ✅ RDS MariaDB (Single-AZ due to sandbox restrictions)
- ✅ Private subnet placement
- ✅ Security group isolation

### Monitoring & Alerts

- ✅ CloudWatch CPU utilization alarms
- ✅ SNS email notifications
- ✅ Health checks for failover

### Disaster Recovery

- ✅ Route 53 failover routing
- ✅ CloudFront origin group failover
- ✅ Automatic failover on primary region failure

---

## 📊 Project Scope Visualization

```
┌─────────────────────────────────────────────────────────────┐
│                    PROJECT SCOPE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Infrastructure Design                                   │
│     ├── Network Architecture                               │
│     ├── Security Configuration                             │
│     └── Load Balancing Strategy                            │
│                                                             │
│  ✅ Infrastructure as Code                                 │
│     ├── Primary Region CloudFormation                      │
│     ├── Spare Region CloudFormation                        │
│     └── Global Components CloudFormation                  │
│                                                             │
│  ✅ Documentation                                           │
│     ├── Architecture Diagrams                              │
│     ├── Technical Rationale                                │
│     ├── Cost & Size Estimation                             │
│     ├── Manual Deployment Guide                            │
│     └── README                                             │
│                                                             │
│  ❌ NOT IN SCOPE:                                           │
│     ├── Application Code Development                       │
│     ├── CI/CD Pipeline Setup                               │
│     ├── Application Monitoring (beyond CloudWatch)         │
│     └── Domain Registration                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---
