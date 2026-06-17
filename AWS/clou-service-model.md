# IaaS vs PaaS vs SaaS

## What is IaaS (Infrastructure as a Service)?

* Provides virtual infrastructure (VMs, Storage, Network).
* You manage the Operating System, Runtime, and Application.

### Example: AWS EC2

Suppose you want to deploy a Spring Boot application.

AWS provides:

* EC2 (Virtual Machine)
* Storage
* Networking

You install:

* Linux
* Java
* Spring Boot
* PostgreSQL
* Your Application

```text id="h6r4xq"
Your App
   ↑
Java
   ↑
Linux
   ↑
AWS EC2
```

**Examples**

* AWS EC2
* Azure Virtual Machines
* Google Compute Engine

---

## What is PaaS (Platform as a Service)?

* Provides a complete platform to develop and deploy applications.
* You only manage your application and data.

### Example: AWS Elastic Beanstalk

You simply upload your Spring Boot JAR.

```bash id="x91w6e"
java -jar app.jar
```

AWS manages:

* Server
* Operating System
* Java Runtime
* Auto Scaling
* Load Balancer
* Deployment

You only write:

```java id="sos9t2"
@RestController
public class EmployeeController {

    @GetMapping("/employees")
    public List<Employee> getEmployees() {
        return service.getEmployees();
    }
}
```

**Examples**

* AWS Elastic Beanstalk
* Google App Engine
* Azure App Service
* Heroku

---

## What is SaaS (Software as a Service)?

* Ready-to-use software.
* You simply use the application.

**Examples**

* Gmail
* Microsoft 365
* Salesforce
* Dropbox

---

## Comparison

| Feature          | IaaS             | PaaS                          | SaaS       |
| ---------------- | ---------------- | ----------------------------- | ---------- |
| You Manage       | OS, Runtime, App | App & Data                    | Nothing    |
| Provider Manages | Infrastructure   | Infrastructure + OS + Runtime | Everything |
| Example          | AWS EC2          | AWS Elastic Beanstalk         | Gmail      |

---

## Easy Analogy

### IaaS = Renting a House 🏠

* You get an empty house.
* You arrange furniture and maintain everything.

### PaaS = Staying in a Hotel 🏨

* Hotel manages the building and services.
* You only use the room.

### SaaS = Watching Netflix 📺

* Just sign in and use it.
* Everything else is managed by the provider.

---

## Interview Answer (30 Seconds)

> **IaaS** provides virtual infrastructure like VMs, storage, and networking. We are responsible for installing and managing the operating system, runtime, and application. A common example is **AWS EC2**.
>
> **PaaS** provides a complete platform where the cloud provider manages the infrastructure, operating system, and runtime. We only deploy and manage our application. A common example is **AWS Elastic Beanstalk**.
>
> **SaaS** provides ready-to-use software over the internet, such as **Gmail** or **Microsoft 365**, where the provider manages everything.
