# Auth Migration: localStorage → HttpOnly Cookies — Interview Prep

Resume bullet this is based on:
> "Migrated frontend token storage from localStorage to HttpOnly, Secure, SameSite cookies, mitigating XSS-based token theft and CSRF exposure across the application."

This doc has two parts: **(1)** how to explain it end-to-end if asked to walk through it, and **(2)** likely cross-questions an interviewer will ask based on this specific line, with answers.

---

## Part 1: How to Explain It (Problem → Fix → Why → Tradeoffs)

**1. The problem:**
Previously, on login, the access token was stored in `localStorage` and manually attached to every API request. `localStorage` is readable by any JavaScript running on the page — including malicious scripts injected via XSS (e.g., through an unsanitized input field or a compromised third-party dependency). If an attacker injected even a small script, they could read the token directly via JS and impersonate the user.

**2. The fix:**
Moved the token into an **HttpOnly cookie**. HttpOnly means JavaScript cannot read the cookie at all — `document.cookie` won't expose it. So even if an XSS vulnerability exists elsewhere on the page, the attacker's injected script can no longer steal the session token, because it's simply not accessible to JS.

**3. The other two flags, and what each solves:**
- **`Secure`** — cookie is only sent over HTTPS, never over plain HTTP, preventing interception over an unencrypted connection.
- **`SameSite=Strict`** — addresses **CSRF**, a separate problem. Without it, if a user is logged in and visits a malicious site in another tab, that site could trigger a request to your API, and the browser would attach the cookie automatically (browsers send cookies regardless of which site initiated the request, by default). `SameSite=Strict` tells the browser not to send the cookie on cross-origin requests, closing that gap.

**4. Precision point (important if pushed):**
HttpOnly doesn't *prevent* XSS — it limits the *impact* of XSS. If an XSS vulnerability exists, an attacker can still do damage on the page itself, but can no longer walk away with the session token specifically. Preventing XSS itself is a separate effort (input sanitization, output encoding, CSP headers). This change specifically hardens **token theft as an XSS payoff**, not XSS as a whole.

**5. Tradeoff to mention proactively:**
Since JS can no longer read the token, the frontend can't manually attach it via an `Authorization` header anymore — the browser sends the cookie automatically based on domain. This required correctly configuring CORS with credentials (`credentials: 'include'` on the frontend, `Access-Control-Allow-Credentials: true` + explicit origin on the backend — wildcard `*` origins don't work with credentialed requests) and making sure cross-origin setups respected `SameSite` rules.

---

## Part 2: Cross-Questions Based on This Resume Point

### Q1: If JavaScript can't read the HttpOnly cookie, how does the frontend know if the user is logged in?

**Answer:**
The frontend doesn't rely on reading the token to determine auth state. Instead:
- On app load, make a lightweight API call (e.g., `/me` or `/session/validate`) — the browser automatically attaches the cookie, and the server responds with the user's identity/profile if the session is valid, or a 401 if not.
- The frontend keeps auth state in memory/app state (e.g., Redux) based on that response, not by inspecting the cookie itself.

---

### Q2: How do you handle token refresh if JS can't read the token or its expiry?

**Answer:**
Two common patterns:
- **Server-driven refresh:** The server issues a short-lived session cookie plus a longer-lived refresh mechanism, also as an HttpOnly cookie. When the access cookie expires, the frontend calls a `/refresh` endpoint; the browser sends the refresh cookie automatically, and the server issues a new session cookie in the response — again, all without JS ever touching the token.
- **Silent refresh via 401 interception:** The frontend's HTTP client intercepts a 401 response, calls `/refresh` transparently, and retries the original request once — the user never notices, and the frontend never needs to know the token's actual expiry time up front.

---

### Q3: Doesn't moving to cookies reintroduce CSRF risk, since cookies are the classic CSRF attack vector?

**Answer:**
Yes — cookie-based auth is specifically why CSRF is a risk to design against, which is exactly why `SameSite=Strict` (or at minimum `Lax`) is included as part of this migration, not an afterthought. For extra defense-in-depth beyond `SameSite`, some systems also add a CSRF token (double-submit cookie pattern) for state-changing requests (POST/PUT/DELETE) as a second layer, though with a well-configured `SameSite=Strict` cookie on a same-origin architecture, the marginal risk is low.

---

### Q4: What's the difference between `SameSite=Strict` and `SameSite=Lax`? Why choose one over the other?

**Answer:**
- **`Strict`** — cookie is never sent on cross-site requests, including top-level navigation (e.g., clicking a link from an external site to yours won't include the cookie on that first request). Maximum CSRF protection, but can break flows like "click an email link and land already logged in."
- **`Lax`** — cookie is sent on top-level navigations (GET requests, like clicking a link) but not on cross-site subrequests (like a form POST or `fetch` triggered from another site). This is the more common default balance between security and usability.
- Choice depends on the app: `Strict` for maximum security-sensitive apps (banking, admin portals), `Lax` when you need smoother cross-site navigation UX.

---

### Q5: What happens on logout — how do you invalidate an HttpOnly cookie you can't read from JS?

**Answer:**
Logout is a server-driven action, not a client-side one:
- The frontend calls a `/logout` endpoint.
- The server responds with a `Set-Cookie` header that overwrites the cookie with an empty value and an expiry in the past, instructing the browser to delete it.
- If using server-side sessions (not pure stateless JWT), the server also invalidates the session record in its store (Redis/DB) at the same time, so even if the cookie somehow persisted, it wouldn't correspond to a valid session anymore.

---

### Q6: Is this approach stateless or stateful? How does it relate to JWT?

**Answer:**
This is independent of the stateless-vs-stateful question — HttpOnly/Secure/SameSite are cookie **transport/storage security properties**, not an auth strategy by themselves. The actual token *inside* the cookie can be either:
- A **stateless JWT** — the cookie just becomes the transport mechanism for the JWT instead of a manual header; the server still verifies it via signature, no DB lookup needed.
- An **opaque session token** — the cookie holds a random session ID, and the server looks up session state (Redis/DB) to resolve it — this is fully stateful.

In practice, many systems combine both: a short-lived stateless JWT for fast validation, stored in an HttpOnly cookie for transport security, with a stateful refresh token (opaque, stored server-side) to allow revocation — since pure stateless JWTs can't be invalidated before their natural expiry without an extra stateful component (a blocklist/deny-list).

---

### Q7: If someone steals the HttpOnly cookie itself (not via XSS, but via physical device access or a MITM without HTTPS), is the user still compromised?

**Answer:**
Yes — HttpOnly protects against **script-based** theft (XSS), not all theft vectors. If an attacker has direct access to the browser's cookie storage (e.g., physical device access, a browser extension with broad permissions, or malware on the machine), HttpOnly doesn't help, since it's a browser-JS boundary, not a device-security boundary. `Secure` protects against network-level interception (MITM) by requiring HTTPS. Defense in depth here would include short token expiry, IP/device fingerprint binding for session validation, and the ability to revoke sessions server-side (which requires some stateful component, per Q6).

---

### Q8: Why not just keep the token in memory (a JS variable) instead of localStorage or cookies?

**Answer:**
In-memory storage (e.g., a React state variable, not persisted anywhere) is actually the most XSS-resistant option, since there's no persistent storage to read at all — but it has a major usability tradeoff: the token is lost on page refresh, forcing a full re-login or a refresh-token flow on every reload. HttpOnly cookies get most of the XSS protection benefit of in-memory storage while surviving page refreshes automatically (since the browser manages and resends the cookie), which is why they're the more common production choice for web apps that need persistence across reloads without constant re-authentication.

---

---

## Part 3: Code — How to Actually Configure This

### Backend (Spring Boot / Java)

**a) Setting the cookie when issuing a token (login endpoint)**

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request, HttpServletResponse response) {
    // Validate credentials, generate token (JWT or opaque session id)
    String token = authService.authenticate(request.getUsername(), request.getPassword());

    ResponseCookie cookie = ResponseCookie.from("access_token", token)
            .httpOnly(true)                 // JS cannot read this cookie
            .secure(true)                   // only sent over HTTPS
            .sameSite("Strict")             // or "Lax" depending on your CSRF/UX tradeoff
            .path("/")                      // cookie sent for all routes
            .maxAge(Duration.ofMinutes(15)) // short-lived access token
            .build();

    response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString());
    return ResponseEntity.ok(Map.of("message", "Login successful"));
}
```

> Note: `ResponseCookie` (Spring 5+) is preferred over the older `javax.servlet.http.Cookie` because it directly supports `SameSite`, which the older API doesn't expose natively.

**b) Reading the cookie on incoming requests (auth filter)**

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain filterChain) throws ServletException, IOException {

        String token = extractTokenFromCookie(request, "access_token");

        if (token != null && jwtUtil.isValid(token)) {
            Authentication auth = jwtUtil.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }

        filterChain.doFilter(request, response);
    }

    private String extractTokenFromCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) return null;
        return Arrays.stream(request.getCookies())
                .filter(c -> cookieName.equals(c.getName()))
                .map(Cookie::getValue)
                .findFirst()
                .orElse(null);
    }
}
```

**c) CORS configuration — required for cookies to work cross-origin**

This is the step people most often miss: with cookie-based auth, `Access-Control-Allow-Origin` **cannot** be `*` — it must be an explicit origin, and `Access-Control-Allow-Credentials` must be `true`.

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("https://app.example.com")  // explicit origin, not "*"
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowCredentials(true)                      // required to send/receive cookies
                .allowedHeaders("*");
    }
}
```

**d) Logout — overwrite the cookie with an expired one**

```java
@PostMapping("/logout")
public ResponseEntity<?> logout(HttpServletResponse response) {
    ResponseCookie cookie = ResponseCookie.from("access_token", "")
            .httpOnly(true)
            .secure(true)
            .sameSite("Strict")
            .path("/")
            .maxAge(0)   // expires immediately
            .build();

    response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString());
    // If using server-side sessions/opaque tokens, also invalidate the session record here (Redis/DB)
    return ResponseEntity.ok(Map.of("message", "Logged out"));
}
```

---

### Frontend (React)

**a) Sending requests with cookies — the critical setting is `credentials`**

By default, browsers **don't** send cookies on cross-origin requests unless explicitly told to. This is the #1 thing people forget when migrating from localStorage (where you manually attached the header) to cookies (where the browser should handle it automatically, but only if configured).

**Using `fetch`:**
```javascript
fetch("https://api.example.com/user/profile", {
  method: "GET",
  credentials: "include",   // tells the browser to send the cookie cross-origin
});
```

**Using `axios`:**
```javascript
// Global config, once, e.g. in an api client setup file
axios.defaults.withCredentials = true;

// or per-request
axios.get("https://api.example.com/user/profile", {
  withCredentials: true,
});
```

**b) Login flow — no manual token handling needed**

```javascript
async function login(username, password) {
  const response = await fetch("https://api.example.com/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    credentials: "include",   // browser stores the Set-Cookie response automatically
    body: JSON.stringify({ username, password }),
  });

  if (!response.ok) throw new Error("Login failed");
  // No token to store manually — the cookie is already set by the browser.
  // Just update app state to reflect "logged in".
}
```

**c) Checking auth state on app load (since JS can't read the cookie)**

```javascript
async function checkAuth() {
  const response = await fetch("https://api.example.com/me", {
    method: "GET",
    credentials: "include",
  });

  if (response.ok) {
    const user = await response.json();
    return user;   // logged in
  }
  return null;      // not logged in / session expired
}

// Typically called once in a top-level useEffect on app mount
useEffect(() => {
  checkAuth().then(user => setUser(user));
}, []);
```

**d) Handling 401s with silent refresh (axios interceptor example)**

```javascript
axios.interceptors.response.use(
  response => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        await axios.post("https://api.example.com/refresh", {}, { withCredentials: true });
        return axios(originalRequest);   // retry original request after refresh
      } catch (refreshError) {
        // refresh failed too — force logout / redirect to login
        window.location.href = "/login";
      }
    }
    return Promise.reject(error);
  }
);
```

**e) Logout**

```javascript
async function logout() {
  await fetch("https://api.example.com/logout", {
    method: "POST",
    credentials: "include",   // needed so the server can identify + clear the right session too
  });
  // update app state — the server already cleared the cookie via Set-Cookie
}
```

---

### Common Gotchas When Implementing This

| Issue | Cause | Fix |
|---|---|---|
| Cookie never gets set in the browser | Missing `credentials: "include"` on the frontend request | Add it to every request that involves auth |
| CORS error: "cannot use wildcard with credentials" | Backend has `allowedOrigins("*")` with `allowCredentials(true)` | Use an explicit origin, never `*`, when credentials are involved |
| Cookie set but not sent back on next request | `SameSite=Strict` blocking it during cross-site navigation, or mismatched `domain`/`path` | Confirm the request is same-site, or relax to `Lax` if the flow requires cross-site navigation |
| Works on `localhost` but not in production | Missing `Secure` flag combined with testing on plain HTTP locally, or domain mismatch between frontend/backend in prod | Use HTTPS in all non-local environments; align cookie `domain` with actual API domain |
| Frontend can't check "is user logged in" instantly on load | JS genuinely can't read HttpOnly cookies — this is by design | Use a `/me` endpoint call on app load, as shown above, instead of trying to read the cookie |

---

## Quick-Reference Table

| Flag/Mechanism | Protects Against | Does NOT Protect Against |
|---|---|---|
| `HttpOnly` | JS reading the cookie (XSS-based token theft) | XSS itself occurring; cookie theft via device/network access |
| `Secure` | Network interception over unencrypted HTTP | Anything once traffic is already HTTPS-decrypted (e.g., XSS) |
| `SameSite=Strict`/`Lax` | CSRF (cross-origin request forgery using the cookie) | XSS; same-site attacks |
| Server-side session store (stateful) | Immediate revocation on logout/compromise | Nothing extra on its own — needs to be paired with the above |
