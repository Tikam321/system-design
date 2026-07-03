# Application Deployment (Interview Notes)

# 1. Frontend Deployment (AWS)

## Deployment Flow

```text
Developer
      │
      ▼
Git Repository
      │
      ▼
Jenkins CI Pipeline
      │
      ▼
Build React Application
      │
      ▼
Deploy to AWS
      │
      ├── Development Environment (EC2)
      │
      └── Staging / Production
                │
                ▼
          Bastion (Jump) Server
                │
                ▼
       Private EC2 Instances (VPC)
```

## Deployment Process

For our frontend application, we use a **Jenkins CI pipeline** to build the React application.

We maintain multiple deployment environments:

* **Development**
* **Staging**
* **Production**

### Development Environment

The frontend build is deployed directly to **EC2 instances** used for development.

### Staging & Production

For **Staging** and **Production**, the application is deployed inside a **Virtual Private Cloud (VPC)**.

The production servers are hosted on **private EC2 instances**, so they are not directly accessible from the internet. Deployment and administrative access are performed through a **Bastion (Jump) Server**, which provides secure access to the private infrastructure.

### Interview Summary

> We build the frontend application using a Jenkins CI pipeline. The build is deployed to different AWS environments. Development deployments are made directly to EC2 instances, while Staging and Production deployments are hosted on private EC2 instances inside a VPC and are accessed securely through a Bastion (Jump) Server.

---

# 2. Backend Deployment (GitOps + Kubernetes)

## Deployment Flow

```text
Developer
      │
      ▼
Git Repository
      │
      ▼
CI Pipeline
      │
      ├── Build Spring Boot Application
      ├── Run Unit Tests
      ├── Create Container Image (Google Jib)
      └── Push Image to Container Registry
                    │
                    ▼
Update Helm values.yaml / Kubernetes Deployment Manifest
                    │
                    ▼
GitOps Repository
                    │
                    ▼
Argo CD
                    │
                    ▼
Kubernetes Cluster
                    │
                    ▼
Rolling Deployment
                    │
                    ▼
New Application Pods
```

## Deployment Process

For backend microservices, we follow a **GitOps-based CI/CD deployment process**.

1. Developers push code to the Git repository.
2. The CI pipeline builds the Spring Boot application.
3. **Google Jib** creates the container image without requiring a Dockerfile.
4. The image is pushed to the **Container Registry**.
5. The CI pipeline updates the **image tag** in the **Helm `values.yaml`** or **Kubernetes Deployment manifest**.
6. The updated configuration is committed to the **GitOps repository**.
7. **Argo CD** continuously monitors the Git repository and detects configuration changes.
8. Argo CD synchronizes the changes with the Kubernetes cluster.
9. Kubernetes performs a **Rolling Deployment**, replacing old pods with new pods with minimal or zero downtime.

---

# Common Interview Answer

### How do you deploy your applications?

> **For the frontend, we use a Jenkins CI pipeline to build the React application. We maintain Development, Staging, and Production environments. Development builds are deployed directly to EC2 instances, while Staging and Production are deployed to private EC2 instances inside a VPC and accessed securely through a Bastion (Jump) Server.**
>
> **For the backend, we follow a GitOps-based deployment process. Google Jib builds the container image, which is pushed to a container registry. The CI pipeline updates the image tag in the Helm configuration, Argo CD automatically synchronizes the changes with the Kubernetes cluster, and Kubernetes performs a rolling deployment with minimal downtime.**
