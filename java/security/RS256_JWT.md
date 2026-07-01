# RS256 JWT Signing & Verification

## Key Rule

-   **Private Key** → **Sign (Create) the JWT**
-   **Public Key** → **Verify the JWT Signature**

> **Remember:** **Private signs, Public verifies.**

------------------------------------------------------------------------

## Flow

``` text
Authentication Server
      (Private Key)
           │
           │ Sign JWT
           ▼
      JWT + Signature
           │
   -----------------------
   │         │          │
   ▼         ▼          ▼
Service A  Service B  Service C
(Public)   (Public)   (Public)
   │         │          │
   └── Verify JWT Signature ──┘
```

------------------------------------------------------------------------

## Why RS256?

-   Only the **Authentication Server** has the **Private Key**.
-   All backend services receive the **Public Key**.
-   Any service can **verify** the JWT.
-   No service can **create or modify** a valid JWT because they do
    **not** have the private key.

This makes RS256 ideal for **microservice architectures**.

------------------------------------------------------------------------

## HS256 vs RS256

  Feature        HS256                RS256
  -------------- -------------------- ---------------
  Signing        Secret Key           Private Key
  Verification   Same Secret Key      Public Key
  Key Type       Symmetric            Asymmetric
  Best For       Single application   Microservices

------------------------------------------------------------------------

## Interview Answer

> "In RS256, the Authentication Server signs the JWT using its **Private
> Key**. The client sends the JWT with every request. Each backend
> service verifies the JWT using the corresponding **Public Key**. Since
> only the Authentication Server owns the Private Key, backend services
> can validate tokens but cannot generate or tamper with them, making
> the system more secure."

------------------------------------------------------------------------

## Common Mistake

❌ JWT Creation → Public Key

❌ JWT Verification → Private Key

✅ JWT Creation → **Private Key**

✅ JWT Verification → **Public Key**
