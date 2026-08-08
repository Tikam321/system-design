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
