# AWS Interview Questions & Answers

A structured set of AWS interview questions covering fundamentals, core services, networking, security, databases, and scenario-based questions.

---

## 1. AWS Fundamentals

**Q1. What is AWS and what are its main advantages?**
AWS (Amazon Web Services) is a cloud computing platform offering on-demand compute, storage, database, networking, and other IT resources over the internet, on a pay-as-you-go basis.
Key advantages:
- No upfront capital expense for hardware
- Elastic scaling (up or down based on demand)
- Global infrastructure (Regions, Availability Zones, Edge Locations)
- Pay only for what you use
- High availability and fault tolerance built into most services

**Q2. What are Regions, Availability Zones (AZs), and Edge Locations?**
- **Region**: A geographic area (e.g., `us-east-1`) containing multiple isolated data centers.
- **Availability Zone**: One or more discrete data centers within a Region, each with independent power, cooling, and networking, connected via low-latency links. Used for high availability.
- **Edge Location**: A CloudFront/Route 53 endpoint used to cache content closer to end users for lower latency.

**Q3. What is the AWS Shared Responsibility Model?**
AWS is responsible for "security **of** the cloud" (physical infrastructure, hardware, global network, virtualization layer). The customer is responsible for "security **in** the cloud" (data encryption, IAM policies, OS patching for EC2, network configuration, application-level security). The split varies by service (e.g., more customer responsibility in EC2, less in Lambda/managed services).

**Q4. Difference between scalability and elasticity?**
- **Scalability**: The ability of a system to handle increased load by adding resources (can be manual or automatic).
- **Elasticity**: The ability to automatically scale resources up **and down** based on demand, so you pay only for what's needed at any given time. Elasticity is a subset of scalability specific to cloud environments.

---

## 2. Compute (EC2, Lambda, ECS/EKS)

**Q5. What is Amazon EC2?**
EC2 (Elastic Compute Cloud) provides resizable virtual servers (instances) in the cloud. You choose the instance type (CPU/memory/network profile), OS, storage, and networking configuration.

**Q6. What are EC2 instance purchasing options?**
- **On-Demand**: Pay per hour/second, no commitment — good for unpredictable workloads.
- **Reserved Instances**: 1 or 3-year commitment for a significant discount — good for steady-state workloads.
- **Spot Instances**: Bid on unused EC2 capacity at up to 90% discount, but can be reclaimed by AWS with short notice — good for fault-tolerant, flexible workloads.
- **Savings Plans**: Flexible pricing model offering discounts in exchange for a consistent usage commitment ($/hour) over 1 or 3 years.
- **Dedicated Hosts/Instances**: Physical servers dedicated to a single customer — used for compliance or licensing requirements.

**Q7. What is Auto Scaling and why is it used?**
Auto Scaling automatically adjusts the number of EC2 instances in a group based on demand (CPU utilization, request count, custom CloudWatch metrics, or schedules). It ensures application availability while optimizing cost by scaling out during high demand and scaling in during low demand.

**Q8. What is an Elastic Load Balancer (ELB)?**
ELB automatically distributes incoming application traffic across multiple targets (EC2 instances, containers, IP addresses) in one or more AZs. Types:
- **Application Load Balancer (ALB)**: Layer 7 (HTTP/HTTPS), supports path/host-based routing.
- **Network Load Balancer (NLB)**: Layer 4 (TCP/UDP), ultra-low latency, handles millions of requests/sec.
- **Gateway Load Balancer (GWLB)**: Used to deploy third-party virtual appliances (firewalls, IDS/IPS).

**Q9. What is AWS Lambda?**
Lambda is a serverless compute service that runs code in response to events (e.g., API calls, S3 uploads, DynamoDB streams) without provisioning or managing servers. You're billed based on execution time and memory used. Ideal for event-driven, short-duration workloads.

**Q10. When would you choose ECS/EKS over Lambda?**
Choose containers (ECS/EKS) when you need long-running processes, complex networking, fine-grained control over the runtime environment, or workloads that exceed Lambda's execution time limit (15 minutes) or need persistent connections/state. Lambda is better for short, event-driven, bursty workloads with minimal operational overhead.

**Q11. What's the difference between ECS and EKS?**
- **ECS (Elastic Container Service)**: AWS's native container orchestration service, simpler to set up, tightly integrated with AWS services.
- **EKS (Elastic Kubernetes Service)**: Managed Kubernetes service — use when you need Kubernetes-specific features, portability across clouds, or already have Kubernetes expertise/tooling.

---

## 3. Storage

**Q12. What are the main AWS storage types?**
- **Object Storage (S3)**: Store files/objects with metadata, accessed via API/HTTP — not a traditional filesystem.
- **Block Storage (EBS)**: Virtual hard disks attached to a single EC2 instance — used for OS/databases needing low-latency access.
- **File Storage (EFS/FSx)**: Shared, POSIX-compliant file systems that can be mounted by multiple EC2 instances simultaneously.

**Q13. What are S3 storage classes?**
- **S3 Standard**: Frequently accessed data, low latency.
- **S3 Intelligent-Tiering**: Automatically moves objects between tiers based on access patterns.
- **S3 Standard-IA / One Zone-IA**: Infrequently accessed data, cheaper storage but retrieval fees.
- **S3 Glacier Instant/Flexible/Deep Archive**: Long-term archival, cheapest storage, retrieval times range from milliseconds to hours.

**Q14. What is S3 versioning and why use it?**
Versioning keeps multiple variants of an object in the same bucket, protecting against accidental deletion/overwrites. Once enabled, it can be suspended but not fully disabled. Often paired with MFA Delete for extra protection.

**Q15. Difference between EBS and S3?**
- **EBS**: Block storage, attached to a single EC2 instance (like a hard drive), used for OS, databases, low-latency workloads.
- **S3**: Object storage, accessed over the network via API, highly durable (11 nines), used for backups, static content, data lakes — not directly mountable as a filesystem by default.

**Q16. What is the durability and availability of S3?**
S3 Standard offers **99.999999999% (11 nines) durability** and **99.99% availability**, achieved by automatically replicating data across multiple AZs within a Region.

---

## 4. Networking (VPC)

**Q17. What is a VPC?**
A Virtual Private Cloud is a logically isolated virtual network within AWS where you can launch resources, define IP address ranges, subnets, route tables, and gateways — giving you full control over your networking environment.

**Q18. Difference between a public and private subnet?**
- **Public subnet**: Has a route to an Internet Gateway (IGW), allowing resources with public IPs to communicate directly with the internet.
- **Private subnet**: No direct route to the internet; outbound internet access (if needed) typically goes through a NAT Gateway/Instance in a public subnet.

**Q19. What is a NAT Gateway used for?**
A NAT Gateway allows instances in a private subnet to initiate outbound connections to the internet (e.g., for updates) while preventing unsolicited inbound connections from the internet.

**Q20. Security Groups vs Network ACLs?**
| | Security Group | Network ACL |
|---|---|---|
| Level | Instance-level | Subnet-level |
| State | Stateful (return traffic auto-allowed) | Stateless (must allow both directions) |
| Rules | Allow rules only | Allow and Deny rules |
| Evaluation | All rules evaluated | Rules evaluated in order (by rule number) |

**Q21. What is VPC Peering?**
A networking connection between two VPCs that allows resources to communicate using private IP addresses as if they were on the same network. It's not transitive — if VPC A peers with B, and B peers with C, A cannot reach C through B.

**Q22. What is a Transit Gateway?**
A hub-and-spoke network service that lets you connect thousands of VPCs and on-premises networks through a single gateway, simplifying network architecture compared to managing many individual VPC peering connections.

---

## 5. Security & Identity

**Q23. What is IAM?**
Identity and Access Management (IAM) lets you securely control access to AWS resources by creating users, groups, roles, and policies that define who can do what.

**Q24. Difference between an IAM Role and an IAM User?**
- **IAM User**: Represents a person or application with long-term credentials (password/access keys).
- **IAM Role**: A set of temporary permissions that can be assumed by users, applications, or AWS services (e.g., an EC2 instance assuming a role to access S3) — no long-term credentials involved, which is more secure.

**Q25. What is the principle of least privilege and how does IAM support it?**
It means granting only the permissions required to perform a task, nothing more. IAM supports this via fine-grained policies (JSON documents specifying allowed/denied actions on specific resources), permission boundaries, and Service Control Policies (SCPs) in AWS Organizations.

**Q26. How does AWS encryption work (at rest and in transit)?**
- **At rest**: Services like S3, EBS, RDS support encryption using AWS KMS (Key Management Service) — either AWS-managed keys or customer-managed keys (CMKs).
- **In transit**: Data is encrypted using TLS/SSL between clients and AWS services.

**Q27. What is AWS KMS?**
Key Management Service is a managed service for creating and controlling encryption keys used to encrypt data across AWS services. It integrates with CloudTrail for auditing key usage.

**Q28. What is AWS WAF and Shield?**
- **WAF (Web Application Firewall)**: Protects web applications from common exploits (SQL injection, XSS) by filtering HTTP/HTTPS traffic based on rules.
- **Shield**: DDoS protection service. Shield Standard is free and automatic; Shield Advanced provides enhanced protection and 24/7 support for a fee.

---

## 6. Databases

**Q29. What is Amazon RDS?**
Relational Database Service is a managed service for relational databases (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora) that handles patching, backups, and failover, reducing operational overhead.

**Q30. What is Multi-AZ vs Read Replicas in RDS?**
- **Multi-AZ**: Synchronous replication to a standby instance in another AZ for **high availability/disaster recovery** — automatic failover, not used for read scaling.
- **Read Replicas**: Asynchronous replication to one or more replicas used for **read scaling**; can be promoted to standalone databases but don't provide automatic failover.

**Q31. What is Amazon DynamoDB?**
A fully managed, serverless NoSQL key-value/document database offering single-digit millisecond latency at any scale, with built-in support for auto-scaling, global tables (multi-region replication), and on-demand or provisioned capacity modes.

**Q32. When would you choose DynamoDB over RDS?**
Choose DynamoDB for highly scalable, low-latency workloads with simple access patterns (key-value lookups), unpredictable/massive scale, or when you don't need complex joins/transactions. Choose RDS for relational data with complex queries, joins, and strong consistency/ACID transaction requirements.

**Q33. What is Amazon Aurora?**
A MySQL/PostgreSQL-compatible relational database built for the cloud, offering up to 5x the throughput of MySQL and 3x that of PostgreSQL, with storage that auto-scales up to 128 TB and is replicated across 3 AZs.

---

## 7. Monitoring, Deployment & Management

**Q34. What is Amazon CloudWatch?**
A monitoring and observability service that collects metrics, logs, and events from AWS resources and applications. Used to set alarms, create dashboards, and trigger automated actions (e.g., Auto Scaling).

**Q35. What is AWS CloudTrail?**
A service that logs API calls and account activity across your AWS infrastructure for auditing, compliance, and security analysis — answers "who did what, when, and from where."

**Q36. Difference between CloudWatch and CloudTrail?**
- **CloudWatch**: Focuses on performance monitoring — metrics, logs, alarms (the "what's happening" of resource behavior).
- **CloudTrail**: Focuses on auditing — records API calls/actions taken by users and services (the "who did what").

**Q37. What is Infrastructure as Code (IaC) in AWS, and what tools support it?**
IaC means defining and provisioning infrastructure using code/config files instead of manual steps. AWS tools:
- **CloudFormation**: AWS-native, JSON/YAML templates.
- **AWS CDK (Cloud Development Kit)**: Define infrastructure using familiar programming languages (Python, TypeScript, etc.), which synthesizes to CloudFormation.
- (Third-party: Terraform is also widely used with AWS.)

**Q38. What is the AWS Well-Architected Framework?**
A framework of best practices across six pillars to help design and evaluate cloud architectures:
1. Operational Excellence
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization
6. Sustainability

---

## 8. Scenario-Based Questions

**Q39. How would you design a highly available web application on AWS?**
- Use an ALB to distribute traffic across EC2 instances/containers in **multiple AZs**.
- Use an Auto Scaling Group to handle variable load.
- Use RDS Multi-AZ (or Aurora) for the database layer.
- Store static assets in S3 and serve via CloudFront (CDN).
- Use Route 53 for DNS with health checks for failover.
- Store secrets in Secrets Manager/Parameter Store, and use IAM roles instead of hardcoded credentials.

**Q40. How would you reduce costs on AWS?**
- Right-size instances based on actual utilization (CloudWatch metrics).
- Use Spot Instances for fault-tolerant/batch workloads.
- Use Reserved Instances/Savings Plans for predictable steady-state workloads.
- Use S3 lifecycle policies to move infrequently accessed data to cheaper storage tiers.
- Use Auto Scaling to avoid over-provisioning.
- Use AWS Cost Explorer/Trusted Advisor to identify idle or underutilized resources.

**Q41. A file uploaded to S3 needs to trigger a processing job automatically. How would you design this?**
Use **S3 Event Notifications** to trigger an **AWS Lambda** function on object creation (or send an event to SQS/SNS for further processing/fan-out). The Lambda function can process the file and store results back in S3 or a database, all without provisioning servers.

**Q42. How would you migrate an on-premises database to AWS with minimal downtime?**
Use **AWS Database Migration Service (DMS)** with **Schema Conversion Tool (SCT)** if changing engines. DMS supports continuous data replication (CDC) so the source database stays live during migration, and you cut over to the target (e.g., RDS/Aurora) once fully synced, minimizing downtime.

**Q43. How would you secure sensitive application secrets (API keys, DB passwords) on AWS?**
Use **AWS Secrets Manager** (or Systems Manager Parameter Store for simpler cases) instead of hardcoding secrets or storing them in environment variables/code. Secrets Manager also supports automatic rotation. Access should be controlled via IAM roles/policies, not shared credentials.

---

## 9. API Gateway, Lambda Deep Dive & Messaging

**Q44. What is Amazon API Gateway?**
A fully managed service to create, publish, monitor, and secure APIs at any scale. It acts as the "front door" for applications to access backend logic, often Lambda functions, exposing them as REST, HTTP, or WebSocket APIs.

**Q45. How does API Gateway work with Lambda?**
API Gateway receives the HTTP request → invokes the Lambda function, passing the request data → Lambda processes it and returns a response → API Gateway forwards that response back to the client.

**Q46. What is Lambda Proxy Integration?**
An API Gateway integration type where the entire HTTP request (headers, query params, path params, body, method, etc.) is passed to Lambda as a single event object with no transformation. Lambda must return a complete response object (`statusCode`, `headers`, `body`). It's simple, flexible, and works well with frameworks like Express/Flask/FastAPI.

**Q47. Difference between Lambda Proxy and Non-Proxy (Custom) Integration?**
| Aspect | Proxy Integration | Non-Proxy (Custom) Integration |
|---|---|---|
| Request Mapping | Automatic (full event passed) | You define mapping templates (VTL) |
| Response Mapping | Lambda returns full HTTP response | You define mapping templates |
| Control | Lambda has full control | API Gateway has more control |
| Complexity | Low | Higher |
| When to use | Most common cases | When specific request/response transformation is needed |

**Q48. What is a Cold Start in Lambda?**
When a Lambda function is invoked after being idle (or for the first time), AWS must initialize a new execution environment, causing a delay — this is a **cold start**.
Ways to reduce it:
- Use Provisioned Concurrency
- Keep functions warm with scheduled invocations
- Minimize package/dependency size
- Use lighter runtimes (Node.js, Python vs Java)
- Avoid attaching Lambda to a VPC unless necessary

**Q49. How do you secure an API Gateway + Lambda API?**
- API Keys
- IAM Authorization
- Amazon Cognito User Pools
- Lambda Authorizers (custom authorizers)
- Resource policies
- AWS WAF
- Throttling and usage plans

**Q50. Difference between SQS and SNS?**
- **SQS (Simple Queue Service)**: A message queue — each message is typically processed by a single consumer (point-to-point). Used to decouple and buffer work between producers and consumers.
- **SNS (Simple Notification Service)**: Pub/Sub messaging — one message can be delivered to multiple subscribers at once (fan-out), e.g., to SQS queues, Lambda, email, or SMS.

**Q51. What is S3 Lifecycle Configuration?**
A set of rules on an S3 bucket that automatically manages objects over time, without manual intervention. It can:
- **Transition** objects to cheaper storage classes after a set period (e.g., Standard → Standard-IA after 30 days → Glacier after 90 days → Deep Archive after 180 days).
- **Expire** (delete) objects after a set period.
- Also applies to noncurrent object versions (if versioning is enabled) and incomplete multipart uploads, to avoid paying for abandoned data.
Rules can be scoped to the whole bucket or filtered by prefix/tags. Main benefit: **automated cost optimization**.

---

## Tips for the Interview
- Be ready to **draw architecture diagrams** on a whiteboard/paper for scenario questions.
- Know the **trade-offs** between services (e.g., SQS vs SNS, RDS vs DynamoDB, ECS vs Lambda) — interviewers often care more about *why* than *what*.
- Have 1–2 **real project examples** ready where you used these services, including challenges faced and how you solved them.
- Brush up on **cost optimization** and **security best practices** — these come up in almost every AWS interview regardless of role.
