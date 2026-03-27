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

13. differencebetween tomact and jetty server
- Tomcat and Jetty are both Java servlet servers.
- Tomcat is the default in Spring Boot, very widely used, and usually the easiest “standard” choice with stronger ecosystem support.
- Jetty is generally lighter and more modular, and many teams prefer it for embedded apps or workloads with lots of long-lived async connections (like WebSocket/SSE).
- In short: choose Tomcat for simplicity and broad support, choose Jetty when lightweight footprint and async connection handling are top priorities.
  
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

It mainly provides:

Spring MVC (@RestController, @RequestMapping, etc.)
An embedded web server (Tomcat by default)
JSON support (Jackson) for request/response conversion
Core web auto-configuration so you can run with minimal setup

















