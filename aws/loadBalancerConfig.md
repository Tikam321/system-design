# 🏗️ AWS Auto Scaling Group (ASG) - Complete Reference Guide

## Region
- **AWS Region:** ap-south-1 (Mumbai)

---

# 1. Learning Environment Configuration

## Auto Scaling Group

| Property | Value |
|----------|-------|
| Launch Template | my-template |
| Instance Type | t3.nano |
| Min Capacity | 1 |
| Desired Capacity | 2 |
| Max Capacity | 4 |
| Health Check | EC2 |
| Grace Period | 300 seconds |
| Cooldown | 300 seconds |

---

## Application Load Balancer

- Internet Facing
- Listener : HTTP (80)
- Cross-Zone Load Balancing Enabled
- Spans Multiple Availability Zones
- DNS Name:

```
http://<alb-name>.ap-south-1.elb.amazonaws.com
```

> **Note:** Route53 is **NOT** required unless using a custom domain.

---

## Target Group

- Protocol : HTTP
- Port : 80
- Health Check : /
- Target Type : Instance

---

## Availability Zones

```
ap-south-1a
    │
    └── EC2 (NGINX)

ap-south-1b
    │
    └── EC2 (NGINX)
```

With Desired Capacity = 2

```
AZ-a -> 1 EC2
AZ-b -> 1 EC2
```

---

# User Data

```bash
#!/bin/bash

dnf update -y
dnf install nginx -y

systemctl enable nginx
systemctl start nginx

echo "<h1>Welcome from Auto Scaling Server</h1>" \
| tee /usr/share/nginx/html/index.html
```

---

# Request Flow

```
                 Client
                    │
                    ▼
        ALB DNS Name (AWS)
                    │
                    ▼
      Application Load Balancer
                    │
                    ▼
           Target Group
             │       │
             ▼       ▼
        EC2(AZ-a) EC2(AZ-b)
             │
             ▼
            NGINX
             │
             ▼
         index.html
```

---

# Complete Architecture

```
                  Client
                     │
                     ▼
        ALB DNS Name (AWS)
                     │
                     ▼
      ┌────────────────────────┐
      │ Application Load Balancer │
      └────────────┬───────────┘
                   │
                   ▼
            Target Group
                   │
      ┌────────────┴────────────┐
      ▼                         ▼

Availability Zone A      Availability Zone B

Public Subnet            Public Subnet

EC2 (NGINX)              EC2 (NGINX)

                   ▲
                   │
          Auto Scaling Group
         Min=1 Desired=2 Max=4
                   │
                   ▼
            Launch Template
```

---

# Production Architecture

```
                     Internet
                         │
                         ▼
                    Amazon Route53
                 (Optional Custom Domain)
                         │
                         ▼
                     AWS WAF
                         │
                         ▼
        Application Load Balancer (HTTPS)
                         │
                         ▼
                   Target Group
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼

   AZ-a              AZ-b              AZ-c

Public Subnet     Public Subnet     Public Subnet
(ALB Node)        (ALB Node)        (ALB Node)

------------------------------------------------------

Private Subnet    Private Subnet    Private Subnet

EC2               EC2               EC2
Spring Boot       Spring Boot       Spring Boot
Docker            Docker            Docker

        ▲
        │
   Auto Scaling Group
 Min=3 Desired=3 Max=12
        │
        ▼
   Launch Template
```

---

# Production ASG Configuration

| Property | Recommended |
|-----------|-------------|
| Instance Type | t3.large / m6i.large |
| Min Capacity | 3 |
| Desired Capacity | 3 |
| Max Capacity | 12 |
| Health Check | ELB |
| Warmup | 120 sec |
| Cooldown | 180 sec |
| Scaling Policy | Target Tracking (CPU 60%) |

---

# Production Security

## ALB Security Group

Inbound

```
443 -> 0.0.0.0/0
80  -> Redirect to HTTPS
```

Outbound

```
All Traffic
```

---

## EC2 Security Group

Inbound

```
80 -> ALB Security Group
```

SSH

```
AWS Systems Manager (Preferred)
```

Outbound

```
All Traffic
```

---

# Typical Production EC2 Types

| Environment | Instance |
|-------------|----------|
| Dev | t3.micro |
| QA | t3.small |
| Staging | t3.medium |
| Production (Small) | t3.large |
| Production (Medium) | m6i.large |
| Compute Intensive | c6i.large |
| Memory Intensive | r6i.large |

---

# Scaling Policy Example

Scale Out

```
CPU > 70% for 5 min

+2 EC2 Instances
```

Scale In

```
CPU < 30% for 10 min

-1 EC2 Instance
```

---

# Best Practices

## Networking

- Use 3 Availability Zones
- Public Subnets for ALB
- Private Subnets for EC2
- NAT Gateway for outbound traffic

---

## Security

- HTTPS only
- AWS WAF
- ACM Certificates
- IAM Roles
- Encrypted EBS
- Security Groups
- AWS Systems Manager instead of SSH

---

## Monitoring

- CloudWatch
- CloudWatch Logs
- SNS Alerts
- AWS X-Ray
- ALB Access Logs

---

# Key Interview Points

✅ One ALB can span multiple Availability Zones.

✅ One Target Group can contain EC2 instances from multiple AZs.

✅ One Auto Scaling Group manages instances across multiple AZs.

✅ Route53 is optional and only needed for custom domains.

✅ ALB provides its own DNS name automatically.

✅ EC2 instances should generally be deployed in private subnets in production.

✅ Health checks should use ELB instead of EC2 in production.

✅ Desired Capacity should ideally be distributed evenly across AZs for high availability.

---

# Request Flow Summary

```
Client
   │
   ▼
Route53 (Optional)
   │
   ▼
AWS WAF
   │
   ▼
Application Load Balancer
   │
   ▼
Target Group
   │
   ▼
Auto Scaling Group
   │
   ▼
EC2 (Spring Boot / NGINX)
   │
   ▼
Response
```


# 🏗️ AWS Auto Scaling Group — Complete Reference Guide
### Mumbai Region (ap-south-1) | Production-Level Infrastructure

---

## 📌 Table of Contents
1. [Your Actual Configuration](#your-actual-configuration)
2. [Architecture Diagram](#architecture-diagram)
3. [Request Flow Diagram](#request-flow-diagram)
4. [Production-Level Ideal Configuration](#production-level-ideal-configuration)
5. [Key Concepts](#key-concepts)
6. [Best Practices](#best-practices)

---

## 1. Your Actual Configuration

### 🔧 Auto Scaling Group — `asg`
| Setting                  | Value                          |
|--------------------------|--------------------------------|
| Region                   | ap-south-1 (Mumbai)            |
| Launch Template          | my-template3                   |
| Instance Type            | t3.nano                        |
| Min Capacity             | 1                              |
| Desired Capacity         | 2                              |
| Max Capacity             | 3                              |
| Health Check Type        | EC2                            |
| Health Check Grace Period| 300 seconds                    |
| Default Cooldown         | 300 seconds                    |
| AZ Distribution Strategy| balanced-best-effort           |
| Deletion Protection      | None                           |

### 🌐 Availability Zones Used
| AZ            | Subnet ID                    | Instances |
|---------------|------------------------------|-----------|
| ap-south-1a   | subnet-0ad2be230b66d6b1e     | 1         |
| ap-south-1b   | subnet-0cac9e3a216423d51     | 1         |

### ⚖️ Load Balancer & Target Group
| Setting        | Value                                          |
|----------------|------------------------------------------------|
| LB Type        | Application Load Balancer (ALB)                |
| LB Count       | 1 (Regional — spans all AZs)                  |
| Target Group   | tg (arn:...:targetgroup/tg/06f207bb0edf11bb)  |
| Health Check   | EC2                                            |

### 🖥️ EC2 Instances (NGINX + index.html)
| Instance ID          | AZ           | Type    | Web Server | Page       |
|----------------------|--------------|---------|------------|------------|
| i-03f029c34b5e1ee89  | ap-south-1a  | t3.nano | NGINX      | index.html |
| i-06d09647e8feaccc0  | ap-south-1b  | t3.nano | NGINX      | index.html |

### 📜 User Data Script (NGINX Setup)
```bash
#!/bin/bash
# Update packages
sudo apt update -y

# Install NGINX
sudo apt install nginx -y

# Create index.html
echo "<html>
  <body>
    <h1>Hello from $(hostname) in $(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)</h1>
  </body>
</html>" | sudo tee /var/www/html/index.html

# Start and enable NGINX
sudo systemctl start nginx
sudo systemctl enable nginx

2. Architecture Diagram
🗺️ Your Setup (2 AZs — ap-south-1a & ap-south-1b)
                        🌍 CLIENT (Browser / User)
                                    |
                                    | HTTP Request (www.example.com)
                                    ▼
                       ┌────────────────────────┐
                       │   Amazon Route 53      │
                       │   (DNS Resolution)     │
                       │                        │
                       │  example.com  ──────►  │
                       │  ALB DNS Name          │
                       └───────────┬────────────┘
                                   |
                                   | Resolves to ALB Endpoint
                                   ▼
              ┌────────────────────────────────────────┐
              │       Application Load Balancer (ALB)  │
              │  ────────────────────────────────────  │
              │  • 1 ALB for entire ap-south-1 region  │
              │  • Listener: Port 80 (HTTP)            │
              │  • ALB Node in ap-south-1a             │
              │  • ALB Node in ap-south-1b             │
              │  • Cross-Zone Load Balancing: ON       │
              └──────────────────┬─────────────────────┘
                                 |
                                 | Forwards traffic
                                 ▼
              ┌────────────────────────────────────────┐
              │          Target Group (tg)             │
              │  ────────────────────────────────────  │
              │  • Protocol : HTTP : Port 80           │
              │  • Target Type: Instance               │
              │  • Health Check: HTTP /                │
              │  • Healthy Threshold: 2                │
              │  • Unhealthy Threshold: 3              │
              └──────────┬─────────────────┬───────────┘
                         |                 |
             ┌───────────┘                 └───────────┐
             ▼                                         ▼
┌─────────────────────────┐           ┌─────────────────────────┐
│    AZ: ap-south-1a      │           │    AZ: ap-south-1b      │
│  ─────────────────────  │           │  ─────────────────────  │
│  Subnet: subnet-0ad2be  │           │  Subnet: subnet-0cac9e  │
│                         │           │                         │
│  ┌─────────────────┐    │           │  ┌─────────────────┐    │
│  │  EC2 (t3.nano)  │    │           │  │  EC2 (t3.nano)  │    │
│  │  NGINX Server   │    │           │  │  NGINX Server   │    │
│  │  index.html     │    │           │  │  index.html     │    │
│  │  Port: 80       │    │           │  │  Port: 80       │    │
│  └─────────────────┘    │           │  └─────────────────┘    │
│                         │           │                         │
│  Auto Scaling Group     │           │  Auto Scaling Group     │
│  Min:1 Des:2 Max:3      │           │  Min:1 Des:2 Max:3      │
└─────────────────────────┘           └─────────────────────────┘

3. Request Flow Diagram
STEP 1:  User types www.example.com in browser
              │
              ▼
STEP 2:  DNS Query → Route 53
         Route 53 returns ALB DNS name
         e.g., my-alb-123456.ap-south-1.elb.amazonaws.com
              │
              ▼
STEP 3:  Request hits ALB on Port 80
         ALB checks Listener Rules
              │
              ▼
STEP 4:  ALB forwards to Target Group (tg)
         Target Group checks healthy instances
              │
         ┌────┴────┐
         ▼         ▼
STEP 5: EC2 (1a)  EC2 (1b)   ← Round Robin / Least Connections
        NGINX     NGINX
        serves    serves
        index.html index.html
              │
              ▼
STEP 6:  Response returned to Client ✅

4. Production-Level Ideal Configuration
🏭 Recommended Production Setup (3 AZs)
                        🌍 CLIENT
                            |
                            ▼
               ┌────────────────────────┐
               │  Amazon Route 53       │
               │  • Latency-based       │
               │    routing             │
               │  • Health checks ON    │
               │  • Failover policy     │
               └───────────┬────────────┘
                           |
                           ▼
          ┌─────────────────────────────────────┐
          │    AWS WAF (Web Application Firewall)│
          │    • SQL Injection protection        │
          │    • XSS protection                 │
          │    • Rate limiting rules            │
          └──────────────────┬──────────────────┘
                             |
                             ▼
          ┌─────────────────────────────────────┐
          │   Application Load Balancer (ALB)   │
          │   • HTTPS Listener (Port 443)       │
          │   • HTTP → HTTPS Redirect           │
          │   • SSL/TLS Certificate (ACM)       │
          │   • Access Logs → S3                │
          │   • Deletion Protection: ON         │
          └──────┬──────────────┬───────────────┘
                 |              |
       ┌─────────┘              └──────────┐
       ▼                                   ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ ap-south-1a  │  │ ap-south-1b  │  │ ap-south-1c  │
│ ──────────── │  │ ──────────── │  │ ──────────── │
│ Private      │  │ Private      │  │ Private      │
│ Subnet       │  │ Subnet       │  │ Subnet       │
│              │  │              │  │              │
│ EC2 (t3.med) │  │ EC2 (t3.med) │  │ EC2 (t3.med) │
│ EC2 (t3.med) │  │ EC2 (t3.med) │  │ EC2 (t3.med) │
│              │  │              │  │              │
│ 2 instances  │  │ 2 instances  │  │ 2 instances  │
└──────────────┘  └──────────────┘  └──────────────┘
       |                 |                 |
       └────────┬────────┘                 |
                └──────────────────────────┘
                           |
                           ▼
          ┌─────────────────────────────────────┐
          │         RDS (Multi-AZ)              │
          │   Primary        Standby            │
          │  ap-south-1a  → ap-south-1b         │
          │  Auto failover in < 60 seconds      │
          └─────────────────────────────────────┘

⚙️ Production ASG Configuration
Setting	Your Setup	Production Ideal
Instance Type	t3.nano	t3.medium / t3.large
Min Capacity	1	3 (1 per AZ)
Desired Capacity	2	6 (2 per AZ)
Max Capacity	3	12 (4 per AZ)
AZs	2	3 (ap-south-1a/b/c)
Health Check Type	EC2	ELB (more accurate)
Health Check Grace Period	300s	120s
Cooldown Period	300s	180s
AZ Distribution Strategy	balanced-best-effort	balanced-only
Deletion Protection	OFF	ON
Instance Warmup	Not set	60-120 seconds
📈 Production Scaling Policies
Policy Type	Trigger	Action
Scale OUT (CPU)	CPU > 70% for 2 minutes	Add 2 instances
Scale IN (CPU)	CPU < 30% for 5 minutes	Remove 1 instance
Scale OUT (Memory)	Memory > 80% for 2 minutes	Add 2 instances
Scale OUT (ALB)	RequestCount > 1000/target/min	Add 2 instances
Scheduled Scale UP	Mon-Fri 9:00 AM IST	Set Desired = 6
Scheduled Scale DN	Mon-Fri 11:00 PM IST	Set Desired = 3
🔒 Production Security Configuration
Component	Your Setup	Production Ideal
EC2 Placement	Public Subnet	Private Subnet
Security Group	Not specified	Allow 80/443 from ALB SG only
ALB Security Group	Not specified	Allow 80/443 from WAF / Internet
SSH Access	Not specified	Via AWS Systems Manager (No SSH)
SSL/TLS	None (HTTP only)	ACM Certificate (HTTPS)
WAF	None	AWS WAF with managed rules
Encryption	None	EBS volumes encrypted (KMS)
🌐 Production VPC Architecture
VPC: 10.0.0.0/16
│
├── Public Subnets (ALB, NAT Gateway)
│   ├── ap-south-1a: 10.0.1.0/24
│   ├── ap-south-1b: 10.0.2.0/24
│   └── ap-south-1c: 10.0.3.0/24
│
├── Private Subnets (EC2 Instances)
│   ├── ap-south-1a: 10.0.11.0/24
│   ├── ap-south-1b: 10.0.12.0/24
│   └── ap-south-1c: 10.0.13.0/24
│
└── Database Subnets (RDS)
    ├── ap-south-1a: 10.0.21.0/24
    ├── ap-south-1b: 10.0.22.0/24
    └── ap-south-1c: 10.0.23.0/24
