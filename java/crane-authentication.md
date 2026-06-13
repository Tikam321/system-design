# Java 17, DI, and Binding Interview Q&A

### Q1: What are the top 3 production-ready features introduced in Java 17?
**Ans:** 
*   **Records:** Fast way to create immutable data classes without boilerplate code.
*   **Sealed Classes:** Lets you explicitly define and restrict which subclasses can extend your class.
*   **Pattern Matching for switch:** Allows you to test specific data types and object states directly inside a switch statement.

---

### Q2: Which Dependency Injection framework do you use in production, and why?
**Ans:** We use **Spring Boot** because it automates object lifecycles via Inversion of Control (IoC), decouples code using interfaces, simplifies unit testing with Mockito, and uses clear annotations like `@Autowired` and `@Component` instead of XML.

---

### Q3: What is the exact difference between Method Overriding and Method Hiding?
**Ans:** 
*   **Method Overriding:** Applies to **instance methods**. It uses **runtime binding**, meaning the JVM runs the method based on the **actual object type**.
*   **Method Hiding:** Applies to **static methods**. It uses **compile-time binding**, meaning the JVM runs the method based on the **declared reference type**.

---

### Q4: How do you differentiate Compile-Time Binding from Run-Time Binding?
**Ans:** 
*   **Compile-Time Binding:** The compiler determines the target method before the app runs. It applies to `static`, `private`, `final`, and overloaded methods.
*   **Run-Time Binding:** The JVM determines the target method at execution time using a virtual method table. It applies exclusively to overridden instance methods.

---

### Q5: How does method binding handle primitive data types?
**Ans:** Primitives use **compile-time binding exclusively**. Since they lack an inheritance hierarchy or methods, the compiler resolves overloaded methods at compile time using exact type matches or automatic type widening (e.g., matching an `int` parameter to a `long` signature).
