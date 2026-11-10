# Kotlin Collections, Streams & Spring Boot Setup: Reference Guide

This guide covers the Spring Boot integration setup for Kotlin, the Kotlin collection hierarchy, functional collection operations compared to Java Streams, and the performance differences between eager collections and lazy `Sequence` pipelines.

---

## Quick Summary

* **Spring Boot Compiler Plugins:** Kotlin classes are `final` by default. Use `kotlin("plugin.spring")` (all-open) for Spring CGLIB proxies, `kotlin("plugin.jpa")` (no-arg) for Hibernate entity instantiation, `jackson-module-kotlin` for JSON deserialization, and `-Xjsr305=strict` for strict framework null-safety.
* **Collection Hierarchy:** Kotlin separates collections at compile time into **read-only** (`List`, `Set`, `Map`) and **mutable** (`MutableList`, `MutableSet`, `MutableMap`) interfaces, backed by standard `java.util` classes at runtime.
* **Functional Operations:** Methods like `.filter()`, `.map()`, `.associate()`, and `.groupBy()` are extension functions directly on collections—no `.stream()` or `.collect()` boilerplate is needed.
* **Eager vs. Lazy (`Sequence`):** Standard Kotlin collection chains evaluate **eagerly** (creating intermediate lists per step). Use `.asSequence()` for **lazy**, single-pass evaluation on large datasets (10,000+ items) or multi-step pipelines.

---

## 1. Spring Boot & Kotlin Integration Setup

Because Kotlin has stricter defaults than Java (classes are `final`, no default zero-arg constructors when primary constructors have arguments), specific plugins and dependencies are required in `build.gradle.kts`.

### A. Gradle Configuration (`build.gradle.kts`)

```kotlin
plugins {
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    
    // Core Kotlin JVM plugin
    kotlin("jvm") version "1.9.20"
    
    // 1. Opens classes/methods for Spring CGLIB proxies (@Service, @Configuration, @Transactional)
    kotlin("plugin.spring") version "1.9.20"
    
    // 2. Generates synthetic zero-arg constructors for Hibernate (@Entity, @MappedSuperclass, @Embeddable)
    kotlin("plugin.jpa") version "1.9.20"
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    
    // Required for Kotlin reflection and bean instantiation
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    
    // Required for deserializing JSON into Kotlin data classes & enforcing non-null types
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

kotlin {
    compilerOptions {
        // Enforces strict null-safety when interacting with Spring's @Nullable/@NonNull annotations
        freeCompilerArgs.addAll("-Xjsr305=strict")
    }
}
```

### B. Application Entry Point

In Kotlin, `main` is a top-level function outside of the application class. The `*` spread operator unpacks the `Array<String>` into `vararg` arguments.

```kotlin
@SpringBootApplication
class Application

fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

---

## 2. Kotlin Collection Hierarchy (Read-Only vs. Mutable)

Unlike Java—where `List.of()` or `Collections.unmodifiableList()` still expose `.add()` and `.remove()` methods that fail at runtime—Kotlin enforces mutability rules at **compile time**.

### Interface & Factory Mapping

| Collection Type | Read-Only Interface (Default) | Creation Function | Mutable Interface | Creation Function |
| :--- | :--- | :--- | :--- | :--- |
| **List** | `List<T>` | `listOf(...)` | `MutableList<T>` | `mutableListOf(...)` |
| **Set** | `Set<T>` | `setOf(...)` | `MutableSet<T>` | `mutableSetOf(...)` |
| **Map** | `Map<K, V>` | `mapOf(k to v)` | `MutableMap<K, V>` | `mutableMapOf(k to v)` |

### Under the Hood (JVM Runtime)
Kotlin does not allocate custom collection data structures. At runtime on the JVM, `listOf()`, `setOf()`, and `mapOf()` instantiate standard `java.util.ArrayList`, `java.util.HashSet`, and `java.util.LinkedHashMap` objects. The read-only interfaces simply hide mutating methods from the compiler.

### Code Example

```kotlin
// 1. Read-Only List (Default & Recommended)
val roles: List<String> = listOf("ADMIN", "USER")
// roles.add("GUEST") // Compile-time error: Unresolved reference: add

// 2. Mutable List
val activeUsers: MutableList<String> = mutableListOf("Alex", "Sam")
activeUsers.add("Taylor") // Allowed

// 3. Map with 'to' infix function and bracket indexing
val statusCodes: Map<Int, String> = mapOf(
    200 to "OK",
    404 to "NOT_FOUND",
    500 to "SERVER_ERROR"
)
val okMessage = statusCodes[200] // Access via [] instead of .get()
```

---

## 3. Functional Operations vs. Java Streams

In Kotlin, functional operations are built directly onto `Iterable<T>` via inline extension functions. You do not need `.stream()` to start a pipeline or `.collect(...)` to terminate it.

### Side-by-Side API Comparison

| Operation | Java Stream API | Kotlin Collection API |
| :--- | :--- | :--- |
| **Filter** | `list.stream().filter(x -> x.isActive()).toList()` | `list.filter { it.isActive }` |
| **Map / Transform** | `list.stream().map(User::getName).toList()` | `list.map { it.name }` |
| **Map & Drop Nulls** | `list.stream().map(...).filter(Objects::nonNull).toList()` | `list.mapNotNull { it.email }` |
| **FlatMap** | `list.stream().flatMap(x -> x.getTags().stream()).toList()` | `list.flatMap { it.tags }` |
| **Find First Match** | `list.stream().filter(...).findFirst().orElse(null)` | `list.firstOrNull { ... }` |
| **Group By Key** | `list.stream().collect(Collectors.groupingBy(User::getDept))` | `list.groupBy { it.dept }` |
| **List to Map (By Key)** | `list.stream().collect(Collectors.toMap(User::getId, u -> u))` | `list.associateBy { it.id }` |
| **List to Map (Key/Val)** | `list.stream().collect(Collectors.toMap(O::getId, O::getAmt))` | `list.associate { it.id to it.amount }` |
| **Sort Ascending** | `list.stream().sorted(Comparator.comparing(User::getAge)).toList()` | `list.sortedBy { it.age }` |
| **Sum Numeric Field** | `list.stream().mapToDouble(Order::getAmount).sum()` | `list.sumOf { it.amount }` |

### Practical Example: Transforming & Filtering Lists into Maps

#### Java
```java
public Map<Long, Double> getCompletedOrderAmounts(List<Order> orders) {
    return orders.stream()
            .filter(o -> "COMPLETED".equals(o.getStatus()))
            .collect(Collectors.toMap(Order::getId, Order::getAmount));
}
```

#### Kotlin
```kotlin
data class Order(val id: Long, val status: String, val amount: Double)

// Idiomatic Kotlin using null-safe '==' and '.associate { key to value }'
fun getCompletedOrderAmounts(orders: List<Order>): Map<Long, Double> =
    orders
        .filter { it.status == "COMPLETED" }
        .associate { it.id to it.amount }
```

---

## 4. Eager Collections vs. Lazy `Sequence`

Understanding evaluation order is critical for JVM memory and performance when migrating from Java `Stream`.

### A. Eager Evaluation (Standard `List` / `Iterable`)
* **How it works:** Every chained function (`filter`, `map`) executes **immediately** across the entire collection and allocates a **new intermediate `ArrayList`** on the heap at each step.
* **Why it is the default:** Because Kotlin's collection functions are `inline`, there is zero anonymous lambda object allocation. For small-to-medium datasets (typical REST API payloads of 10–1,000 items), eager evaluation is fast and cache-friendly.

### B. Lazy Evaluation (`Sequence<T>`)
* **How it works:** Calling `.asSequence()` mirrors Java's `.stream()`. Operations are evaluated **lazily, element-by-element** in a single pass. No intermediate lists are allocated, and execution only starts when a terminal operation (`.toList()`, `.firstOrNull()`, `.count()`) is called.
* **Short-Circuiting:** Operations like `.take(5)` or `.firstOrNull()` stop processing immediately once the condition is met.

### Code Comparison

```kotlin
// Eager: Creates an intermediate list after filter, and another after map
val eagerEmails = employees
    .filter { it.department == "ENG" }
    .mapNotNull { it.email }

// Lazy (Java Stream equivalent): Single-pass, zero intermediate lists, short-circuits at 5 items
val lazyTopEmails = employees
    .asSequence()                         // Equivalent to .stream()
    .filter { it.department == "ENG" }
    .filter { it.salary > 100_000 }
    .mapNotNull { it.email }
    .take(5)                              // Stops evaluating remaining items once 5 are found
    .toList()                             // Terminal operation (equivalent to .collect())
```

### Decision Matrix for Backend Development

| Scenario | Recommended Approach | Reason |
| :--- | :--- | :--- |
| **Small collections (< 1,000 items) & 1–2 steps** | Standard Eager (`list.filter { ... }.map { ... }`) | Avoids `Sequence` wrapper overhead; `inline` functions eliminate lambda allocation. |
| **Large collections (10,000+ items)** | Lazy Sequence (`list.asSequence()...`) | Prevents allocating massive intermediate lists on the JVM heap. |
| **Long chains (3+ transformations) or `.take(N)`** | Lazy Sequence (`list.asSequence()...`) | Processes item-by-item and short-circuits early without wasted work. |
| **CPU-heavy parallel processing** | Java Parallel Stream (`list.parallelStream()`) | Kotlin `Sequence` is single-threaded; standard `java.util.List` methods remain accessible. |
