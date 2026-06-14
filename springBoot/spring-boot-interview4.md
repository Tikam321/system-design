# 1. # Difference Between Lazy Loading and Eager Loading

| Lazy Loading | Eager Loading |
|---|---|
| Data loads only when needed | Data loads immediately |
| Improves initial performance | More data fetched upfront |
| Reduces memory usage | Higher memory usage |
| May cause additional queries | Fewer additional queries |

---

# Lazy Loading

Related data is fetched only when accessed.

## Example

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;
```

Orders load only when:

```java
user.getOrders()
```

is called.

---

# Eager Loading

Related data loads immediately with parent entity.

## Example

```java
@OneToMany(fetch = FetchType.EAGER)
private List<Order> orders;
```

Orders load together with User entity.

---

# LazyInitializationException

Occurs when lazy-loaded data is accessed after Hibernate session is closed.

---

# Example

```java
User user = userRepository.findById(1).get();

session.close();

user.getOrders();
```

Output:

```text
LazyInitializationException
```

---

# Why?

Because:
- Orders were not loaded initially
- Session already closed
- Hibernate cannot fetch data now

---

# Solutions

- Use `JOIN FETCH`
- Use transactional boundaries properly
- DTO mapping
- Open Session In View (less preferred)

---

# Interview Ready Answer

> Lazy loading fetches related data only when required, while eager loading fetches related data immediately with the parent entity. LazyInitializationException occurs when a lazy-loaded association is accessed after the Hibernate session is closed, preventing Hibernate from loading the required data.

# 2. # `AutoConfiguration.imports` in Spring Boot

Spring Boot uses:

```text
META-INF/spring/
org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

to know which auto-configuration classes should be loaded during application startup.

---

# What Does This File Contain?

It contains list of auto-configuration classes.

Example:

```text
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
```

---

# What Are Auto-Configuration Classes?

These are predefined Spring Boot configuration classes that automatically create beans based on conditions.

Example:

```java
@Configuration
public class DataSourceAutoConfiguration {

}
```

---

# What Do They Do?

They automatically configure:
- DataSource
- Tomcat
- DispatcherServlet
- JPA
- Security
- Jackson
- Redis
etc.

based on:
- Dependencies in classpath
- Properties
- Existing beans

---

# Internal Flow

```text
1. Spring Boot starts
2. Reads AutoConfiguration.imports
3. Loads auto-configuration classes
4. Checks conditions
5. Creates required beans automatically
```

---

# Example

If project contains:

```xml
spring-boot-starter-data-jpa
```

Spring Boot loads:

```text
DataSourceAutoConfiguration
HibernateJpaAutoConfiguration
```

and automatically creates:
- DataSource
- EntityManager
- TransactionManager

---

# Conditional Annotations

Auto-config classes use conditions like:

```java
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnProperty
```

---

# Example

```java
@ConditionalOnClass(DataSource.class)
```

Configuration loads only if:
- DataSource dependency exists

---

# Important Point

If developer already defines custom bean:

```java
@Bean
DataSource dataSource()
```

Spring Boot usually skips auto-config using:

```java
@ConditionalOnMissingBean
```

---

# Interview Ready Answer

> `AutoConfiguration.imports` is a Spring Boot metadata file that contains the list of auto-configuration classes to load during startup. Auto-configuration classes are predefined configuration classes that automatically create beans like DataSource, DispatcherServlet, JPA, and Security components based on classpath dependencies, properties, and conditional annotations such as `@ConditionalOnClass` and `@ConditionalOnMissingBean`.


# 3. # `AutoConfiguration.imports` in Spring Boot

Spring Boot uses:

```text
META-INF/spring/
org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

to know which auto-configuration classes should be loaded during application startup.

---

# What Does This File Contain?

It contains list of auto-configuration classes.

Example:

```text
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
```

---

# What Are Auto-Configuration Classes?

These are predefined Spring Boot configuration classes that automatically create beans based on conditions.

Example:

```java
@Configuration
public class DataSourceAutoConfiguration {

}
```

---

# What Do They Do?

They automatically configure:
- DataSource
- Tomcat
- DispatcherServlet
- JPA
- Security
- Jackson
- Redis
etc.

based on:
- Dependencies in classpath
- Properties
- Existing beans

---

# Internal Flow

```text
1. Spring Boot starts
2. Reads AutoConfiguration.imports
3. Loads auto-configuration classes
4. Checks conditions
5. Creates required beans automatically
```

---

# Example

If project contains:

```xml
spring-boot-starter-data-jpa
```

Spring Boot loads:

```text
DataSourceAutoConfiguration
HibernateJpaAutoConfiguration
```

and automatically creates:
- DataSource
- EntityManager
- TransactionManager

---

# Conditional Annotations

Auto-config classes use conditions like:

```java
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnProperty
```

---

# Example

```java
@ConditionalOnClass(DataSource.class)
```

Configuration loads only if:
- DataSource dependency exists

---

# Important Point

If developer already defines custom bean:

```java
@Bean
DataSource dataSource()
```

Spring Boot usually skips auto-config using:

```java
@ConditionalOnMissingBean
```

---

# Interview Ready Answer

> `AutoConfiguration.imports` is a Spring Boot metadata file that contains the list of auto-configuration classes to load during startup. Auto-configuration classes are predefined configuration classes that automatically create beans like DataSource, DispatcherServlet, JPA, and Security components based on classpath dependencies, properties, and conditional annotations such as `@ConditionalOnClass` and `@ConditionalOnMissingBean`.


# 4. # Spring Boot Auto-Configuration Internal Flow

```text
1. Application starts
2. @EnableAutoConfiguration gets triggered
3. Spring Boot scans dependencies available in classpath
4. Spring Boot reads:

META-INF/spring/
org.springframework.boot.autoconfigure.AutoConfiguration.imports

5. Auto-configuration classes get loaded
6. Conditional annotations are checked:
   - @ConditionalOnClass
   - @ConditionalOnMissingBean
   - @ConditionalOnProperty

7. Based on:
   - Dependencies
   - Properties
   - Existing beans

   Spring automatically creates required beans

8. Beans are registered into IoC Container
```

---

# Example

If classpath contains:

```xml
spring-boot-starter-web
```

Spring Boot detects:
- Tomcat
- Spring MVC
- DispatcherServlet

Then corresponding auto-config classes create beans automatically.

---

# Important Point

Spring Boot creates beans only when conditions match.

Example:

```java
@ConditionalOnMissingBean
```

means:

```text
Create bean only if user has not already created one.
```

---

# Interview Ready Answer

> Yes, during application startup Spring Boot scans dependencies available in the classpath and reads auto-configuration classes from `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`. Based on conditional annotations, properties, dependencies, and existing beans, Spring Boot automatically creates and registers required beans into the IoC container.


# 6. # Why `peek()` Should Not Be Used for Business Logic?

`peek()` is mainly designed for:

```text
Debugging and logging
```

not for modifying application/business state.

---

# Why?

Because `peek()`:
- Is an intermediate operation
- May not execute if terminal operation is missing
- Can behave unpredictably in parallel streams
- Makes code harder to understand

---

# Bad Practice

```java
list.stream()
    .peek(user -> user.setName("Java"))
    .collect(Collectors.toList());
```

Using `peek()` for modifying data/business logic is discouraged.

---

# Correct Usage

```java
list.stream()
    .peek(System.out::println)
    .collect(Collectors.toList());
```

Used for:
- Debugging
- Logging
- Inspecting stream elements

---

# Use map() Instead

For transformation/business logic:

```java
list.stream()
    .map(user -> {
        user.setName("Java");
        return user;
    })
    .collect(Collectors.toList());
```

---

# Interview Ready Answer

> `peek()` is intended mainly for debugging and logging, not business logic. Since it is an intermediate operation, its execution depends on terminal operations and can behave unpredictably, especially in parallel streams. For transformations or business logic, `map()` should be used instead.


# 7. # Challenges Faced While Upgrading Java Version

# 1. Dependency Compatibility Issues

Some libraries/frameworks may not support newer Java versions.

Example:
- Older Spring/Hibernate versions
- Deprecated APIs

---

# 2. Removed/Deprecated APIs

Certain APIs get removed in newer Java versions.

Example:

```text
javax.* → jakarta.*
```

---

# 3. Build Tool Compatibility

Need compatible:
- Maven
- Gradle
- Plugins

versions.

---

# 4. JVM Behavior Changes

Garbage collector or JVM optimizations may behave differently.

Can affect:
- Performance
- Memory usage

---

# 5. Reflection and Module Issues

Java 9+ module system may block reflective access.

Example:

```text
Illegal reflective access warning
```

---

# 6. Docker/Base Image Updates

Need updated:
- JDK image
- Runtime environment

---

# 7. CI/CD Pipeline Changes

Pipelines may require:
- New JDK setup
- Updated build agents

---

# 8. Regression Testing

Need extensive testing to ensure:
- Existing functionality works
- No production issues occur

---

# 9. Production Performance Validation

Need to monitor:
- CPU usage
- GC behavior
- Startup time
- Throughput

after upgrade.

---

# Interview Ready Answer

> While upgrading Java versions, common challenges include dependency incompatibility, removed/deprecated APIs, module system restrictions, build tool compatibility, JVM behavior changes, and regression testing. We usually validate library support, update build configurations, perform extensive testing, and monitor application performance after deployment.

# 8 Difference Between Comparable and Comparator

| Comparable | Comparator |
|---|---|
| Used for natural sorting | Used for custom sorting |
| Present in `java.lang` package | Present in `java.util` package |
| Class itself implements it | Separate class/lambda implements it |
| Has `compareTo()` method | Has `compare()` method |
| Supports single sorting logic | Supports multiple sorting logics |

---

# Comparable

Used when class defines its own default sorting.

## Example

```java
class Employee implements Comparable<Employee> {

    int salary;

    @Override
    public int compareTo(Employee e) {

        return this.salary - e.salary;
    }
}
```

Sorting:

```java
Collections.sort(list);
```

---

# Comparator

Used for custom/dynamic sorting.

## Example

```java
Comparator<Employee> comparator =
    (e1, e2) -> e1.salary - e2.salary;
```

Sorting:

```java
Collections.sort(list, comparator);
```

---

# Main Difference

## Comparable

```text
Sorting logic inside class
```

## Comparator

```text
Sorting logic outside class
```

---

# Example Use Case

## Comparable

```text
Default sorting by salary
```

## Comparator

```text
Sort by:
- salary
- name
- age
```

Multiple sorting options possible.

---

# Interview Ready Answer

> Comparable is used for natural/default sorting and requires the class itself to implement `compareTo()`. Comparator is used for custom sorting and implements the `compare()` method externally, allowing multiple sorting strategies.

# 10. What is the difference between PUT and POST in REST APIs?
- POST is used to create a new resource, and the server usually generates the resource ID.
- PUT is used to update a resource when the client already knows the resource identifier.
- POST is not idempotent - if you send the same POST twice, you may create two records.
- PUT is idempotent - sending the same request multiple times results in the same final state.
- Example: POST /users creates a new user, PUT /users/101 updates user 101.
- In real APIs, we use POST for create, PUT for full update, and PATCH for partial update.
