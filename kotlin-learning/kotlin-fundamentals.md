# Kotlin Fundamentals for Java Developers: Cheatsheet & Reference Guide

This reference guide summarizes the essential Kotlin language features, type safety mechanisms, and idiomatic syntax patterns covered prior to transitioning into Spring Boot development.

---

## 1. Variables & Immutability (`val` vs `var`)

Kotlin enforces immutability by default and relies on compile-time type inference.

* **`val` (Value):** Immutable/read-only reference. Identical to Java's `final`. It cannot be reassigned once initialized.
* **`var` (Variable):** Mutable reference. Can be reassigned at runtime.
* **Type Inference:** Explicit type declarations can be omitted when the compiler can deduce the type.

### Java vs. Kotlin Comparison

#### Java
```java
final String company = "Anaplan";
String role = "Backend Engineer";
role = "Senior Engineer";
// company = "Other"; // Compilation error: cannot assign a value to final variable
```

#### Kotlin
```kotlin
val company = "Anaplan"               // Inferred as String, immutable (final)
var role: String = "Backend Engineer" // Explicit type, mutable
role = "Senior Engineer"              // Allowed

// company = "Other"                  // Compilation error: Val cannot be reassigned
```

---

## 2. Compile-Time Null Safety & Operators

Kotlin distinguishes between nullable and non-nullable types directly in the type system, preventing runtime `NullPointerException` (NPE) errors.

### The Type Hierarchy
* **`T` (Non-nullable):** Cannot hold `null`. Guaranteed safe for method calls.
* **`T?` (Nullable):** Can hold either a valid instance of `T` or `null`.

### Core Null Safety Operators

| Operator | Name | Behavior | Java Equivalent |
| :--- | :--- | :--- | :--- |
| `?` | **Nullable Type Declaration** | Marks a variable as capable of holding `null`. | Default Java reference behavior / `@Nullable` |
| `?.` | **Safe Call Operator** | Executes the call only if the receiver is not `null`; evaluates to `null` otherwise. | `if (obj != null) { obj.method(); }` |
| `?:` | **Elvis Operator** | Returns the expression on the left if non-null; otherwise evaluates and returns the right side. | `Optional.ofNullable(a).orElse(b)` |
| `!!` | **Not-Null Assertion** | Asserts that a value is non-null. Throws a runtime NPE if the value is `null`. | Direct unchecked method call in Java (discouraged). |

### Example Implementation

#### Java
```java
public String getCity(User user) {
    if (user != null && user.getAddress() != null && user.getAddress().getCity() != null) {
        return user.getAddress().getCity();
    }
    return "Unknown";
}
```

#### Kotlin
```kotlin
fun getCity(user: User?): String {
    return user?.address?.city ?: "Unknown"
}
```

#### Combined Practice Example (`getDisplayName`)
```kotlin
fun getDisplayName(firstName: String? = null, lastName: String? = null): String =
    if (firstName == null && lastName == null) {
        "Anonymous"
    } else {
        "${firstName ?: ""} ${lastName ?: ""}".trim()
    }
```

---

## 3. String Literals, Templates & Formatting

Kotlin provides native support for single-line strings, multiline raw strings, and variable interpolation.

### A. Escaped Strings (Single-line)
Uses standard double quotes (`"..."`). Standard escape sequences such as `\n` and `\t` are evaluated.

```kotlin
val singleLine = "Line 1\nLine 2"
```

### B. Raw Strings (Multiline)
Uses triple double quotes (`"""..."""`). Preserves line breaks and formatting without requiring escape characters. Use `.trimIndent()` or `.trimMargin()` to remove unwanted indentation.

```kotlin
val rawJson = """
    {
        "status": "UP",
        "service": "core-engine"
    }
""".trimIndent()
```

### C. String Templates
Embed variables using `$variableName` or evaluate complex logic using `${expression}`.

```kotlin
val username = "Alex"
val items = listOf("Task 1", "Task 2")

// Simple interpolation
println("Hello, $username")

// Expression interpolation
println("Remaining items: ${items.size}")
println("Status: ${if (items.isEmpty()) "Done" else "Active"}")
```

### D. `String.format()` (Printf-style Formatting)
Used for strict decimal rounding, padding, or locale-specific numeric formatting:

```kotlin
val price = 14.5
val product = "Subscription"
val formatted = "Product: %s, Price: $%.2f".format(product, price)
```

### E. Programmatic String Construction (`buildString`)
Replaces manual `StringBuilder` creation. Under the hood, it allocates an unsynchronized `StringBuilder`, runs the block as an in-place DSL (Lambda with Receiver), and calls `.toString()`.

```kotlin
val report = buildString {
    append("Tenant: Anaplan")
    appendLine()
    append("Active Nodes: 12")
}
```

---

## 4. Data Classes vs. Java POJOs / Records

The `data class` modifier automatically provides standard structural methods based on primary constructor properties.

### Auto-Generated Methods
* `equals()` and `hashCode()` based on structural equality.
* `toString()` formatted as `ClassName(prop1=val1, prop2=val2)`.
* `copy()` for non-destructive, immutable updates.
* `componentN()` functions for destructuring declarations.

### Example Implementation

#### Java POJO (Without Lombok)
```java
public final class UserDto {
    private final Long id;
    private final String email;

    public UserDto(Long id, String email) {
        this.id = id;
        this.email = email;
    }

    public Long getId() { return id; }
    public String getEmail() { return email; }
    // plus equals(), hashCode(), toString()
}
```

#### Kotlin
```kotlin
data class UserDto(
    val id: Long,
    val email: String
)

// Usage & Immutability via copy():
val user1 = UserDto(1L, "user1@company.com")
val user2 = user1.copy(email = "user2@company.com") // id remains 1L

// Destructuring:
val (userId, userEmail) = user2
```

---

## 5. Default Arguments & Named Arguments

Kotlin eliminates the need for constructor overloading and the traditional Builder pattern.

```kotlin
data class ServerConfig(
    val host: String,
    val port: Int = 8080,           // Default argument
    val useTls: Boolean = true      // Default argument
)

// Standard instantiation using defaults:
val defaultConfig = ServerConfig(host = "localhost")

// Explicit named arguments (order independent):
val customConfig = ServerConfig(
    useTls = false,
    host = "10.0.0.1"
    // port falls back to 8080
)
```

---

## 6. Single-Expression Functions

When a function consists of a single expression or delegates directly to an underlying call, the body braces `{ ... }` and `return` keyword can be replaced with `=`. The return type can often be inferred automatically.

#### Java
```java
public int calculateTotal(int base, int tax) {
    return base + tax;
}
```

#### Kotlin
```kotlin
// Full syntax:
fun calculateTotal(base: Int, tax: Int): Int = base + tax

// With type inference:
fun calculateTotal(base: Int, tax: Int) = base + tax

// With conditional logic:
fun getRoleTier(role: String) = when (role) {
    "ADMIN" -> 1
    "MODERATOR" -> 2
    else -> 3
}
```

---

## 7. Extension Functions

Extension functions allow adding new functionality to third-party or standard library classes without inheriting or using utility classes (e.g., `StringUtils`).

```kotlin
// Defining an extension function on String
fun String.isValidDomain(): Boolean {
    return this.endsWith(".com") || this.endsWith(".io")
}

// Usage:
val domain = "service.io"
val isValid = domain.isValidDomain() // Evaluates to true
```

Inside the function body, `this` refers to the **receiver object** on which the function was invoked.

---

## 8. Smart Casts and the `when` Expression

### Smart Casts
Once a type check is executed using `is` (equivalent to `instanceof`), the compiler automatically casts the reference within that scope. Explicit casting (`(Type) obj`) is unnecessary.

```kotlin
fun evaluatePayload(obj: Any) {
    if (obj is String) {
        // obj is automatically cast to String
        println(obj.uppercase())
    }
}
```

### The `when` Expression
A complete replacement for Java's `switch` that can act as an expression returning a value.

```kotlin
fun parseStatusCode(status: String): Int = when (status) {
    "OK" -> 200
    "ACCEPTED", "PROCESSING" -> 202
    "UNAUTHORIZED" -> 401
    else -> 500
}
```

---

## 9. Scope Functions (`let` and `apply`)

### A. `let` (Null-Checks & Transformations)
Executes a block only when non-null (when combined with `?.`). Replaces `Optional.map()`.

```kotlin
user?.email?.let { email ->
    sendEmail(email)
}
```

### B. `apply` (Object Configuration)
Executes a block with the object as `this` and returns the object itself.

```kotlin
val restTemplate = RestTemplate().apply {
    errorHandler = CustomErrorHandler()
}
```

---

## 10. Lambdas, Receivers & Memory Mechanics

### A. Standard Lambda (`(T) -> Unit`)
A standard lambda receives an instance as an argument. By default, Kotlin provides the implicit identifier **`it`** if a single parameter is not explicitly named.

```kotlin
fun configureUser(action: (User) -> Unit): User {
    val user = User()
    action(user)
    return user
}

// Using implicit 'it':
val user1 = configureUser {
    it.setName("Alex")
    it.setAge(30)
}

// Using explicit parameter name:
val user2 = configureUser { customUser ->
    customUser.setName("Alex")
    customUser.setAge(30)
}
```

### B. Lambda with Receiver (`T.() -> Unit`)
A lambda with receiver shifts the context so that the target instance becomes **`this`** inside the block. Methods and properties of `T` can be accessed directly without prefixing with `it.`.

```kotlin
fun configureUserWithReceiver(action: User.() -> Unit): User {
    val user = User()
    user.action() // Executes block with user as 'this'
    return user
}

// Usage: Direct member access inside the block
val user = configureUserWithReceiver {
    setName("Alex")
    setAge(30)
}
```

### C. JVM Memory & `inline` Functions
