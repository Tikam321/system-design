# 1. Spring Bean Lifecycle

```text
1. Bean Instantiation
   → Spring creates the bean object.

2. Dependency Injection
   → Spring injects required dependencies.

3. BeanPostProcessor Before Initialization
   → Spring processes bean before initialization.

4. @PostConstruct / afterPropertiesSet()
   → Initialization logic executes.

5. Bean Ready for Use
   → Bean is available for application usage.

6. BeanPostProcessor After Initialization
   → Spring performs post-initialization processing.

7. @PreDestroy / destroy()
   → Cleanup logic executes before bean destruction.
```
# 2. Why Spring Boot Does Not Rollback for Checked Exceptions

By default, Spring transaction management rolls back transactions only for:

- Runtime Exceptions (`RuntimeException`)
- Errors (`Error`)

It does NOT rollback for checked exceptions because checked exceptions are considered recoverable/business exceptions.

---

# Example

```java
@Transactional
public void createUser() throws Exception {

    userRepository.save(user);

    throw new Exception("Checked Exception");
}
```

In this case:

- Exception occurs
- But transaction does NOT rollback
- Data may still get committed

---

# Why?

Spring follows EJB transaction behavior rules:

- Unchecked exceptions → System failures → Rollback
- Checked exceptions → Business/recoverable exceptions → No rollback

Spring assumes checked exceptions can be handled safely.

---

# How to Enable Rollback for Checked Exceptions

Use:

```java
@Transactional(rollbackFor = Exception.class)
```

---

# Example

```java
@Transactional(rollbackFor = Exception.class)
public void createUser() throws Exception {

    userRepository.save(user);

    throw new Exception("Checked Exception");
}
```

Now transaction will rollback properly.

---

# Interview Ready Answer

> By default, Spring does not rollback transactions for checked exceptions because checked exceptions are treated as recoverable business exceptions. Spring only automatically rolls back for unchecked exceptions like RuntimeException and Error. To enable rollback for checked exceptions, we use `@Transactional(rollbackFor = Exception.class)`.
# 3. # Does volatile Guarantee Atomicity?

No, `volatile` does NOT guarantee atomicity.

## volatile guarantees:
- Visibility
- Ordering

## volatile does NOT guarantee:
- Atomicity
- Thread safety for updates like `count++`

---

# Example

```java
volatile int count = 0;

count++;
```

`count++` internally does:

```text
1. Read
2. Increment
3. Write
```

Multiple threads can overlap these steps and cause race conditions.

---

# Correct Solution

Use `AtomicInteger`.

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

---

# Interview Ready Answer

> No, volatile only guarantees visibility and ordering, not atomicity. Multiple threads updating a volatile variable can still cause race conditions because operations like `count++` are not atomic.

# 4. # Difference Between `==` and `.equals()` in Java

| `==` | `.equals()` |
|---|---|
| Compares reference/address | Compares object content/value |
| Checks if both references point to same object | Checks logical equality |
| Works for primitives and objects | Mainly used for objects |
| Can compare memory location | Can compare actual data |

---

# Example with String

```java
String s1 = new String("Java");
String s2 = new String("Java");

System.out.println(s1 == s2);        // false
System.out.println(s1.equals(s2));   // true
```

---

# Why?

## `==`

Checks:

```text
Do both variables point to same object?
```

## `.equals()`

Checks:

```text
Do both objects contain same value?
```

---

# Primitive Example

```java
int a = 10;
int b = 10;

System.out.println(a == b); // true
```

For primitives, `==` compares actual values.

---

# Interview Ready Answer

> `==` compares references or memory addresses, while `.equals()` compares object content or logical equality. For primitives, `==` compares actual values, but for objects it checks whether both references point to the same object.

# 5. # Have You Used `@Query` (JPQL) in Spring?

Yes, `@Query` is used in Spring Data JPA to write custom JPQL or native SQL queries when derived query methods are not sufficient.

---

# Example Using JPQL

```java
@Repository
public interface UserRepository
        extends JpaRepository<User, Long> {

    @Query("SELECT u FROM User u WHERE u.email = :email")
    User findByEmail(@Param("email") String email);
}
```

---

# Why Use `@Query`?

- Complex queries
- Custom joins
- Aggregations
- Dynamic filtering
- Better readability for complex logic

---

# JPQL vs Native Query

| JPQL | Native SQL |
|---|---|
| Works with Entity names | Works with table names |
| Database independent | Database specific |
| Uses object fields | Uses actual column names |

---

# Native Query Example

```java
@Query(
 value = "SELECT * FROM users WHERE email = :email",
 nativeQuery = true
)
User findUser(@Param("email") String email);
```

---

# Interview Ready Answer

> Yes, I have used `@Query` in Spring Data JPA for writing custom JPQL and native SQL queries when derived query methods were not enough. JPQL works with entity objects and fields instead of table names and is useful for complex joins, filtering, and optimized database queries.

# 6. # What Happens When Two HashMap Keys Have Same hashCode? (Collision)

When two keys produce the same hashCode, it is called a collision.

HashMap handles collisions using:

- Linked List (before Java 8)
- Linked List + Balanced Tree (Java 8+)

---

# Flow Inside HashMap

```text
1. hashCode() calculates bucket index
2. Both keys go to same bucket
3. HashMap uses equals() to differentiate keys
4. Entries are stored in linked list/tree inside bucket
```

---

# Example

```java
Map<String, Integer> map = new HashMap<>();

map.put("FB", 1);
map.put("Ea", 2);
```

Both `"FB"` and `"Ea"` have same hashCode.

---

# Internal Structure

```text
Bucket
  ↓
[FB=1] -> [Ea=2]
```

HashMap traverses bucket and uses:

```java
equals()
```

to identify correct key.

---

# In Java 8+

If collisions become too large:

```text
Linked List → Red Black Tree
```

This improves lookup performance.

---

# Time Complexity

| Scenario | Complexity |
|---|---|
| Normal | O(1) |
| Heavy Collision (Linked List) | O(n) |
| Heavy Collision (Tree) | O(log n) |

---

# Important Point

Same `hashCode()` does NOT mean objects are equal.

HashMap always verifies using:

```java
equals()
```

---

# Interview Ready Answer

> When two HashMap keys generate the same hashCode, a collision occurs. Both entries are stored in the same bucket, and HashMap uses `equals()` to differentiate between them. Before Java 8, collisions were handled using linked lists, while Java 8+ converts heavily populated buckets into Red-Black Trees for better performance.

#7. # Difference Between JVM, JRE, and JDK

| Component | Purpose |
|---|---|
| JVM | Runs Java bytecode |
| JRE | Provides environment to run Java applications |
| JDK | Provides tools to develop and run Java applications |

---

# JVM (Java Virtual Machine)

- Executes `.class` bytecode
- Platform dependent
- Provides:
  - Memory management
  - Garbage collection
  - JIT compilation

---

# JRE (Java Runtime Environment)

- Used to run Java applications
- Contains:
  - JVM
  - Libraries
  - Runtime files

```text
JRE = JVM + Libraries
```

---

# JDK (Java Development Kit)

- Used to develop Java applications
- Contains:
  - JRE
  - Compiler (`javac`)
  - Debugging tools
  - Development tools

```text
JDK = JRE + Development Tools
```

---

# Simple Flow

```text
Java Code (.java)
      ↓
Compiler (javac)
      ↓
Bytecode (.class)
      ↓
JVM Executes
```

---

# Interview Ready Answer

> JVM executes Java bytecode, JRE provides the runtime environment to run Java applications, and JDK provides development tools like compiler and debugger along with JRE to build and run Java applications.


# 7. # Primitive vs Non-Primitive Memory Allocation in Java

| Primitive Data Types | Non-Primitive Data Types |
|---|---|
| Stores actual value | Stores reference/address |
| Mostly stored in Stack | Object stored in Heap |
| Fixed memory size | Dynamic memory size |
| Faster access | Slightly slower due to reference lookup |

---

# Primitive Example

```java
int a = 10;
```

Memory:

```text
Stack
-----
a = 10
```

Primitive variable directly stores value.

---

# Non-Primitive Example

```java
String s = new String("Java");
```

Memory:

```text
Stack                Heap
-----                -----
s  --------->      "Java"
```

- Stack stores reference
- Actual object stored in Heap

---

# Important Point

## Primitive Types

Examples:

```text
int
char
double
boolean
```

Store actual data directly.

---

## Non-Primitive Types

Examples:

```text
String
Array
Object
Class
List
```

Store object references.

---

# Why Heap for Objects?

Objects are:

- Dynamic
- Larger in size
- Shared across methods/threads

Heap allows dynamic memory allocation.

---

# Interview Ready Answer

> Primitive data types store actual values directly and are usually allocated in stack memory, while non-primitive data types store references in stack memory and their actual objects are allocated in heap memory. Primitives have fixed size, whereas objects use dynamic memory allocation.


# 8. can you expexplain the internal worlking of springboot at high level ?
1. application.run()
2. application.context
3. component scan
4. autoconfiguratoin start configuraiton based on the dependency like base on jdbc driver and web dependcy it automatically configure tomcat and datasource obbjct.
5. start the embedded server

# 9. # Difference Between JVM and JIT

| JVM | JIT |
|---|---|
| Java Virtual Machine | Just-In-Time Compiler |
| Runs Java bytecode | Converts bytecode into native machine code |
| Part of JRE | Part of JVM |
| Handles memory, GC, execution | Improves execution performance |
| Interprets bytecode | Compiles frequently used code |

---

# JVM (Java Virtual Machine)

- Executes Java bytecode
- Provides platform independence
- Handles:
  - Memory management
  - Garbage Collection
  - Class loading
  - Security

---

# JIT (Just-In-Time Compiler)

- Part of JVM
- Converts frequently executed bytecode into native machine code
- Improves performance by avoiding repeated interpretation

---

# Flow

```text
Java Code
   ↓
Bytecode (.class)
   ↓
JVM
   ↓
JIT converts hot code to native machine code
   ↓
Faster execution
```

---

# Interview Ready Answer

> JVM is the runtime environment that executes Java bytecode and manages memory, garbage collection, and class loading. JIT is a component inside JVM that improves performance by converting frequently executed bytecode into native machine code.


# 9. What is Parallel Stream API in Java?

Parallel Stream allows processing stream operations using multiple threads simultaneously to improve performance on large datasets.

It uses:

```text
ForkJoinPool (Common Thread Pool)
```

internally.

---

# Example

```java
List<Integer> numbers =
    Arrays.asList(1, 2, 3, 4, 5);

numbers.parallelStream()
       .forEach(System.out::println);
```

---

# How It Works

```text
Collection
   ↓
Split into multiple parts
   ↓
Multiple threads process data in parallel
   ↓
Combine results
```

---

# Sequential vs Parallel Stream

| Sequential Stream | Parallel Stream |
|---|---|
| Single thread | Multiple threads |
| Slower for huge data | Faster for large computations |
| Ordered execution | Order may vary |

---

# Example

## Sequential

```java
list.stream()
```

## Parallel

```java
list.parallelStream()
```

or

```java
list.stream().parallel()
```

---

# Benefits

- Better CPU utilization
- Faster processing for large datasets
- Easy parallelism with minimal code

---

# Drawbacks

- Thread overhead
- Not always faster
- Can cause race conditions with shared mutable data
- Order is not guaranteed

---

# Best Use Cases

- Large datasets
- CPU-intensive operations
- Independent tasks

---

# Avoid Using For

- Small datasets
- Database/API calls
- Shared mutable state

---

# Interview Ready Answer

> Parallel Stream API in Java allows stream operations to run concurrently using multiple threads through the ForkJoinPool framework. It improves performance for large CPU-intensive tasks by splitting data into multiple parts and processing them in parallel.


# .10  What is Service Discovery?

Service Discovery is a mechanism in microservices architecture that helps services dynamically find and communicate with each other without hardcoding IP addresses or ports.

Since microservice instances scale dynamically, their IPs and ports keep changing.

Service Discovery solves this problem.

---

# Why Do We Need It?

Suppose:

```text
Order Service → Needs to call Payment Service
```

But Payment Service instances may:

- Scale up/down
- Restart
- Change IP/port

Instead of hardcoding:

```text
http://192.168.1.10:8080
```

We use:

```text
http://PAYMENT-SERVICE
```

Service discovery resolves actual instance automatically.

---

# How It Works

```text
1. Service starts
2. Registers itself with Service Registry
3. Client asks registry for service location
4. Registry returns available instances
5. Client calls service
```

---

# Architecture

```text
Order Service
      ↓
Service Registry (Eureka)
      ↓
Payment Service Instances
```

---

# Common Service Discovery Tools

| Tool | Usage |
|---|---|
| Eureka | Spring Cloud |
| Consul | HashiCorp |
| Zookeeper | Distributed systems |
| Kubernetes DNS | Kubernetes |
| Nacos | Alibaba ecosystem |

---

# Types of Service Discovery

---

# 1. Client-Side Discovery

Client directly asks registry.

```text
Client → Registry → Service
```

Example:
- Eureka + Ribbon

---

# 2. Server-Side Discovery

Load balancer queries registry.

```text
Client → Load Balancer → Service
```

Example:
- Kubernetes
- AWS ELB

---

# Spring Boot Eureka Example

---

# Eureka Server

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

}
```

---

# Register Microservice

```properties
eureka.client.service-url.defaultZone=
http://localhost:8761/eureka
```

---

# Enable Discovery Client

```java
@EnableDiscoveryClient
@SpringBootApplication
public class PaymentServiceApplication {

}
```

---

# Call Service Using Name

```java
http://PAYMENT-SERVICE/api/pay
```

---

# Benefits

- Dynamic scaling
- Fault tolerance
- Load balancing
- No hardcoded IPs
- Better microservice communication

---

# Interview Ready Answer

> Service Discovery is a mechanism used in microservices architecture to dynamically locate service instances without hardcoding IP addresses or ports. Services register themselves with a service registry like Eureka or Consul, and other services discover them using service names. It helps in dynamic scaling, load balancing, and fault-tolerant communication between microservices.

# 11 Authentication and Authorization in Microservices

# Authentication

Authentication verifies:

```text
Who the user is
```

Example:
- Username/password
- JWT token
- OAuth login

---

# Authorization

Authorization verifies:

```text
What the user can access
```

Example:
- ADMIN can delete user
- USER can only view profile

---

# Common Approach in Microservices

Usually implemented using:

- API Gateway
- JWT Token
- OAuth2 / Keycloak / Auth Server

---

# Flow

```text
1. User logs in
2. Auth Service validates credentials
3. JWT token generated
4. Client sends JWT with every request
5. API Gateway validates token
6. Request forwarded to microservice
7. Microservice checks roles/permissions
```

---

# Architecture

```text
Client
   ↓
API Gateway
   ↓
Auth Service
   ↓
Microservices
```

---

# JWT-Based Authentication

## Login

```text
username + password
```

## Server Generates

```text
JWT Token
```

## Client Sends

```text
Authorization: Bearer <token>
```

---

# Authorization Example

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser() {

}
```

Only ADMIN can access API.

---

# Why JWT in Microservices?

- Stateless
- Scalable
- No session sharing required
- Easy service-to-service communication

---

# Common Tools

| Tool | Purpose |
|---|---|
| Spring Security | Security framework |
| JWT | Token authentication |
| OAuth2 | Authorization framework |
| Keycloak | Identity provider |
| API Gateway | Central authentication layer |

---

# Interview Ready Answer

> In microservices, authentication and authorization are commonly implemented using JWT and Spring Security. A dedicated Auth Service validates user credentials and generates JWT tokens. Clients send the token with every request, and the API Gateway or microservices validate the token before processing requests. Authorization is handled using roles and permissions such as `ADMIN` or `USER`.


# 12  What Does `alg` Mean in JWT Header?

`alg` means:

```text
Signing Algorithm
```

It tells which algorithm was used to create the JWT signature.

---

# Example Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Here:

```text
HS256 = HMAC SHA-256
```

---

# Why Is It Needed?

When server receives JWT:

1. Reads `alg`
2. Uses same algorithm + secret key
3. Recreates signature
4. Compares with JWT signature

If signatures match:

```text
Token is valid
```

Else:

```text
Token is tampered
```

---

# How Signature Is Created

```text
Signature =
Hash(
   Header + Payload + SecretKey
)
```

using algorithm mentioned in:

```text
alg
```

---

# Common JWT Algorithms

| Algorithm | Type |
|---|---|
| HS256 | Symmetric |
| HS512 | Symmetric |
| RS256 | Asymmetric (Public/Private Key) |
| ES256 | Elliptic Curve |

---

# Most Common

## HS256

Uses:

```text
Same secret key
```

for:
- Signing
- Verification

---

# RS256

Uses:

```text
Private Key → Sign
Public Key → Verify
```

More secure for distributed systems.

---

# Interview Ready Answer

> The `alg` field in JWT header represents the signing algorithm used to generate the token signature. When the server creates a JWT, it hashes the header and payload using the specified algorithm and a secret/private key to generate the signature. During verification, the server uses the same algorithm to validate token integrity and detect tampering.


# 12. How Do You Secure Communication Between Microservices?

- Use HTTPS/TLS to encrypt communication
- Use JWT/OAuth2 for authentication and authorization
- Use API Gateway for centralized security
- Use mTLS for secure service-to-service authentication
- Use roles/permissions for access control
- Use private networks/firewalls for network security

---

# Interview Ready Answer

> Microservice communication is secured using HTTPS/TLS for encryption, JWT or OAuth2 for authentication and authorization, API Gateway for centralized security, and mTLS for secure service-to-service communication.



# 13. Approach to Blue-Green Deployment

Blue-Green Deployment is a deployment strategy where:

- Blue = Current production environment
- Green = New version environment

Both environments run simultaneously.

---

# Approach

```text
1. Deploy new version to Green environment
2. Test Green environment
3. Switch traffic from Blue → Green
4. Monitor application
5. Rollback to Blue if issues occur
```

---

# Benefits

- Zero downtime deployment
- Easy rollback
- Safer releases
- Reduced production risk

---

# Commonly Used With

- Load Balancer
- Kubernetes
- Docker
- Cloud platforms (AWS, Azure)

---

# Interview Ready Answer

> In Blue-Green deployment, we maintain two identical environments: Blue for current production and Green for the new release. We deploy and test the new version in Green, then switch traffic using a load balancer. If any issue occurs, traffic can quickly be rolled back to Blue, ensuring zero downtime and safer deployments.


# 14. # Example: equals() True but Different hashCode() (Wrong Implementation)

This violates Java contract and breaks HashMap behavior.

---

# Wrong Example

```java
class User {

    int id;

    User(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object obj) {

        User u = (User) obj;

        return this.id == u.id;
    }

    @Override
    public int hashCode() {

        return (int)(Math.random() * 1000);
    }
}
```

---

# Usage

```java
User a = new User(1);
User b = new User(1);

System.out.println(a.equals(b)); // true

System.out.println(a.hashCode());
System.out.println(b.hashCode());
```

Possible output:

```text
true
101
567
```

---

# Why Is This Wrong?

Because:

```java
a.equals(b) == true
```

but:

```java
a.hashCode() != b.hashCode()
```

This breaks HashMap contract.

---

# HashMap Problem

```java
Map<User, String> map = new HashMap<>();

map.put(a, "Java");

System.out.println(map.get(b));
```

Output:

```text
null
```

Even though:

```java
a.equals(b) == true
```

---

# Why null?

Because:

```text
a → stored in Bucket 101
b → searched in Bucket 567
```

HashMap never reaches correct bucket to call `equals()`.

---

# Correct Implementation

```java
@Override
public int hashCode() {
    return Objects.hash(id);
}
```

---

# Interview Ready Answer

> If `equals()` returns true but hashCodes are different, HashMap breaks because it uses hashCode to locate the bucket first. If objects go into different buckets, HashMap never reaches the object to compare using `equals()`, causing retrieval failures.


# 15. Can Two Equal Objects Have Different hashCode?

No.

If two objects are equal according to `equals()`, then their `hashCode()` must also be equal.

---

# Java Contract

```text
If a.equals(b) == true
then
a.hashCode() == b.hashCode()
```

Otherwise collections like:

- HashMap
- HashSet

will not work correctly.

---

# Example

```java
String s1 = new String("Java");
String s2 = new String("Java");

System.out.println(s1.equals(s2));     // true
System.out.println(s1.hashCode() == s2.hashCode()); // true
```

---

# Important Point

The reverse is NOT mandatory.

```text
Same hashCode does NOT mean objects are equal
```

Hash collisions can happen.

---

# Interview Ready Answer

> No, if two objects are equal according to `equals()`, they must return the same `hashCode()`. This is part of the Java hashCode-equals contract required for collections like HashMap and HashSet to function correctly.

# 15. Application Goes Down at 3 AM Due to DB Overload — Actions Taken

# Immediate Actions

## 1. Identify Root Cause

Check:
- DB CPU usage
- Memory usage
- Slow queries
- Connection pool exhaustion
- Sudden traffic spike
- Locks/deadlocks

Use:
- APM tools
- DB monitoring dashboards
- Slow query logs

---

# 2. Stabilize System

Temporarily:
- Restart failed services if needed
- Increase DB resources
- Scale application instances
- Limit incoming traffic/rate limit

---

# 3. Check Slow Queries

Run:

```sql
EXPLAIN ANALYZE
```

on heavy queries.

Look for:
- Full table scans
- Missing indexes
- Expensive joins

---

# 4. Optimize Database

- Add indexes
- Kill long-running queries
- Optimize joins
- Reduce unnecessary reads

---

# 5. Reduce DB Load

Use:
- Redis caching
- Read replicas
- Pagination
- Batch processing

---

# 6. Check Connection Pool

Verify:
- HikariCP settings
- Max pool size
- Connection leaks

---

# 7. Traffic Handling

If sudden spike:
- Enable autoscaling
- Use load balancer
- Apply circuit breaker/rate limiting

---

# 8. Post-Incident Actions

After recovery:
- RCA (Root Cause Analysis)
- Add monitoring/alerts
- Capacity planning
- Load testing
- Query optimization review

---

# Interview Ready Answer

> If the application goes down due to DB overload, I would first stabilize the system by checking database metrics, slow queries, connection pool usage, and traffic spikes. I would optimize heavy queries using EXPLAIN ANALYZE, add indexes if needed, and reduce DB load using caching or read replicas. I would also scale services temporarily, monitor recovery, and later perform RCA and capacity planning to prevent future incidents.
