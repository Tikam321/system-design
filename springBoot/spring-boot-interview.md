### What is Spring boot?
- Sprint boot is a Java-based spring framework used for Rapid Application Development (to build stand-alone microservices). 
It has extra support of auto-configuration and embedded application servers like tomcat, jetty, etc.

### Features of Spring Boot that make it different?
- Creates stand-alone spring application with minimal configuration needed.
- It has embedded tomcat, jetty which makes it just code and run the application.
- Provide production-ready features such as metrics, health checks, and externalized configuration.
- Absolutely no requirement for XML configuration.

1. adavtantage
- Easy to understand and develop spring applications.
- Spring Boot is nothing but an existing framework with the addition of an embedded HTTP server and annotation configuration which makes it easier to understand and faster the process of development.
- Increases productivity and reduces development time.
- Minimum configuration.
- We don’t need to write any XML configuration, only a few annotations are required to do the configuration.

2. What are the Spring Boot key components?
-  Below are the four key components of spring-boot:
- Spring Boot auto-configuration.
- Spring Boot CLI.
- Spring Boot starter POMs.
- Spring Boot Actuators.

3. Why Spring Boot over Spring?
-Below are some key points which spring boot offers but spring doesn’t:

-Starter POM.
-Version Management.
-Auto Configuration.
-Component Scanning.
-Embedded server.
-InMemory DB.
-Actuators
-Spring Boot simplifies the spring feature for the user:

4. What is the starter dependency of the Spring boot module?
Spring boot provides numbers of starter dependency, here are the most commonly used -

-Data JPA starter.
-Test Starter.
-Security starter.
-Web starter.
-Mail starter.
-Thymeleaf starter.

5. How does Spring Boot works?
-Spring Boot automatically configures your application based on the dependencies you have added to the project by using annotation.
-The entry point of the spring boot application is the class that contains @SpringBootApplication annotation and the main method.

Spring Boot automatically scans all the components included in the project by using @ComponentScan annotation.

5. How does Spring Boot works?
-Spring Boot automatically configures your application based on the dependencies you have added to the project by using annotation. 
-The entry point of the spring boot application is the class that contains @SpringBootApplication annotation and the main method.
-Spring Boot automatically scans all the components included in the project by using @ComponentScan annotation.

6. What does the @SpringBootApplication annotation do internally?
- The @SpringBootApplication annotation is equivalent to using @Configuration, @EnableAutoConfiguration, and @ComponentScan with their default attributes.
- Spring Boot enables the developer to use a single annotation instead of using multiple. But, as we know, Spring provided loosely coupled features that we can use for each annotation as per our project needs.

7. @ComponentScan in Spring Boot tells Spring which packages to scan to find beans like:
@Component
@Service
@Repository
@Controller
@Configuration

8. How does a spring boot application get started?
Just like any other Java program, a Spring Boot application must have a main method. This method serves as an entry point,
which invokes the SpringApplication#run method to bootstrap the application.

```java
@SpringBootApplication 
public class MyApplication { 
   
       public static void main(String[] args) {    
    
             SpringApplication.run(MyApplication.class);        
               // other statements     
       } 
}

```

9. How to get the list of all the beans in your Spring boot application?
- Spring Boot actuator “/Beans” is used to get the list of all the spring beans in your application.

10. What is an IOC container?
IoC Container is a framework for implementing automatic dependency injection. It manages object creation and its life-time and also injects dependencies into the class.

11. What is dependency Injection?
- The process of injecting dependent bean objects into target bean objects is called dependency injection.
- Setter Injection: The IOC container will inject the dependent bean object into the target bean object by calling the setter method.
- Constructor Injection: The IOC container will inject the dependent bean object into the target bean object by calling the target bean constructor.
- Field Injection: The IOC container will inject the dependent bean object into the target bean object by Reflection API.

12. Where do we define properties in the Spring Boot application?
You can define both application and Spring boot-related properties into a file called application.properties.
You can create this file manually or use Spring Initializer to create this file. You don’t need to do any special configuration to instruct Spring Boot to load this file, If it exists in classpath then spring boot automatically loads it and configure itself and the application code accordingly.

13. difference between tomact and jetty server
- Tomcat and Jetty are both Java servlet servers.
- Tomcat is the default in Spring Boot, very widely used, and usually the easiest “standard” choice with stronger ecosystem support.
- Jetty is generally lighter and more modular, and many teams prefer it for embedded apps or workloads with lots of long-lived async connections (like WebSocket/SSE).
- In short: choose Tomcat for simplicity and broad support, choose Jetty when lightweight footprint and async connection handling are top priorities.
```java
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.5'
    id 'io.spring.dependency-management' version '1.1.6'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    implementation 'org.springframework.boot:spring-boot-starter-jetty'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

'```
  
14. How to check the environment properties in your Spring boot application?
- Spring Boot actuator “/env” returns the list of all the environment properties of running the spring boot application.

15. Can we override or replace the Embedded tomcat server in Spring Boot?
- Yes, we can replace the Embedded Tomcat server with any server by using the Starter dependency in the pom.xml file.
- Like you can use spring-boot-starter-jetty as a dependency for using a jetty server in your project.

16. What Are the Basic Annotations that Spring Boot Offers?
- The primary annotations that Spring Boot offers reside in its org.springframework.boot.autoconfigure and its sub-packages. Here are a couple of basic ones:
- @EnableAutoConfiguration – to make Spring Boot look for auto-configuration beans on its classpath and automatically apply them.
- @SpringBootApplication – used to denote the main class of a Boot Application. This annotation combines @Configuration, @EnableAutoConfiguration,
- and @ComponentScan annotations with their default attributes.

17.  What is Spring Boot dependency management?
- Spring Boot dependency management is used to manage dependencies and configuration automatically without you specifying the version for any of that dependencies.

18. Can we create a non-web application in Spring Boot?
- Yes, we can create a non-web application by removing the web dependencies from the classpath along with changing the way Spring Boot creates the application context.

19. Is it possible to change the port of the embedded Tomcat server in Spring Boot?
- Yes, it is possible. By using the server.port in the application.properties.

20. What are the actuator-provided endpoints used for monitoring the Spring boot application?
- Actuators provide below pre-defined endpoints to monitor our application.
- Health
- Info
- Beans
- Mappings
- Configprops
- Httptrace
- Heapdump
- Threaddump
- Shutdown

21. Can we disable the default web server in the Spring boot application?
- Yes, we can use application.properties to configure the web application type i.e spring.main.web-application-type=none.

22. The main functionality of spring-boot-starter-web is to let you build HTTP web apps and REST APIs quickly with auto-configuration.

- It mainly provides:
Spring MVC (@RestController, @RequestMapping, etc.)
- An embedded web server (Tomcat by default)
- JSON support (Jackson) for request/response conversion
- Core web auto-configuration so you can run with minimal setup

23. How to disable a specific auto-configuration class?
- You can use exclude attribute of @EnableAutoConfiguration if you want auto-configuration not to apply to any specific class.
- //use of exclude
- @EnableAutoConfiguration(exclude={className})

24. @Controller and @RestController differ in what they return by default:

- @Controller:
- Used for MVC web pages.
- Methods usually return a view name (like home.html).
- To return JSON/text from a method, you must add @ResponseBody.

- @RestController:
- Used for REST APIs.
- It is basically @Controller + @ResponseBody.
- Methods return data (JSON/XML/text) directly in the HTTP response body.
- So: use @Controller for server-rendered views, and @RestController for API endpoints.

25. Spring Boot follows MVC like this:
- Client sends request to a URL.
- DispatcherServlet (front controller) receives it.
- It routes to a Controller method (@Controller / @RestController).
- Controller calls Service/Repository for business logic and data.
- Result is put into the Model.
- Response:
- With @Controller: returns a View (Thymeleaf/JSP page).
- With @RestController: returns JSON/XML directly (no view page).
- So in Spring Boot MVC:
- Model = data objects (Entity, DTO, Model)
- View = UI template (or JSON response in REST)
- Controller = request handler + flow control

26. @RequestMapping is a general-purpose mapping annotation, while @GetMapping is specific to HTTP GET.

- @RequestMapping
- Can map any HTTP method (GET, POST, PUT, etc.)
- You must specify method if needed: method = RequestMethod.GET
- Can be used at class level and method level
- @GetMapping

- Shortcut/composed annotation for RequestMapping(method = RequestMethod.GET)
- More readable for GET endpoints
- Used at method level (typically)
- Example:
- @RequestMapping(value="/users", method=RequestMethod.GET) is equivalent to: @GetMapping("/users")

27. What is Spring Actuator? What are its advantages?
- An actuator is an additional feature of Spring that helps you to monitor and manage your application when you push it to production. These actuators include auditing, health, CPU - usage, HTTP hits, and metric gathering, and many more that are automatically applied to your application.

27. How to enable Actuator in Spring boot application?
To enable the spring actuator feature, we need to add the dependency of “spring-boot-starter-actuator” in pom.xml.
```java
<dependency>
<groupId> org.springframework.boot</groupId>
<artifactId> spring-boot-starter-actuator </artifactId>
</dependency>
```
- Spring Boot Annotations Interview Questions
- Spring Actuator works through auto-configuration + endpoint discovery + endpoint invocation.
- Internal flow:
  
-   You add spring-boot-starter-actuator.
-   At startup, Spring Boot auto-config creates actuator beans (HealthEndpoint, InfoEndpoint, metrics, etc.).
-   Boot discovers endpoint operations (@ReadOperation, @WriteOperation, @DeleteOperation) and wraps them as invokable operations.
-   Web layer maps exposed endpoints to URLs (usually /actuator/**) using MVC/WebFlux handler mappings.
-   When a request comes (example /actuator/health), Actuator calls the matching operation invoker.
-   That operation pulls data from contributors:
-   Health from HealthContributor beans (DB, disk, custom checks)
-   Metrics from Micrometer meter registries
-   Result is converted to JSON and returned.
-   Security/exposure rules (management.endpoints.web.exposure.include, Spring Security) decide what is visible.
-   So internally, Actuator is not “magic”; it is a set of Spring beans, discovered endpoints, and HTTP mappings managed by Boot auto-configuration.

# Spring Boot Annotations Interview Questions

28. What are stereotype annotations in Spring and how are they different?
- Stereotype annotations include:
- @Component
- @Service
- @Repository
- @Controller
- They are functionally similar but convey intent.
- Differences:
- @Service represents business logic
- @Repository represents data access layer
- @Controller represents web layer
- @Component is a generic stereotype
- Using specific stereotypes improves readability and enables additional behavior such as exception translation.

29. Why is @Repository important beyond being a marker annotation?
- @Repository does more than label a class.
- Main reasons it matters:
- It is a @Component specialization, so Spring auto-detects it as a bean.
- It enables persistence exception translation: low-level exceptions (JPA/Hibernate/JDBC vendor errors) are converted into Spring’s DataAccessException hierarchy.
- That gives you consistent, database-agnostic error handling in service layers.
- It also clearly marks DAO/data-access intent, which helps architecture, reviews, and AOP targeting.
- So the big “beyond marker” value is: automatic exception translation + consistent data-access semantics.

30. What is Configuration and what problem does it solve?
- @Configuration marks a class as a source of bean definitions.
- It ensures:
- Beans are singletons by default
- Method calls to @Bean are intercepted
- Same bean instance is returned every time
- Without @Configuration, multiple bean instances can be created unintentionally.

 31. What is Configuration and what problem does it solve?
- @Configuration marks a class as a source of bean definitions.
- It ensures:
- Beans are singletons by default
- Method calls to @Bean are intercepted
- Same bean instance is returned every time
- Without @Configuration, multiple bean instances can be created unintentionally.

32. What is the difference between RequestParam, PathVariable, and RequestBody?
- @RequestParam extracts query parameters
- @PathVariable extracts values from URL path
- @RequestBody binds request payload to an object
- Each is used based on API design and HTTP semantics.

# Spring Boot Tricky Interview Questions
32. What are common reasons for slow Spring Boot application startup?
Common causes include excessive auto configuration, large classpath scanning, slow database initialization, unnecessary dependencies, and heavy logging. Using actuator startup metrics helps identify bottlenecks.


33. Why does field injection cause testing and immutability issues(Field injection (@Autowired on fields) hurts both testing and immutability because)?
- Dependencies are hidden (not in constructor), so unit tests can’t easily pass mocks with new MyService(mockRepo).
- You often need a Spring context or reflection to set fields, making tests slower and more fragile.
- Injected fields usually can’t be final, so the object is not truly immutable.
- The object can exist in a partially initialized state before Spring injects fields.



34. Circular dependency means Spring finds a loop while creating beans.

Example that fails:
```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}

@Service
public class PaymentService {
    private final OrderService orderService;

    public PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```
Why it fails:

To create OrderService, Spring needs PaymentService.
To create PaymentService, Spring needs OrderService.
Loop => BeanCurrentlyInCreationException.
Clean fix (break the cycle by redesign):

Keep dependency one-way.
```java
@Service
public class PaymentService {
    public boolean pay(Order order) {
        return true; // call gateway
    }
}

@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void checkout(Order order) {
        if (paymentService.pay(order)) {
            // mark order as paid
        }
    }
}
```
Quick workaround (not ideal design):

public PaymentService(@Lazy OrderService orderService) { ... }
Best practice: refactor responsibilities so services don’t mutually depend on each other.

### @Lazy is a workaround for circular dependency.
It tells Spring: inject a proxy now, create the real bean only when first used.
```java
@Service
class UserService {
    private final OrderService orderService;

    UserService(@Lazy OrderService orderService) {   // lazy side
        this.orderService = orderService;
    }

    public void deactivateUser(Long userId) {
        orderService.cancelOrders(userId); // real OrderService created here (first use)
    }
}

@Service
class OrderService {
    private final UserService userService;

    OrderService(UserService userService) {
        this.userService = userService;
    }

    public void cancelOrders(Long userId) {
        // cancel logic
    }
}

```
- How it works internally:

- Spring starts and registers the bean definition.
- Instead of creating the real object immediately, Spring keeps it uninitialized.
- If @Lazy is used at an injection point, Spring injects a proxy object.
- On the first method call, that proxy asks the container for the real bean.
- Spring creates + initializes the real bean, then the proxy forwards calls to it.
- Later calls reuse the same bean (for singleton scope).

35. Why does Transactional sometimes not work in Spring Boot?
Transactional may not work due to self invocation, private method usage, or when the class is not managed by Spring. Transaction management relies on proxy based interception, which requires public methods and external calls.

36. Summary: Spring Boot variable priority (with Render example)

- Render env vars are injected as OS-level environment variables for your running service.
- Spring Boot reads many config sources, and for the same key, higher priority value wins.
- Practical priority (high to low):
- Command-line args (--server.port=9090)
- Java system properties (-Dserver.port=9090)
- OS env vars (like Render dashboard vars)
- application-{profile}.yml / application-{profile}.properties
- application.yml / application.properties
- Render example

- In Render: SPRING_DATASOURCE_URL=jdbc:postgresql://prod-db/...
- In application.yml: spring.datasource.url: jdbc:postgresql://localhost/...
- Result in production on Render: Spring uses the Render value.
- Best practice

- Put secrets and deploy-specific values in Render env vars.
- Keep safe defaults in application.yml / application.properties.

- Example (server.port):

- CLI: --server.port=9090
- JVM: -Dserver.port=8081
- Render env: SERVER_PORT=8080
- application.yml: server.port: 7070
- Final used value = 9090 (CLI wins).

37. What is the difference between application.properties and bootstrap.properties?
- bootstrap.properties is loaded before application.properties and is mainly used in Spring Cloud applications for external configuration sources.
- In most Spring Boot applications, application.properties or application.yml is sufficient.

38. Why does LazyInitializationException occur in Spring Boot?
LazyInitializationException occurs when a lazily loaded JPA association is accessed outside an active Hibernate session or transaction. This commonly happens when entities are returned directly from controllers instead of using DTOs.

39. What is the role of ConfigurationProperties in Spring Boot?
ConfigurationProperties binds external configuration to strongly typed Java objects. It provides type safety, better structure, and supports validation, making it preferable over multiple @Value annotations for complex configuration.

40. Why is spring.jpa.hibernate.ddl-auto=update not recommended in production?
- Using ddl-auto=update can cause unintended schema changes and data loss in production.
- Schema changes should be handled using controlled migration tools such as Flyway or Liquibase.

41. In Spring Boot, “DDL” usually means Hibernate auto schema management via:
- spring.jpa.hibernate.ddl-auto
- It controls how Spring/Hibernate handles database tables from your @Entity classes at app startup.
spring:
  jpa:
    hibernate:
      ddl-auto: update
- Values and functionality:

- none: do nothing to schema
- validate: check schema matches entities; fail startup if mismatch
- update: try to update schema to match entities (safe-ish for dev, not ideal for prod)
- create: drop existing schema and recreate on startup
- create-drop: same as create, then drop again when app stops
- Best practice:

- Dev: update (or create)
- Test: create-drop
- Prod: validate or none, and use Flyway/Liquibase for real migrations

41. JPA and Spring Data JPA are related, but not the same:
- JPA:
- A Java specification (standard), not a library by itself.
- Defines ORM concepts: @Entity, EntityManager, JPQL, relationships.
- Needs an implementation (like Hibernate).

- Spring Data JPA:
- A Spring module built on top of JPA.
- Reduces boilerplate by giving JpaRepository interfaces.
- Auto-generates queries from method names (findByEmail(...)), paging/sorting, etc.
- So:
- Use plain JPA when you want low-level control.
- Use Spring Data JPA for faster development with less CRUD code.

42. what  is the difference betwen spring boot  and spring MVC ?
- Spring MVC is a web framework (inside Spring) for building web apps and REST APIs using controllers, request mapping, model/view, etc.
- Spring Boot is a setup/productivity layer on top of Spring (including MVC) that gives:

- auto-configuration
- starter dependencies
- embedded server (Tomcat/Jetty)
- minimal boilerplate
- production features (Actuator, externalized config)
- So:
- Spring MVC = web framework feature
- Spring Boot = faster way to build/run Spring apps (often using MVC)


 
























