# Kotlin Learning 1 — Kotlin Basics vs Java

## Why Kotlin
- Compiles to JVM bytecode — same runtime as Java (threads, GC, classloaders, memory model identical)
- Null safety, concision, first-class functions, coroutines
- Official Android language (2017), first-class Spring Boot support

## Hello World: Java vs Kotlin

Java:
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, Kotlin!");
    }
}
```

Kotlin:
```kotlin
fun main() {
    println("Hello, Kotlin!")
}
```

## Key Differences (Quick Map)

| Java | Kotlin |
|---|---|
| `String name = null` (NPE risk) | `String?` must be declared nullable |
| `final String x = "a"` | `val x = "a"` |
| mutable variable | `var y = 1` |
| `static` keyword | top-level functions / `object` / `companion object` |
| getters/setters | properties (`val`/`var` with `get()`/`set()`) |
| `instanceof` + cast | `is` + smart cast |
| `switch` | `when` expression (exhaustive) |
| POJO + Lombok | `data class` |
| checked exceptions | none (any exception can be thrown) |
| builder pattern everywhere | default + named arguments |

## Notes

(Notes from today's session go here)
