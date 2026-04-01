security-notes-cors-csrf-xss.md
# CORS, CSRF, and XSS - Quick Notes

## 1) CORS (Cross-Origin Resource Sharing)

### What it is
- A browser security mechanism that controls whether frontend code from one origin can call APIs on another origin.
- Origin = `scheme + host + port` (for example, `https://app.example.com:443`).

### Why it exists
- By default, browsers block many cross-origin requests from JavaScript.
- Server must explicitly allow trusted origins.

### Typical safe setup
- Allow only known frontend origins (no `*` in production with credentials).
- Allow only needed methods and headers.
- If cookies are used cross-site:
- `Access-Control-Allow-Credentials: true`
- exact allowed origin (not wildcard).

### Example headers
```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

---

## 2) CSRF (Cross-Site Request Forgery)

### What it is
- An attacker tricks a logged-in user’s browser into sending unwanted requests to a trusted site.
- Browser automatically includes cookies, so server may think request is valid.

### Simple flow
1. User logs in to `bank.com` (session cookie stored).
2. User visits `evil.com`.
3. `evil.com` submits a hidden request to `bank.com/transfer`.
4. Browser sends `bank.com` cookie automatically.
5. If no CSRF protection, action may succeed.

### Important point
- `HttpOnly` protects cookie reading by JS, but does NOT stop CSRF by itself.

### Prevention
- CSRF tokens (synchronizer token or double-submit cookie pattern).
- `SameSite` cookies (`Lax`/`Strict` where possible).
- Verify `Origin`/`Referer` for state-changing requests.
- Use custom headers for API mutations.

---

## 3) XSS (Cross-Site Scripting)

### What it is
- Injection of malicious JavaScript into a trusted page.
- Script runs in victim’s browser under your site’s origin.

### Common types
- Stored XSS: payload saved in DB and shown to many users.
- Reflected XSS: payload comes from request and is reflected in response.
- DOM XSS: client-side JS inserts unsafe data into DOM.

### Unsafe example
```js
result.innerHTML = "Hello " + userInput; // unsafe
```

### Safe approach
```js
result.textContent = "Hello " + userInput; // safer
```

### Prevention
- Output encode/escape user input.
- Avoid unsafe HTML sinks (`innerHTML`) unless sanitized.
- Validate/sanitize inputs server-side.
- Use Content Security Policy (CSP).
- Keep dependencies updated.

---

## Quick Difference

- CORS: browser rule for cross-origin API access.
- CSRF: attacker forces victim browser to send authenticated request.
- XSS: attacker injects JS that runs in victim browser.

---

## Fast Interview Lines

- CORS is a controlled cross-origin access mechanism, not an attack.
- CSRF abuses automatic cookie sending; stop it with CSRF token + SameSite + origin checks.
- XSS is script injection; stop it with escaping/sanitization/CSP and safe DOM APIs.
