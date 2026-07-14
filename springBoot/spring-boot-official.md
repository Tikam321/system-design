# Spring Boot Interview Questions & Answers (Complete Guide)

A complete, structured set of Spring Boot interview questions — from basics to advanced/microservices — with clear answers. Organized by topic so you can review section by section.

---

## Table of Contents
1. [Spring Boot Basics](#1-spring-boot-basics)
2. [Auto-Configuration & Starters](#2-auto-configuration--starters)
3. [Annotations](#3-annotations)
4. [Application Configuration & Profiles](#4-application-configuration--profiles)
5. [Dependency Injection & Beans](#5-dependency-injection--beans)
6. [Spring Boot Actuator](#6-spring-boot-actuator)
7. [REST APIs & Web Layer](#7-rest-apis--web-layer)
8. [Exception Handling & Validation](#8-exception-handling--validation)
9. [Spring Data JPA & Database](#9-spring-data-jpa--database)
10. [Spring Security](#10-spring-security)
11. [Testing](#11-testing)
12. [Microservices with Spring Boot](#12-microservices-with-spring-boot)
13. [Spring Boot with Messaging/Caching](#13-spring-boot-with-messagingcaching)
14. [Deployment, Packaging & Performance](#14-deployment-packaging--performance)
15. [Miscellaneous / Advanced](#15-miscellaneous--advanced)

---

## 1. Spring Boot Basics

**Q1. What is Spring Boot? How is it different from Spring Framework?**
Spring Boot is a framework built on top of the Spring Framework that simplifies bootstrapping and developing new Spring applications. It removes boilerplate configuration by providing auto-configuration, embedded servers, starter dependencies, and production-ready features (Actuator).
- Spring Framework requires manual configuration (XML/Java config) for beans, MVC dispatcher servlets, view resolvers, etc.
- Spring Boot auto-configures most of this based on the classpath, provides embedded Tomcat/Jetty/Undertow so you don't need to deploy a WAR to an external server, and offers a single executable JAR.

**Q2. What are the main features of Spring Boot?**
- Auto-configuration
- Standalone (embedded servers: Tomcat, Jetty, Undertow)
- Opinionated "starter" dependencies
- No XML configuration needed
- Production-ready features via Actuator (health checks, metrics)
- Externalized configuration (properties/YAML, profiles)
- CLI support and DevTools for faster development

**Q3. What is the difference between Spring, Spring MVC, and Spring Boot?**
- **Spring**: Core framework providing IoC container, DI, AOP, etc.
- **Spring MVC**: A module within Spring for building web applications following MVC pattern (DispatcherServlet, controllers, view resolvers).
- **Spring Boot**: A tool that sits on top of Spring/Spring MVC to simplify configuration, dependency management, and deployment.

**Q4. What is the entry point of a Spring Boot application?**
A class annotated with `@SpringBootApplication` containing a `main` method that calls `SpringApplication.run(Application.class, args)`.

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

**Q5. What happens internally when `SpringApplication.run()` is called?**
1. Creates an `ApplicationContext` (usually `AnnotationConfigServletWebServerApplicationContext` for web apps).
2. Registers `ApplicationContextInitializers` and `ApplicationListeners`.
3. Determines the application type (Servlet, Reactive, or None) based on classpath.
4. Loads configuration from `application.properties`/`application.yml`.
5. Triggers auto-configuration classes based on classpath dependencies.
6. Starts the embedded web server (if it's a web application).
7. Publishes `ApplicationReadyEvent` once startup finishes.

**Q6. What is Spring Initializr?**
A web tool (start.spring.io) and IDE plugin used to bootstrap Spring Boot projects by selecting build tool (Maven/Gradle), language, Spring Boot version, and dependencies. It generates a ready-to-use project skeleton.

**Q7. Can we override the default embedded server in Spring Boot?**
Yes. By excluding the default starter (`spring-boot-starter-tomcat`) and adding another (`spring-boot-starter-jetty` or `spring-boot-starter-undertow`) in the `spring-boot-starter-web` dependency.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

**Q8. What is Spring Boot DevTools?**
A module (`spring-boot-devtools`) that improves developer productivity: automatic restart on code changes, live reload of browser, disabling caching for template engines, and sensible default properties during development.

---

## 2. Auto-Configuration & Starters

**Q9. What is auto-configuration in Spring Boot?**
Auto-configuration automatically configures Spring beans based on the jars present on the classpath, existing beans, and property settings, so developers don't need to write manual configuration. It is enabled via `@EnableAutoConfiguration` (included in `@SpringBootApplication`).

**Q10. How does auto-configuration actually work internally?**
Spring Boot scans `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (or `spring.factories` in older versions) for a list of auto-configuration classes. Each class is annotated with conditional annotations like `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty` so it only applies when specific conditions are true.

**Q11. What are Spring Boot Starters?**
Starters are curated dependency descriptors that bundle a set of compatible dependencies for a specific feature so you don't have to manage individual versions. Examples:
- `spring-boot-starter-web` – Spring MVC + embedded Tomcat
- `spring-boot-starter-data-jpa` – Spring Data JPA + Hibernate
- `spring-boot-starter-security` – Spring Security
- `spring-boot-starter-test` – JUnit, Mockito, AssertJ, Spring Test

**Q12. What is the purpose of `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`?**
- `@ConditionalOnClass`: Configuration applies only if a specified class is present on the classpath.
- `@ConditionalOnMissingBean`: Applies only if a bean of that type is not already defined (lets users override defaults).
- `@ConditionalOnProperty`: Applies only if a specific property has a given value (or is present).

**Q13. How can you exclude a specific auto-configuration class?**
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
```
Or via properties: `spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration`

**Q14. How do you create a custom Spring Boot starter?**
Create two modules:
1. An **autoconfigure** module containing `@Configuration` classes with conditional annotations and a `AutoConfiguration.imports` file registering them.
2. A **starter** module that has no code — just declares dependencies (the autoconfigure module + any required libraries) so consumers only add one dependency.

---

## 3. Annotations

**Q15. What does `@SpringBootApplication` do?**
It's a composite/meta-annotation equivalent to:
```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```
- `@Configuration`: Marks the class as a source of bean definitions.
- `@EnableAutoConfiguration`: Enables Spring Boot's auto-configuration mechanism.
- `@ComponentScan`: Scans the package (and sub-packages) of the annotated class for components.

**Q16. Difference between `@Component`, `@Service`, `@Repository`, `@Controller`?**
All are stereotypes of `@Component` and get registered as Spring beans, but they add semantic meaning and extra behavior:
- `@Component`: Generic stereotype for any Spring-managed component.
- `@Service`: Marks a service-layer class (business logic); semantically indicates a service.
- `@Repository`: Marks a DAO/persistence-layer class; enables automatic exception translation (converts JDBC/Hibernate exceptions into Spring's `DataAccessException` hierarchy).
- `@Controller`: Marks a Spring MVC controller that returns views.
- `@RestController`: `@Controller` + `@ResponseBody`, returns data (JSON/XML) directly instead of a view name.

**Q17. Difference between `@RequestMapping` and `@GetMapping`/`@PostMapping`?**
`@RequestMapping` is generic and requires specifying the HTTP method (`method = RequestMethod.GET`). `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping` are shortcut/composed annotations for specific HTTP methods, introduced for readability.

**Q18. Difference between `@RequestParam`, `@PathVariable`, `@RequestBody`, `@RequestHeader`?**
- `@RequestParam`: Binds a query parameter (`?name=value`) or form data to a method argument.
- `@PathVariable`: Binds a URI template variable (`/users/{id}`) to a method argument.
- `@RequestBody`: Binds the HTTP request body (typically JSON) to a Java object, using `HttpMessageConverter`.
- `@RequestHeader`: Binds an HTTP header value to a method argument.

**Q19. What is `@Autowired`? What are its limitations?**
`@Autowired` injects a dependency automatically by type. It can be applied on constructors, setters, or fields.
Limitations: field injection makes unit testing harder (can't set final fields, needs reflection) and hides dependencies; if multiple beans of the same type exist, it throws `NoUniqueBeanDefinitionException` unless combined with `@Qualifier` or `@Primary`. Constructor injection is recommended nowadays as it makes dependencies explicit and supports immutability (`final` fields), and Spring itself recommends it (constructor injection doesn't even need `@Autowired` if there's only one constructor).

**Q20. Difference between `@Qualifier` and `@Primary`?**
- `@Primary`: Marks a bean as the default choice when multiple candidates exist.
- `@Qualifier("beanName")`: Explicitly specifies which bean to inject at the injection point, overriding the default `@Primary` choice if needed.

**Q21. What is `@Value` used for?**
Injects values from properties files, environment variables, or SpEL expressions into fields.
```java
@Value("${server.port}")
private int port;

@Value("#{2 * 10}")
private int computed;
```

**Q22. What is `@ConfigurationProperties`? How is it different from `@Value`?**
`@ConfigurationProperties` binds a whole group of external properties (prefixed) to a POJO in a type-safe way, supporting nested objects, lists, and maps — better for structured configuration. `@Value` is for injecting single values and supports SpEL, but doesn't scale well for many related properties.
```java
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {
    private String host;
    private int port;
    // getters/setters
}
```
Needs `@EnableConfigurationProperties(MailProperties.class)` or `@ConfigurationPropertiesScan`.

**Q23. What is the difference between `@Bean` and `@Component`?**
- `@Component` is a class-level annotation used with component scanning — Spring auto-detects and instantiates the class.
- `@Bean` is a method-level annotation used inside a `@Configuration` class to manually create/configure a bean, typically used for third-party classes you can't annotate directly.

**Q24. What is `@Transactional`?**
Declares that a method (or all methods in a class) should run within a database transaction. Spring wraps the method with proxy logic to begin a transaction, commit on success, and roll back on a runtime exception (by default; checked exceptions don't trigger rollback unless configured via `rollbackFor`).

**Q25. What's the difference between `@ControllerAdvice` and `@RestControllerAdvice`?**
`@ControllerAdvice` allows centralized exception handling, model attributes, and data binding across multiple controllers. `@RestControllerAdvice` is `@ControllerAdvice` + `@ResponseBody`, so returned objects are serialized directly as the response body (typically JSON), useful for REST APIs.

---

## 4. Application Configuration & Profiles

**Q26. What is the difference between `application.properties` and `application.yml`?**
Both configure application settings externally. YAML is more readable for hierarchical/nested data and supports lists naturally, while `.properties` uses flat key-value pairs. Functionally equivalent — Spring Boot supports both.

**Q27. What are Spring Profiles? Why use them?**
Profiles let you define environment-specific configuration (dev, test, prod) that can be activated conditionally. Beans can be scoped to a profile using `@Profile("dev")`, and properties can be split into `application-dev.properties`, `application-prod.properties`, activated via `spring.profiles.active=dev`.

**Q28. What is the property loading order / precedence in Spring Boot?**
From highest to lowest priority (partial list, later overrides earlier if higher precedence):
1. Command-line arguments
2. `SPRING_APPLICATION_JSON` (environment variable/system property)
3. JNDI attributes
4. Java System properties
5. OS environment variables
6. Profile-specific properties outside the packaged jar (`application-{profile}.properties`)
7. Profile-specific properties packaged inside the jar
8. Application properties outside the packaged jar
9. Application properties packaged inside the jar
10. `@PropertySource` annotations
11. Default properties (`SpringApplication.setDefaultProperties`)

**Q29. How do you externalize configuration in Spring Boot?**
Via `application.properties`/`.yml`, OS environment variables, command-line args, `@ConfigurationProperties`, config servers (Spring Cloud Config), or external files referenced with `spring.config.location`/`spring.config.import`.

**Q30. What is `spring.config.import` used for?**
Introduced in Spring Boot 2.4+ to import additional configuration sources (e.g., another properties file, a Config Server, or `optional:` sources) declaratively from within `application.properties`.

**Q31. How can you access environment variables/config programmatically?**
By injecting `Environment`:
```java
@Autowired
private Environment env;
String port = env.getProperty("server.port");
```

---

## 5. Dependency Injection & Beans

**Q32. What is Inversion of Control (IoC) and Dependency Injection (DI)?**
IoC is a design principle where the control of object creation and lifecycle is transferred from the application code to a container/framework. DI is the pattern used to implement IoC — dependencies are "injected" into a class rather than the class creating them itself.

**Q33. What are the types of Dependency Injection in Spring?**
- Constructor Injection (recommended)
- Setter Injection
- Field Injection (via `@Autowired` directly on fields)

**Q34. What are Spring Bean Scopes?**
- `singleton` (default): one instance per Spring container.
- `prototype`: new instance every time it's requested.
- `request`: one instance per HTTP request (web-aware).
- `session`: one instance per HTTP session.
- `application`: one instance per ServletContext.
- `websocket`: one instance per WebSocket session.

**Q35. What is the Spring Bean lifecycle?**
1. Instantiation
2. Populate properties (DI)
3. `BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware` callbacks (if implemented)
4. `BeanPostProcessor.postProcessBeforeInitialization()`
5. `@PostConstruct` / `InitializingBean.afterPropertiesSet()` / custom `init-method`
6. `BeanPostProcessor.postProcessAfterInitialization()`
7. Bean ready to use
8. On shutdown: `@PreDestroy` / `DisposableBean.destroy()` / custom `destroy-method`

**Q36. What is a circular dependency, and how does Spring handle it?**
A circular dependency occurs when Bean A depends on Bean B, and Bean B depends on Bean A. Spring can resolve this for singleton beans using setter/field injection (via early bean references in a three-level cache), but it **cannot** resolve circular dependencies with constructor injection — it throws `BeanCurrentlyInCreationException`. Best practice is to redesign to avoid circular dependencies (e.g., via `@Lazy` or refactoring shared logic into a third bean).

**Q37. What's the difference between `BeanFactory` and `ApplicationContext`?**
`BeanFactory` is the basic IoC container providing DI. `ApplicationContext` is a more advanced container built on top of `BeanFactory`, adding features like event propagation, AOP integration, internationalization, and easier integration with Spring's declarative features. Spring Boot always uses an `ApplicationContext`.

---

## 6. Spring Boot Actuator

**Q38. What is Spring Boot Actuator?**
A module that provides production-ready features to monitor and manage an application: health checks, metrics, environment info, thread dumps, and more, exposed via HTTP endpoints or JMX.

**Q39. Name common Actuator endpoints.**
- `/actuator/health` – application health status
- `/actuator/info` – arbitrary app info
- `/actuator/metrics` – application metrics
- `/actuator/env` – environment properties
- `/actuator/beans` – all Spring beans
- `/actuator/mappings` – all request mappings
- `/actuator/loggers` – view/change log levels at runtime
- `/actuator/threaddump`, `/actuator/heapdump`

**Q40. How do you enable/expose specific Actuator endpoints?**
```properties
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
```
By default, only `health` and `info` are exposed over HTTP for security reasons.

**Q41. How do you create a custom Actuator health indicator?**
Implement `HealthIndicator`:
```java
@Component
public class CustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        boolean isHealthy = checkSomething();
        return isHealthy ? Health.up().build()
                          : Health.down().withDetail("Error", "Service down").build();
    }
}
```

**Q42. How is Actuator secured in production?**
Typically restrict endpoint exposure, put Actuator on a separate management port (`management.server.port`), and secure endpoints with Spring Security (require authentication/authorization for sensitive endpoints).

---

## 7. REST APIs & Web Layer

**Q43. How do you build a REST API in Spring Boot?**
Using `@RestController`, mapping annotations (`@GetMapping`, etc.), and returning POJOs/DTOs that get serialized to JSON via Jackson (included by default with `spring-boot-starter-web`).

**Q44. What is `ResponseEntity`, and why use it?**
`ResponseEntity<T>` represents the full HTTP response (status code, headers, body). It gives fine-grained control over the response rather than just returning a body with an implicit 200 status.
```java
return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
```

**Q45. How do you handle content negotiation in Spring Boot (JSON vs XML)?**
Spring uses `HttpMessageConverter`s based on the `Accept` header. By default Jackson handles JSON; adding `jackson-dataformat-xml` enables XML support, and Spring will pick the converter matching the requested media type.

**Q46. What is HATEOAS, and how is it supported in Spring Boot?**
HATEOAS (Hypermedia as the Engine of Application State) is a REST constraint where responses include links to related resources/actions. Spring Boot supports it via the `spring-boot-starter-hateoas` module with `EntityModel`, `CollectionModel`, and `WebMvcLinkBuilder`.

**Q47. How do you version REST APIs in Spring Boot?**
Common strategies:
- URI versioning: `/api/v1/users`
- Request parameter: `/api/users?version=1`
- Header versioning: custom header like `X-API-VERSION`
- Media type/content negotiation versioning: `Accept: application/vnd.company.v1+json`

**Q48. What is CORS, and how do you enable it in Spring Boot?**
CORS (Cross-Origin Resource Sharing) allows/restricts requests from different origins. Enable via `@CrossOrigin` on a controller/method, or globally:
```java
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**").allowedOrigins("https://example.com");
        }
    };
}
```

**Q49. How does Spring Boot handle file upload/download?**
File upload via `MultipartFile`:
```java
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile file) { ... }
```
Configured via `spring.servlet.multipart.max-file-size` and `max-request-size`. Download via returning a `Resource` wrapped in `ResponseEntity` with appropriate `Content-Disposition` headers.

---

## 8. Exception Handling & Validation

**Q50. How do you handle exceptions globally in Spring Boot?**
Using `@RestControllerAdvice` combined with `@ExceptionHandler` methods:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage()));
    }
}
```

**Q51. What is the default exception handling behavior in Spring Boot?**
Spring Boot provides a default `/error` endpoint (via `BasicErrorController`) that returns a JSON error response with fields like `timestamp`, `status`, `error`, `message`, and `path` when an unhandled exception occurs.

**Q52. How do you validate request payloads in Spring Boot?**
Using Bean Validation (JSR-380/Jakarta Validation) annotations like `@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Min`, `@Max` on DTO fields, combined with `@Valid`/`@Validated` on the controller method parameter. Errors are captured in `BindingResult` or thrown as `MethodArgumentNotValidException`, which can be handled globally.

**Q53. Difference between `@Valid` and `@Validated`?**
`@Valid` is a standard Java Bean Validation annotation triggering validation cascading. `@Validated` is a Spring-specific annotation that additionally supports validation groups and can be applied at the class level to enable method-level validation on non-body parameters.

---

## 9. Spring Data JPA & Database

**Q54. What is Spring Data JPA?**
An abstraction over JPA (Java Persistence API, implemented typically by Hibernate) that drastically reduces boilerplate DAO code by providing repository interfaces with built-in CRUD operations and derived query methods.

**Q55. What is the difference between `CrudRepository`, `JpaRepository`, and `PagingAndSortingRepository`?**
- `CrudRepository`: Basic CRUD operations (save, findById, findAll, delete).
- `PagingAndSortingRepository`: Extends CrudRepository, adds pagination and sorting.
- `JpaRepository`: Extends `PagingAndSortingRepository`, adds JPA-specific methods (batch operations, flushing).

**Q56. How do derived query methods work?**
Spring Data parses method names following a convention (`findBy`, `countBy`, `deleteBy` + property names + keywords like `And`, `OrderBy`, `GreaterThan`) and auto-generates the query.
```java
List<User> findByLastNameAndAgeGreaterThan(String lastName, int age);
```

**Q57. What is `@Query`, and when would you use it over derived methods?**
`@Query` lets you write custom JPQL or native SQL when the query is too complex for method-name derivation, or for performance-tuned queries.
```java
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmailCustom(@Param("email") String email);
```

**Q58. What is the difference between JPQL and native SQL queries in `@Query`?**
JPQL operates on entity objects/fields (database-agnostic, translated to SQL by Hibernate). Native SQL (`nativeQuery = true`) uses actual table/column names and is DB-specific, useful for DB-specific functions.

**Q59. What is the N+1 select problem, and how do you avoid it?**
It happens when fetching a list of entities triggers one query for the list, then one additional query per entity to fetch a related/lazy-loaded association, resulting in N+1 queries. Solutions: use `JOIN FETCH` in JPQL, `@EntityGraph`, or batch fetching configuration.

**Q60. What is the difference between `FetchType.LAZY` and `FetchType.EAGER`?**
- `LAZY`: Related entity/collection is loaded on first access (default for `@OneToMany`/`@ManyToMany`).
- `EAGER`: Related entity is loaded immediately along with the parent (default for `@ManyToOne`/`@OneToOne`).
Overusing EAGER can cause performance issues and unwanted joins.

**Q61. What is Hibernate's first-level and second-level cache?**
- First-level cache: session-scoped, always enabled, caches entities within a single persistence context (session).
- Second-level cache: optional, shared across sessions/SessionFactory, needs explicit configuration (e.g., Ehcache, Caffeine) to cache entities across sessions/transactions.

**Q62. How does Spring Boot auto-configure a DataSource?**
If a JDBC driver and `spring-boot-starter-data-jpa`/`spring-boot-starter-jdbc` are on the classpath, and connection properties (`spring.datasource.url`, `username`, `password`) are set, Spring Boot auto-configures a `DataSource` (using HikariCP as default connection pool since Boot 2.x).

**Q63. What is `spring.jpa.hibernate.ddl-auto` used for? What are its values?**
Controls automatic schema generation/validation by Hibernate:
- `none`: no action
- `validate`: validates schema, makes no changes
- `update`: updates schema based on entities (not recommended for production)
- `create`: drops and recreates schema on startup
- `create-drop`: creates schema on startup, drops it on shutdown (useful for tests)

**Q64. How do you manage database migrations in Spring Boot?**
Using tools like **Flyway** or **Liquibase**, which apply versioned SQL/changelog scripts automatically on startup, tracked in a metadata table (e.g., `flyway_schema_history`). Preferred over `ddl-auto=update` for production environments.

**Q65. What is optimistic locking, and how do you implement it in JPA?**
A concurrency control strategy where conflicts are detected (not prevented) using a version column. Implemented via `@Version` on an entity field; JPA checks the version at update time and throws `OptimisticLockException` if it has changed since it was read.

---

## 10. Spring Security

**Q66. How does Spring Security integrate with Spring Boot?**
Adding `spring-boot-starter-security` auto-configures a default security setup: all endpoints require authentication, a default user is generated with a random password logged at startup, and basic form login/HTTP Basic auth is enabled by default.

**Q67. How do you customize Spring Security configuration?**
By defining a `SecurityFilterChain` bean (Spring Security 5.7+/6.x style, replacing `WebSecurityConfigurerAdapter`):
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .anyRequest().authenticated())
        .formLogin(Customizer.withDefaults());
    return http.build();
}
```

**Q68. What is the difference between Authentication and Authorization?**
- **Authentication**: Verifying who the user is (login/identity).
- **Authorization**: Verifying what the authenticated user is allowed to do (permissions/roles).

**Q69. How does JWT-based authentication work in a Spring Boot app?**
1. User logs in with credentials; server validates and generates a signed JWT (containing claims like username, roles, expiry).
2. Client stores the JWT and sends it in the `Authorization: Bearer <token>` header on subsequent requests.
3. A custom filter (`OncePerRequestFilter`) intercepts requests, validates the JWT signature/expiry, and sets the `Authentication` object in the `SecurityContext` if valid.
4. No server-side session state is needed (stateless authentication).

**Q70. What is CSRF, and how does Spring Security handle it?**
CSRF (Cross-Site Request Forgery) tricks a logged-in user's browser into submitting unwanted requests. Spring Security enables CSRF protection by default for stateful (session-based, browser) apps, requiring a CSRF token on state-changing requests. For stateless REST APIs (JWT-based), CSRF protection is typically disabled since there's no session/cookie-based auth to exploit.

**Q71. What is method-level security in Spring? How do you enable it?**
Enables authorization checks directly on service/controller methods using annotations like `@PreAuthorize`, `@PostAuthorize`, `@Secured`, `@RolesAllowed`. Enabled via `@EnableMethodSecurity` (Spring Security 6) on a configuration class.
```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }
```

**Q72. How does password encoding work in Spring Security?**
Passwords should never be stored in plaintext. `PasswordEncoder` (commonly `BCryptPasswordEncoder`) hashes passwords with a salt before storage, and Spring Security uses it to verify credentials during authentication by re-hashing input and comparing.

---

## 11. Testing

**Q73. What testing support does Spring Boot provide?**
`spring-boot-starter-test` bundles JUnit 5, Mockito, AssertJ, Hamcrest, and Spring Test utilities like `@SpringBootTest`, `MockMvc`, and test slices.

**Q74. What is `@SpringBootTest`?**
Loads the full application context for integration testing. Can start an embedded server with `webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT` for full end-to-end HTTP testing.

**Q75. What are "test slices" (e.g., `@WebMvcTest`, `@DataJpaTest`)?**
Annotations that load only a relevant slice of the application context for faster, focused tests:
- `@WebMvcTest`: Loads only web layer (controllers, `MockMvc`), mocks service layer.
- `@DataJpaTest`: Loads only JPA-related components, uses an in-memory DB by default.
- `@JsonTest`: Tests JSON serialization/deserialization.
- `@RestClientTest`: Tests REST clients.

**Q76. How do you mock dependencies in Spring Boot tests?**
Using Mockito's `@Mock`/`@InjectMocks` for plain unit tests, or Spring's `@MockBean` (registers a Mockito mock in the Spring context, replacing the real bean) for integration-style tests where you need to mock a specific bean in a loaded context.

**Q77. How do you test REST controllers without starting a full server?**
Using `MockMvc` with `@WebMvcTest` or combined with `@SpringBootTest` and `@AutoConfigureMockMvc`:
```java
mockMvc.perform(get("/api/users/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("John"));
```

**Q78. What is Testcontainers, and why is it used with Spring Boot?**
A library that spins up real Docker containers (e.g., PostgreSQL, Kafka, Redis) for integration tests, giving more realistic test environments than in-memory/mocked alternatives. Spring Boot 3.1+ has built-in support via `@ServiceConnection` and `@SpringBootTest` integration.

---

## 12. Microservices with Spring Boot

**Q79. Why is Spring Boot popular for building microservices?**
It provides fast startup, embedded servers (no need for shared app servers), production-ready features (Actuator), easy integration with Spring Cloud for service discovery, config management, circuit breakers, and API gateways — well suited to independently deployable services.

**Q80. What is Spring Cloud, and how does it relate to Spring Boot?**
Spring Cloud is a set of tools built on top of Spring Boot to address common microservices/distributed-system patterns: service discovery (Eureka/Consul), client-side load balancing, centralized configuration (Config Server), API gateway (Spring Cloud Gateway), circuit breakers (Resilience4j), and distributed tracing.

**Q81. What is service discovery, and how is it implemented (e.g., Eureka)?**
Service discovery allows microservices to find and communicate with each other without hardcoding hostnames/ports. Services register themselves with a discovery server (e.g., Netflix Eureka, Consul) at startup, and other services query the registry to locate instances, often combined with client-side load balancing.

**Q82. What is a Circuit Breaker pattern? How is it implemented in Spring Boot?**
A pattern that prevents cascading failures by "opening" (short-circuiting) calls to a failing downstream service after a failure threshold, allowing it time to recover, then gradually testing it again (half-open state). Commonly implemented using **Resilience4j** with `@CircuitBreaker` annotation or programmatic API (Hystrix is now legacy/deprecated).

**Q83. What is an API Gateway, and why use one with microservices?**
A single entry point that routes client requests to appropriate backend microservices, handling cross-cutting concerns like authentication, rate limiting, logging, and load balancing centrally. Spring Cloud Gateway is Spring's reactive gateway implementation.

**Q84. How does distributed configuration management work with Spring Cloud Config?**
A centralized Config Server stores configuration (often backed by a Git repo) for all microservices/environments. Client services fetch their configuration from the Config Server at startup (and optionally refresh it at runtime via `/actuator/refresh` or Spring Cloud Bus).

**Q85. How do microservices typically communicate with each other in Spring Boot apps?**
- Synchronous: REST calls via `RestTemplate` (legacy), `WebClient` (reactive), or declarative clients like OpenFeign.
- Asynchronous: Messaging systems like Kafka, RabbitMQ for event-driven communication.

**Q86. What is distributed tracing, and how is it achieved in Spring Boot microservices?**
Distributed tracing tracks a request as it flows across multiple services, correlating logs/spans with a trace ID. Achieved using **Micrometer Tracing** (formerly Spring Cloud Sleuth) combined with tools like Zipkin or Jaeger for visualization.

---

## 13. Spring Boot with Messaging/Caching

**Q87. How does Spring Boot integrate with Kafka/RabbitMQ?**
Via `spring-kafka` or `spring-boot-starter-amqp`, which auto-configure connection factories, template beans (`KafkaTemplate`, `RabbitTemplate`) for producing messages, and `@KafkaListener`/`@RabbitListener` annotations for consuming messages declaratively.

**Q88. How do you enable caching in Spring Boot?**
Add `@EnableCaching` on a configuration class, then annotate methods:
- `@Cacheable`: Caches the method's return value.
- `@CachePut`: Updates the cache without skipping method execution.
- `@CacheEvict`: Removes entries from the cache.
Spring Boot auto-configures a cache provider (simple in-memory `ConcurrentHashMap` by default, or Redis/Ehcache/Caffeine if on the classpath).

**Q89. What is the difference between `@Cacheable` and `@CachePut`?**
`@Cacheable` skips executing the method if a cached value already exists for the given key. `@CachePut` always executes the method and updates the cache with the new result — useful for update operations where you want the cache refreshed.

---

## 14. Deployment, Packaging & Performance

**Q90. What is a "fat/uber JAR", and how does Spring Boot create one?**
A single JAR that bundles the application code along with all its dependencies and an embedded server, made runnable via `java -jar app.jar`. Created using the Spring Boot Maven/Gradle plugin (`spring-boot-maven-plugin`), which repackages the JAR with a nested-JAR loader (`JarLauncher`).

**Q91. How do you containerize a Spring Boot application with Docker?**
Write a Dockerfile that copies the built jar and runs it, e.g.:
```dockerfile
FROM eclipse-temurin:17-jre
COPY target/app.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
Spring Boot also supports building optimized OCI images directly via `./mvnw spring-boot:build-image` (using Cloud Native Buildpacks), and layered JARs for better Docker layer caching.

**Q92. What are Spring Boot layered JARs, and why are they useful?**
Spring Boot splits the fat JAR into layers (dependencies, spring-boot-loader, snapshot-dependencies, application code) so Docker can cache unchanged layers (like dependencies) separately from frequently-changing application code, speeding up image rebuilds.

**Q93. How do you improve Spring Boot application startup time?**
- Use lazy initialization (`spring.main.lazy-initialization=true`)
- Reduce unnecessary auto-configuration (exclude unused starters)
- Use Spring Boot's AOT (Ahead-Of-Time) processing / GraalVM native image compilation
- Minimize component scanning scope
- Use Spring Native/GraalVM for near-instant startup in serverless contexts

**Q94. What is GraalVM Native Image support in Spring Boot?**
Spring Boot 3+ supports compiling applications into native executables using GraalVM, producing near-instant startup and lower memory footprint by doing ahead-of-time compilation and eliminating JVM warm-up, at the cost of longer build times and some reflection/dynamic-proxy limitations.

**Q95. How can you monitor a Spring Boot application in production?**
Via Actuator endpoints (`/health`, `/metrics`) exposed to monitoring tools, Micrometer for metrics (integrated with Prometheus, Datadog, New Relic, etc.), centralized logging (ELK/EFK stack), and distributed tracing (Zipkin/Jaeger).

---

## 15. Miscellaneous / Advanced

**Q96. What is the difference between Spring Boot 2.x and Spring Boot 3.x?**
Spring Boot 3.x requires Java 17+ as a baseline, migrates from `javax.*` to `jakarta.*` namespace (due to Jakarta EE migration), adds native support for GraalVM/AOT processing, and uses Spring Framework 6.

**Q97. What is `WebClient`, and how is it different from `RestTemplate`?**
`WebClient` is a non-blocking, reactive HTTP client (part of Spring WebFlux) supporting asynchronous calls and reactive streams (`Mono`/`Flux`). `RestTemplate` is the traditional synchronous, blocking HTTP client — now in maintenance mode, with `WebClient` (or `RestClient` in Boot 3.2+) recommended for new projects.

**Q98. What is `RestClient` (Spring Boot 3.2+)?**
A newer, synchronous HTTP client with a fluent API, meant to be a simpler synchronous alternative to `RestTemplate`, built on the same underlying infrastructure as `WebClient`.

**Q99. Difference between Spring MVC and Spring WebFlux?**
Spring MVC is a traditional, synchronous/blocking, thread-per-request model built on the Servlet API. Spring WebFlux is a reactive, non-blocking framework built on Project Reactor, capable of handling higher concurrency with fewer threads — useful for I/O-heavy, high-throughput applications.

**Q100. What is AOP (Aspect-Oriented Programming), and how is it used in Spring Boot?**
AOP allows separating cross-cutting concerns (logging, security, transactions) from business logic using aspects. Spring AOP uses proxies (JDK dynamic proxies or CGLIB) to intercept method calls matched by pointcut expressions and apply advice (`@Before`, `@After`, `@Around`, etc.). `@Transactional` and Spring Security's method security are built on this mechanism.

**Q101. What is the difference between `@RestController` returning an object directly vs `ResponseEntity`?**
Returning a plain object always responds with HTTP 200 (unless an exception/`@ResponseStatus` says otherwise) and serializes the object as the body. `ResponseEntity` gives explicit control over status code, headers, and body — preferred when you need to vary the response (e.g., 201 Created, 404 Not Found).

**Q102. How does Spring Boot handle logging?**
Uses Commons Logging facade internally, with **Logback** as the default underlying implementation (auto-configured). Log levels/output can be configured via `application.properties` (`logging.level.*`, `logging.file.name`) or a custom `logback-spring.xml`.

**Q103. What is the difference between `CommandLineRunner` and `ApplicationRunner`?**
Both are interfaces to run code right after the application context is loaded/application starts. `CommandLineRunner.run(String... args)` gives raw command-line args, while `ApplicationRunner.run(ApplicationArguments args)` gives a parsed `ApplicationArguments` object (with named option/non-option arguments).

**Q104. How do you schedule tasks in Spring Boot?**
Enable via `@EnableScheduling`, then annotate methods with `@Scheduled`:
```java
@Scheduled(fixedRate = 5000)
public void runTask() { ... }

@Scheduled(cron = "0 0 * * * *")
public void hourlyTask() { ... }
```

**Q105. How do you run asynchronous methods in Spring Boot?**
Enable via `@EnableAsync`, then annotate a method with `@Async`. It runs on a separate thread (from a configured `TaskExecutor`), and can return `void`, `Future<T>`, or `CompletableFuture<T>`.

**Q106. What is the difference between `@RequestScope`, `@SessionScope`, `@ApplicationScope`?**
Convenience annotations (introduced in Spring 4.3) that are shorthand for `@Scope("request")`, `@Scope("session")`, and `@Scope("application")` respectively, applied to beans that should live for the duration of a single HTTP request, an HTTP session, or the whole `ServletContext`.

**Q107. What are Spring Boot's embedded server options, and how do you switch between them?**
Tomcat (default), Jetty, and Undertow. Switch by excluding the default Tomcat starter from `spring-boot-starter-web` and adding the desired server's starter dependency instead (see Q7).

**Q108. What is idempotency, and why does it matter for REST APIs?**
An operation is idempotent if performing it multiple times has the same effect as performing it once. `GET`, `PUT`, and `DELETE` are expected to be idempotent by HTTP semantics; `POST` typically is not. This matters for retry-safety in distributed systems (e.g., a client retrying a timed-out request shouldn't create duplicate resources).

**Q109. How would you handle multi-tenancy in a Spring Boot application?**
Common approaches: separate databases per tenant, a shared database with a tenant discriminator column (row-level multi-tenancy), or separate schemas per tenant — often implemented with Hibernate's multi-tenancy support (`MultiTenantConnectionProvider`, `CurrentTenantIdentifierResolver`) combined with a `TenantContext` (e.g., ThreadLocal) populated from a request header or subdomain.

**Q110. What are some Spring Boot best practices for production-grade applications?**
- Use constructor injection over field injection
- Externalize configuration and use profiles per environment
- Never expose sensitive Actuator endpoints publicly without security
- Use DB migration tools (Flyway/Liquibase) instead of `ddl-auto=update`
- Implement proper global exception handling and structured logging
- Add health checks, metrics, and tracing for observability
- Write unit + integration tests (including Testcontainers for realistic DB/queue tests)
- Use DTOs instead of exposing JPA entities directly in the API layer
- Keep secrets out of source control (use vaults/secret managers)

---

## Quick-Revision Cheat Sheet

| Topic | Key Point |
|---|---|
| `@SpringBootApplication` | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| Default embedded server | Tomcat |
| Default bean scope | Singleton |
| Default cache implementation | `ConcurrentHashMap` (unless Redis/Ehcache present) |
| Recommended DI style | Constructor injection |
| Default connection pool | HikariCP |
| Actuator endpoints exposed by default | `health`, `info` |
| Recommended migration tool | Flyway or Liquibase |
| Modern reactive HTTP client | `WebClient` |
| Modern synchronous HTTP client (3.2+) | `RestClient` |
| Namespace change in Boot 3.x | `javax.*` → `jakarta.*` |

---

*Good luck with your interview! Review each section, and be ready to back up answers with small code examples — interviewers often ask you to write a quick snippet (a REST controller, a repository method, or an exception handler) live.*
