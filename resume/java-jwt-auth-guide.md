# Java JWT Authentication & Authorization — Complete Implementation Guide

This guide covers a production-grade authentication flow in Java: password hashing with BCrypt, login API, JWT access token generation/validation, and refresh token handling with rotation.

> **Why BCrypt, if asked in an interview**: BCrypt is a deliberately slow, adaptive hashing algorithm purpose-built for passwords. Unlike general-purpose hashes (SHA-256, MD5), which are optimized to be *fast* — a property that helps attackers brute-force stolen hashes cheaply on GPUs — BCrypt has a tunable **cost factor** that makes each hash computation expensive on purpose (~100ms+ per hash at cost factor 12). It also generates and embeds the salt automatically, so there's no separate salt column to manage or accidentally mishandle. This is the same category of algorithm (adaptive, memory/CPU-hard) as Argon2 and PBKDF2 — all three are acceptable answers; BCrypt is simply the most widely adopted and has first-class support in Spring Security.

---

## Table of Contents
1. [Dependencies](#1-dependencies)
2. [Password Hashing (BCrypt)](#2-password-hashing-bcrypt)
3. [User Registration API](#3-user-registration-api)
4. [Login API — Verifying Password Hash](#4-login-api--verifying-password-hash)
5. [JWT Utility — Generate & Validate Tokens](#5-jwt-utility--generate--validate-tokens)
6. [Refresh Token Flow](#6-refresh-token-flow)
7. [JWT Authentication Filter (Spring Security)](#7-jwt-authentication-filter-spring-security)
8. [Full Request Flow Summary](#8-full-request-flow-summary)

---

## 1. Dependencies

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- JWT library (jjwt) -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.5</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

---

## 2. Password Hashing (BCrypt)

Spring Security ships `BCryptPasswordEncoder` out of the box — it **automatically generates and embeds a random salt internally**, so there's no separate salt column to manage or risk mishandling.

```java
package com.flashsale.security;

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Component;

@Component
public class PasswordHasherBCrypt {

    // Strength/cost factor: higher = slower = more secure but more CPU.
    // 10-12 is the common production range.
    private final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12);

    public String hashPassword(String plainPassword) {
        // Salt is generated internally and embedded in the returned hash string
        // e.g: $2a$12$eImiTXuWVxfM37uY4JANjQ==...
        return encoder.encode(plainPassword);
    }

    public boolean verifyPassword(String plainPassword, String storedHash) {
        // BCrypt extracts the salt from storedHash automatically
        return encoder.matches(plainPassword, storedHash);
    }
}
```

**DB schema — only ONE column needed (no separate salt column):**

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,  -- BCrypt hash (salt embedded)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 3. User Registration API

```java
package com.flashsale.controller;

import com.flashsale.dto.RegisterRequest;
import com.flashsale.entity.User;
import com.flashsale.repository.UserRepository;
import com.flashsale.security.PasswordHasherBCrypt;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordHasherBCrypt passwordHasher;

    @PostMapping("/register")
    public ResponseEntity<?> register(@RequestBody RegisterRequest request) {
        if (userRepository.findByEmail(request.getEmail()).isPresent()) {
            return ResponseEntity.badRequest().body("Email already registered");
        }

        String hashedPassword = passwordHasher.hashPassword(request.getPassword());

        User user = new User();
        user.setEmail(request.getEmail());
        user.setPasswordHash(hashedPassword); // never store plaintext
        userRepository.save(user);

        return ResponseEntity.ok("User registered successfully");
    }
}
```

---

## 4. Login API — Verifying Password Hash

```java
package com.flashsale.controller;

import com.flashsale.dto.LoginRequest;
import com.flashsale.dto.LoginResponse;
import com.flashsale.entity.User;
import com.flashsale.entity.RefreshToken;
import com.flashsale.repository.UserRepository;
import com.flashsale.security.JwtUtil;
import com.flashsale.security.PasswordHasherBCrypt;
import com.flashsale.service.RefreshTokenService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Optional;

@RestController
@RequestMapping("/api/auth")
public class LoginController {

    @Autowired private UserRepository userRepository;
    @Autowired private PasswordHasherBCrypt passwordHasher;
    @Autowired private JwtUtil jwtUtil;
    @Autowired private RefreshTokenService refreshTokenService;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {

        Optional<User> userOpt = userRepository.findByEmail(request.getEmail());

        // Use a generic error message for both "user not found" and
        // "wrong password" — being specific here leaks which emails are registered.
        if (userOpt.isEmpty()) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid email or password");
        }

        User user = userOpt.get();

        boolean passwordMatches = passwordHasher.verifyPassword(
                request.getPassword(),
                user.getPasswordHash()
        );

        if (!passwordMatches) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid email or password");
        }

        // Generate short-lived access token
        String accessToken = jwtUtil.generateAccessToken(user.getEmail(), user.getId());

        // Generate long-lived refresh token, persist it server-side
        RefreshToken refreshToken = refreshTokenService.createRefreshToken(user.getId());

        LoginResponse response = new LoginResponse(
                accessToken,
                refreshToken.getToken(),
                "Bearer",
                jwtUtil.getAccessTokenExpiryMs()
        );

        return ResponseEntity.ok(response);
    }
}
```

---

## 5. JWT Utility — Generate & Validate Tokens

```java
package com.flashsale.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.util.Date;

@Component
public class JwtUtil {

    // In production, load this from a secrets manager (AWS Secrets Manager,
    // Vault, etc.) — NEVER hardcode or commit the secret key.
    @Value("${jwt.secret}")
    private String secretKeyString;

    // Access tokens should be SHORT-lived: 15 minutes is typical
    private final long ACCESS_TOKEN_EXPIRY_MS = 15 * 60 * 1000;

    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(secretKeyString.getBytes());
    }

    public long getAccessTokenExpiryMs() {
        return ACCESS_TOKEN_EXPIRY_MS;
    }

    /**
     * Generates a signed JWT access token containing user identity claims.
     */
    public String generateAccessToken(String email, Long userId) {
        Date now = new Date();
        Date expiry = new Date(now.getTime() + ACCESS_TOKEN_EXPIRY_MS);

        return Jwts.builder()
                .subject(email)
                .claim("userId", userId)
                .claim("type", "access")
                .issuedAt(now)
                .expiration(expiry)
                .signWith(getSigningKey())
                .compact();
    }

    /**
     * Validates a JWT's signature and expiry.
     * Throws an exception if invalid/expired/tampered.
     */
    public Claims validateToken(String token) {
        try {
            return Jwts.parser()
                    .verifyWith(getSigningKey())
                    .build()
                    .parseSignedClaims(token)
                    .getPayload();
        } catch (ExpiredJwtException e) {
            throw new RuntimeException("Token expired", e);
        } catch (SignatureException e) {
            throw new RuntimeException("Invalid token signature", e);
        } catch (JwtException e) {
            throw new RuntimeException("Invalid token", e);
        }
    }

    public String extractEmail(String token) {
        return validateToken(token).getSubject();
    }

    public Long extractUserId(String token) {
        return validateToken(token).get("userId", Long.class);
    }

    public boolean isTokenExpired(String token) {
        try {
            Date expiration = validateToken(token).getExpiration();
            return expiration.before(new Date());
        } catch (Exception e) {
            return true;
        }
    }
}
```

---

## 6. Refresh Token Flow

**Why refresh tokens exist:** Access tokens are short-lived (15 min) so that if one is stolen, the exposure window is small. But you don't want users re-entering their password every 15 minutes. The refresh token is long-lived (days/weeks), stored server-side, and used **only** to mint new access tokens — it's never sent on regular API calls.

### 7.1 Refresh Token Entity

```java
package com.flashsale.entity;

import jakarta.persistence.*;
import java.time.Instant;

@Entity
@Table(name = "refresh_tokens")
public class RefreshToken {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String token; // random opaque string (UUID), not a JWT

    @Column(name = "user_id", nullable = false)
    private Long userId;

    @Column(nullable = false)
    private Instant expiryDate;

    @Column(nullable = false)
    private boolean revoked = false;

    // getters and setters omitted for brevity
}
```

### 7.2 Refresh Token Service

```java
package com.flashsale.service;

import com.flashsale.entity.RefreshToken;
import com.flashsale.repository.RefreshTokenRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.Instant;
import java.util.Optional;
import java.util.UUID;

@Service
public class RefreshTokenService {

    @Autowired
    private RefreshTokenRepository refreshTokenRepository;

    // Refresh tokens live much longer than access tokens: e.g. 7 days
    private final long REFRESH_TOKEN_DURATION_MS = 7L * 24 * 60 * 60 * 1000;

    public RefreshToken createRefreshToken(Long userId) {
        // Optional: revoke old tokens for this user to enforce single active session,
        // or skip this to allow multiple concurrent device sessions.
        // refreshTokenRepository.revokeAllByUserId(userId);

        RefreshToken refreshToken = new RefreshToken();
        refreshToken.setUserId(userId);
        refreshToken.setToken(UUID.randomUUID().toString());
        refreshToken.setExpiryDate(Instant.now().plusMillis(REFRESH_TOKEN_DURATION_MS));
        refreshToken.setRevoked(false);

        return refreshTokenRepository.save(refreshToken);
    }

    public Optional<RefreshToken> verifyAndGet(String token) {
        Optional<RefreshToken> tokenOpt = refreshTokenRepository.findByToken(token);

        if (tokenOpt.isEmpty()) return Optional.empty();

        RefreshToken refreshToken = tokenOpt.get();

        if (refreshToken.isRevoked() || refreshToken.getExpiryDate().isBefore(Instant.now())) {
            return Optional.empty();
        }

        return Optional.of(refreshToken);
    }

    public void revokeToken(String token) {
        refreshTokenRepository.findByToken(token)
                .ifPresent(rt -> {
                    rt.setRevoked(true);
                    refreshTokenRepository.save(rt);
                });
    }
}
```

### 7.3 Refresh Token API Endpoint

```java
package com.flashsale.controller;

import com.flashsale.dto.RefreshTokenRequest;
import com.flashsale.dto.LoginResponse;
import com.flashsale.entity.RefreshToken;
import com.flashsale.entity.User;
import com.flashsale.repository.UserRepository;
import com.flashsale.security.JwtUtil;
import com.flashsale.service.RefreshTokenService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Optional;

@RestController
@RequestMapping("/api/auth")
public class RefreshTokenController {

    @Autowired private RefreshTokenService refreshTokenService;
    @Autowired private UserRepository userRepository;
    @Autowired private JwtUtil jwtUtil;

    @PostMapping("/refresh")
    public ResponseEntity<?> refreshAccessToken(@RequestBody RefreshTokenRequest request) {

        Optional<RefreshToken> tokenOpt = refreshTokenService.verifyAndGet(request.getRefreshToken());

        if (tokenOpt.isEmpty()) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body("Invalid or expired refresh token, please log in again");
        }

        RefreshToken refreshToken = tokenOpt.get();
        User user = userRepository.findById(refreshToken.getUserId())
                .orElseThrow(() -> new RuntimeException("User not found"));

        // Issue a brand new access token
        String newAccessToken = jwtUtil.generateAccessToken(user.getEmail(), user.getId());

        // Rotation pattern (recommended): revoke old refresh token,
        // issue a brand new one each time — limits damage if a refresh
        // token is ever stolen (it becomes single-use).
        refreshTokenService.revokeToken(refreshToken.getToken());
        RefreshToken newRefreshToken = refreshTokenService.createRefreshToken(user.getId());

        LoginResponse response = new LoginResponse(
                newAccessToken,
                newRefreshToken.getToken(),
                "Bearer",
                jwtUtil.getAccessTokenExpiryMs()
        );

        return ResponseEntity.ok(response);
    }

    @PostMapping("/logout")
    public ResponseEntity<?> logout(@RequestBody RefreshTokenRequest request) {
        refreshTokenService.revokeToken(request.getRefreshToken());
        return ResponseEntity.ok("Logged out successfully");
    }
}
```

---

## 7. JWT Authentication Filter (Spring Security)

This filter intercepts every incoming request, extracts the JWT from the `Authorization` header, validates it, and sets the authenticated user in the Spring Security context — so your controllers can trust `@AuthenticationPrincipal` / `SecurityContextHolder`.

```java
package com.flashsale.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.Collections;

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUtil jwtUtil;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7); // strip "Bearer "

        try {
            if (!jwtUtil.isTokenExpired(token)) {
                String email = jwtUtil.extractEmail(token);
                Long userId = jwtUtil.extractUserId(token);

                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(
                                email, null, Collections.emptyList()
                        );
                authentication.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            // Invalid/expired token — let the request continue unauthenticated;
            // downstream security config will reject it with 401/403 as appropriate.
            SecurityContextHolder.clearContext();
        }

        filterChain.doFilter(request, response);
    }
}
```

**Registering the filter in Spring Security config:**

```java
package com.flashsale.config;

import com.flashsale.security.JwtAuthenticationFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // stateless JWT APIs typically disable CSRF
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll() // login/register/refresh are public
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

---

## 8. Full Request Flow Summary

```
┌──────────────┐  1. POST /api/auth/register (email, password)
│   Client     │─────────────────────────────────────────────►
└──────────────┘                                          ┌─────────────┐
                                                            │ AuthController│
                                                            │ hash password │
                                                            │ (BCrypt)      │
                                                            └──────┬───────┘
                                                                   ▼
                                                            ┌─────────────┐
                                                            │   users DB   │
                                                            │ (hash stored)│
                                                            └─────────────┘

┌──────────────┐  2. POST /api/auth/login (email, password)
│   Client     │─────────────────────────────────────────────►
└──────────────┘                                          ┌─────────────────┐
                                                            │ verify password  │
                                                            │ (BCrypt.matches) │
                                                            └────────┬────────┘
                                                                     ▼ valid
                                                    ┌────────────────────────────┐
                                                    │ generate:                  │
                                                    │  - Access Token (JWT, 15m) │
                                                    │  - Refresh Token (UUID,7d) │
                                                    │  save refresh token in DB  │
                                                    └────────────┬───────────────┘
◄───────────────────────────────────────────────────────────────┘
    { accessToken, refreshToken }

┌──────────────┐  3. GET /api/orders  (Authorization: Bearer <accessToken>)
│   Client     │─────────────────────────────────────────────►
└──────────────┘                                          ┌─────────────────────┐
                                                            │ JwtAuthenticationFilter│
                                                            │ validate signature +  │
                                                            │ expiry                │
                                                            └──────────┬───────────┘
                                                                       ▼ valid
                                                              request proceeds
                                                              to OrderController

┌──────────────┐  4. Access token expires after 15 min
│   Client     │  POST /api/auth/refresh { refreshToken }
└──────────────┘─────────────────────────────────────────────►
                                                    ┌─────────────────────────┐
                                                    │ verify refresh token in DB│
                                                    │ (not expired, not revoked)│
                                                    │ issue NEW access token +  │
                                                    │ NEW refresh token (rotate)│
                                                    │ revoke OLD refresh token  │
                                                    └────────────┬──────────────┘
◄───────────────────────────────────────────────────────────────┘
    { new accessToken, new refreshToken }

┌──────────────┐  5. POST /api/auth/logout { refreshToken }
│   Client     │─────────────────────────────────────────────►
└──────────────┘                                          revoke refresh token in DB
```

## Key Security Practices Recap — Interview Talking Points

| Design decision | Why (the "why not X" answer) |
|---|---|
| **BCrypt for password hashing** | Adaptive/slow by design (tunable cost factor) — resists GPU brute-force in a way fast general-purpose hashes (SHA, MD5) don't. Salt is generated and embedded automatically, so there's no separate salt column to manage. |
| **Access token = JWT, short-lived (~15 min)** | Stateless — any service can validate it locally via signature, no DB round-trip, so it scales horizontally without a shared session store. Short expiry limits the damage window if one is ever stolen. |
| **Refresh token = opaque UUID, NOT a JWT** | Needs to be **revocable** server-side (e.g., on logout, or if compromised). A stateless JWT can't be revoked before its expiry without extra infrastructure (a blocklist), so the refresh token is intentionally stored in the DB instead. |
| **Refresh token rotation** (new refresh token issued on every refresh call, old one revoked) | Makes stolen refresh tokens single-use — if an attacker replays a stolen refresh token after the legitimate user has already rotated it, the old token is already revoked and the theft is detectable. |
| **Stateless session management (`SessionCreationPolicy.STATELESS`)** | Avoids server-side session storage entirely, which is what lets the auth layer scale horizontally behind a load balancer without sticky sessions. |
| **Generic error message on login failure** | Prevents user enumeration — a different message for "wrong password" vs "no such user" lets an attacker discover which emails are registered. |
| **JWT signing secret in a secrets manager, not source code** | Standard secret-management hygiene — a leaked signing key lets an attacker forge valid tokens for any user. |

This structure is deliberately **not** "here are two options, pick one" — it's a single, defensible design where every choice has a specific failure mode it's protecting against. If asked "why not store the JWT server-side too, for revocability," the answer is: that defeats the point of a stateless access token — you'd be back to a DB lookup on every request, which is exactly what the refresh-token split avoids.
