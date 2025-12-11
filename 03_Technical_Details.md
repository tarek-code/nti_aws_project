# Technical Details

## Design Rationale for High Availability, Disaster Recovery, and Scalability

---

## 🎯 Design Principles

```
┌─────────────────────────────────────────────────────────────┐
│                    DESIGN PRINCIPLES                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  🎯 High Availability                                       │
│     ├── Multi-AZ deployment (Primary region)                │
│     ├── Load balancers for traffic distribution             │
│     └── Auto Scaling Groups for instance resilience        │
│                                                              │
│  🎯 Disaster Recovery                                        │
│     ├── Multi-region architecture                           │
│     ├── Route 53 failover routing                           │
│     └── CloudFront origin group failover                    │
│                                                              │
│  🎯 Scalability                                              │
│     ├── Auto Scaling Groups                                  │
│     ├── Dynamic scaling policies                            │
│     └── Load balancer distribution                          │
│                                                              │
│  🎯 Security                                                 │
│     ├── 3-tier network isolation                            │
│     ├── Security group defense in depth                     │
│     └── Private subnets for sensitive tiers                 │
│                                                              │
│  🎯 Cost Optimization                                        │
│     ├── Regional NAT Gateway (vs multiple zonal)            │
│     ├── Single-AZ spare region                              │
│     └── Right-sized instances                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏗️ High Availability Architecture

### Multi-AZ Deployment Strategy

```
┌─────────────────────────────────────────────────────────────┐
│              PRIMARY REGION - MULTI-AZ DESIGN                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────┐    ┌──────────────────────┐      │
│  │   Availability Zone A│    │  Availability Zone B│      │
│  │                      │    │                      │      │
│  │  Web Subnet          │    │  Web Subnet          │      │
│  │  App Subnet          │    │  App Subnet          │      │
│  │  DB Subnet           │    │  DB Subnet           │      │
│  │                      │    │                      │      │
│  │  ┌──────────────┐   │    │  ┌──────────────┐   │      │
│  │  │ Web Instance │   │    │  │ Web Instance │   │      │
│  │  │ App Instance │   │    │  │ App Instance │   │      │
│  │  └──────────────┘   │    │  └──────────────┘   │      │
│  └──────────────────────┘    └──────────────────────┘      │
│           │                            │                    │
│           └────────────┬───────────────┘                    │
│                        │                                     │
│                        ▼                                     │
│              ┌──────────────────┐                           │
│              │  Application     │                           │
│              │  Load Balancer   │                           │
│              │  (2 AZs Required)│                           │
│              └──────────────────┘                           │
│                                                              │
│  ✅ If AZ-A fails → Traffic routes to AZ-B                  │
│  ✅ If AZ-B fails → Traffic routes to AZ-A                  │
│  ✅ ALB automatically distributes traffic                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why Multi-AZ?**

- ✅ **ALB Requirement:** Application Load Balancer requires subnets in at least 2 AZs
- ✅ **Fault Tolerance:** If one AZ fails, services continue in the other AZ
- ✅ **High Availability:** No single point of failure at AZ level
- ✅ **Best Practice:** AWS recommends Multi-AZ for production workloads

---

## 🌐 Regional NAT Gateway Design

```
┌─────────────────────────────────────────────────────────────┐
│              REGIONAL NAT GATEWAY BENEFITS                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Traditional Approach (Zonal NAT):                          │
│  ┌──────────────┐    ┌──────────────┐                      │
│  │  NAT Gateway │    │  NAT Gateway │                      │
│  │   (AZ-A)     │    │   (AZ-B)     │                      │
│  │              │    │              │                      │
│  │  Cost: $66/mo│    │  Cost: $66/mo│                      │
│  └──────────────┘    └──────────────┘                      │
│         Total: $132/month                                   │
│                                                              │
│  Our Approach (Regional NAT):                               │
│  ┌──────────────────────────────────────┐                  │
│  │     Regional NAT Gateway              │                  │
│  │  (Automatic Multi-AZ Expansion)       │                  │
│  │                                       │                  │
│  │  Cost: $33/month                      │                  │
│  │  ✅ Automatic HA                       │                  │
│  │  ✅ No EIP Required                    │                  │
│  │  ✅ Simplified Management              │                  │
│  └──────────────────────────────────────┘                  │
│                                                              │
│  💰 Cost Savings: 50% reduction ($66/month saved)           │
│  ✅ High Availability: Built-in Multi-AZ                    │
│  ✅ Simplicity: Single NAT Gateway to manage                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why Regional NAT Gateway?**

- ✅ **Cost Efficiency:** 50% cost savings vs zonal NAT Gateways
- ✅ **Automatic HA:** Automatically expands to all AZs where workloads exist
- ✅ **No EIP Required:** Simplifies configuration
- ✅ **Simplified Management:** One NAT Gateway instead of multiple

---

## 🔄 Disaster Recovery Strategy

### Multi-Region Failover Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    DR ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal Operation:                                           │
│  ┌──────────┐                                               │
│  │   User   │                                               │
│  └────┬─────┘                                               │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────┐                                       │
│  │   Route 53       │ ──► Primary ALB (us-east-1) ✅        │
│  │   Health Check   │                                       │
│  │   Status: Healthy│                                       │
│  └──────────────────┘                                       │
│                                                              │
│  ────────────────────────────────────────────────────────   │
│                                                              │
│  Failover Scenario:                                          │
│  ┌──────────┐                                               │
│  │   User   │                                               │
│  └────┬─────┘                                               │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────┐                                       │
│  │   Route 53       │ ──► Spare NLB (us-west-2) ✅         │
│  │   Health Check   │                                       │
│  │   Status: Failed │  (Primary region down)                │
│  └──────────────────┘                                       │
│                                                              │
│  ✅ Automatic Failover: < 3 minutes                         │
│  ✅ Zero Manual Intervention                                 │
│  ✅ Geographic Redundancy                                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why Multi-Region DR?**

- ✅ **Geographic Redundancy:** Protects against regional outages
- ✅ **Automatic Failover:** Route 53 health checks trigger failover automatically
- ✅ **RTO (Recovery Time Objective):** < 3 minutes
- ✅ **RPO (Recovery Point Objective):** Near-zero (depends on data replication)

---

## 🌐 Route 53 Failover Design

```
┌─────────────────────────────────────────────────────────────┐
│              ROUTE 53 FAILOVER MECHANISM                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  HOW IT WORKS                                         │  │
│  │  ───────────────────────────────────────────────────  │  │
│  │                                                       │  │
│  │  1. Health Check Configuration:                       │  │
│  │     • Monitors Primary ALB DNS name                   │  │
│  │     • Protocol: HTTPS (port 443)                      │  │
│  │     • Path: / (or /health)                           │  │
│  │     • Interval: 30 seconds                           │  │
│  │     • Failure Threshold: 3 consecutive failures     │  │
│  │     • Regions: Multiple regions for redundancy       │  │
│  │                                                       │  │
│  │  2. Primary Record (Failover: PRIMARY):               │  │
│  │     • Always checked first                           │  │
│  │     • Health check attached                           │  │
│  │     • If healthy → Route traffic to Primary ALB      │  │
│  │     • If unhealthy → Don't route (failover)          │  │
│  │                                                       │  │
│  │  3. Secondary Record (Failover: SECONDARY):          │  │
│  │     • Only used when Primary is unhealthy            │  │
│  │     • No health check needed                          │  │
│  │     • Automatic failover when Primary fails          │  │
│  │                                                       │  │
│  │  4. Failover Timeline:                                │  │
│  │     • Health check detects failure: ~90 seconds      │  │
│  │     • Route 53 updates DNS: ~60 seconds              │  │
│  │     • DNS propagation: ~60 seconds                    │  │
│  │     • Total: < 3 minutes                              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ✅ Benefits:                                                │
│     • Automatic failover (no manual intervention)           │
│     • Health check based (not just DNS)                     │
│     • Fast failover (< 3 minutes)                           │
│     • Geographic redundancy                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why Route 53 Failover?**
- ✅ **Automatic:** No manual intervention required
- ✅ **Health-Based:** Monitors actual application health, not just DNS
- ✅ **Fast:** Failover in < 3 minutes
- ✅ **Reliable:** Multiple health check regions for redundancy

---

## ☁️ CloudFront CDN Design

```
┌─────────────────────────────────────────────────────────────┐
│              CLOUDFRONT ORIGIN GROUP FAILOVER                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  HOW IT WORKS                                         │  │
│  │  ───────────────────────────────────────────────────  │  │
│  │                                                       │  │
│  │  1. Origin Group Configuration:                      │  │
│  │     • Primary Origin: Primary ALB (HTTPS)            │  │
│  │     • Secondary Origin: Spare NLB (HTTP)             │  │
│  │     • Failover Criteria: 500, 502, 503, 504 errors  │  │
│  │                                                       │  │
│  │  2. Request Flow:                                     │  │
│  │     • User requests content from CloudFront          │  │
│  │     • CloudFront checks cache first                  │  │
│  │     • If cache miss → Forward to Primary Origin     │  │
│  │     • If Primary returns 5xx → Failover to Secondary │  │
│  │                                                       │  │
│  │  3. Caching Benefits:                                 │  │
│  │     • Static content cached at edge locations        │  │
│  │     • Reduced latency for users                      │  │
│  │     • Reduced load on origin servers                 │  │
│  │     • Lower data transfer costs                      │  │
│  │                                                       │  │
│  │  4. Security Features:                                │  │
│  │     • DDoS protection                                │  │
│  │     • SSL/TLS termination                            │  │
│  │     • WAF integration (optional)                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ✅ Benefits:                                                │
│     • Global content delivery (low latency)                 │
│     • Origin failover (redundancy)                          │
│     • DDoS protection                                       │
│     • Cost optimization (caching)                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why CloudFront?**
- ✅ **Performance:** Global edge locations reduce latency
- ✅ **Reliability:** Origin group failover provides redundancy
- ✅ **Security:** Built-in DDoS protection
- ✅ **Cost:** Caching reduces origin load and data transfer costs

---

## 📈 Scalability Design

### Auto Scaling Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                    AUTO SCALING ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              DYNAMIC SCALING POLICY                   │  │
│  │                                                       │  │
│  │  Metric: Average CPU Utilization                     │  │
│  │  Target: 50%                                         │  │
│  │                                                       │  │
│  │  ┌──────────────┐                                    │  │
│  │  │ CPU > 50%    │ ──► Scale Out (+1 instance)       │  │
│  │  └──────────────┘                                    │  │
│  │                                                       │  │
│  │  ┌──────────────┐                                    │  │
│  │  │ CPU < 50%    │ ──► Scale In (-1 instance)        │  │
│  │  └──────────────┘                                    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Capacity Limits:                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Web ASG:  Min: 2  │  Desired: 2  │  Max: 4         │  │
│  │  App ASG:  Min: 2  │  Desired: 2  │  Max: 4         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ✅ Automatic scaling based on demand                        │
│  ✅ Cost optimization (scale down during low traffic)       │
│  ✅ Performance optimization (scale up during high traffic)  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why Auto Scaling?**

- ✅ **Automatic Scaling:** Responds to traffic changes automatically
- ✅ **Cost Optimization:** Scales down during low traffic periods
- ✅ **Performance:** Scales up to handle traffic spikes
- ✅ **High Availability:** Maintains minimum capacity for redundancy

---

## 🔒 Security Architecture

### 3-Tier Network Isolation

```
┌─────────────────────────────────────────────────────────────┐
│                    SECURITY ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  TIER 1: WEB (Public Subnets)                         │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │ Security Group: Web-SG                          │  │  │
│  │  │ Inbound:  80, 443 from 0.0.0.0/0               │  │  │
│  │  │ Outbound: All traffic                           │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ✅ Public-facing, internet accessible               │  │
│  └──────────────────────────────────────────────────────┘  │
│                    │                                        │
│                    │ Port 80 (SG reference)                │
│                    ▼                                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  TIER 2: APP (Private Subnets)                       │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │ Security Group: App-SG                          │  │  │
│  │  │ Inbound:  80 from App-ALB-SG                   │  │  │
│  │  │ Outbound: All traffic                           │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ✅ Isolated from internet                            │  │
│  │  ✅ Only accessible from Web tier                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                    │                                        │
│                    │ Port 3306 (SG reference)              │
│                    ▼                                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  TIER 3: DB (Private Subnets)                        │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │ Security Group: DB-SG                           │  │  │
│  │  │ Inbound:  3306 from App-SG                     │  │  │
│  │  │ Outbound: All traffic                           │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ✅ Complete isolation from internet                  │  │
│  │  ✅ Only accessible from App tier                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ✅ Defense in Depth: Multiple security layers              │
│  ✅ Least Privilege: Only necessary ports open              │
│  ✅ Security Group References: No CIDR-based rules          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why 3-Tier Security?**

- ✅ **Defense in Depth:** Multiple security layers
- ✅ **Least Privilege:** Each tier only receives necessary traffic
- ✅ **Isolation:** Database completely isolated from internet
- ✅ **Security Group References:** More secure than CIDR-based rules

---

## 💰 Cost Optimization Strategies

```
┌─────────────────────────────────────────────────────────────┐
│                    COST OPTIMIZATION                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Strategy 1: Regional NAT Gateway                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Traditional: 2 × Zonal NAT = $132/month            │  │
│  │  Our Design:  1 × Regional NAT = $66/month           │  │
│  │  Savings: $66/month (50%)                            │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Strategy 2: Single-AZ Spare Region                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Primary: 2 AZs = Full redundancy                    │  │
│  │  Spare:   1 AZ  = Cost-optimized standby            │  │
│  │  Savings: 50% reduction in spare region costs        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Strategy 3: Right-Sized Instances                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Web/App: t3.micro (1 vCPU, 1 GB RAM)                │  │
│  │  DB:      db.t3.micro (2 vCPU, 1 GB RAM)             │  │
│  │  ✅ Sufficient for sandbox/testing                    │  │
│  │  ✅ Can scale up for production                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Strategy 4: Auto Scaling                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Scale down during low traffic → Cost savings        │  │
│  │  Scale up during high traffic → Performance          │  │
│  │  ✅ Pay only for what you use                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Design Decision Summary

| Design Aspect               | Choice                      | Rationale                                         |
| --------------------------- | --------------------------- | ------------------------------------------------- |
| **Multi-AZ**          | ✅ Primary Region           | ALB requirement + High availability               |
| **Single-AZ**         | ✅ Spare Region             | Cost optimization for standby                     |
| **NAT Gateway**       | Regional                    | 50% cost savings + Automatic HA                   |
| **Web Load Balancer** | ALB (Primary) / NLB (Spare) | L7 features (Primary) / Single-AZ support (Spare) |
| **App Load Balancer** | Internal ALB (Primary)      | Proper load balancing for app tier                |
| **Security Groups**   | SG References               | More secure than CIDR-based rules                 |
| **Auto Scaling**      | Target Tracking (CPU 50%)   | Automatic scaling based on demand                 |
| **Database**          | Single-AZ MariaDB           | Sandbox restriction + Cost optimization           |

---
