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
   
