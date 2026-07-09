# CI/CD Guide

## What is CI (Continuous Integration)?

**Continuous Integration (CI)** is the practice of frequently integrating code changes into a shared repository. Whenever a developer pushes code, an automated pipeline is triggered to validate the changes.

The CI pipeline typically performs the following tasks:

- Checkout the latest source code
- Compile the application
- Execute unit tests
- Perform static code analysis
- Build the application
- Publish the build artifact

### CI Workflow

```text
Developer
    │
git push
    │
    ▼
Git Repository (GitHub/GitLab/Bitbucket)
    │
    ▼
CI Server (Jenkins/GitHub Actions/GitLab CI)
    │
    ├── Checkout Code
    ├── Compile
    ├── Run Unit Tests
    ├── Static Code Analysis
    ├── Build Application
    └── Publish Artifact
```

### Benefits of CI

- Detects bugs early
- Prevents broken code from reaching the main branch
- Reduces merge conflicts
- Ensures every commit is buildable
- Improves overall code quality

---

# What is CD?

CD stands for either:

- **Continuous Delivery**
- **Continuous Deployment**

Although both automate software delivery, they differ in the final production deployment step.

---

# Continuous Delivery

Continuous Delivery extends CI by automatically deploying the validated application to development, testing, or staging environments.

The application is always **production-ready**, but **deploying to production requires manual approval**.

### Continuous Delivery Workflow

```text
Developer Push
       │
       ▼
CI Pipeline
       │
       ▼
Deploy to Development
       │
       ▼
Integration Tests
       │
       ▼
Deploy to QA / Staging
       │
       ▼
Manual Approval
       │
       ▼
Production
```

### Benefits

- Production-ready builds at all times
- Reduced deployment risk
- Human approval before production
- Faster release cycles

---

# Continuous Deployment

Continuous Deployment is the next step after Continuous Delivery.

Every successful build is automatically deployed to production without any manual intervention.

### Continuous Deployment Workflow

```text
Developer Push
       │
       ▼
CI Pipeline
       │
       ▼
Automated Testing
       │
       ▼
Deploy to Production
```

### Benefits

- Fully automated releases
- Faster feature delivery
- No manual deployment effort
- Frequent production deployments

---

# CI vs Continuous Delivery vs Continuous Deployment

| Feature | Continuous Integration | Continuous Delivery | Continuous Deployment |
|----------|------------------------|---------------------|-----------------------|
| Build Automatically | ✅ | ✅ | ✅ |
| Run Automated Tests | ✅ | ✅ | ✅ |
| Deploy to Development | Optional | ✅ | ✅ |
| Deploy to QA/Staging | Optional | ✅ | ✅ |
| Manual Approval Before Production | ❌ | ✅ | ❌ |
| Automatic Production Deployment | ❌ | ❌ | ✅ |

---

# Real-World Spring Boot CI/CD Example

Suppose you develop a new REST API.

## Step 1: Push Code

```bash
git add .
git commit -m "Add User Profile API"
git push origin feature/profile-api
```

---

## Step 2: Continuous Integration

The CI server automatically performs:

1. Checkout source code
2. Build the project

```bash
mvn clean install
```

3. Execute JUnit tests
4. Perform SonarQube analysis
5. Build Docker image
6. Push Docker image to the container registry

If any step fails, the pipeline stops and the developer is notified.

---

## Step 3: Continuous Delivery

After a successful CI build:

- Deploy the Docker image to the Development environment
- Run integration tests
- Deploy to QA/Staging
- Wait for manual approval
- Deploy to Production

Workflow:

```text
Docker Image
      │
      ▼
Development
      │
      ▼
Integration Testing
      │
      ▼
QA/Staging
      │
      ▼
Manual Approval
      │
      ▼
Production
```

---

## Step 4: Continuous Deployment

The process is identical to Continuous Delivery except that the manual approval step is removed.

Workflow:

```text
Docker Image
      │
      ▼
Development
      │
      ▼
Testing
      │
      ▼
Production
```

---

# Common CI/CD Tools

| Category | Tools |
|----------|-------|
| Version Control | Git, GitHub, GitLab, Bitbucket |
| CI Servers | Jenkins, GitHub Actions, GitLab CI/CD, CircleCI |
| Build Tools | Maven, Gradle |
| Testing | JUnit, Mockito, TestNG |
| Code Quality | SonarQube, Checkstyle, SpotBugs |
| Artifact Repository | Nexus, Artifactory |
| Containerization | Docker |
| Container Registry | Docker Hub, Amazon ECR, Google Artifact Registry |
| Deployment | Kubernetes, Helm, Argo CD, Spinnaker |

---

# Interview Answer (1–2 Minutes)

> **Continuous Integration (CI)** is the practice of frequently integrating code into a shared repository. Every code commit automatically triggers a pipeline that compiles the application, executes automated tests, performs static code analysis, and generates build artifacts. This helps detect integration issues early and ensures the application remains in a healthy state.

> **Continuous Delivery** extends CI by automatically deploying validated builds to development, testing, or staging environments. The application is always ready for production, but deployment to production requires manual approval.

> **Continuous Deployment** goes one step further by automatically deploying every successful build to production without any manual intervention. As long as all automated tests and quality checks pass, the release is deployed automatically.

---

# Key Takeaways

- **CI** = Automatically build and test every code change.
- **Continuous Delivery** = Automatically prepare releases; production deployment requires manual approval.
- **Continuous Deployment** = Automatically release every successful build to production.
- CI improves code quality, while CD accelerates and standardizes software delivery.
