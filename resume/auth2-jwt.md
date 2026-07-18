# OAuth 2.0, JWT & Spring Security — Interview Revision Notes

*Authentication & Authorization in Distributed Microservices*

---

## 1. The Problem This Solves

In a microservices architecture, many independent services (Order Service, User Service, Payment Service, etc.) all need to answer: **who is this request from, and are they allowed to do this?**

Each service maintaining its own login/session system doesn't scale. The goal is a security mechanism that is:

- **Centralized for authentication** — one place issues identity.
- **Distributed / stateless for authorization** — any service can verify a request on its own, without calling back to a central server.

---

## 2. OAuth 2.0 — The Authorization Framework

OAuth 2.0 defines *how* a client gets permission (a token) to access resources on a user's behalf, without ever handling or storing the user's raw password.

- The user authenticates once against an **Authorization Server** (e.g. Keycloak, Okta, Auth0, or a custom Spring service).
- That server issues a token representing: *this user, with these permissions, for this duration.*
- The client then presents that token to any microservice instead of re-authenticating every time.

> **Key point for interviews:** the client never sees or stores the user's actual login credentials — only a scoped, time-limited token. That's the core security benefit.

### Common OAuth 2.0 Flows

- **Authorization Code Flow** — used for user-facing apps (browser/mobile) logging in a real person.
- **Client Credentials Flow** — used for service-to-service calls with no user involved.

---

## 3. JWT — The Token Format That Makes It Stateless

JWT (JSON Web Token) is the specific token format that makes OAuth 2.0 work **without server-side session storage**. It's a signed piece of JSON, structured in three parts:

| Part | Contains | Example |
|---|---|---|
| **Header** | Token type and signing algorithm | `{"alg":"RS256","typ":"JWT"}` |
| **Payload** | Claims: user id, roles/scopes, issuer, expiry | `{"sub":"123","role":"ADMIN","exp":...}` |
| **Signature** | Cryptographic signature proving the token wasn't tampered with | `HMACSHA256(header+payload, privateKey)` |

**How verification works:**

- The Authorization Server **signs** the JWT using its **PRIVATE key** at the time of login.
- Every microservice **verifies** that signature using the Authorization Server's **PUBLIC key** (often fetched once from a `/.well-known/jwks.json` endpoint and cached).
- Because the signature check happens locally, no service needs to call back to the Authorization Server or maintain shared session state — that's what **stateless** means here.

> ⚠️ **Common mistake to avoid:** don't describe this using session language ("a session ID is created"). Sessions and JWTs are *opposite* models — sessions require server-side state, JWTs are self-contained and require none.

---

## 4. Spring Security — Enforcing It in Each Microservice

Spring Security is the framework layer that intercepts requests inside each Java microservice and enforces the policy, per request:

1. A filter (`OAuth2ResourceServerConfigurer` / `BearerTokenAuthenticationFilter`) extracts the JWT from the `Authorization: Bearer <token>` header.
2. It validates the signature against the issuer's public key.
3. It extracts claims (roles/scopes) and builds a Spring `Authentication` object.
4. Access rules are then declared declaratively:

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .requestMatchers("/orders/**").hasAuthority("SCOPE_orders.read")
    .anyRequest().authenticated()
);
```

---

## 5. Full Request Flow (Diagram)

End-to-end flow: login, token issuance, and independent verification at each downstream microservice.

![OAuth2 + JWT Flow](oauth_flow.png)

**Flow summary:**

1. User logs in → Authorization Server issues a signed JWT (access token + refresh token).
2. Client attaches the JWT to every API call as a Bearer token.
3. Each microservice (or the API Gateway) independently validates the JWT via Spring Security's resource-server support.
4. Each service enforces its own authorization rules (roles/scopes) from the token's claims — no cross-service session lookups needed.
5. Token expires after a set time; the client uses a refresh token to get a new one without forcing a full re-login.

---

## 6. Session-Based Auth vs. JWT — Quick Comparison

| Aspect | Session-based Auth (old) | JWT Auth (stateless) |
|---|---|---|
| Where identity lives | Server-side session store (memory/Redis) | Inside the token itself (claims) |
| What client holds | Session ID (cookie) | Signed JWT (Bearer token) |
| Verifying a request | Server looks up session store | Service verifies signature locally |
| Scaling across microservices | Needs shared/sticky session store | No shared state needed |
| Revocation | Easy — delete session server-side | Harder — needs short expiry / blocklist |

---

## 7. One-Paragraph Answer (Say This In The Interview)

> "I implemented authentication and authorization using OAuth 2.0 and JWT with Spring Security across our microservices. OAuth 2.0 handles the authorization flow — it lets a client access resources on a user's behalf without ever exposing the user's actual credentials. Once the user authenticates, the Authorization Server issues a JWT signed with its private key, carrying the user's identity and role-based claims plus an expiry. Because it's self-contained, every downstream microservice can independently verify the token's signature using the Authorization Server's public key — no shared session store, no server-side session state, no network call back to a central server. That's what makes it stateless: each request carries everything a service needs to authenticate and authorize it on its own, which scales cleanly across distributed services."

---

## 8. Q&A — Likely Interview Questions

### Conceptual

**Q: What's the difference between authentication and authorization?**
A: Authentication answers "who are you?" (verifying identity — login). Authorization answers "what are you allowed to do?" (checking permissions/roles/scopes once identity is known). OAuth 2.0 is fundamentally an *authorization* framework; authentication is layered on top via something like OpenID Connect or a custom login step.

**Q: Is OAuth 2.0 itself an authentication protocol?**
A: Not strictly — OAuth 2.0 was designed for delegated *authorization* (letting an app act on a user's behalf). Authentication is technically handled by **OpenID Connect (OIDC)**, which is built on top of OAuth 2.0 and adds an ID token (also a JWT) specifically to prove identity.

**Q: Why JWT instead of opaque tokens?**
A: An opaque token is just a random string the server must look up in a database to know anything about it (introspection call needed every time). A JWT is self-describing — the claims are embedded and cryptographically verifiable, so no database lookup is needed on every request. Tradeoff: opaque tokens are trivially revocable; JWTs are not, by design.

### JWT-Specific

**Q: Why sign with a private key and verify with a public key instead of a shared secret?**
A: Asymmetric signing (RS256) means only the Authorization Server holds the private key and can *create* valid tokens. Every microservice only needs the public key to *verify* — if a microservice were compromised, an attacker still couldn't forge new tokens, only read the public key. With a shared secret (HMAC/HS256), any service holding the secret could both verify *and* forge tokens — a bigger blast radius if one service is breached.

**Q: How do you handle token revocation if there's no session to invalidate?**
A: Keep access tokens short-lived (e.g., 5–15 minutes) so a compromised token has a small window of danger. Use longer-lived refresh tokens stored securely, and maintain a revocation/blocklist (e.g., in Redis) checked only for refresh tokens or for emergency invalidation — this keeps most requests fast (no lookup) while still allowing a "kill switch."

**Q: What's the difference between an access token and a refresh token?**
A: Access token — short-lived, sent with every API request, used directly for authorization. Refresh token — longer-lived, stored securely (httpOnly cookie or secure storage), used only to request a new access token without forcing the user to log in again.

**Q: What happens if the JWT is stolen?**
A: Whoever holds it can use it until it expires — that's why short expiry matters. Best practices: use HTTPS everywhere (prevent interception), store tokens in httpOnly cookies rather than localStorage (mitigates XSS token theft), and keep refresh tokens especially well-protected since they're longer-lived.

**Q: What's inside the JWT payload — should sensitive data go there?**
A: No — JWT payloads are Base64-encoded, *not encrypted*, by default (unless you specifically use JWE). Anyone can decode and read the claims. Never put passwords or sensitive PII in a JWT; only put non-sensitive identity/authorization claims (user id, roles, scopes, expiry).

### Spring Security / Architecture

**Q: Where should token validation happen — API Gateway or each individual microservice?**
A: Either is valid, tradeoffs differ. Gateway-level validation centralizes the logic (one place to update, less duplication) but creates a single choke point and means internal service-to-service calls behind the gateway are implicitly trusted. Per-service validation (defense in depth) is more resilient — even if a request somehow bypasses the gateway, each service independently checks — at the cost of slight redundancy. Many production systems do both: gateway does a first pass, services still validate/authorize independently.

**Q: How does Spring Security know how to verify the token without a shared secret at each service?**
A: Each resource-server microservice is configured with the Authorization Server's `issuer-uri` or `jwk-set-uri` in `application.yml`. Spring Security fetches and caches the public keys (JWKS) from that endpoint at startup, then verifies signatures against them locally — no per-request network round trip.

**Q: How do you enforce role-based vs scope-based access control in Spring Security?**
A: `hasRole("ADMIN")` checks a `ROLE_` prefixed authority (typically derived from a custom `JwtAuthenticationConverter` mapping a `roles` claim). `hasAuthority("SCOPE_orders.read")` checks OAuth2 scopes directly from the token's `scope`/`scp` claim. Which one you use depends on whether your system models permissions as coarse-grained roles or fine-grained scopes — many systems use both together.

**Q: What happens if clocks are out of sync between the Authorization Server and a microservice?**
A: Clock skew can make a still-valid token appear expired (or not-yet-valid) when checking the `exp`/`nbf` claims. Spring Security allows configuring a small clock skew leeway (a few seconds/minutes) to tolerate minor drift between servers.

**Q: How would you test this setup?**
A: Unit test the Spring Security filter chain config using `@WithMockUser` or `SecurityMockMvcRequestPostProcessors.jwt()` to simulate authenticated requests with specific claims/roles, without needing a real Authorization Server. For integration tests, spin up a test Authorization Server (e.g., a lightweight OIDC provider in Docker) or use WireMock to stub the JWKS endpoint.

---

*Tip for revision: re-read Section 3 and Section 6 twice — that's where session-based auth and JWT auth most commonly get blended together in explanations, and interviewers listen closely for that distinction.*
