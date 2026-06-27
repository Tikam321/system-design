# CompletableFuture

## What is CompletableFuture?

`CompletableFuture` is an implementation of the `Future` interface introduced in **Java 8** for **asynchronous, non-blocking programming**. It allows tasks to run in the background, supports chaining, combining multiple tasks, and provides better exception handling.

---

## Why use CompletableFuture?

`Future` has some limitations:

* `get()` blocks the calling thread.
* Cannot chain asynchronous tasks.
* Cannot combine multiple tasks.
* Limited exception handling.

`CompletableFuture` overcomes these limitations.

---

## Basic Example

```java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> "Hello");

System.out.println(future.join());
```

---

## Common Methods

* `runAsync()` – Execute a task without returning a result.
* `supplyAsync()` – Execute a task and return a result.
* `thenApply()` – Transform the result.
* `thenAccept()` – Consume the result.
* `thenCompose()` – Chain dependent tasks.
* `thenCombine()` – Combine two independent tasks.
* `allOf()` – Wait for all tasks.
* `exceptionally()` – Handle exceptions.

---

## Interview Answer

> **CompletableFuture** is a Java 8 class that extends `Future` to support **asynchronous, non-blocking programming**. It enables task chaining, combining multiple asynchronous operations, and better exception handling without blocking the calling thread.
