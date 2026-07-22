# FastAPI Interview Preparation Guide

A comprehensive Q&A reference covering FastAPI fundamentals, async, dependency injection, Pydantic, security, testing, performance, and real-world system design — useful for backend and AI Engineer roles (since FastAPI is the dominant framework for serving LLM/agent APIs).

---

## Table of Contents
1. [FastAPI Fundamentals](#1-fastapi-fundamentals)
2. [Path Operations & Request Handling](#2-path-operations--request-handling)
3. [Pydantic & Data Validation](#3-pydantic--data-validation)
4. [Dependency Injection](#4-dependency-injection)
5. [Async & Concurrency](#5-async--concurrency)
6. [Middleware, CORS & Error Handling](#6-middleware-cors--error-handling)
7. [Authentication & Security](#7-authentication--security)
8. [Background Tasks & Streaming](#8-background-tasks--streaming)
9. [Database Integration](#9-database-integration)
10. [Testing](#10-testing)
11. [Performance & Deployment](#11-performance--deployment)
12. [FastAPI for AI/LLM Applications (Specific)](#12-fastapi-for-aillm-applications-specific)
13. [Coding Practice Questions](#13-coding-practice-questions)

---

## 1. FastAPI Fundamentals

**Q1: What is FastAPI, and why has it become popular for building AI/LLM backends?**
A: FastAPI is a modern Python web framework built on top of **Starlette** (for the web/ASGI layer) and **Pydantic** (for data validation). It's popular for AI backends because: it's natively async (critical for concurrent LLM API calls), it auto-generates interactive API docs (Swagger/OpenAPI), it has built-in request/response validation via type hints, and it's fast enough to handle high-throughput streaming responses (like token-by-token LLM output).

**Q2: What is the difference between FastAPI and Flask?**
A: Flask is a WSGI (synchronous) framework by default; FastAPI is ASGI (asynchronous) native, meaning it can handle concurrent I/O-bound requests (like calling an LLM API) far more efficiently without blocking. FastAPI also has built-in data validation via Pydantic and automatic OpenAPI docs generation, whereas Flask requires extensions (e.g., Flask-RESTful, Marshmallow) to achieve similar functionality.

**Q3: What is ASGI, and how does it differ from WSGI?**
A: **WSGI** (Web Server Gateway Interface) is a synchronous spec — one request is handled at a time per worker, blocking on I/O. **ASGI** (Asynchronous Server Gateway Interface) supports async request handling, WebSockets, and long-lived connections, allowing a single worker to handle many concurrent requests by yielding control during I/O waits (e.g., while waiting on an LLM API response).

**Q4: What server do you run FastAPI with, and why?**
A: FastAPI apps are run with an ASGI server like **Uvicorn** (often with **Gunicorn** as a process manager in production, using `uvicorn.workers.UvicornWorker`). Uvicorn implements the ASGI protocol and handles the event loop that enables FastAPI's async capabilities.

**Q5: How does FastAPI generate automatic API documentation?**
A: FastAPI inspects your path operation function signatures, type hints, and Pydantic models to auto-generate an OpenAPI schema, which powers two built-in UIs: **Swagger UI** (`/docs`) and **ReDoc** (`/redoc`) — no extra annotation work required beyond normal type-hinted code.

---

## 2. Path Operations & Request Handling

**Q6: Write a basic FastAPI app with a GET and POST endpoint.**
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}

@app.post("/items/")
async def create_item(item: Item):
    return {"name": item.name, "price": item.price}
```

**Q7: What's the difference between path parameters, query parameters, and the request body?**
A: **Path parameters** (`/items/{item_id}`) are part of the URL and required by definition. **Query parameters** (`?q=value`) come after `?` in the URL and are typically optional. **Request body** is sent as JSON in the HTTP body, typically parsed into a Pydantic model — used for POST/PUT/PATCH when sending structured data.

**Q8: How do you set default values and mark parameters as optional?**
```python
from fastapi import Query

@app.get("/items/")
async def list_items(skip: int = 0, limit: int = 10, q: str | None = Query(default=None, max_length=50)):
    return {"skip": skip, "limit": limit, "q": q}
```

**Q9: How do you handle file uploads in FastAPI?**
```python
from fastapi import File, UploadFile

@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    contents = await file.read()
    return {"filename": file.filename, "size": len(contents)}
```

**Q10: What is the difference between `UploadFile` and `bytes` for file uploads?**
A: `bytes` loads the entire file into memory at once — fine for small files. `UploadFile` uses a spooled temporary file (memory up to a threshold, then disk), and supports async read/write, making it much better for large file uploads (e.g., uploading documents for a RAG ingestion pipeline).

---

## 3. Pydantic & Data Validation

**Q11: What is Pydantic, and how does FastAPI use it?**
A: Pydantic is a data validation library that uses Python type hints to validate, serialize, and document data structures at runtime. FastAPI uses Pydantic models to automatically validate incoming request bodies, serialize response data, and generate the OpenAPI schema — if incoming data doesn't match the model, FastAPI automatically returns a 422 error with details.

**Q12: How do you add custom validation to a Pydantic model?**
```python
from pydantic import BaseModel, field_validator

class UserSignup(BaseModel):
    username: str
    age: int

    @field_validator("age")
    @classmethod
    def check_age(cls, v):
        if v < 18:
            raise ValueError("User must be 18 or older")
        return v
```

**Q13: What's the difference between Pydantic v1 and v2 in terms of validators?**
A: Pydantic v2 renamed `@validator` to `@field_validator` and rewrote the core validation engine in Rust (via `pydantic-core`) for a significant performance boost. v2 also introduces `model_config` (replacing the old `class Config`) and `model_dump()` (replacing `.dict()`).

**Q14: How do you create nested Pydantic models?**
```python
class Address(BaseModel):
    city: str
    zip_code: str

class User(BaseModel):
    name: str
    address: Address  # nested model
```

**Q15: How do you make a field optional vs. required with a default in Pydantic?**
```python
from pydantic import BaseModel

class Config(BaseModel):
    temperature: float = 0.7          # optional, has default
    max_tokens: int                   # required, no default
    system_prompt: str | None = None  # optional, defaults to None
```

**Q16: What is `model_dump()` used for, and how do you exclude fields from serialization?**
A: `model_dump()` converts a Pydantic model instance to a Python dict. You can exclude sensitive fields (e.g., API keys) using `model.model_dump(exclude={"api_key"})`, or exclude unset fields with `exclude_unset=True` — useful for PATCH-style partial updates.

---

## 4. Dependency Injection

**Q17: What is FastAPI's dependency injection system, and why is it useful?**
A: FastAPI lets you declare reusable "dependencies" (functions or classes) that are automatically called and injected into path operations — useful for shared logic like authentication, DB session management, or rate limiting, without duplicating code across endpoints.
```python
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
async def list_users(db=Depends(get_db)):
    return db.query(User).all()
```

**Q18: What's the difference between a dependency with `return` vs. `yield`?**
A: A dependency using `return` provides a value and finishes immediately. A dependency using `yield` supports setup/teardown logic (like a context manager) — code after `yield` runs after the request completes, useful for closing DB connections or releasing resources even if the request raised an exception.

**Q19: How do you use dependencies for authentication across multiple endpoints?**
```python
from fastapi import Depends, HTTPException, Header

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != "expected-secret":
        raise HTTPException(status_code=401, detail="Invalid API key")
    return x_api_key

@app.get("/protected/")
async def protected_route(api_key: str = Depends(verify_api_key)):
    return {"message": "authenticated"}
```

**Q20: Can dependencies depend on other dependencies?**
A: Yes — dependencies can be chained/nested. FastAPI resolves the full dependency tree and caches each dependency's result **per request** by default (so if two different path parameters depend on the same sub-dependency, it's only computed once per request unless `use_cache=False` is set).

**Q21: How do you apply a dependency to an entire router or app, rather than one endpoint at a time?**
```python
from fastapi import APIRouter, Depends

router = APIRouter(dependencies=[Depends(verify_api_key)])

# or, for the whole app:
app = FastAPI(dependencies=[Depends(verify_api_key)])
```

---

## 5. Async & Concurrency

**Q22: When should a path operation function be `async def` vs. plain `def`?**
A: Use `async def` when the function performs non-blocking I/O (e.g., calling an LLM API with an async HTTP client, an async DB driver). Use plain `def` if the function does blocking/CPU-bound work (e.g., calling a synchronous library) — FastAPI automatically runs `def` functions in a separate thread pool so they don't block the event loop.

**Q23: What happens if you call a blocking (synchronous) function inside an `async def` endpoint?**
A: It blocks the entire event loop, stalling all other concurrent requests being handled by that worker — a very common performance bug. Fix: either make the call async (use an async client/driver), or wrap the blocking call with `run_in_threadpool` / `asyncio.to_thread`.
```python
from fastapi.concurrency import run_in_threadpool

@app.get("/data/")
async def get_data():
    result = await run_in_threadpool(blocking_function)
    return result
```

**Q24: How would you call multiple LLM APIs concurrently inside one endpoint?**
```python
import asyncio
import httpx

@app.post("/multi-model-compare/")
async def compare_models(prompt: str):
    async with httpx.AsyncClient() as client:
        tasks = [
            client.post(OPENAI_URL, json={"prompt": prompt}),
            client.post(ANTHROPIC_URL, json={"prompt": prompt}),
        ]
        responses = await asyncio.gather(*tasks)
    return [r.json() for r in responses]
```

**Q25: What is the GIL's impact on a FastAPI app, and how does async help despite it?**
A: The GIL still limits true CPU-bound parallelism within a single process. But async doesn't rely on multiple CPU cores for I/O-bound work — it relies on the event loop switching between tasks while waiting on I/O (like network calls), so many concurrent LLM API calls can be "in flight" at once even on a single core, since most of the wait time is the API round-trip, not CPU work.

---

## 6. Middleware, CORS & Error Handling

**Q26: What is middleware in FastAPI, and how do you add custom logging middleware?**
```python
import time
from fastapi import Request

@app.middleware("http")
async def log_requests(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    duration = time.time() - start
    print(f"{request.method} {request.url.path} took {duration:.3f}s")
    return response
```

**Q27: How do you enable CORS in FastAPI?**
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://myfrontend.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Q28: How do you implement custom exception handling?**
```python
from fastapi import Request
from fastapi.responses import JSONResponse

class LLMTimeoutError(Exception):
    pass

@app.exception_handler(LLMTimeoutError)
async def llm_timeout_handler(request: Request, exc: LLMTimeoutError):
    return JSONResponse(
        status_code=504,
        content={"detail": "The model took too long to respond. Please try again."},
    )
```

**Q29: How does FastAPI handle validation errors by default, and how do you customize them?**
A: By default, invalid request data raises a `RequestValidationError` and returns HTTP 422 with details on which field failed. You can override this with a custom handler:
```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(status_code=422, content={"error": "Invalid input", "details": exc.errors()})
```

---

## 7. Authentication & Security

**Q30: What are the common authentication methods FastAPI supports?**
A: API keys via headers, **OAuth2 with JWT bearer tokens** (FastAPI has built-in `OAuth2PasswordBearer` support), Basic Auth, and session cookies. Most production LLM API wrappers use API-key-based auth or JWT.

**Q31: Show a basic OAuth2 + JWT authentication flow in FastAPI.**
```python
from fastapi import Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer
import jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
SECRET_KEY = "your-secret-key"

async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload.get("sub")
    except jwt.PyJWTError:
        raise HTTPException(status_code=401, detail="Invalid credentials")

@app.get("/me")
async def read_current_user(user=Depends(get_current_user)):
    return {"user": user}
```

**Q32: How do you rate-limit an endpoint in FastAPI (important for LLM APIs to control cost)?**
A: Common approach: use `slowapi` (a FastAPI-compatible wrapper around `limits`) or implement a custom dependency using Redis to track request counts per API key/IP within a time window.
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.get("/generate")
@limiter.limit("5/minute")
async def generate(request: Request):
    ...
```

**Q33: How would you securely store and use an LLM provider's API key in a FastAPI app?**
A: Never hardcode it — load from environment variables (e.g., via `python-dotenv` or a secrets manager like AWS Secrets Manager/GCP Secret Manager). Use Pydantic's `BaseSettings` for typed config loading:
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str
    anthropic_api_key: str

    class Config:
        env_file = ".env"

settings = Settings()
```

---

## 8. Background Tasks & Streaming

**Q34: What are FastAPI `BackgroundTasks`, and when would you use them?**
A: `BackgroundTasks` let you run a function *after* returning a response to the client — useful for logging, sending notification emails, or writing analytics without making the user wait for those side effects.
```python
from fastapi import BackgroundTasks

def log_usage(prompt: str, tokens: int):
    with open("usage.log", "a") as f:
        f.write(f"{prompt[:50]}... used {tokens} tokens\n")

@app.post("/generate")
async def generate(prompt: str, background_tasks: BackgroundTasks):
    result = call_llm(prompt)
    background_tasks.add_task(log_usage, prompt, result["tokens"])
    return result
```

**Q35: How do you stream an LLM's token-by-token output to the client in FastAPI?**
A: Use `StreamingResponse` with an async generator — this is the standard pattern for streaming chat completions.
```python
from fastapi.responses import StreamingResponse

async def token_stream(prompt: str):
    async for chunk in llm_client.stream(prompt):
        yield chunk.text

@app.post("/chat/stream")
async def chat_stream(prompt: str):
    return StreamingResponse(token_stream(prompt), media_type="text/event-stream")
```

**Q36: What's the difference between `BackgroundTasks` and a proper task queue (e.g., Celery, RQ)?**
A: `BackgroundTasks` runs within the same process/worker after the response is sent — fine for lightweight, fast operations. For long-running, resource-intensive, or retryable jobs (e.g., processing a large document for RAG ingestion), use a dedicated task queue with separate workers, so a crash or slowdown doesn't affect your API's ability to serve other requests.

---

## 9. Database Integration

**Q37: How do you integrate SQLAlchemy with FastAPI (async)?**
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session

@app.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(User).where(User.id == user_id))
    return result.scalar_one_or_none()
```

**Q38: How would you store conversation history for a chat application in a database via FastAPI?**
A: Model a `Conversation` table (id, user_id, created_at) and a `Message` table (id, conversation_id, role, content, created_at, token_count). On each request, fetch prior messages by `conversation_id`, append the new user message, call the LLM, then persist both the user message and assistant response.

**Q39: How do you integrate a vector database (e.g., for RAG) into a FastAPI service?**
```python
from qdrant_client import QdrantClient

qdrant = QdrantClient(url="http://localhost:6333")

@app.post("/search")
async def search(query: str):
    query_vector = embed(query)
    results = qdrant.search(collection_name="docs", query_vector=query_vector, limit=5)
    return [{"text": r.payload["text"], "score": r.score} for r in results]
```

---

## 10. Testing

**Q40: How do you test FastAPI endpoints?**
A: Use `TestClient` (built on `httpx`), which lets you call endpoints synchronously in tests without running a live server.
```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_item():
    response = client.get("/items/1")
    assert response.status_code == 200
    assert response.json()["item_id"] == 1
```

**Q41: How do you test async endpoints and mock external LLM API calls?**
```python
import pytest
from httpx import AsyncClient, ASGITransport
from unittest.mock import patch

@pytest.mark.asyncio
async def test_generate_endpoint():
    with patch("main.call_llm", return_value={"text": "mocked response"}):
        async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as ac:
            response = await ac.post("/generate", json={"prompt": "hello"})
    assert response.status_code == 200
    assert response.json()["text"] == "mocked response"
```

**Q42: How do you override a dependency (e.g., a DB session or auth check) during testing?**
```python
app.dependency_overrides[get_db] = lambda: mock_db_session
app.dependency_overrides[verify_api_key] = lambda: "test-key"
```
This lets you swap real dependencies for test doubles without changing endpoint code.

---

## 11. Performance & Deployment

**Q43: How do you run FastAPI in production for high-throughput workloads?**
A: Run with **Uvicorn workers managed by Gunicorn** (multiple worker processes to use multiple CPU cores, since the GIL limits a single process), behind a reverse proxy like **Nginx** for TLS termination and load balancing.
```bash
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

**Q44: What is connection pooling, and why does it matter for a FastAPI app calling external APIs/DBs?**
A: Connection pooling reuses existing TCP/HTTP connections instead of opening a new one per request, reducing latency and resource overhead. For LLM API calls, reusing an `httpx.AsyncClient` instance (rather than creating a new one per request) is a common performance win — configure it once at app startup.
```python
@app.on_event("startup")
async def startup():
    app.state.http_client = httpx.AsyncClient()

@app.on_event("shutdown")
async def shutdown():
    await app.state.http_client.aclose()
```
(Note: `on_event` is being replaced by the `lifespan` context manager in newer FastAPI versions.)

**Q45: What is the `lifespan` context manager, and why replace `on_event`?**
```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.http_client = httpx.AsyncClient()
    app.state.model = load_embedding_model()  # load once at startup
    yield
    await app.state.http_client.aclose()

app = FastAPI(lifespan=lifespan)
```
A: `lifespan` consolidates startup/shutdown logic into a single, more testable async context manager, and is the currently recommended pattern (the older `@app.on_event("startup")` / `"shutdown"` decorators are deprecated).

**Q46: How would you scale a FastAPI service that serves LLM requests horizontally?**
A: Run multiple stateless replicas behind a load balancer, keep session/conversation state in an external store (Redis/Postgres) rather than in-process memory, use a shared cache (Redis) for repeated LLM responses, and separate long-running/streaming connections from quick request-response endpoints if traffic patterns differ significantly.

---

## 12. FastAPI for AI/LLM Applications (Specific)

**Q47: Design a FastAPI endpoint for a RAG-based Q&A system.**
```python
from pydantic import BaseModel

class QueryRequest(BaseModel):
    question: str
    top_k: int = 5

class QueryResponse(BaseModel):
    answer: str
    sources: list[str]

@app.post("/ask", response_model=QueryResponse)
async def ask(request: QueryRequest, db=Depends(get_vector_db)):
    query_embedding = await embed_query(request.question)
    chunks = await db.search(query_embedding, top_k=request.top_k)
    context = "\n".join(c.text for c in chunks)
    answer = await call_llm(f"Context: {context}\n\nQuestion: {request.question}")
    return QueryResponse(answer=answer, sources=[c.source for c in chunks])
```

**Q48: How would you implement request timeouts for slow LLM calls in FastAPI?**
```python
import asyncio

@app.post("/generate")
async def generate(prompt: str):
    try:
        result = await asyncio.wait_for(call_llm(prompt), timeout=30.0)
        return result
    except asyncio.TimeoutError:
        raise HTTPException(status_code=504, detail="LLM request timed out")
```

**Q49: How do you handle multiple concurrent users each with their own streaming chat session?**
A: Each streaming request runs its own async generator independent of other requests (since FastAPI/Starlette handles each connection as a separate task on the event loop), but session/history state should not live in local Python variables at the module level (that would leak across users) — store per-user conversation state in a DB or cache keyed by session ID, fetched fresh on each request.

**Q50: How would you validate and sanitize user input before sending it to an LLM (basic prompt-injection defense)?**
A: Use Pydantic validators to enforce length limits and reject obviously malicious patterns, keep user input clearly delimited from system instructions in the prompt template (e.g., using explicit tags), and never let raw user input control which system prompt/tools get used.
```python
class ChatRequest(BaseModel):
    message: str

    @field_validator("message")
    @classmethod
    def validate_length(cls, v):
        if len(v) > 4000:
            raise ValueError("Message too long")
        return v
```

---

## 13. Coding Practice Questions

**Q51: Build a simple FastAPI app that wraps an LLM call with caching (avoid recomputation for identical prompts).**
```python
from fastapi import FastAPI
from pydantic import BaseModel
import hashlib

app = FastAPI()
cache: dict[str, str] = {}

class PromptRequest(BaseModel):
    prompt: str

def call_llm(prompt: str) -> str:
    return f"Response to: {prompt}"  # placeholder for real LLM call

@app.post("/generate")
async def generate(request: PromptRequest):
    key = hashlib.sha256(request.prompt.encode()).hexdigest()
    if key in cache:
        return {"response": cache[key], "cached": True}
    result = call_llm(request.prompt)
    cache[key] = result
    return {"response": result, "cached": False}
```

**Q52: Implement a health-check endpoint that verifies the app can reach its dependencies (DB + LLM provider).**
```python
@app.get("/health")
async def health_check(db=Depends(get_db)):
    checks = {"db": False, "llm": False}
    try:
        await db.execute(text("SELECT 1"))
        checks["db"] = True
    except Exception:
        pass
    try:
        await ping_llm_provider()
        checks["llm"] = True
    except Exception:
        pass
    status = "ok" if all(checks.values()) else "degraded"
    return {"status": status, "checks": checks}
```

**Q53: Write a dependency that extracts and validates a Bearer token, returning a 401 if missing/invalid.**
```python
from fastapi import Header, HTTPException

async def get_bearer_token(authorization: str = Header(...)):
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Missing or invalid Authorization header")
    return authorization.removeprefix("Bearer ")
```

**Q54: Implement pagination for a `/conversations` endpoint.**
```python
@app.get("/conversations")
async def list_conversations(
    skip: int = Query(0, ge=0),
    limit: int = Query(20, ge=1, le=100),
    db=Depends(get_db),
):
    result = await db.execute(
        select(Conversation).offset(skip).limit(limit)
    )
    return result.scalars().all()
```

**Q55: Write a WebSocket endpoint for a real-time chat with an LLM.**
```python
from fastapi import WebSocket, WebSocketDisconnect

@app.websocket("/ws/chat")
async def websocket_chat(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            user_message = await websocket.receive_text()
            async for chunk in llm_client.stream(user_message):
                await websocket.send_text(chunk)
    except WebSocketDisconnect:
        print("Client disconnected")
```

---

## Quick Reference: Study Priority for AI Engineer / Backend Interviews

| Priority | Topic Area |
|---|---|
| **High** | Async fundamentals (async/await, blocking vs non-blocking), Pydantic validation, dependency injection |
| **High** | Streaming responses, background tasks, LLM API integration patterns |
| **Medium** | Auth (OAuth2/JWT, API keys), rate limiting, error handling |
| **Medium** | Database integration (async SQLAlchemy), testing with `TestClient` |
| **Lower (but know basics)** | Deployment specifics (Gunicorn/Uvicorn workers), WebSockets |

**Final tip:** For AI Engineer roles specifically, be ready to design a complete endpoint from scratch — e.g., "build me a `/chat` endpoint that streams responses, handles rate limiting, and logs usage" — combining several of the patterns above. Interviewers care more about whether you understand *why* something is async or cached than whether you memorize exact syntax.

---

*Good luck with your FastAPI interview preparation!*
