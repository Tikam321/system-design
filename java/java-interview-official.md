# Java Interview Questions & Answers

A complete guide to core Java interview questions — covering fundamentals, OOP, collections, exception handling, memory management, Java 8+ features, and more.

> **Note:** Multithreading & concurrency questions are covered separately in `multithreading-interview-questions.md`.

---

## Table of Contents
1. [Java Basics](#java-basics)
2. [OOP Concepts](#oop-concepts)
3. [Data Types, Variables & Operators](#data-types-variables--operators)
4. [Strings](#strings)
5. [Exception Handling](#exception-handling)
6. [Collections Framework](#collections-framework)
7. [Generics](#generics)
8. [Memory Management & Garbage Collection](#memory-management--garbage-collection)
9. [Java 8+ Features](#java-8-features)
10. [Object Class Methods & equals/hashCode](#object-class-methods--equalshashcode)
11. [Serialization](#serialization)
12. [I/O & NIO](#io--nio)
13. [JVM Internals](#jvm-internals)
14. [Design Patterns in Java](#design-patterns-in-java)
15. [Common Coding Questions](#common-coding-questions)
16. [Miscellaneous / Advanced](#miscellaneous--advanced)

---

## Java Basics

### 1. What are the main features of Java?
Platform independence (write once, run anywhere via JVM bytecode), object-oriented, robust (strong memory management, exception handling), secure (no explicit pointers, bytecode verification), multithreaded, high performance (JIT compilation), and automatic garbage collection.

### 2. What is JVM, JRE, and JDK?
- **JVM (Java Virtual Machine)**: Executes Java bytecode; provides platform independence by abstracting the underlying OS/hardware.
- **JRE (Java Runtime Environment)**: JVM + core libraries needed to *run* Java applications.
- **JDK (Java Development Kit)**: JRE + development tools (compiler `javac`, debugger, etc.) needed to *develop* Java applications.

### 3. Why is Java called "platform independent"?
Java source code is compiled into an intermediate form called **bytecode** (`.class` files), which any JVM (on any OS/hardware) can interpret/execute — so the same compiled code runs unmodified across platforms, as long as a compatible JVM is available.

### 4. What is the difference between JIT and interpreter?
The JVM initially interprets bytecode line-by-line (slower). The **Just-In-Time (JIT) compiler** identifies frequently executed ("hot") code paths and compiles them directly into native machine code at runtime, significantly boosting performance for repeated execution.

### 5. What is the difference between `==` and `.equals()`?
- `==` compares references (memory addresses) for objects, or actual values for primitives.
- `.equals()` compares logical/content equality, and its behavior depends on whether the class has overridden it (default `Object.equals()` also just compares references).

```java
String a = new String("hello");
String b = new String("hello");
a == b;        // false (different objects)
a.equals(b);   // true (same content)
```
## Q: What is the difference between `==` and `equals()`?

**Answer:**

- `==`
  - Primitives → compares values.
  - Objects → compares references (memory).

- `equals()`
  - Default (`Object`) → compares references.
  - Overridden (`String`, Wrapper, Record, Custom Class) → compares values.

---

## Q: Does `equals()` always compare values?

**Answer:**

No. `Object.equals()` compares references. Value comparison happens only if the class overrides it.

---

## Q: Why do we override `equals()`?

**Answer:**

To compare objects based on their fields instead of memory addresses.

---

## Q: What is the default implementation of `Object.equals()`?

**Answer:**

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

---

## Q: How does `String.equals()` work?

**Answer:**

1. Check `this == obj`
2. Check object type
3. Compare length
4. Compare characters

---

## Q: Why check `this == obj` first?

**Answer:**

It's a fast path. If both references are the same, they're already equal.

---

## Q: What is the standard `equals()` implementation?

**Answer:**

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;

    Employee other = (Employee) obj;
    return id == other.id &&
           Objects.equals(name, other.name);
}
```

---

## Q: One-line interview answer?

**Answer:**

> `==` compares references for objects and values for primitives. `equals()` compares references by default, but overridden implementations compare object values.
### 6. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?

| | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutability | Immutable | Mutable | Mutable |
| Thread-safe | N/A (immutable) | No | Yes (synchronized methods) |
| Performance | Slower for repeated modification | Fast | Slower than StringBuilder due to sync overhead |
| Use case | Fixed/rarely-changing text | Single-threaded string building | Multi-threaded string building |

### 7. Why is `String` immutable in Java?
- **Security**: Strings are used for things like class names, file paths, network connections — immutability prevents tampering.
- **String pool caching**: Since strings are immutable, the JVM can safely reuse/share string literals in the String Constant Pool, saving memory.
- **Thread safety**: Immutable objects are inherently thread-safe — no synchronization needed.
- **Hashcode caching**: Since content never changes, `hashCode()` can be computed once and cached, making `String` efficient as `HashMap` keys.

### 8. What is the String pool (String Constant Pool)?
A special memory region in the heap where the JVM stores unique string literals. When you create a string via a literal (`"hello"`), the JVM checks the pool first and reuses an existing object if the content matches, rather than creating a new one — saving memory. Strings created via `new String("hello")` bypass the pool (unless `.intern()` is called).

### 9. What is autoboxing and unboxing?
- **Autoboxing**: automatic conversion of a primitive type to its corresponding wrapper object (e.g., `int` → `Integer`).
- **Unboxing**: the reverse — converting a wrapper object back to its primitive type.

```java
Integer boxed = 10;   // autoboxing
int unboxed = boxed;  // unboxing
```

### 10. What is the Integer cache in Java?
Java caches `Integer` objects for values from **-128 to 127** (via `Integer.valueOf()`). Within this range, autoboxed integers reference the same cached objects, so `==` comparisons may unexpectedly return `true`; outside this range, new objects are created each time.

```java
Integer a = 100, b = 100;
a == b; // true (cached)
Integer c = 200, d = 200;
c == d; // false (not cached, different objects)
```

### 11. What is the difference between `final`, `finally`, and `finalize()`?
- **`final`**: keyword used to declare constants, prevent method overriding, or prevent class inheritance.
- **`finally`**: a block that always executes after a `try`/`catch`, used for cleanup (e.g., closing resources), regardless of whether an exception occurred.
- **`finalize()`**: a method (deprecated since Java 9) called by the garbage collector before reclaiming an object's memory — its execution timing is not guaranteed.

### 12. What is the difference between method overloading and method overriding?

| Overloading | Overriding |
|---|---|
| Same method name, different parameter list, same class | Same method signature, subclass redefines parent's method |
| Resolved at compile time (static polymorphism) | Resolved at runtime (dynamic polymorphism) |
| Return type can differ | Return type must be same/covariant |
| No inheritance required | Requires inheritance |

### 13. Can we overload `main()` method?
Yes — you can have multiple `main()` methods with different signatures, but the JVM will only invoke the standard `public static void main(String[] args)` as the program's entry point.

### 14. What is the difference between `static` and `instance` members?
- **Static members** belong to the class itself — shared across all instances, loaded once at class loading, accessed via the class name.
- **Instance members** belong to each individual object — a separate copy exists for each instance.

### 15. Can a `static` method access non-static members?
No, not directly — static methods belong to the class (not any particular instance), so they can't reference instance variables/methods without first creating an object.

---

## OOP Concepts

### 16. What are the four pillars of OOP?
1. **Encapsulation** — bundling data and methods together, restricting direct access via access modifiers (getters/setters).
2. **Abstraction** — hiding implementation details, exposing only essential features (via abstract classes/interfaces).
3. **Inheritance** — a class acquiring properties/behavior from a parent class, enabling code reuse.
4. **Polymorphism** — the ability of an object to take multiple forms (method overloading = compile-time, overriding = runtime).

### 17. What is the difference between an abstract class and an interface?

| Abstract Class | Interface |
|---|---|
| Can have both abstract and concrete methods | Traditionally only abstract methods; Java 8+ allows `default`/`static` methods |
| Can have constructors | Cannot have constructors |
| Can have instance variables of any access modifier | Fields are implicitly `public static final` (constants) |
| Supports single inheritance | A class can implement multiple interfaces |
| Use when classes share a common base with some shared implementation | Use to define a contract/capability across unrelated classes |

### 18. Why doesn't Java support multiple inheritance of classes?
To avoid the **Diamond Problem** — ambiguity when a class inherits conflicting method implementations from two parent classes. Java sidesteps this by allowing multiple *interface* implementation (contracts only, no state) while limiting class inheritance to a single parent.

### 19. How does Java handle the diamond problem with interfaces and default methods?
Since Java 8 allows `default` methods in interfaces, a class implementing two interfaces with conflicting default methods must explicitly override the method to resolve the ambiguity, or explicitly specify which interface's version to use (`InterfaceName.super.methodName()`).

### 20. What is polymorphism? Explain with an example.
The ability of a single interface/method to represent different underlying behaviors depending on the actual object type.

```java
class Animal { void sound() { System.out.println("Some sound"); } }
class Dog extends Animal { void sound() { System.out.println("Bark"); } }
class Cat extends Animal { void sound() { System.out.println("Meow"); } }

Animal a = new Dog();
a.sound(); // "Bark" — resolved at runtime (dynamic dispatch)
```

### 21. What is dynamic method dispatch?
The mechanism by which a call to an overridden method is resolved at **runtime** based on the actual object type (not the reference type) — the foundation of runtime polymorphism in Java.

### 22. What is the difference between composition and inheritance? When would you prefer one over the other?
- **Inheritance** ("is-a" relationship): a subclass extends a superclass, inheriting its behavior — tightly coupled.
- **Composition** ("has-a" relationship): a class contains a reference to another class and delegates behavior to it — more flexible, loosely coupled.

**Best practice**: "Favor composition over inheritance" — composition avoids fragile base-class problems and offers better flexibility to change implementations at runtime.

### 23. What are access modifiers in Java?
- `private` — accessible only within the same class.
- *(default/package-private)* — accessible within the same package.
- `protected` — accessible within the same package + subclasses (even in different packages).
- `public` — accessible from anywhere.

### 24. What is constructor overloading? Can constructors be inherited?
Constructor overloading means defining multiple constructors with different parameter lists in the same class. Constructors are **not inherited** by subclasses, but a subclass constructor can invoke a parent constructor using `super(...)`.

### 25. What is the difference between `this` and `super`?
- `this` refers to the current object instance — used to resolve ambiguity or call another constructor in the same class (`this(...)`).
- `super` refers to the immediate parent class — used to call the parent's constructor (`super(...)`) or access parent's overridden methods/fields.

### 26. Can we instantiate an abstract class?
No, abstract classes cannot be instantiated directly — they must be subclassed, and the subclass must implement all abstract methods (or itself be declared abstract).

### 27. What is a marker interface? Give examples.
An interface with no methods, used to "tag"/mark a class as having a certain property, which other code can check via `instanceof`. Examples: `Serializable`, `Cloneable`, `Remote`.

---

## Data Types, Variables & Operators

### 28. What are primitive data types in Java?
`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean` — these hold actual values directly (not references) and are stored on the stack (for local variables).

### 29. What is the difference between primitive types and wrapper classes?
Primitives store raw values directly and are more memory/performance efficient. Wrapper classes (`Integer`, `Double`, `Boolean`, etc.) wrap primitives as objects, enabling their use in collections (which require objects), providing utility methods, and supporting `null` values.

### 30. What is the default value of local variables vs instance variables?
Local variables have **no default value** — they must be explicitly initialized before use, or the compiler throws an error. Instance variables (fields) get default values automatically (`0`/`0.0` for numeric types, `false` for boolean, `null` for objects).

### 31. What is type casting? Explain implicit vs explicit casting.
- **Implicit (widening) casting**: automatic conversion from a smaller to a larger data type (e.g., `int` → `long`) — no data loss.
- **Explicit (narrowing) casting**: manual conversion from a larger to a smaller type (e.g., `double` → `int`), which may cause data/precision loss, requiring an explicit cast operator.

```java
int i = 100;
long l = i;          // implicit widening
double d = 100.04;
int x = (int) d;     // explicit narrowing, x = 100
```

### 32. What is the difference between `&` and `&&` (or `|` and `||`)?
`&`/`|` always evaluate both operands. `&&`/`||` use **short-circuit evaluation** — the second operand isn't evaluated if the result can already be determined from the first (e.g., `false && x` skips evaluating `x`).

---

## Strings

### 33. How does `String.intern()` work?
Returns a reference to the canonical (pooled) instance of the string. If the string pool already has an equal string, that reference is returned; otherwise the string is added to the pool and its reference returned.

### 34. Why should you use `StringBuilder` instead of `String` concatenation in a loop?
Since `String` is immutable, every concatenation (`+=`) creates a brand-new `String` object, discarding the old one — leading to significant memory churn and performance overhead in loops. `StringBuilder` mutates an internal buffer in place, avoiding repeated allocations.

### 35. How do you reverse a string in Java?
```java
String reversed = new StringBuilder("hello").reverse().toString();
```

### 36. How do you check if a string is a palindrome?
```java
static boolean isPalindrome(String s) {
    String clean = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    return clean.equals(new StringBuilder(clean).reverse().toString());
}
```

---

## Exception Handling

### 37. What is the difference between checked and unchecked exceptions?
- **Checked exceptions**: subclasses of `Exception` (excluding `RuntimeException`), verified at compile time — the method must either handle them (`try/catch`) or declare them (`throws`). E.g., `IOException`, `SQLException`.
- **Unchecked exceptions**: subclasses of `RuntimeException` — not checked at compile time, typically indicate programming bugs. E.g., `NullPointerException`, `ArrayIndexOutOfBoundsException`.

### 38. What is the exception hierarchy in Java?
```
Throwable
├── Error (serious JVM-level issues, e.g., OutOfMemoryError, StackOverflowError — not meant to be caught)
└── Exception
    ├── RuntimeException (unchecked, e.g., NullPointerException, ArithmeticException)
    └── Other checked exceptions (e.g., IOException, SQLException)
```

### 39. What is the difference between `throw` and `throws`?
- `throw` is used to explicitly throw an exception instance within code.
- `throws` is used in a method signature to declare that the method might throw certain checked exceptions, delegating handling to the caller.

### 40. Can a `finally` block prevent an exception from propagating?
Yes — if a `finally` block itself returns a value or throws a new exception, it can **suppress** the original exception/return value from the `try`/`catch` block. This is considered bad practice.

### 41. What happens if both `try` and `finally` blocks have `return` statements?
The `finally` block's `return` statement wins — it overrides any return value set in the `try` or `catch` block. This is a common trick question and best avoided in real code.

```java
static int test() {
    try {
        return 1;
    } finally {
        return 2; // this value is actually returned
    }
}
```

### 42. What is a custom exception? How do you create one?
A user-defined exception class extending `Exception` (checked) or `RuntimeException` (unchecked), used to represent domain-specific error conditions.

```java
class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

### 43. What is try-with-resources?
A Java 7+ feature that automatically closes resources (implementing `AutoCloseable`) at the end of the block, eliminating the need for manual `finally`-based cleanup.

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
}
```

### 44. What is exception chaining?
Wrapping a lower-level exception inside a higher-level one to preserve the original cause while providing more context, via the constructor's `cause` parameter or `initCause()`.

```java
try {
    // low-level operation
} catch (SQLException e) {
    throw new RuntimeException("Failed to save user", e);
}
```

---

## Collections Framework

### 45. What is the Java Collections Framework?
A unified architecture of interfaces (`List`, `Set`, `Map`, `Queue`) and implementations (`ArrayList`, `HashSet`, `HashMap`, etc.) for storing, manipulating, and retrieving groups of objects, along with algorithms (`Collections.sort()`, etc.).

### 46. Difference between `ArrayList` and `LinkedList`?

| `ArrayList` | `LinkedList` |
|---|---|
| Backed by a dynamic array | Backed by a doubly linked list |
| Fast random access — O(1) `get(index)` | Slow random access — O(n) traversal |
| Slow insertion/deletion in the middle — O(n) shifting | Fast insertion/deletion at known nodes — O(1) once positioned |
| Less memory overhead per element | More memory overhead (node pointers) |

### 47. Difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?

| | `HashMap` | `LinkedHashMap` | `TreeMap` |
|---|---|---|---|
| Ordering | No guaranteed order | Insertion order (or access order) | Sorted by natural order / comparator |
| Performance | O(1) average for get/put | Slightly slower than HashMap | O(log n) for get/put |
| Null keys | One `null` key allowed | One `null` key allowed | No `null` keys (throws NPE) |
| Backing structure | Array of buckets (linked list/tree per bucket) | HashMap + doubly linked list | Red-Black tree |

### 48. How does `HashMap` work internally?
- Keys are hashed via `hashCode()`, and the hash is used to determine a bucket index in an internal array.
- Multiple keys hashing to the same bucket (collisions) are stored as a linked list within that bucket (Java 8+ converts to a balanced tree/Red-Black tree if a bucket grows beyond a threshold of 8 entries, for better worst-case performance).
- `equals()` is used to differentiate keys within the same bucket.
- Resizing (rehashing) occurs when the load factor threshold (default 0.75) is exceeded, doubling the internal array's capacity.

### 49. Why are `hashCode()` and `equals()` important for custom objects used as `HashMap` keys?
`HashMap` relies on `hashCode()` to locate the correct bucket and `equals()` to identify the specific key within that bucket. If you override `equals()` without properly overriding `hashCode()` (or vice versa), objects considered "equal" may end up in different buckets, breaking map lookups.

### 50. What is the contract between `equals()` and `hashCode()`?
- If two objects are equal according to `equals()`, they **must** have the same `hashCode()`.
- Two objects with the same `hashCode()` are **not required** to be equal (hash collisions are allowed).
- `hashCode()` must consistently return the same value across multiple invocations, as long as no field used in `equals()`/`hashCode()` changes.

### 51. Difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?
Mirrors the Map equivalents:
- `HashSet` — no ordering guarantee, backed by a `HashMap` internally.
- `LinkedHashSet` — maintains insertion order.
- `TreeSet` — sorted order (natural or via `Comparator`), backed by a `TreeMap`.

### 52. What is the difference between `Comparable` and `Comparator`?

| `Comparable` | `Comparator` |
|---|---|
| `compareTo(T o)` defined inside the class itself | `compare(T o1, T o2)` defined in a separate class/lambda |
| Defines a class's single "natural ordering" | Allows multiple different sorting strategies |
| `java.lang` package | `java.util` package |

```java
class Employee implements Comparable<Employee> {
    int salary;
    public int compareTo(Employee other) { return this.salary - other.salary; }
}

// Comparator (external, flexible)
Comparator<Employee> byNameDesc = (e1, e2) -> e2.name.compareTo(e1.name);
```

### 53. What is the difference between `Iterator` and `ListIterator`?
- `Iterator` — traverses forward-only, works with any `Collection` (`List`, `Set`, `Queue`), supports `remove()`.
- `ListIterator` — extends `Iterator`, works only with `List`, supports bidirectional traversal (`hasPrevious()`/`previous()`), and supports `add()`/`set()` during iteration.

### 54. What is `ConcurrentModificationException`? How do you avoid it?
Thrown when a collection is structurally modified (add/remove) while being iterated using a regular `Iterator`, other than through the iterator's own `remove()` method. Avoid it by:
- Using `Iterator.remove()` instead of `Collection.remove()` during iteration.
- Using `CopyOnWriteArrayList` or `ConcurrentHashMap` for concurrent scenarios.
- Collecting items to remove into a separate list, then removing after iteration.

### 55. Difference between `Array` and `ArrayList`?

| Array | `ArrayList` |
|---|---|
| Fixed size | Dynamically resizable |
| Can hold primitives or objects | Can only hold objects (autoboxed primitives) |
| No built-in utility methods | Rich API (`add`, `remove`, `contains`, etc.) |
| Slightly better raw performance | Small overhead due to dynamic resizing/wrapper objects |

### 56. What is the difference between `Queue`, `Deque`, and `Stack`?
- **Queue**: FIFO (First-In-First-Out) ordering — `offer()`/`poll()`.
- **Deque** (Double-Ended Queue): supports insertion/removal at both ends — can act as both a queue and a stack (`ArrayDeque` is generally preferred over the legacy `Stack` class).
- **Stack**: LIFO (Last-In-First-Out) — legacy class, extends `Vector` (synchronized, considered somewhat obsolete in favor of `Deque`/`ArrayDeque`).

### 57. What is the fail-fast vs fail-safe iterator behavior?
- **Fail-fast** iterators (e.g., `ArrayList`, `HashMap`) throw `ConcurrentModificationException` if the underlying collection is structurally modified during iteration — detected via a `modCount` field.
- **Fail-safe** iterators (e.g., `CopyOnWriteArrayList`, `ConcurrentHashMap`) operate on a cloned/snapshot copy of the collection, so concurrent modifications don't throw exceptions (though the iterator may not reflect the latest changes).

---

## Generics

### 58. What are generics, and why use them?
Generics allow classes, interfaces, and methods to operate on types specified as parameters, providing compile-time type safety and eliminating the need for explicit casting.

```java
List<String> list = new ArrayList<>(); // only Strings allowed, compile-time checked
```

### 59. What is type erasure?
Java implements generics via **type erasure** — generic type information exists only at compile time for type-checking purposes; at runtime, all generic type parameters are erased and replaced with their bounds (or `Object` if unbounded). This is why you can't do `new T()` or check `instanceof List<String>` at runtime.

### 60. What is the difference between `<? extends T>` and `<? super T>` (bounded wildcards)?
- **`<? extends T>`** (upper bound): accepts `T` or any subtype — used when you only need to **read** from the structure (producer). E.g., `List<? extends Number>`.
- **`<? super T>`** (lower bound): accepts `T` or any supertype — used when you need to **write** into the structure (consumer). E.g., `List<? super Integer>`.
- Mnemonic: **PECS** — "Producer Extends, Consumer Super."

---

## Memory Management & Garbage Collection

### 61. What is the difference between heap and stack memory?
- **Stack**: stores method call frames, local variables, and references — memory is automatically reclaimed when a method returns (LIFO), thread-specific.
- **Heap**: stores all objects and their instance variables — shared across all threads, managed by the garbage collector.

### 62. What is garbage collection? How does it work in Java?
Automatic memory management that reclaims memory occupied by objects no longer reachable from any live reference, freeing developers from manual memory deallocation. Common algorithms include generational GC (dividing heap into Young/Old generations), mark-and-sweep, and mark-compact.

### 63. What are the generations in the JVM heap (Young, Old, Metaspace)?
- **Young Generation** (further split into Eden + Survivor spaces): where new objects are allocated; collected frequently via **Minor GC**.
- **Old (Tenured) Generation**: holds long-lived objects that survived multiple young-generation collections; collected less frequently via **Major/Full GC**.
- **Metaspace** (replaced PermGen since Java 8): stores class metadata, method info — grows dynamically, using native memory instead of the heap.

### 64. What causes a memory leak in Java despite having a garbage collector?
Even with GC, memory leaks occur when objects remain **reachable** (referenced) even though they're logically no longer needed — e.g., static collections that keep growing, unclosed resources, listeners/callbacks never deregistered, or improper caching without eviction policies.

### 65. What is a `WeakReference`? How does it differ from a strong reference?
A `WeakReference` allows the garbage collector to reclaim the referenced object even if the `WeakReference` itself is still reachable — useful for caches where you don't want to prevent garbage collection. Normal ("strong") references always prevent GC as long as they're reachable. Java also has `SoftReference` (collected only under memory pressure) and `PhantomReference` (used for post-mortem cleanup tracking).

### 66. What is `OutOfMemoryError` vs `StackOverflowError`?
- **`OutOfMemoryError`**: the heap (or another memory area like Metaspace) is exhausted and no more objects can be allocated.
- **`StackOverflowError`**: the call stack exceeds its limit, typically caused by excessive/infinite recursion.

---

## Java 8+ Features

### 67. What are the major features introduced in Java 8?
Lambda expressions, functional interfaces, the Stream API, default/static methods in interfaces, the new `java.time` Date/Time API, `Optional`, and method references.

### 68. What is a lambda expression?
A concise way to represent an anonymous function — an implementation of a functional interface — without boilerplate.

```java
Runnable r = () -> System.out.println("Running");
Comparator<Integer> cmp = (a, b) -> a - b;
```

### 69. What is a functional interface?
An interface with exactly **one** abstract method (though it can have multiple default/static methods), which can be implemented via a lambda expression. Annotated with `@FunctionalInterface` (optional but recommended). Examples: `Runnable`, `Callable`, `Comparator`, `Function<T,R>`, `Predicate<T>`, `Supplier<T>`, `Consumer<T>`.

### 70. What is the Stream API? Why use it?
An abstraction for processing sequences of elements in a functional, declarative style — supporting operations like `filter`, `map`, `reduce`, `sorted`, `collect` — often more concise and readable than imperative loops, and can be parallelized easily via `parallelStream()`.

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "Dave");
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

### 71. Difference between intermediate and terminal operations in streams?
- **Intermediate operations** (`filter`, `map`, `sorted`, `distinct`) are **lazy** — they don't execute until a terminal operation is invoked, and return a new stream.
- **Terminal operations** (`collect`, `forEach`, `reduce`, `count`) trigger actual processing of the stream and produce a result or side-effect; a stream can only be consumed once.

### 72. What is the difference between `map()` and `flatMap()`?
- `map()` transforms each element into another single element (1-to-1 mapping).
- `flatMap()` transforms each element into a stream of elements and then **flattens** all those streams into one — useful when dealing with nested structures (e.g., `List<List<T>>` → `List<T>`).

```java
List<List<Integer>> nested = List.of(List.of(1,2), List.of(3,4));
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// [1, 2, 3, 4]
```

### 73. What is `Optional`? Why was it introduced?
A container object that may or may not hold a non-null value, introduced to reduce `NullPointerException`s and make the possibility of "no value" explicit in method signatures, encouraging safer, more functional handling of absent values.

```java
Optional<String> name = Optional.ofNullable(getName());
String result = name.map(String::toUpperCase).orElse("UNKNOWN");
```

### 74. What are method references? Give examples of the four types.
A shorthand syntax for lambdas that just call an existing method:
1. **Static method reference**: `ClassName::staticMethod` (e.g., `Integer::parseInt`)
2. **Instance method reference on a particular object**: `object::instanceMethod` (e.g., `System.out::println`)
3. **Instance method reference on an arbitrary object of a type**: `ClassName::instanceMethod` (e.g., `String::toUpperCase`)
4. **Constructor reference**: `ClassName::new` (e.g., `ArrayList::new`)

### 75. What is the difference between `default` and `static` methods in interfaces?
- **`default` methods**: provide a method body that implementing classes inherit but can override; enables adding new methods to interfaces without breaking existing implementations.
- **`static` methods**: belong to the interface itself, called via the interface name, not inherited by implementing classes.

### 76. What are the key improvements from Java 9 to recent LTS versions (11, 17, 21)?
- **Java 9**: Module system (Project Jigsaw), `JShell` (REPL).
- **Java 10**: `var` for local variable type inference.
- **Java 11 (LTS)**: New `HttpClient`, `String` methods (`isBlank`, `strip`, `repeat`).
- **Java 14–16**: Records (compact immutable data carriers), pattern matching for `instanceof`, sealed classes (preview).
- **Java 17 (LTS)**: Sealed classes finalized, pattern matching for `switch` (preview).
- **Java 21 (LTS)**: Virtual threads (Project Loom), record patterns, pattern matching for switch (finalized), sequenced collections.

### 77. What are records (Java 14+)?
A concise way to declare immutable data-carrier classes — the compiler auto-generates the constructor, `equals()`, `hashCode()`, `toString()`, and accessor methods.

```java
public record Point(int x, int y) {}
// Equivalent to a full class with fields, constructor, getters, equals, hashCode, toString
```

### 78. What are sealed classes (Java 17+)?
Classes/interfaces that restrict which other classes/interfaces may extend or implement them, using the `sealed` keyword and a `permits` clause — enabling exhaustive pattern matching and more controlled class hierarchies.

```java
public sealed interface Shape permits Circle, Square, Triangle {}
```

---

## Object Class Methods & equals/hashCode

### 79. What methods does every Java object inherit from `Object`?
`equals()`, `hashCode()`, `toString()`, `getClass()`, `clone()`, `wait()`, `notify()`, `notifyAll()`, `finalize()` (deprecated).

### 80. What is the default implementation of `toString()`? Why override it?
By default, it returns `ClassName@hexHashCode`, which isn't very readable. Overriding it provides a meaningful, human-readable string representation of an object — extremely useful for debugging and logging.

### 81. What is object cloning? Difference between shallow and deep copy?
Cloning creates a copy of an object via the `Cloneable` interface and `clone()` method.
- **Shallow copy**: copies the object's fields as-is — if a field is a reference to another object, both the original and copy share the same referenced object.
- **Deep copy**: recursively copies referenced objects too, so the copy is fully independent of the original.

---

## Serialization

### 82. What is serialization and deserialization?
**Serialization** converts an object's state into a byte stream (for storage or transmission), and **deserialization** reconstructs the object from that byte stream. Achieved by implementing the `Serializable` marker interface.

### 83. What is `serialVersionUID`, and why is it important?
A unique identifier for a serializable class's version, used during deserialization to verify that the sender and receiver of a serialized object have loaded compatible class definitions. If not explicitly declared, the JVM auto-generates one based on class details — risky, since even minor class changes can invalidate previously serialized data.

### 84. What happens to `static` and `transient` fields during serialization?
- `static` fields belong to the class, not the instance, so they are **not serialized**.
- `transient` fields are explicitly excluded from serialization (e.g., for sensitive data like passwords, or non-serializable fields like threads/sockets).

---

## I/O & NIO

### 85. Difference between `InputStream`/`OutputStream` and `Reader`/`Writer`?
- `InputStream`/`OutputStream` handle **byte-based** I/O (binary data — images, files).
- `Reader`/`Writer` handle **character-based** I/O (text, handles character encoding properly).

### 86. What is the difference between traditional I/O (`java.io`) and NIO (`java.nio`)?
- **java.io**: stream-oriented, blocking I/O — reads data sequentially.
- **java.nio**: buffer-oriented and channel-based, supports **non-blocking I/O** and **selectors** (allowing a single thread to manage multiple channels) — better suited for scalable, high-throughput I/O applications.

---

## JVM Internals

### 87. What are the main components of JVM architecture?
- **Class Loader Subsystem**: loads, links, and initializes classes.
- **Runtime Data Areas**: Method Area, Heap, Stack, PC Registers, Native Method Stack.
- **Execution Engine**: Interpreter, JIT Compiler, Garbage Collector.
- **Native Method Interface (JNI)**: allows interaction with native (C/C++) code.

### 88. What are the different class loaders in Java?
- **Bootstrap ClassLoader**: loads core Java classes (`java.lang.*`, etc.) from the JDK's core libraries.
- **Extension/Platform ClassLoader**: loads classes from extension directories.
- **Application/System ClassLoader**: loads classes from the application's classpath.
This follows a **delegation model** — each loader delegates to its parent first before attempting to load a class itself.

### 89. What is the difference between compile-time and runtime polymorphism?
- **Compile-time (static) polymorphism**: method overloading — resolved by the compiler based on method signature.
- **Runtime (dynamic) polymorphism**: method overriding — resolved at runtime based on the actual object type via dynamic dispatch.

---

## Design Patterns in Java

### 90. Explain the Singleton pattern and its common implementations.
Ensures a class has only one instance and provides a global access point. Common approaches:
- Eager initialization (instance created at class loading).
- Lazy initialization with double-checked locking (thread-safe, `volatile` instance).
- Using an `enum` (simplest, inherently thread-safe and serialization-safe).

```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
```

### 91. What is the Factory pattern?
A creational pattern that delegates object instantiation to a factory method/class, decoupling client code from concrete implementation classes — useful when the exact type of object needed isn't known until runtime.

### 92. What is the Builder pattern? Why use it?
Used to construct complex objects step-by-step, especially useful when a class has many optional parameters — avoids "telescoping constructors" (multiple overloaded constructors) and improves readability.

```java
Pizza pizza = new Pizza.Builder()
    .size("Large")
    .addTopping("Cheese")
    .addTopping("Mushroom")
    .build();
```

### 93. What is the Observer pattern? Where is it used in Java?
Defines a one-to-many dependency where multiple "observer" objects are notified automatically when a "subject" object's state changes. Used in Java's event listener model (e.g., GUI event handling) and reactive programming frameworks.

### 94. What is Dependency Injection, and why is it useful?
A design pattern where an object's dependencies are provided (injected) externally rather than the object creating them itself — improves testability (easy to mock dependencies), decoupling, and adherence to the Single Responsibility/Inversion of Control principles. (Frameworks like Spring build heavily on this.)

---

## Common Coding Questions

### 95. Check if two strings are anagrams.
```java
static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    char[] c1 = a.toCharArray(), c2 = b.toCharArray();
    Arrays.sort(c1);
    Arrays.sort(c2);
    return Arrays.equals(c1, c2);
}
```

### 96. Find the first non-repeated character in a string.
```java
static Character firstNonRepeated(String s) {
    Map<Character, Integer> counts = new LinkedHashMap<>();
    for (char c : s.toCharArray()) counts.merge(c, 1, Integer::sum);
    for (Map.Entry<Character, Integer> e : counts.entrySet()) {
        if (e.getValue() == 1) return e.getKey();
    }
    return null;
}
```

### 97. Find duplicate elements in an array.
```java
static Set<Integer> findDuplicates(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    Set<Integer> duplicates = new HashSet<>();
    for (int num : arr) {
        if (!seen.add(num)) duplicates.add(num);
    }
    return duplicates;
}
```

### 98. Implement a basic LRU cache using `LinkedHashMap`.
```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

### 99. Reverse a linked list.
```java
class Node { int val; Node next; Node(int val) { this.val = val; } }

static Node reverse(Node head) {
    Node prev = null;
    while (head != null) {
        Node next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

### 100. Find the missing number in an array of 1 to N.
```java
static int findMissing(int[] arr, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = Arrays.stream(arr).sum();
    return expectedSum - actualSum;
}
```

---

## Miscellaneous / Advanced

### 101. What is the difference between `Comparable`'s natural ordering being used implicitly by `Collections.sort()` vs explicit `Comparator`?
`Collections.sort(list)` uses the class's own `compareTo()` (natural ordering) if it implements `Comparable`. `Collections.sort(list, comparator)` overrides that with custom sort logic — useful for sorting by different criteria without modifying the class itself.

### 102. What is reflection in Java? What are its use cases and downsides?
Reflection (`java.lang.reflect`) allows inspecting and manipulating classes, methods, fields, and constructors at runtime — even private ones. Used heavily by frameworks (Spring, Hibernate, JUnit) for dependency injection, ORM mapping, and annotation processing. Downsides: performance overhead, breaks encapsulation, bypasses compile-time type safety, and can be a security risk.

### 103. What are annotations? How do custom annotations work?
Metadata attached to code (classes, methods, fields) that doesn't directly affect execution but can be processed by tools/frameworks at compile time or runtime (often via reflection). Custom annotations are defined with `@interface` and can specify retention policy (`SOURCE`, `CLASS`, `RUNTIME`) and target elements.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LogExecutionTime {}
```

### 104. What is the difference between `equals()` and `compareTo()`?
`equals()` checks for exact logical equality (returns boolean). `compareTo()` (from `Comparable`) determines relative ordering, returning a negative, zero, or positive integer — used for sorting rather than strict equality checks. Note: it's best practice for `compareTo() == 0` to be consistent with `equals()`.

### 105. What is the difference between a shallow understanding of "pass by value" vs Java's actual behavior with objects?
Java is **strictly pass-by-value** — always. When passing an object, the *reference* (memory address) is passed by value, meaning the method can modify the object's internal state through that reference, but reassigning the parameter itself inside the method won't affect the caller's original reference.

```java
void modify(StringBuilder sb) {
    sb.append(" world");   // affects caller's object (mutation through reference)
    sb = new StringBuilder("new"); // does NOT affect caller's reference
}
```

### 106. What is the difference between `List.of()` and `Arrays.asList()`?
- `List.of(...)` (Java 9+) creates a **truly immutable** list — any modification attempt throws `UnsupportedOperationException`, and it disallows `null` elements.
- `Arrays.asList(...)` returns a **fixed-size** list backed by the array — you can't add/remove elements, but you *can* use `set()` to modify existing elements (which also reflects back into the original array).

### 107. What is the significance of the `transient` keyword outside serialization context — is there any?
`transient` is specific to serialization — marking a field so it's skipped during the default serialization process. It has no effect outside of `Serializable` classes.

---

## Quick Cheat-Sheet Summary

| Concept | Key Point |
|---|---|
| `==` vs `.equals()` | Reference vs content comparison |
| `String` immutability | Security, pooling, thread-safety, hashcode caching |
| Checked vs Unchecked exceptions | Compile-time enforced vs runtime-only |
| `HashMap` internals | hash → bucket index, collision handling via list/tree |
| `equals`/`hashCode` contract | Equal objects must have equal hash codes |
| Generics | Type erasure at runtime, compile-time safety |
| PECS | Producer Extends, Consumer Super (wildcards) |
| Stream API | Lazy intermediate ops, eager terminal ops |
| Records | Immutable data carriers, auto-generated boilerplate |
| Pass-by-value | Java always passes references by value |

---
### 108. # Java Collections Framework - Interview Notes

## Java Collections Hierarchy

```text
                           Iterable
                               │
                          Collection
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
      List                   Set                  Queue
        │                      │                      │
        ├── ArrayList          ├── HashSet           ├── PriorityQueue
        ├── LinkedList         ├── LinkedHashSet     └── ArrayDeque
        ├── Vector             └── TreeSet
        └── Stack (Legacy)

Map (Separate Hierarchy)
│
├── HashMap
├── LinkedHashMap
├── TreeMap
├── Hashtable (Legacy)
└── ConcurrentHashMap
```

> **Note:** `Map` is **not** a subtype of the `Collection` interface. It belongs to a separate hierarchy because it stores **key-value pairs**, whereas `Collection` stores individual elements.

---

## Q1. What is the Java Collections Framework?

The **Java Collections Framework (JCF)** provides interfaces and classes to efficiently store, manipulate, and retrieve groups of objects.

---

## Q2. What is a List?

- Ordered collection
- Allows duplicates
- Supports indexing

| Collection | Best Use |
|------------|----------|
| ArrayList | Fast random access |
| LinkedList | Frequent insert/delete |
| Vector | Thread-safe (Legacy) |
| Stack | LIFO (Legacy) |

---

## Q3. What is a Set?

- Stores unique elements
- No duplicates

| Collection | Best Use |
|------------|----------|
| HashSet | Fast lookup |
| LinkedHashSet | Maintains insertion order |
| TreeSet | Maintains sorted order |

---

## Q4. What is a Queue?

- Follows **FIFO (First In, First Out)**

| Collection | Best Use |
|------------|----------|
| PriorityQueue | Priority-based processing |
| ArrayDeque | Queue + Stack implementation |

---

## Q5. What is a Map?

- Stores **key-value pairs**
- Keys are unique
- Values can be duplicated

| Collection | Best Use |
|------------|----------|
| HashMap |

*Good luck with your interview! Be ready to explain the "why" behind each answer, not just the definition — interviewers often follow up with "why does Java do it this way?"*
