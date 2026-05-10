# 1. # Difference Between Lazy Loading and Eager Loading

| Lazy Loading | Eager Loading |
|---|---|
| Data loads only when needed | Data loads immediately |
| Improves initial performance | More data fetched upfront |
| Reduces memory usage | Higher memory usage |
| May cause additional queries | Fewer additional queries |

---

# Lazy Loading

Related data is fetched only when accessed.

## Example

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;
```

Orders load only when:

```java
user.getOrders()
```

is called.

---

# Eager Loading

Related data loads immediately with parent entity.

## Example

```java
@OneToMany(fetch = FetchType.EAGER)
private List<Order> orders;
```

Orders load together with User entity.

---

# LazyInitializationException

Occurs when lazy-loaded data is accessed after Hibernate session is closed.

---

# Example

```java
User user = userRepository.findById(1).get();

session.close();

user.getOrders();
```

Output:

```text
LazyInitializationException
```

---

# Why?

Because:
- Orders were not loaded initially
- Session already closed
- Hibernate cannot fetch data now

---

# Solutions

- Use `JOIN FETCH`
- Use transactional boundaries properly
- DTO mapping
- Open Session In View (less preferred)

---

# Interview Ready Answer

> Lazy loading fetches related data only when required, while eager loading fetches related data immediately with the parent entity. LazyInitializationException occurs when a lazy-loaded association is accessed after the Hibernate session is closed, preventing Hibernate from loading the required data.
