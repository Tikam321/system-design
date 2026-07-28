# REST API Interview Questions & Answers (Spring Boot Focus)

A curated set of the most commonly asked REST API interview questions, from fundamentals to Spring Boot–specific implementation details.

---

## 1. Fundamentals

### Q1. What is REST?
REST (Representational State Transfer) is an architectural style for designing networked applications. It's not a protocol or standard — it's a set of constraints that, when followed, make a system scalable, simple, and easy to maintain. An API that follows these constraints is called "RESTful."

### Q2. What are the constraints/principles of REST?
1. **Client-Server** – separation of concerns between UI (client) and data storage (server).
2. **Statelessness** – each request from client to server must contain all information needed to understand it; server stores no client session state between requests.
3. **Cacheability** – responses must define themselves as cacheable or not, to improve performance.
4. **Uniform Interface** – a consistent way to interact with resources (this is what makes REST "RESTful"). Sub-constraints:
   - Resource identification (URIs)
   - Manipulation via representations (JSON/XML)
   - Self-descriptive messages
   - HATEOAS (Hypermedia as the Engine of Application State)
5. **Layered System** – client can't tell whether it's connected directly to the server or through an intermediary (proxy, load balancer, gateway).
6. **Code on Demand (optional)** – server can send executable code (e.g., JavaScript) to the client.

### Q3. What is a "resource" in REST?
Anything that can be named and identified — a user, an order, a document. Each resource is identified by a URI, e.g., `/users/123`. The resource itself is an abstract concept; what you send over the wire is a *representation* of it (JSON, XML, etc.).

### Q4. Difference between REST and SOAP?
| REST | SOAP |
|---|---|
| Architectural style | Protocol |
| Uses HTTP verbs, lightweight (JSON) | XML-based, heavier payloads |
| Stateless by design | Can be stateful |
| No strict standard | Strict standard (WSDL, XML Schema) |
| Faster, easier to scale | Better for enterprise-grade security (WS-Security), formal contracts |

### Q5. Is REST stateless? What does that actually mean in practice?
Yes. The server doesn't store any client context between requests. Every request must contain everything needed to process it (auth token, params, etc.). In practice this means:
- No server-side session tied to a specific client between calls.
- Authentication is typically done via a token (JWT) sent with every request, not a server session.
- This enables horizontal scaling — any server instance can handle any request.

---

## 2. HTTP Basics

### Q6. What are the common HTTP methods and their purpose?
| Method | Purpose | Idempotent? | Safe? |
|---|---|---|---|
| GET | Retrieve a resource | Yes | Yes |
| POST | Create a resource / trigger processing | No | No |
| PUT | Replace a resource entirely | Yes | No |
| PATCH | Partially update a resource | No (typically) | No |
| DELETE | Remove a resource | Yes | No |
| HEAD | Like GET but no body, just headers | Yes | Yes |
| OPTIONS | Discover allowed methods on a resource | Yes | Yes |

### Q7. What does "idempotent" mean, and why does it matter?
An operation is idempotent if calling it once has the same effect as calling it multiple times. `PUT /users/1` with the same body always results in the same state, no matter how many times you call it. `POST /users` is NOT idempotent — calling it twice creates two resources.

This matters for retry logic: if a network call times out, a client can safely retry an idempotent request (like PUT/DELETE) without worrying about duplicate side effects, but retrying a POST could create duplicates.

### Q8. What's the difference between PUT and PATCH?
- **PUT** replaces the *entire* resource. If you omit a field, it's typically treated as null/removed.
- **PATCH** applies a *partial* update — only the fields you send are updated; the rest remain untouched.

### Q9. Why is GET considered "safe" but not "idempotent" in the same breath — what's the difference between safe and idempotent?
- **Safe** = doesn't change server state at all (read-only).
- **Idempotent** = repeating it produces the same result, but it *could* still change state (e.g., DELETE is idempotent but not safe — deleting once or ten times leaves the resource gone, but state did change on the first call).

All safe methods are idempotent, but not all idempotent methods are safe.

### Q10. What are the important HTTP status code categories?
- **1xx** – Informational
- **2xx** – Success (`200 OK`, `201 Created`, `202 Accepted`, `204 No Content`)
- **3xx** – Redirection (`301 Moved Permanently`, `304 Not Modified`)
- **4xx** – Client error (`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `405 Method Not Allowed`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests`)
- **5xx** – Server error (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`)

### Q11. What's the difference between 401 and 403?
- **401 Unauthorized** — you're not authenticated (no/invalid credentials). "I don't know who you are."
- **403 Forbidden** — you're authenticated, but not allowed to access this resource. "I know who you are, but you can't do this."

### Q12. Difference between 200, 201, and 204?
- **200 OK** – generic success, response has a body.
- **201 Created** – resource was created (typically after POST); should include a `Location` header pointing to the new resource.
- **204 No Content** – success, but no body to return (common for DELETE or PUT with nothing to send back).

---

## 3. Design & Best Practices

### Q13. What are good practices for naming REST endpoints/URIs?
- Use **nouns**, not verbs: `/users` not `/getUsers`.
- Use plural nouns for collections: `/orders`, `/orders/123`.
- Represent hierarchy through nesting: `/users/123/orders`.
- Use hyphens for readability, lowercase paths: `/order-items`.
- Avoid deep nesting beyond 2–3 levels; use query params for filtering instead: `/orders?status=shipped`.
- Let the HTTP method express the action, not the URL.

### Q14. How should you handle API versioning?
Common strategies:
1. **URI versioning**: `/api/v1/users` — simplest, most visible, widely used.
2. **Query parameter**: `/api/users?version=1`
3. **Header versioning**: custom header like `X-API-Version: 1`
4. **Content negotiation (media type)**: `Accept: application/vnd.myapp.v1+json`

URI versioning is the most common in practice due to simplicity, even though header-based is considered "more RESTful" by purists.

### Q15. How do you handle pagination in REST APIs?
- **Offset-based**: `/users?page=2&size=20` — simple, but can have consistency issues if data changes between requests.
- **Cursor-based**: `/users?after=xyz123&limit=20` — more stable for large/changing datasets.
- Response often includes metadata: `totalElements`, `totalPages`, `hasNext`, links to next/prev pages (HATEOAS style).

### Q16. What is HATEOAS?
Hypermedia As The Engine Of Application State. The response includes links describing possible next actions, so the client doesn't need to hardcode URLs.

```json
{
  "id": 123,
  "status": "PENDING",
  "_links": {
    "self": { "href": "/orders/123" },
    "cancel": { "href": "/orders/123/cancel" }
  }
}
```

In Spring, this is implemented via **Spring HATEOAS** (`EntityModel`, `WebMvcLinkBuilder`).

### Q17. How do you design error responses consistently?
Use a standard error object structure, and ideally follow **RFC 7807 (Problem Details)**:
```json
{
  "type": "https://example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Email field is required",
  "instance": "/users"
}
```
Spring Boot 3+ has built-in support for this via `ProblemDetail`.

### Q18. What is content negotiation?
The client tells the server what format it wants via the `Accept` header (e.g., `application/json`, `application/xml`), and the server responds accordingly. In Spring, `HttpMessageConverter`s handle this automatically based on `@RequestMapping(produces = ...)` or the `Accept` header.

### Q19. How do you handle filtering, sorting, and searching in REST?
Via query parameters, keeping URIs resource-focused:
```
GET /products?category=electronics&sort=price,desc&minPrice=100
```

### Q20. What's the difference between a REST API being "RESTful" vs just "HTTP API" / RPC-style?
Many "REST APIs" in the wild are actually RPC-style over HTTP (e.g., `/getUserDetails`, `/createOrderNow`) and don't follow the uniform interface or HATEOAS constraints. True RESTful design is rarer in practice — most production APIs are "REST-ish."

---

## 4. Security

### Q21. How do you secure a REST API?
- **HTTPS/TLS** for transport security.
- **Authentication**: JWT tokens, OAuth2, API keys.
- **Authorization**: role/permission-based access control.
- **Input validation** to prevent injection attacks.
- **Rate limiting / throttling** to prevent abuse.
- **CORS** configuration to control which origins can call the API.

### Q22. How does JWT-based authentication work?
1. Client logs in with credentials.
2. Server validates and issues a signed JWT (contains claims like user id, roles, expiry).
3. Client sends the JWT in the `Authorization: Bearer <token>` header on each subsequent request.
4. Server validates the token's signature and expiry on each request — no session lookup needed, which keeps things stateless.

### Q23. How do you implement security in Spring Boot?
Using **Spring Security**:
- Define a `SecurityFilterChain` bean to configure which endpoints require authentication.
- Use `JwtAuthenticationFilter` (custom) to parse and validate tokens.
- Use `@PreAuthorize("hasRole('ADMIN')")` for method-level authorization.
- Configure `PasswordEncoder` (e.g., `BCryptPasswordEncoder`) for storing passwords.

### Q24. What is CORS and why does it matter for REST APIs?
Cross-Origin Resource Sharing — a browser security mechanism that blocks JavaScript from making requests to a different origin (domain/port/protocol) unless the server explicitly allows it via headers like `Access-Control-Allow-Origin`. In Spring Boot, configured via `@CrossOrigin` or a global `CorsConfigurationSource` bean.

---

## 5. Spring Boot Specific

### Q25. What annotations are commonly used to build REST controllers in Spring Boot?
- `@RestController` – combines `@Controller` + `@ResponseBody`; return values are serialized directly into the response body.
- `@RequestMapping` / `@GetMapping` / `@PostMapping` / `@PutMapping` / `@PatchMapping` / `@DeleteMapping`
- `@PathVariable` – extract values from the URI path.
- `@RequestParam` – extract query parameters.
- `@RequestBody` – deserialize the request body into a Java object.
- `@ResponseStatus` – set a specific HTTP status for a response.

### Q26. How does Spring Boot handle exceptions globally?
Using `@ControllerAdvice` (or `@RestControllerAdvice`) combined with `@ExceptionHandler`:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```
This centralizes error handling instead of repeating try-catch blocks in every controller.

### Q27. What is `ResponseEntity` and why use it over returning a plain object?
`ResponseEntity<T>` lets you control the full HTTP response — status code, headers, and body — not just the body. Useful when you need to return `201 Created` with a `Location` header, or `404` conditionally.

```java
return ResponseEntity.status(HttpStatus.CREATED)
        .location(uri)
        .body(savedUser);
```

### Q28. How does request validation work in Spring Boot?
Using **Bean Validation (JSR-380)** annotations on DTOs (`@NotNull`, `@Size`, `@Email`, etc.) combined with `@Valid` in the controller method:
```java
@PostMapping("/users")
public ResponseEntity<User> create(@Valid @RequestBody UserDto dto) { ... }
```
Validation errors are caught by a `MethodArgumentNotValidException` handler in a `@ControllerAdvice`.

### Q29. What's the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`/`@RestController`?
All are stereotypes of `@Component` (so all become Spring-managed beans), but they signal intent and get extra behavior:
- `@Repository` – adds automatic exception translation for persistence-layer exceptions.
- `@Service` – marks business/service-layer logic (semantic only, no extra behavior).
- `@Controller` / `@RestController` – marks web layer, handles HTTP requests.

### Q30. How do you document a REST API in Spring Boot?
Typically with **springdoc-openapi** (successor to Springfox), which auto-generates OpenAPI/Swagger docs from your controllers and annotations (`@Operation`, `@Parameter`, `@Schema`), exposing a UI at `/swagger-ui.html`.

### Q31. How do you test REST APIs in Spring Boot?
- **Unit tests**: mock the service layer, test controller logic with `MockMvc`.
- **Integration tests**: `@SpringBootTest` with `TestRestTemplate` or `WebTestClient` to hit real endpoints.
- **Contract testing**: tools like Spring Cloud Contract for consumer-driven contracts between services.

Example with `MockMvc`:
```java
mockMvc.perform(get("/users/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("John"));
```

### Q32. How does Spring Boot achieve statelessness with sessions typically used in traditional web apps?
By disabling session creation in the security config:
```java
.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
```
and relying entirely on token-based auth (JWT) instead of `HttpSession`.

---

## 6. Scenario / Practical Questions

### Q33. How would you design a REST API for an e-commerce "Order" system?
```
POST   /orders                 -> create an order
GET    /orders/{id}            -> get order details
GET    /orders?userId=123      -> list orders for a user
PATCH  /orders/{id}            -> update order status
DELETE /orders/{id}            -> cancel an order
GET    /orders/{id}/items      -> list items in an order
```
Discuss: status codes for each, validation, idempotency for POST (e.g., via an `Idempotency-Key` header for payment-related calls), pagination for the list endpoint.

### Q34. How do you handle long-running operations in a REST API (e.g., report generation)?
Use the **202 Accepted** pattern: client calls POST, server responds immediately with 202 and a status URL (`/jobs/123/status`). Client polls that URL until the job completes, or use webhooks/callbacks for async notification.

### Q35. How would you prevent duplicate order creation if a client retries a POST request due to a network timeout?
Use an **idempotency key** — client generates a unique key per logical operation and sends it as a header (`Idempotency-Key: abc-123`). Server checks if it has already processed that key; if so, returns the original response instead of creating a duplicate.

### Q36. How do you handle rate limiting in a Spring Boot REST API?
- Use a library like **Bucket4j** or **Resilience4j**, or an API gateway (Spring Cloud Gateway, Kong, Nginx) in front of the service.
- Return `429 Too Many Requests` with a `Retry-After` header when limits are exceeded.

### Q37. How would you design an API to support both mobile and web clients that need different amounts of data (over-fetching/under-fetching)?
- Use sparse fieldsets: `/users/1?fields=name,email`
- Or provide different endpoints/DTOs per client need.
- Or consider **GraphQL** alongside REST if this becomes a recurring pain point — good to mention you understand the trade-off, even in a REST-focused interview.

---

## 7. Quick-Fire Round (rapid recall)

| Question | Short Answer |
|---|---|
| Is HTTP stateless? | Yes, by protocol design |
| Which method is not safe and not idempotent? | POST |
| Which status code for a validation failure? | 400 or 422 |
| Which header specifies the format client wants? | `Accept` |
| Which header specifies the format of the request body? | `Content-Type` |
| Where do you put the new resource's URL after creation? | `Location` header, with `201 Created` |
| What does ETag support? | Caching / conditional requests (`If-None-Match`) |
| Default Spring Boot embedded server? | Tomcat |
| Annotation to bind JSON body to Java object? | `@RequestBody` |
| How to return a custom status code with a body? | `ResponseEntity` |

---

## Tips for the Interview
- Be ready to **design an API on a whiteboard** (endpoints, methods, status codes) for a given domain — this is one of the most common practical rounds.
- Know the **trade-offs**, not just definitions (e.g., why PUT vs PATCH, why JWT vs session).
- Be able to explain **how you'd debug** a REST API issue (checking status codes, logs, Postman, curl).
- Mention real Spring Boot code when possible — interviewers like concrete examples over textbook definitions.

Good luck with your interview!
