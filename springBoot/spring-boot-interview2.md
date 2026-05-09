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
