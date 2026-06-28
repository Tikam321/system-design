# Java Optional - Part 1 (`of()` & `ofNullable()`)

## Why was `Optional` introduced?

Before Java 8, developers had to write multiple null checks:

```java
if (user != null) {
    if (user.getEmail() != null) {
        sendEmail(user.getEmail());
    }
}
```

Problems:

* ❌ Too many null checks
* ❌ Code becomes difficult to read
* ❌ Easy to get `NullPointerException`
* ❌ Business logic gets hidden behind boilerplate

`Optional` was introduced to explicitly represent **"a value may or may not be present."**

---

# Mental Model

Think of `Optional` as a **container (box)**.

```
Optional<String>

+------------------+
|     "Java"       |
+------------------+

OR

+------------------+
|      Empty       |
+------------------+
```

Instead of returning `null`, return an **empty Optional**.

---

# `Optional.of()`

## Syntax

```java
Optional<String> optional = Optional.of(value);
```

## Purpose

Use when you are **100% sure** the value is **NOT null**.

## Example

```java
String name = "Tikam";

Optional<String> optional = Optional.of(name);

System.out.println(optional);
```

**Output**

```text
Optional[Tikam]
```

## If value is null

```java
String name = null;

Optional.of(name);
```

**Result**

```text
NullPointerException
```

## Production Rule

✅ Use only when the value is guaranteed to be non-null.

Examples:

* Hardcoded values
* Constants
* Objects already validated

---

# `Optional.ofNullable()`

## Syntax

```java
Optional<String> optional = Optional.ofNullable(value);
```

## Purpose

Use when the value **may be null**.

## Example (Value Exists)

```java
String name = "Tikam";

Optional<String> optional = Optional.ofNullable(name);
```

**Output**

```text
Optional[Tikam]
```

---

## Example (Value is Null)

```java
String name = null;

Optional<String> optional = Optional.ofNullable(name);
```

**Output**

```text
Optional.empty
```

No exception is thrown.

## Production Rule

✅ Use when data comes from:

* Database
* REST API
* User Input
* External Service
* File
* Kafka
* Redis

---

# `of()` vs `ofNullable()`

| Feature          | `Optional.of()`               | `Optional.ofNullable()`    |
| ---------------- | ----------------------------- | -------------------------- |
| Accepts null     | ❌ No                          | ✅ Yes                      |
| If value is null | Throws `NullPointerException` | Returns `Optional.empty()` |
| Production Usage | Rare                          | Very Common                |

---

# Best Practice

✅ Use `Optional.of()`

* When you are **100% certain** the value cannot be null.

✅ Use `Optional.ofNullable()`

* Whenever there is **any possibility** of a null value.

---

# Interview Tip

**Q: What is the difference between `Optional.of()` and `Optional.ofNullable()`?**

**Answer:**

* `Optional.of()` expects a non-null value and throws `NullPointerException` if the value is null.
* `Optional.ofNullable()` safely handles nullable values by returning `Optional.empty()` instead of throwing an exception.

---

# Quick Summary

```java
Optional.of("Java");
// -> Optional[Java]

Optional.of(null);
// -> NullPointerException

Optional.ofNullable("Java");
// -> Optional[Java]

Optional.ofNullable(null);
// -> Optional.empty()
```

---

# Key Takeaway

> **`Optional.of()` = "I guarantee the value is NOT null."**

> **`Optional.ofNullable()` = "The value may or may not be null."**
