# 1. Challenges When Migrating Monolith to Microservices

# 1. Distributed System Complexity

In monolith:

```text
Everything runs in one application
```

In microservices:

```text
Multiple services communicate over network
```

Challenges:
- Network latency
- Service failures
- Retry handling

---

# 2. Data Management

Monolith usually has:

```text
Single database
```

Microservices prefer:

```text
Database per service
```

Challenges:
- Data consistency
- Distributed transactions
- Data duplication

---

# 3. Service Communication

Need communication using:

- REST
- Kafka
- gRPC

Challenges:
- Timeout handling
- Service discovery
- API versioning

---

# 4. Deployment Complexity

Now multiple services need:
- CI/CD
- Containerization
- Monitoring

Deployment becomes harder.

---

# 5. Distributed Transactions

ACID transactions across services become difficult.

Need:
- Saga pattern
- Event-driven architecture

---

# 6. Monitoring and Debugging

Harder to trace requests across services.

Need:
- Centralized logging
- Distributed tracing
- Monitoring tools

---

# 7. Security Challenges

Need:
- JWT/OAuth2
- API Gateway
- Secure inter-service communication

---

# 8. Performance Issues

More network calls can increase:
- Latency
- Resource usage

---

# 9. Testing Complexity

Integration and end-to-end testing become more difficult.

---

# 10. Team and Service Boundaries

Improper service splitting can cause:
- Tight coupling
- Chatty communication
- Dependency issues

---

# Interview Ready Answer

> Migrating from monolith to microservices introduces challenges like distributed system complexity, service communication, data consistency, distributed transactions, deployment complexity, monitoring, and security. Managing inter-service communication, database separation, and debugging across multiple services becomes more difficult and requires tools like API Gateway, service discovery, centralized logging, and event-driven patterns.

# 2. How Does Spring Boot Simplify Web Application Development?

Spring Boot simplifies development by reducing configuration and providing production-ready features out of the box.

---

# Key Features

## 1. Auto Configuration

Automatically configures:
- Database
- DispatcherServlet
- Jackson
- Tomcat

based on dependencies.

---

# 2. Embedded Server

Provides embedded:
- Tomcat
- Jetty

No need for external server deployment.

---

# 3. Starter Dependencies

Simplifies dependency management.

Example:

```xml
spring-boot-starter-web
```

includes:
- Spring MVC
- Tomcat
- Jackson

---

# 4. Minimal Configuration

No XML configuration required.

Uses:
- Annotations
- Convention over configuration

---

# 5. Production Ready Features

Provides:
- Actuator
- Health checks
- Metrics
- Monitoring

---

# 6. Easy Microservices Development

Supports:
- REST APIs
- Service discovery
- Config server
- API Gateway

---

# Interview Ready Answer

> Spring Boot simplifies web application development through auto-configuration, embedded servers, starter dependencies, and minimal configuration. It reduces boilerplate code and provides production-ready features like monitoring, health checks, and easy REST API development.


# 3.  # How Does Spring Boot Simplify Web Application Development?

Spring Boot simplifies development by reducing configuration and providing production-ready features out of the box.

---

# Key Features

## 1. Auto Configuration

Automatically configures:
- Database
- DispatcherServlet
- Jackson
- Tomcat

based on dependencies.

---

# 2. Embedded Server

Provides embedded:
- Tomcat
- Jetty

No need for external server deployment.

---

# 3. Starter Dependencies

Simplifies dependency management.

Example:

```xml
spring-boot-starter-web
```

includes:
- Spring MVC
- Tomcat
- Jackson

---

# 4. Minimal Configuration

No XML configuration required.

Uses:
- Annotations
- Convention over configuration

---

# 5. Production Ready Features

Provides:
- Actuator
- Health checks
- Metrics
- Monitoring

---

# 6. Easy Microservices Development

Supports:
- REST APIs
- Service discovery
- Config server
- API Gateway

---

# Interview Ready Answer

> Spring Boot simplifies web application development through auto-configuration, embedded servers, starter dependencies, and minimal configuration. It reduces boilerplate code and provides production-ready features like monitoring, health checks, and easy REST API development.


# 4. Externalized Configuration in Spring Boot

Spring Boot externalized configuration allows application configuration to be kept outside the code so values can change without rebuilding the application.

---

# Why Needed?

Different environments need different configs:

- Dev
- QA
- Prod

Example:
- DB URL
- API keys
- Ports

---

# Common Configuration Sources

| Source | Example |
|---|---|
| application.properties | Local config |
| application.yml | YAML config |
| Environment variables | OS-level config |
| Command-line arguments | Runtime override |
| Spring Cloud Config | Centralized config |

---

# Example

## application.properties

```properties
server.port=8081

spring.datasource.url=jdbc:mysql://localhost:3306/test
```

---

# Accessing Values

```java
@Value("${server.port}")
private String port;
```

---

# Using @ConfigurationProperties

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppConfig {

    private String name;

}
```

---

# Environment Specific Config

```text
application-dev.properties
application-prod.properties
```

Activate profile:

```properties
spring.profiles.active=prod
```

---

# Benefits

- No hardcoded values
- Easy environment management
- Better security
- Easy deployment configuration

---

# Interview Ready Answer

> Spring Boot provides externalized configuration support by allowing configuration values to be stored outside the application code using files like `application.properties`, environment variables, command-line arguments, and profile-specific configuration files. This helps manage different environments such as dev, QA, and production without changing the code.


# 5. Embedded Servers Supported by Spring Boot

Spring Boot supports:

- Tomcat
- Jetty
- Undertow

---

# Default Embedded Server

```text
Apache Tomcat
```

is the default embedded server in Spring Boot.

---

# Example

When using:

```xml
spring-boot-starter-web
```

Spring Boot automatically includes:

```text
Embedded Tomcat
```

---

# Changing Server

## Use Jetty

```xml
spring-boot-starter-jetty
```

## Use Undertow

```xml
spring-boot-starter-undertow
```

---

# Benefit of Embedded Servers

- No external deployment needed
- Easy standalone execution
- Simplified deployment

Run directly using:

```bash
java -jar app.jar
```

---

# Interview Ready Answer

> Spring Boot supports embedded servers like Tomcat, Jetty, and Undertow. By default, Spring Boot uses Apache Tomcat as the embedded web server.


# 6. # Does Spring Boot Internally Run JAR in Tomcat?

Yes.

When you run a Spring Boot application:

```bash
java -jar app.jar
```

Spring Boot internally starts the embedded Tomcat server inside the same JVM process.

---

# What Happens Internally?

```text
1. JVM starts
2. Spring Boot application starts
3. Embedded Tomcat initializes
4. DispatcherServlet gets configured
5. Application starts listening on port
```

---

# Important Point

You do NOT separately install or start Tomcat.

Tomcat is already packaged inside:

```text
app.jar
```

---

# Traditional Spring Application

```text
WAR file → External Tomcat server
```

---

# Spring Boot

```text
JAR file → Embedded Tomcat inside application
```

---

# Interview Ready Answer

> Yes, when we run a Spring Boot application, it internally starts the embedded Tomcat server within the same JVM process. The Tomcat server is packaged inside the executable JAR, so no external Tomcat installation is required.

# 7.# Spring Profiles and Their Usage

Spring Profiles allow loading different configurations for different environments like:

- Dev
- QA
- Prod

This helps manage environment-specific properties without changing code.

---

# Example

## application-dev.properties

```properties
server.port=8081
```

## application-prod.properties

```properties
server.port=9090
```

---

# Activate Profile

```properties
spring.profiles.active=dev
```

or

```bash
--spring.profiles.active=prod
```

---

# Usage

Different environments can have:
- Different DB URLs
- Different API keys
- Different logging levels
- Different ports

---

# Using @Profile

```java
@Profile("dev")
@Component
public class DevConfig {

}
```

Bean loads only in `dev` profile.

---

# Benefits

- Environment-specific configuration
- Cleaner code
- Easier deployment management
- Better security/config separation

---

# Interview Ready Answer

> Spring Profiles are used to load environment-specific configurations such as dev, QA, and production settings. Spring Boot supports profile-specific property files and `@Profile` annotation to activate beans or configurations based on the active environment.

# 8. # How Enum Singleton Works Internally

## Enum Singleton

```java
public enum Singleton {

    INSTANCE;

    public void show() {
        System.out.println("Singleton Object");
    }
}
```

Usage:

```java
public class Main {

    public static void main(String[] args) {

        Singleton s1 = Singleton.INSTANCE;
        Singleton s2 = Singleton.INSTANCE;

        System.out.println(s1 == s2);

        s1.show();
    }
}
```

---

# Output

```text
true
Singleton Object
```

---

# How INSTANCE Gets Created?

When JVM loads enum class:

```text
JVM automatically creates enum objects
```

Internally JVM roughly converts it like this:

```java
final class Singleton extends Enum<Singleton> {

    public static final Singleton INSTANCE =
            new Singleton();

    private Singleton() {
        super("INSTANCE", 0);
    }
}
```

---

# Why Single Instance Is Guaranteed?

Because:

## 1. Constructor is Private Internally

You cannot do:

```java
new Singleton();
```

---

# 2. JVM Controls Object Creation

Only JVM can create enum instances during class loading.

---

# 3. Reflection Cannot Create Enum Objects

JVM blocks reflective enum creation.

---

# 4. Serialization Safe

During deserialization:

```text
JVM returns existing enum instance
```

instead of creating new object.

---

# Important Point

Enum instances are created:

```text
Only once during class loading
```

similar to static final objects.

---

# Interview Ready Answer

> In enum singleton, JVM automatically creates the enum instance during class loading. The enum constructor is internally private, and JVM ensures only one instance exists. Enums are also protected against reflection, serialization, and multithreading issues, making them the safest singleton implementation in Java.

# 9. Can Spring Boot Application Be Deployed as WAR File?

Yes.

Spring Boot applications can be deployed as:

- Executable JAR (embedded server)
- WAR file (external server)

---

# WAR Deployment

For WAR deployment:

```text
External Tomcat/Jetty server is required
```

---

# Required Changes

## 1. Change Packaging

### Maven

```xml
<packaging>war</packaging>
```

---

# 2. Extend SpringBootServletInitializer

```java
@SpringBootApplication
public class Application
        extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(
            SpringApplicationBuilder application) {

        return application.sources(Application.class);
    }
}
```

---

# 3. Mark Tomcat as Provided

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>

    <artifactId>
        spring-boot-starter-tomcat
    </artifactId>

    <scope>provided</scope>
</dependency>
```

---

# Deployment Flow

```text
Generate WAR
    ↓
Deploy WAR into external Tomcat
    ↓
Tomcat starts application
```

---

# JAR vs WAR

| JAR | WAR |
|---|---|
| Embedded server | External server |
| Easy deployment | Traditional deployment |
| Standalone | Server dependent |

---

# Interview Ready Answer

> Yes, Spring Boot applications can be deployed as WAR files on external servers like Tomcat or Jetty. For this, we change packaging to WAR, extend `SpringBootServletInitializer`, and mark embedded Tomcat dependency as provided.

# 10. Significance of Extending `SpringBootServletInitializer`

When deploying Spring Boot as a WAR file in an external Tomcat server, Tomcat needs a way to start the Spring Boot application.

`SpringBootServletInitializer` provides that integration.

---

# Why Is It Needed?

In executable JAR:

```text
main() method starts application
```

But in external Tomcat:

```text
Tomcat starts application
```

So Tomcat needs initialization configuration.

---

# What This Code Does

```java
extends SpringBootServletInitializer
```

Makes Spring Boot application compatible with external servlet containers like:
- Tomcat
- Jetty

---

# Role of configure()

```java
@Override
protected SpringApplicationBuilder configure(
        SpringApplicationBuilder application) {

    return application.sources(Application.class);
}
```

This tells Tomcat:

```text
Which Spring Boot main class should be loaded
```

during application startup.

---

# Internal Flow

```text
External Tomcat starts
      ↓
Tomcat loads WAR
      ↓
SpringBootServletInitializer executes
      ↓
configure() registers Application.class
      ↓
Spring Context starts
```

---

# Without This

External Tomcat cannot properly bootstrap Spring Boot application.

Application startup may fail.

---

# Interview Ready Answer

> `SpringBootServletInitializer` is required when deploying a Spring Boot application as a WAR file to an external servlet container like Tomcat. It helps the external server bootstrap and initialize the Spring Boot application, and the `configure()` method specifies the main application class to load into the Spring context.


# 11. What is Strangler Pattern?

Strangler Pattern is a migration approach used to gradually transform a monolithic application into microservices without rewriting the entire system at once.

---

# Why Is It Called Strangler?

Inspired by:

```text
Strangler Fig Tree
```

which slowly grows around a tree and replaces it over time.

Similarly:

```text
New microservices gradually replace parts of monolith
```

---

# Why Do We Use It?

Directly rewriting a large monolith is risky because:

- Huge downtime risk
- Difficult testing
- High failure chances
- Long development time

Strangler pattern enables:

```text
Incremental migration
```

with lower risk.

---

# How It Works

```text
1. Keep monolith running
2. Create new microservice for one module
3. Route specific traffic to new service
4. Gradually replace monolith modules
5. Eventually remove monolith completely
```

---

# Example

Suppose monolith has:

```text
- User Module
- Payment Module
- Order Module
```

Migration:

```text
Step 1 → Move Payment to microservice
Step 2 → Move Order to microservice
Step 3 → Move User module
```

---

# Architecture

```text
Client
   ↓
API Gateway
   ↓
Monolith + New Microservices
```

Gateway routes requests accordingly.

---

# Benefits

- Low migration risk
- Zero/minimal downtime
- Gradual modernization
- Easier testing and rollback

---

# Challenges

- Temporary hybrid architecture
- Complex routing
- Data consistency issues

---

# Interview Ready Answer

> Strangler Pattern is a gradual migration strategy used to convert a monolithic application into microservices incrementally instead of rewriting the entire system at once. New microservices slowly replace monolith modules over time, reducing migration risk, downtime, and deployment failures.

# 12. # What Happens If We Place return Statement in finally Block?

If a `return` statement is placed inside the `finally` block, it overrides any return value or exception from the `try` or `catch` block.

This is generally considered bad practice.

---

# Example

```java
public class Test {

    public static int test() {

        try {
            return 10;
        } finally {
            return 20;
        }
    }

    public static void main(String[] args) {

        System.out.println(test());
    }
}
```

---

# Output

```text
20
```

---

# Why?

Execution Flow:

```text
1. try returns 10
2. finally executes before method exits
3. finally returns 20
4. Original return gets overridden
```

---

# Exception Case

```java
try {
    throw new RuntimeException();
} finally {
    return;
}
```

Here exception gets suppressed.

This makes debugging difficult.

---

# Important Point

```text
finally block always executes before method completes
```

unless JVM crashes or System.exit() is called.

---

# Interview Ready Answer

> If we place a return statement inside the finally block, it overrides any return value or exception from the try or catch block. This can suppress exceptions and create unexpected behavior, so using return inside finally is considered bad practice.
