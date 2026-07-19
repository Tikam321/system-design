
# Spring AOP (Aspect-Oriented Programming)

## What is AOP?

Aspect-Oriented Programming (AOP) is a programming paradigm that separates **cross-cutting concerns** (such as logging, security, transactions, caching, auditing, etc.) from the core business logic.

Instead of writing the same code in multiple methods, AOP allows you to write it once and apply it wherever required.

---

## Why do we need AOP?

Imagine you have hundreds of service methods.

Without AOP, you may end up writing logging, security checks, execution time calculation, and exception handling in every method.

### Without AOP

```java
public void transferMoney() {

    System.out.println("Request Started");

    long start = System.currentTimeMillis();

    // Business Logic

    long end = System.currentTimeMillis();

    System.out.println("Execution Time : " + (end - start));
}
```

Imagine writing the above logging and timing code in **500 methods**.

### With AOP

```java
public void transferMoney() {

    // Only Business Logic

}
```

The logging, timing, security, and transaction management happen automatically through an **Aspect**.

---

# Core Terminology

## 1. Aspect

An **Aspect** is simply a Java class that contains the cross-cutting logic.

Example:

```java
@Aspect
@Component
public class LoggingAspect {

}
```

Here, `LoggingAspect` is an Aspect.

Think of it as:

```
Aspect
    |
    +-- Logging
    +-- Security
    +-- Performance Monitoring
    +-- Auditing
```

---

## 2. Join Point

A **Join Point** is a point during program execution where an Aspect can be applied.

In **Spring AOP**, the only supported Join Point is **method execution**.

Example:

```java
paymentService.charge();
```

When the `charge()` method executes, that execution is the **Join Point**.

> **Note:** AspectJ supports constructor execution, field access, object initialization, etc., but Spring AOP only supports **method execution**.

---

## 3. Pointcut

A **Pointcut** tells Spring **which Join Points should be intercepted**.

Example:

```java
@Pointcut("execution(* com.example.service..*(..))")
public void serviceMethods() {}
```

### Understanding the expression

| Expression | Meaning |
|------------|---------|
| `execution` | Intercept method execution |
| `*` | Any return type |
| `com.example.service..` | Any class inside the `service` package and its sub-packages |
| `*` | Any method name |
| `(..)` | Any number of parameters |

This Pointcut matches every method inside the `service` package.

---

## 4. Advice

An **Advice** is the code that executes when a Pointcut matches.

Spring provides **five types of Advice**.

---

### 4.1 @Before

Runs **before** the target method executes.

```java
@Before("serviceMethods()")
public void logStart() {
    System.out.println("Method Started");
}
```

Execution Flow

```
Caller
   |
@Before
   |
Target Method
```

Use Cases

- Logging
- Authentication
- Input Validation

---

### 4.2 @After

Runs after the target method completes, whether it succeeds or throws an exception.

```java
@After("serviceMethods()")
public void cleanup() {
    System.out.println("Completed");
}
```

Execution Flow

```
Target Method
      |
   @After
```

It behaves similarly to a **finally block**.

---

### 4.3 @AfterReturning

Runs **only when the target method completes successfully**.

```java
@AfterReturning("serviceMethods()")
public void success() {
    System.out.println("Method executed successfully");
}
```

Execution Flow

```
Target Method
      |
Returns Successfully
      |
@AfterReturning
```

Use Cases

- Response Logging
- Auditing
- Cache Updates

---

### 4.4 @AfterThrowing

Runs **only if the target method throws an exception**.

```java
@AfterThrowing("serviceMethods()")
public void error() {
    System.out.println("Exception occurred");
}
```

Execution Flow

```
Target Method
      |
 Exception
      |
@AfterThrowing
```

Use Cases

- Error Logging
- Alerts
- Monitoring

---

### 4.5 @Around

The most powerful Advice.

It executes **before** and **after** the target method.

```java
@Around("serviceMethods()")
public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {

    long start = System.currentTimeMillis();

    Object result = pjp.proceed();

    long end = System.currentTimeMillis();

    System.out.println("Execution Time : " + (end - start));

    return result;
}
```

### What is `ProceedingJoinPoint`?

`ProceedingJoinPoint` represents the target method being intercepted.

The statement

```java
pjp.proceed();
```

actually invokes the target method.

If you don't call `proceed()`, the target method **never executes**.

Execution Flow

```
Caller
   |
@Around (Before)
   |
Target Method
   |
@Around (After)
   |
Caller
```

Capabilities of `@Around`

- Execute code before the method
- Execute code after the method
- Modify method arguments
- Modify return value
- Retry failed operations
- Skip method execution completely

---

## 5. Target

The **Target** is the actual business object whose methods are intercepted.

Example

```java
@Service
public class PaymentService {

    public void charge() {
        // Business Logic
    }

}
```

Here, `PaymentService` is the Target object.

---

## 6. Proxy

Spring never exposes the original Target object directly.

Instead, it creates a **Proxy**.

```
Caller
   |
 Proxy
   |
Target
```

Whenever another bean calls

```java
paymentService.charge();
```

the request first reaches the Proxy.

The Proxy executes all matching Advices and then invokes the target method.

### Types of Proxies

#### JDK Dynamic Proxy

Used when the target class implements an interface.

```
Caller
   |
JDK Proxy
   |
Interface
   |
Target
```

#### CGLIB Proxy

Used when the target class does not implement an interface.

CGLIB creates a subclass of the target class.

```
PaymentServiceProxy
        extends
PaymentService
```

---

## 7. Weaving

**Weaving** is the process of connecting Aspects with Target objects.

Spring performs **Runtime Weaving** using Proxies.

AspectJ supports

- Compile-Time Weaving
- Load-Time Weaving
- Runtime Weaving

---

# Complete Example

## Aspect

```java
@Aspect
@Component
public class TimingAspect {

    @Around("execution(* com.example.service..*(..))")
    public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {

        long start = System.currentTimeMillis();

        Object result = pjp.proceed();

        long end = System.currentTimeMillis();

        System.out.println("Execution Time : " + (end - start));

        return result;
    }

}
```

## Business Class

```java
@Service
public class PaymentService {

    public void charge() {
        System.out.println("Charging customer...");
    }

}
```

### Execution Flow

```
Caller
   |
Proxy
   |
@Around (Before)
   |
Target Method
   |
@Around (After)
   |
Caller
```

Output

```
Charging customer...
Execution Time : 15 ms
```

---

# How Spring AOP Works Internally

When the Spring Application starts:

### Step 1

Spring scans all beans.

```
@Component
@Service
@Repository
@Controller
@Aspect
```

---

### Step 2

Spring creates the `PaymentService` bean.

---

### Step 3

Spring checks whether any Pointcut matches `PaymentService`.

For example,

```java
@Pointcut("execution(* com.example.service..*(..))")
```

Since `PaymentService` is inside the `service` package, it matches.

---

### Step 4

Instead of exposing the original bean,

Spring creates a **Proxy**.

Depending on the class,

- JDK Dynamic Proxy (if an interface exists)
- CGLIB Proxy (if no interface exists)

---

### Step 5

Other beans receive the Proxy instead of the original object.

```
@Autowired
private PaymentService paymentService;
```

Actually, `paymentService` is a Proxy object.

---

### Step 6

When

```java
paymentService.charge();
```

is called,

Execution Flow

```
Caller
   |
Proxy
   |
@Before
   |
@Around (Before)
   |
Target Method
   |
@Around (After)
   |
@AfterReturning / @AfterThrowing
   |
@After
   |
Caller
```

---

# Self Invocation Problem

One of the most common Spring AOP interview questions.

Example

```java
@Service
public class PaymentService {

    public void processPayment() {
        validatePayment();
    }

    public void validatePayment() {
        System.out.println("Validating Payment");
    }

}
```

Suppose `validatePayment()` has a `@Before` Advice.

Now,

```java
paymentService.processPayment();
```

calls

```java
this.validatePayment();
```

internally.

Execution Flow

```
Caller
   |
Proxy
   |
processPayment()
   |
this.validatePayment()
   |
Target Object
```

Notice that `validatePayment()` is called using `this`, **not through the Proxy**.

Therefore,

- No interception occurs.
- No Advice executes.

### Why?

Spring AOP is **Proxy-Based**.

Only method calls that pass through the Proxy are intercepted.

Internal method calls (`this.method()`) bypass the Proxy.

---

# How to Solve Self Invocation?

### Solution 1 (Recommended)

Move the method to another Spring Bean.

```
PaymentService
      |
NotificationService
      |
Proxy
      |
Advice
```

---

### Solution 2

Inject the Proxy of the same bean.

```java
@Autowired
private PaymentService self;
```

Instead of

```java
this.validatePayment();
```

use

```java
self.validatePayment();
```

Now the call goes through the Proxy.

---

### Solution 3

Use AspectJ.

AspectJ performs **bytecode weaving**, so even internal method calls are intercepted.

---

# Spring AOP vs AspectJ

| Spring AOP | AspectJ |
|------------|----------|
| Proxy-Based | Bytecode Weaving |
| Runtime Weaving | Compile-Time, Load-Time, Runtime |
| Supports only Method Execution | Supports all Join Points |
| Easy to configure | More powerful |
| Self Invocation not supported | Self Invocation supported |

---

# Interview Tips

- AOP separates cross-cutting concerns from business logic.
- Spring AOP is **Proxy-Based**.
- Spring supports only **Method Execution** Join Points.
- A **Pointcut** decides which methods should be intercepted.
- An **Advice** defines what should execute.
- `ProceedingJoinPoint.proceed()` invokes the target method.
- Spring uses **JDK Dynamic Proxy** if an interface exists.
- Spring uses **CGLIB Proxy** if no interface exists.
- Internal method calls (`this.method()`) bypass the Proxy, so Advice does not execute.
- `@Around` is the most powerful Advice because it can execute code before and after the method, modify arguments, modify return values, retry execution, or even skip method execution.


