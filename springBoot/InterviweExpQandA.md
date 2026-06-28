# Java, Spring Boot & System Design Interview Questions (Short & Medium Answers)

## Questions Set 1

### 1. How does Spring AOP intercept and handle exceptions?

Spring AOP creates a proxy around the target object. When a method is invoked, the call first goes through the proxy, which executes the configured advices. If the target method throws an exception, the proxy catches it and invokes the `@AfterThrowing` advice, where you can log the exception, perform auditing, or execute cleanup logic. After the advice executes, the exception is rethrown unless it is explicitly handled.

---

### 2. What is a default method? What is the difference between a default method and a static method?

A **default method** is a method with an implementation inside an interface, introduced in Java 8. It allows adding new methods to interfaces without breaking existing implementations. A **static method** also belongs to an interface but is associated with the interface itself rather than its implementing classes. Default methods can be overridden by implementing classes, whereas static methods cannot be overridden and are accessed using the interface name.

---

### 3. What are the Java 8 features?

Major Java 8 features include Lambda Expressions, Functional Interfaces, Stream API, Default and Static methods in interfaces, Optional class, Method References, Date and Time API (`java.time`), CompletableFuture, Nashorn JavaScript Engine, and improvements to Collections with methods like `forEach()`.

---

### 4. What is the difference between Error and Exception?

An **Exception** represents conditions that an application can recover from, such as `IOException` or `SQLException`. They can be handled using try-catch blocks. An **Error** represents serious problems in the JVM, such as `OutOfMemoryError` or `StackOverflowError`, which applications generally should not handle because they indicate critical system failures.

---

### 5. What is the difference between method overriding and method overloading?

**Method Overloading** means having multiple methods with the same name but different parameter lists within the same class. It is resolved at compile time (compile-time polymorphism). Constructors can also be overloaded. **Method Overriding** occurs when a subclass provides its own implementation of a method already defined in the parent class using the same signature. It is resolved at runtime (runtime polymorphism).

---

### 6. Can multiple threads be created by calling `run()` multiple times on the same Thread object?

No. Calling `run()` directly does not create a new thread; it simply executes the method like a normal method call on the current thread. A new thread is created only when `start()` is invoked. The `start()` method asks the JVM to create a new thread and internally calls `run()`. Also, a Thread object can be started only once; calling `start()` again throws `IllegalThreadStateException`.

---

### 7. What is Spring Security?

Spring Security is a framework that provides authentication, authorization, session management, CSRF protection, password encryption, and protection against common security vulnerabilities. It integrates with JWT, OAuth2, LDAP, and role-based access control to secure Spring applications.

---

### 8. What are AI Prompts?

An AI prompt is the instruction or input given to an AI model to generate a response. A good prompt clearly defines the task, provides necessary context, specifies constraints such as output format or length, and may include examples to improve response quality.

---

### 9. In which Kubernetes component are secrets stored?

Secrets are stored as Kubernetes **Secret** objects. Internally, Kubernetes stores them in **etcd**, which is the cluster's distributed key-value store. Since etcd contains sensitive data, encryption at rest should be enabled for better security.

---

### 10. What is the difference between Waterfall and Agile methodology?

Waterfall is a sequential development model where each phase is completed before moving to the next. It is suitable for projects with stable requirements. Agile is an iterative approach where development happens in short sprints with continuous feedback, allowing frequent changes and faster delivery.

---

### 11. What is etcd?

etcd is a distributed, highly available key-value database used by Kubernetes to store cluster configuration, metadata, secrets, and the current state of the cluster. The Kubernetes API Server reads and writes all cluster information to etcd.

---

### 12. Which collections are mostly used in Java?

The most commonly used collections are **ArrayList** for fast indexed access, **LinkedList** for frequent insertions and deletions, **HashMap** for fast key-value lookups, **HashSet** for storing unique elements, **TreeMap** for sorted key-value pairs, **TreeSet** for sorted unique elements, and **PriorityQueue** for priority-based processing.

---

### 13. What is Optional? Explain methods like `get()` and `isPresent()`.

`Optional` is a container object introduced in Java 8 to avoid `NullPointerException` by explicitly representing the presence or absence of a value. `isPresent()` checks whether a value exists, `get()` returns the value but throws `NoSuchElementException` if empty, `orElse()` provides a default value, `orElseGet()` computes a default lazily, and `orElseThrow()` throws a custom exception when the value is absent.

---

### 14. What is a deadlock? How can it be avoided?

A deadlock occurs when two or more threads wait indefinitely for resources held by each other. It can be avoided by acquiring locks in a consistent order, minimizing nested synchronization, using timeout-based locks (`tryLock()`), reducing lock scope, and preferring concurrent collections when possible.

---

### 15. Which data structures come under the Java Collection Framework?

The Java Collection Framework includes **List**, **Set**, **Queue**, and **Deque** interfaces along with their implementations such as `ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `PriorityQueue`, and `ArrayDeque`. Maps are part of the Collections Framework but do not extend the `Collection` interface.

---

### 16. Which data structures come under the Java Collections Library?

The Collections Library includes all collection interfaces and implementations such as **List, Set, Queue, Deque, Map**, along with utility classes like `Collections`, `Arrays`, iterators, comparators, and concurrent collections like `ConcurrentHashMap` and `CopyOnWriteArrayList`.

---

# Questions Set 2

### 1. What are Quality Gates in Jenkins?

Quality Gates are checkpoints in a CI/CD pipeline that ensure code meets predefined quality standards before deployment. They are commonly integrated with SonarQube to verify metrics like code coverage, bugs, vulnerabilities, code smells, and duplication. If a quality gate fails, Jenkins can stop the pipeline.

---

### 2. What is Thread Pooling?

Thread pooling is a technique where a fixed or dynamic pool of reusable threads executes multiple tasks instead of creating a new thread for every request. It reduces thread creation overhead, improves performance, controls resource usage, and is implemented in Java using the Executor Framework.

---

### 3. What is the difference between Reactive Programming and MVC?

Spring MVC follows a synchronous, thread-per-request model where one thread handles one request until completion. Reactive Programming uses a non-blocking, event-driven model where a small number of threads handle many concurrent requests asynchronously using publishers like `Mono` and `Flux`, making it suitable for high-concurrency applications.

---

### 4. What is React API Context?

The React Context API provides a way to share data across components without passing props through every intermediate component. It is commonly used for themes, authentication, user information, and global application state.

---

### 5. What monitoring tools have you used?

For frontend performance, I have used **Google Lighthouse** to measure metrics like Performance, Accessibility, Best Practices, SEO, Core Web Vitals, and page load times. In cloud environments, **AWS CloudWatch** is commonly used for monitoring application logs, CPU usage, memory, and infrastructure metrics.

---

### 6. What are different Jenkins plugins?

Common Jenkins plugins include Git Plugin, Maven Plugin, Pipeline Plugin, Docker Plugin, Kubernetes Plugin, SonarQube Scanner Plugin, JUnit Plugin, Email Extension Plugin, Credentials Plugin, Blue Ocean Plugin, Slack Plugin, and Artifactory Plugin.

---

# Questions Set 3

### 1. What is Lazy Loading?

Lazy loading is a technique where an object or resource is loaded only when it is actually needed instead of during initialization. In Hibernate, related entities marked with `FetchType.LAZY` are fetched from the database only when accessed, improving performance and reducing unnecessary database queries.

---

### 2. What is a CI/CD Pipeline?

A CI/CD pipeline automates software delivery. **Continuous Integration (CI)** automatically builds and tests code whenever changes are committed. **Continuous Delivery/Deployment (CD)** automates deployment to testing or production environments, ensuring faster and more reliable releases.

---

### 3. Which cloud platform are you using?

In our project, we deploy applications on **AWS EC2** instances. We use **AWS CloudWatch** for monitoring logs, application health, CPU utilization, memory usage, and infrastructure metrics, which helps identify issues and monitor application performance.

---

### 4. How do you support different `Accept` headers for the same endpoint?

Spring supports content negotiation using the `produces` attribute in `@RequestMapping` or `@GetMapping`. Based on the client's `Accept` header, Spring automatically selects the appropriate response format, such as JSON or XML, using the configured message converters.

---

### 5. How is the security layer implemented in your application?

Our application uses JWT-based authentication. Once the user logs in, a JWT token is generated and sent with every request. Spring Security validates the token, authenticates the user, and then the application performs authorization checks based on the user's permissions stored in the database before allowing access to resources.

---

### 6. How do you implement role-based access control?

Authentication is handled using JWT, while authorization is database-driven. After validating the JWT, we query the database to determine which companies and sub-organization codes the logged-in admin is authorized to manage. These permissions are then enforced in the service layer, ensuring the admin can only access or modify data for the assigned organizations.

---

### 7. Explain the request flow from frontend to backend.

The frontend sends an HTTP request with the JWT token in the Authorization header. The request reaches Spring Security filters, where the JWT is validated and the user is authenticated. The request is then forwarded to the controller, which delegates to the service layer for business logic. The service interacts with the repository layer, which communicates with the database. Finally, the response is returned through the controller back to the frontend.

---

### 8. Can you handle exceptions at the consumer level?

Yes. In messaging systems like Kafka, exceptions can be handled at the consumer level using try-catch blocks, error handlers, retry mechanisms, Dead Letter Topics (DLT), or Spring Kafka's `DefaultErrorHandler`, ensuring failed messages are retried or redirected without stopping message consumption.

---

### 9. What is the difference between `@Primary` and `@Qualifier`?

`@Primary` specifies the default bean to inject when multiple beans of the same type exist. `@Qualifier` explicitly tells Spring which bean to inject by its name or qualifier. `@Primary` works at the bean definition level, while `@Qualifier` is used at the injection point for precise bean selection.

---

### 10. What is Lazy Loading?

Lazy loading delays the loading of objects until they are actually required. It improves application performance and reduces memory usage by avoiding unnecessary database queries during initial object retrieval.

---

### 11. What is the `@Transactional` annotation?

`@Transactional` is used to define a transaction boundary. All database operations inside a transactional method execute as a single unit of work. If all operations succeed, the transaction is committed; if an exception occurs, Spring rolls back the transaction, ensuring data consistency.

### 12.  spring bean life cycle
Spring first creates the bean by calling its constructor. Then it injects all the required dependencies. After dependency injection, it executes the initialization logic, such as a method annotated with @PostConstruct. Once initialization is complete, the bean is ready to use. Finally, when the application context is closed, Spring invokes methods annotated with @PreDestroy before removing the bean."
