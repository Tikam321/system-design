# Python & AI Engineer Interview Preparation Guide

A comprehensive Q&A reference covering Core Python, Data Structures, OOP, Python for Data/ML, Machine Learning, Deep Learning, LLMs & GenAI, System Design, and Coding Questions — tailored for AI Engineer role preparation.

---

## Table of Contents
1. [Core Python Fundamentals](#1-core-python-fundamentals)
2. [Data Structures & Algorithms](#2-data-structures--algorithms)
3. [Object-Oriented Programming](#3-object-oriented-programming)
4. [Python for Data Science (NumPy/Pandas)](#4-python-for-data-science-numpypandas)
5. [Machine Learning Fundamentals](#5-machine-learning-fundamentals)
6. [Deep Learning](#6-deep-learning)
7. [LLMs & Generative AI (AI Engineer Specific)](#7-llms--generative-ai-ai-engineer-specific)
8. [RAG & Vector Databases](#8-rag--vector-databases)
9. [AI Agents & Tool Use](#9-ai-agents--tool-use)
10. [MLOps / LLMOps & Deployment](#10-mlops--llmops--deployment)
11. [System Design for AI Applications](#11-system-design-for-ai-applications)
12. [Coding Practice Questions](#12-coding-practice-questions)

---

## 1. Core Python Fundamentals

**Q1: What is the difference between a list, tuple, and set in Python?**
A: A **list** is mutable, ordered, and allows duplicates. A **tuple** is immutable, ordered, and allows duplicates — used when data shouldn't change (e.g., function returns, dictionary keys). A **set** is mutable, unordered, and stores only unique elements — useful for membership tests and removing duplicates.

**Q2: What are Python's mutable and immutable data types?**
A: Immutable: `int`, `float`, `str`, `tuple`, `frozenset`, `bool`. Mutable: `list`, `dict`, `set`, custom objects (by default). Immutability matters for hashability (only immutable types can be dict keys/set members) and for avoiding unintended side effects when passing objects around.

**Q3: Explain Python's GIL (Global Interpreter Lock).**
A: The GIL is a mutex that allows only one thread to execute Python bytecode at a time, even on multi-core machines. It simplifies memory management (reference counting) but limits true parallelism for CPU-bound multi-threaded code. For CPU-bound work, use `multiprocessing` instead of `threading`. For I/O-bound work (API calls, file I/O), `threading` or `asyncio` still helps because the GIL is released during I/O waits.

**Q4: What's the difference between `*args` and `**kwargs`?**
A: `*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dict. Example:
```python
def func(*args, **kwargs):
    print(args)    # tuple
    print(kwargs)  # dict
func(1, 2, name="Alice")  # (1, 2) {'name': 'Alice'}
```

**Q5: What are decorators? Give an example relevant to AI engineering.**
A: A decorator wraps a function to extend its behavior without modifying its source. Common in AI pipelines for logging, timing, retries, and caching LLM calls.
```python
import time
from functools import wraps

def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

@timing_decorator
def call_llm(prompt):
    ...
```

**Q6: What is a generator, and why would you use one when processing large datasets?**
A: A generator produces values lazily using `yield`, keeping only one item in memory at a time instead of building a full list. This is critical for AI engineering when streaming tokens from an LLM, processing large training datasets, or batching embeddings without loading everything into RAM.
```python
def batch_generator(data, batch_size):
    for i in range(0, len(data), batch_size):
        yield data[i:i + batch_size]
```

**Q7: What is the difference between `deepcopy` and `copy`?**
A: `copy.copy()` creates a shallow copy — nested objects are still shared references. `copy.deepcopy()` recursively copies everything, so nested objects are fully independent. Relevant when duplicating config dicts or model state without mutating the original.

**Q8: Explain Python's context managers (`with` statement).**
A: Context managers handle setup/teardown automatically (e.g., closing files, releasing GPU memory, managing DB sessions). Implemented via `__enter__`/`__exit__` or `contextlib.contextmanager`.
```python
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.time()
    yield
    print(f"Elapsed: {time.time() - start}s")

with timer():
    train_model()
```

**Q9: What is the difference between `@staticmethod`, `@classmethod`, and instance methods?**
A: Instance methods take `self` and operate on instance data. `@classmethod` takes `cls` and operates on the class itself (useful for alternative constructors, e.g., `Model.from_pretrained(...)`). `@staticmethod` takes neither — it's a plain function namespaced inside a class for organizational purposes.

**Q10: What are Python's async/await used for, and why do they matter for AI engineers?**
A: `async`/`await` enable asynchronous (non-blocking) I/O — critical when calling multiple LLM APIs concurrently, serving multiple users in a FastAPI backend, or streaming responses without blocking the event loop.
```python
import asyncio
import httpx

async def call_model(prompt):
    async with httpx.AsyncClient() as client:
        response = await client.post(API_URL, json={"prompt": prompt})
        return response.json()

async def main():
    results = await asyncio.gather(*[call_model(p) for p in prompts])
```

**Q11: What is a Python virtual environment, and why is it important?**
A: A virtual environment isolates project dependencies so different projects can use different (potentially conflicting) package versions — essential in AI work where different projects may need different versions of `torch`, `transformers`, or `numpy`. Tools: `venv`, `conda`, `poetry`, `uv`.

**Q12: Explain exception handling and custom exceptions in Python.**
A: `try/except/else/finally` blocks handle errors gracefully. Custom exceptions let you build meaningful error hierarchies — useful for distinguishing, say, a rate-limit error from a model-timeout error when calling an LLM API.
```python
class LLMTimeoutError(Exception):
    pass

class LLMRateLimitError(Exception):
    pass

try:
    response = call_llm(prompt)
except LLMRateLimitError:
    time.sleep(retry_delay)
except LLMTimeoutError:
    log_and_retry()
finally:
    cleanup()
```

**Q13: What's the difference between `==` and `is`?**
A: `==` checks value equality; `is` checks identity (same object in memory). Common bug: using `is` to compare strings/numbers can behave inconsistently due to Python's interning, so always use `==` for value comparison.

**Q14: What is memoization, and how does `functools.lru_cache` help?**
A: Memoization caches function outputs for previously seen inputs, avoiding recomputation. Useful for caching embeddings or repeated LLM calls with identical prompts.
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_embedding(text):
    return embedding_model.encode(text)
```

---

## 2. Data Structures & Algorithms

**Q15: What is the time complexity of common operations on a Python list vs. dict?**
A: List: indexing O(1), search O(n), append O(1) amortized, insert/delete at arbitrary index O(n). Dict: get/set/delete O(1) average (hash table), but O(n) worst case with hash collisions.

**Q16: How would you implement an LRU cache from scratch?**
A: Use a combination of a hash map (for O(1) lookup) and a doubly linked list (for O(1) reordering of recently-used items). Python's `OrderedDict` makes this simple:
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key):
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

**Q17: Explain how a hash table works and why it matters for embeddings/vector lookups.**
A: A hash table maps keys to array indices via a hash function, giving average O(1) lookup. In AI systems, similar concepts (hashing/indexing) underlie approximate nearest-neighbor search (e.g., LSH — Locality Sensitive Hashing) used to speed up vector similarity search at scale.

**Q18: What's the difference between BFS and DFS, and when would each be used in an agent's task planning?**
A: BFS explores all nodes level-by-level (good for shortest-path problems); DFS explores as deep as possible before backtracking (good for exhaustive search/backtracking). In agent planning, DFS-like exploration is common in tree-of-thought reasoning where an agent explores one plan deeply before backtracking to try alternatives.

**Q19: What is Big-O notation, and why does it matter when choosing a data structure for a RAG pipeline?**
A: Big-O describes how runtime/memory scales with input size. For RAG at scale, choosing a vector index (e.g., HNSW at O(log n) approx. search) instead of brute-force cosine similarity (O(n)) is the difference between a system that scales to millions of documents and one that doesn't.

---

## 3. Object-Oriented Programming

**Q20: Explain the four pillars of OOP with AI-relevant examples.**
A:
- **Encapsulation**: Bundling data and methods together, e.g., a `Tokenizer` class hiding its internal vocabulary logic behind `.encode()`/`.decode()`.
- **Abstraction**: Exposing only essential details, e.g., calling `model.generate(prompt)` without needing to know the underlying attention mechanism.
- **Inheritance**: A `BaseAgent` class with shared logic, extended by `ResearchAgent`, `CodingAgent`, etc.
- **Polymorphism**: Different agent subclasses implementing their own `.run()` method, callable through the same interface.

**Q21: What is the difference between an abstract base class and a regular class?**
A: An abstract base class (via Python's `abc` module) defines a contract that subclasses must implement, preventing instantiation of the base class itself. Useful for defining a common interface across multiple LLM providers.
```python
from abc import ABC, abstractmethod

class BaseLLMProvider(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str:
        ...

class OpenAIProvider(BaseLLMProvider):
    def generate(self, prompt: str) -> str:
        return call_openai(prompt)

class AnthropicProvider(BaseLLMProvider):
    def generate(self, prompt: str) -> str:
        return call_anthropic(prompt)
```

**Q22: What is method resolution order (MRO), and why does it matter in multiple inheritance?**
A: MRO determines the order in which base classes are searched when calling a method. Python uses the C3 linearization algorithm. Matters when combining mixins (e.g., a `LoggingMixin` + `RetryMixin` on an API client class) to avoid ambiguous method calls.

---

## 4. Python for Data Science (NumPy/Pandas)

**Q23: Why is NumPy faster than plain Python lists for numerical operations?**
A: NumPy arrays are stored in contiguous memory blocks of a single data type and leverage vectorized C operations (SIMD), avoiding Python's per-element overhead and type-checking. This makes operations like matrix multiplication orders of magnitude faster — foundational for all ML/DL computation.

**Q24: What is broadcasting in NumPy?**
A: Broadcasting lets NumPy perform element-wise operations on arrays of different shapes by implicitly expanding the smaller array without copying data, e.g., adding a bias vector to every row of a matrix without a loop.
```python
import numpy as np
matrix = np.random.rand(100, 10)
bias = np.random.rand(10)
result = matrix + bias  # bias broadcast across all 100 rows
```

**Q25: How would you handle missing data in a Pandas DataFrame before feeding it into a model?**
A: Options: `df.dropna()` to remove rows/columns, `df.fillna(value)` for constant/mean/median imputation, or model-based imputation (e.g., `sklearn.impute.KNNImputer`). Choice depends on how much data is missing and whether missingness is random or informative.

**Q26: What's the difference between `.loc[]` and `.iloc[]` in Pandas?**
A: `.loc[]` selects by label (column/index name); `.iloc[]` selects by integer position. Mixing them up is a very common bug source.

---

## 5. Machine Learning Fundamentals

**Q27: What is the bias-variance tradeoff?**
A: **Bias** is error from overly simplistic assumptions (underfitting); **variance** is error from sensitivity to training data noise (overfitting). The goal is minimizing total error by balancing model complexity — too simple underfits, too complex overfits and fails to generalize.

**Q28: Explain overfitting and how to prevent it.**
A: Overfitting is when a model learns training data noise instead of generalizable patterns, performing well on training data but poorly on unseen data. Prevention: more training data, regularization (L1/L2, dropout), early stopping, cross-validation, simpler models, data augmentation.

**Q29: What's the difference between L1 and L2 regularization?**
A: L1 (Lasso) adds the absolute value of weights to the loss, encouraging sparsity (some weights become exactly zero — useful for feature selection). L2 (Ridge) adds the squared value of weights, shrinking them smoothly toward zero without eliminating them entirely.

**Q30: Explain precision, recall, and F1-score. When would you prioritize one over another?**
A: **Precision** = TP / (TP + FP) — how many predicted positives are actually correct. **Recall** = TP / (TP + FN) — how many actual positives were caught. **F1** is the harmonic mean of both. Prioritize recall in high-cost-of-miss scenarios (e.g., fraud/medical detection); prioritize precision when false positives are costly (e.g., spam filters blocking legitimate emails).

**Q31: What is cross-validation, and why use k-fold instead of a single train/test split?**
A: Cross-validation splits data into k folds, training on k-1 folds and validating on the remaining fold, rotating through all folds. This gives a more robust estimate of model performance than a single split, reducing the risk that results are a fluke of one particular split.

**Q32: What is the difference between bagging and boosting?**
A: **Bagging** (e.g., Random Forest) trains multiple models in parallel on random data subsets and averages results to reduce variance. **Boosting** (e.g., XGBoost, LightGBM) trains models sequentially, each correcting the errors of the previous one, reducing bias.

**Q33: Explain gradient descent and its variants.**
A: Gradient descent iteratively adjusts model parameters in the direction that reduces the loss function. **Batch GD** uses the whole dataset per update (slow, stable). **SGD** (Stochastic) uses one sample per update (fast, noisy). **Mini-batch GD** is the practical middle ground used in almost all modern deep learning. **Adam** adds momentum and adaptive learning rates on top of this.

---

## 6. Deep Learning

**Q34: What is backpropagation?**
A: Backpropagation computes the gradient of the loss function with respect to each weight by applying the chain rule backward through the network, enabling gradient descent to update weights layer by layer.

**Q35: What is the vanishing/exploding gradient problem, and how is it addressed?**
A: In deep networks, gradients can shrink toward zero (vanishing) or grow uncontrollably (exploding) as they propagate backward through many layers, stalling or destabilizing training. Fixes: ReLU activations (vs. sigmoid/tanh), batch normalization, residual/skip connections (as in ResNets and Transformers), gradient clipping, careful weight initialization.

**Q36: Explain the Transformer architecture at a high level.**
A: Transformers process sequences using **self-attention** instead of recurrence, letting every token attend to every other token in parallel. Core components: multi-head self-attention, positional encoding (since there's no inherent order awareness), feed-forward layers, layer normalization, and residual connections. This parallelism is what made training on massive datasets feasible, unlike RNNs/LSTMs.

**Q37: What is self-attention, in plain terms?**
A: For each token, self-attention computes a weighted combination of all other tokens' representations, where the weights (attention scores) reflect how relevant each other token is to understanding this one. Computed via Query, Key, and Value matrices: `Attention(Q,K,V) = softmax(QK^T / √d_k) V`.

**Q38: What's the difference between encoder-only, decoder-only, and encoder-decoder models?**
A: **Encoder-only** (e.g., BERT) builds bidirectional representations, good for classification/understanding tasks. **Decoder-only** (e.g., GPT, Claude, Llama) generates text autoregressively (left-to-right), good for generation. **Encoder-decoder** (e.g., T5, original Transformer) encodes input then decodes output — good for translation/summarization where input and output are distinct sequences.

**Q39: What is dropout, and why does it help?**
A: Dropout randomly zeroes a fraction of neurons during training, preventing co-adaptation of neurons and reducing overfitting by forcing the network to learn redundant, robust representations.

**Q40: What is batch normalization?**
A: Batch norm normalizes layer inputs (zero mean, unit variance) per mini-batch, which stabilizes and speeds up training by reducing internal covariate shift, and allows higher learning rates.

---

## 7. LLMs & Generative AI (AI Engineer Specific)

**Q41: What is a token, and why does tokenization matter for cost/performance?**
A: A token is a chunk of text (often a sub-word unit) that an LLM processes as its atomic unit — not the same as a word or character. Cost and context-window limits are measured in tokens, so tokenization efficiency directly affects API cost and how much content fits in a prompt. Different models use different tokenizers (e.g., BPE-based), so token counts vary by model for the same text.

**Q42: What is the context window, and what are its practical implications?**
A: The context window is the maximum number of tokens (input + output combined) a model can process in one call. Practical implications: long documents must be chunked or summarized for RAG; conversation history must be truncated or summarized once it exceeds the window; larger context windows increase cost and latency even if not fully used.

**Q43: Explain temperature, top-p, and top-k sampling.**
A: **Temperature** scales the probability distribution before sampling — lower values (e.g., 0.2) make output more deterministic/focused, higher values (e.g., 1.0+) make it more random/creative. **Top-k** restricts sampling to the k most likely next tokens. **Top-p (nucleus sampling)** restricts sampling to the smallest set of tokens whose cumulative probability exceeds p. These are usually tuned together depending on whether the task needs precision (low temp) or creativity (higher temp).

**Q44: What is prompt engineering, and what are some core techniques?**
A: Prompt engineering is designing inputs to reliably get useful outputs from an LLM without changing model weights. Core techniques: few-shot examples, chain-of-thought prompting ("think step by step"), explicit output format specification (e.g., requesting JSON/XML), role/system prompts, and providing negative examples of what not to do.

**Q45: What is fine-tuning, and when would you choose it over prompting/RAG?**
A: Fine-tuning updates model weights on a custom dataset to change behavior/style/domain knowledge permanently. Choose fine-tuning when: you need consistent formatting/behavior at scale, prompting alone can't achieve the needed accuracy, or you need to bake in a very large volume of stylistic/domain patterns. Choose RAG/prompting instead when: knowledge changes frequently, you need traceability/citations, or you have limited labeled training data — fine-tuning is more expensive and doesn't help with fast-changing facts.

**Q46: What is LoRA (Low-Rank Adaptation), and why is it popular?**
A: LoRA fine-tunes a model efficiently by freezing the original weights and injecting small trainable low-rank matrices into each layer, dramatically reducing the number of trainable parameters (often <1% of the full model). This makes fine-tuning feasible on consumer GPUs and allows swapping lightweight "adapters" for different tasks without duplicating the whole model.

**Q47: What is hallucination in LLMs, and how do you mitigate it?**
A: Hallucination is when a model generates plausible-sounding but factually incorrect or fabricated content, because LLMs predict statistically likely text rather than verified facts. Mitigation: RAG (grounding responses in retrieved documents), lowering temperature, asking the model to cite sources, output verification/fact-checking pipelines, and explicit instructions to say "I don't know" when uncertain.

**Q48: What is the difference between zero-shot, one-shot, and few-shot prompting?**
A: **Zero-shot**: no examples given, relying purely on the model's pretrained knowledge and instructions. **One-shot**: one example provided to demonstrate the expected format/behavior. **Few-shot**: multiple examples provided, which generally improves accuracy on structured/niche tasks at the cost of using more context tokens.

**Q49: What is chain-of-thought (CoT) prompting, and why does it improve performance on reasoning tasks?**
A: CoT prompts the model to generate intermediate reasoning steps before the final answer (e.g., "let's think step by step"), which improves accuracy on multi-step math/logic tasks because it gives the model "room" to build up to the answer incrementally rather than jumping straight to a (possibly wrong) conclusion.

**Q50: What is model distillation?**
A: Distillation trains a smaller "student" model to mimic a larger "teacher" model's outputs, producing a faster/cheaper model that retains much of the teacher's capability — used to create smaller, deployable versions of large models.

**Q51: How do you evaluate an LLM application in production?**
A: Combination of: automated eval sets (golden Q&A pairs with expected outputs), LLM-as-a-judge scoring, human evaluation/spot-checks, task-specific metrics (BLEU/ROUGE for summarization, exact-match for QA), latency/cost tracking, and user feedback signals (thumbs up/down, retry rate). Production systems also need regression testing so a model/prompt update doesn't silently degrade quality.

**Q52: What is function calling / tool use in LLMs?**
A: Function calling lets an LLM output a structured request (function name + arguments) instead of free text when it determines an external action/tool is needed (e.g., calling a weather API, querying a database). The application layer executes the actual function and returns the result to the model, which then incorporates it into its final response — this is the foundation of agentic behavior.

---

## 8. RAG & Vector Databases

**Q53: What is RAG (Retrieval-Augmented Generation), and why is it used?**
A: RAG retrieves relevant documents/chunks from an external knowledge base at query time and injects them into the LLM's prompt as context, so the model can generate grounded, up-to-date answers without needing to be retrained on that data. Solves two core LLM limitations: knowledge cutoff and hallucination on domain-specific facts.

**Q54: Describe the typical RAG pipeline end-to-end.**
A: 1) **Chunking**: split documents into manageable pieces. 2) **Embedding**: convert chunks into vector representations using an embedding model. 3) **Indexing**: store vectors in a vector database (e.g., Pinecone, Weaviate, pgvector, FAISS). 4) **Retrieval**: at query time, embed the user's query and find the most similar chunks (cosine similarity/ANN search). 5) **Augmentation**: insert retrieved chunks into the LLM prompt. 6) **Generation**: LLM produces an answer grounded in the retrieved context.

**Q55: What is chunking strategy, and why does chunk size matter?**
A: Chunking splits documents into pieces small enough to embed meaningfully and fit in context, but large enough to preserve semantic coherence. Too small → loses context/meaning; too large → dilutes relevance and wastes context window. Common strategies: fixed-size with overlap, sentence/paragraph-based, semantic chunking (splitting at topic boundaries).

**Q56: What is a vector embedding?**
A: A vector embedding is a dense numerical representation of text (or image/audio) such that semantically similar inputs are located close together in vector space, enabling similarity search via distance metrics like cosine similarity.

**Q57: What is Approximate Nearest Neighbor (ANN) search, and why not use exact search?**
A: ANN algorithms (e.g., HNSW, IVF) trade a small amount of accuracy for massive speed gains when searching millions/billions of vectors, since exact brute-force nearest-neighbor search (O(n)) doesn't scale. HNSW builds a navigable graph structure enabling approximate O(log n) search.

**Q58: How do you improve RAG retrieval quality beyond basic vector search?**
A: Hybrid search (combining vector similarity with keyword/BM25 search), re-ranking retrieved results with a cross-encoder model, query rewriting/expansion, metadata filtering, and chunk-overlap tuning. Also: evaluating retrieval separately from generation (recall@k) to isolate whether failures are retrieval or generation issues.

**Q59: What's the difference between a vector database and a traditional relational database?**
A: Vector DBs are optimized for similarity search over high-dimensional embeddings (via ANN indexes) rather than exact-match queries over structured rows. Some systems (e.g., pgvector) add vector search capability on top of a traditional relational database, letting you combine structured filtering with semantic search.

---

## 9. AI Agents & Tool Use

**Q60: What defines an "AI agent" as opposed to a simple LLM call?**
A: An agent uses an LLM as a reasoning engine that can autonomously decide which tools to call, in what order, evaluate intermediate results, and iterate — as opposed to a single fixed input→output call. Agents typically have: a planning/reasoning loop, tool access, memory (short and/or long-term), and some mechanism to know when the task is complete.

**Q61: What is the ReAct pattern?**
A: ReAct (Reason + Act) interleaves reasoning traces with actions: the model reasons about what to do, takes an action (tool call), observes the result, then reasons again — repeating until the task is solved. This structured loop improves reliability over asking a model to do everything in one shot.

**Q62: What is MCP (Model Context Protocol), and why does it matter for agent building?**
A: MCP is a standardized protocol for connecting LLM applications to external tools, data sources, and services, so developers don't need to write custom integration code for every tool/every model provider. It separates "the agent" from "the tools it can use," making tool integrations reusable and portable across different LLM applications.

**Q63: How do you prevent an agent from getting stuck in infinite tool-call loops?**
A: Set a maximum iteration/step count, implement timeout budgets, track repeated identical tool calls and break the loop, and design clear "task complete" signals the agent can emit. Also useful: cost/token budgets per task run.

**Q64: What's the difference between short-term and long-term memory in an agent system?**
A: Short-term memory is the current conversation/context window — temporary and lost after the session. Long-term memory persists across sessions (e.g., stored in a vector DB or key-value store) so the agent can recall user preferences or past interactions beyond a single context window.

**Q65: How do you handle errors/failures in a multi-step agent workflow?**
A: Wrap tool calls in try/except with retries and exponential backoff, validate tool outputs before passing them back to the model, give the model the error message so it can self-correct (rather than silently failing), and set hard limits on retries to avoid runaway costs.

---

## 10. MLOps / LLMOps & Deployment

**Q66: What is the difference between MLOps and LLMOps?**
A: MLOps covers the general lifecycle of ML models (training, versioning, deployment, monitoring, retraining). LLMOps applies similar discipline specifically to LLM-based applications, with added focus on prompt versioning, embedding pipeline management, cost/token tracking, and evaluation of non-deterministic generative outputs (harder to test than traditional ML classification accuracy).

**Q67: How would you monitor an LLM application in production?**
A: Track latency (p50/p95/p99), token usage and cost per request, error rates (timeouts, rate limits), output quality via sampled human review or LLM-as-judge scoring, user feedback signals, and drift — e.g., whether retrieved documents or query patterns are shifting over time in ways that degrade relevance.

**Q68: What is model versioning, and why does it matter for reproducibility?**
A: Model versioning tracks exactly which model weights/version, prompt template, and configuration produced a given output, so results are reproducible and you can roll back if a new version regresses quality — critical since providers frequently update models behind the same API endpoint name.

**Q69: How do you reduce LLM API costs in a production system?**
A: Cache repeated/identical queries, use a smaller/cheaper model for simple sub-tasks and reserve the largest model for complex reasoning (model routing), reduce prompt/context size via better chunking, batch requests where possible, and set token limits on outputs.

---

## 11. System Design for AI Applications

**Q70: Design a customer support chatbot backed by a company's internal documentation. Walk through your architecture.**
A: Key components: (1) **Ingestion pipeline** — periodically crawl/parse internal docs, chunk them, generate embeddings, store in a vector DB. (2) **Retrieval layer** — on each user query, embed the query, retrieve top-k relevant chunks (possibly with hybrid search + re-ranking). (3) **Generation layer** — construct a prompt with system instructions + retrieved context + conversation history, call the LLM. (4) **Guardrails** — filter for off-topic/harmful queries, add citation of sources in the response. (5) **Feedback loop** — capture thumbs up/down to identify weak spots in retrieval or generation. (6) **Observability** — log latency, cost, and failure cases for continuous improvement.

**Q71: How would you design a system to handle 10,000 concurrent LLM requests without excessive cost or latency?**
A: Use a request queue with rate-limiting per provider, implement caching for repeated queries, route requests across multiple model tiers based on task complexity, use streaming responses to improve perceived latency, employ async/non-blocking I/O throughout the backend, and consider batching where the provider supports it.

**Q72: How would you design an evaluation pipeline for a RAG system before shipping changes to production?**
A: Build a golden dataset of representative queries with expected answers/sources. Evaluate retrieval separately (recall@k — did the right chunk get retrieved?) from generation (did the model use the context correctly, and is the answer factually grounded?). Run automated regression tests on every prompt/model/pipeline change, supplemented by periodic human spot-checks, before promoting changes to production.

---

## 12. Coding Practice Questions

**Q73: Write a function to compute cosine similarity between two vectors.**
```python
import numpy as np

def cosine_similarity(vec_a, vec_b):
    a, b = np.array(vec_a), np.array(vec_b)
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
```

**Q74: Implement a simple token-bucket rate limiter (useful for LLM API call throttling).**
```python
import time

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.last_check = time.time()

    def allow_request(self):
        now = time.time()
        elapsed = now - self.last_check
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)
        self.last_check = now
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False
```

**Q75: Write a function to chunk a long text into overlapping windows for RAG ingestion.**
```python
def chunk_text(text, chunk_size=500, overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap
    return chunks
```

**Q76: Implement a basic retry decorator with exponential backoff (for flaky LLM API calls).**
```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    wait = base_delay * (2 ** attempt)
                    print(f"Retry {attempt+1}/{max_retries} after {wait}s: {e}")
                    time.sleep(wait)
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3)
def call_llm_api(prompt):
    ...
```

**Q77: Given a list of (text, embedding) pairs and a query embedding, return the top-k most similar texts.**
```python
import numpy as np

def top_k_similar(query_embedding, documents, k=3):
    # documents: list of (text, embedding) tuples
    scored = [
        (text, cosine_similarity(query_embedding, emb))
        for text, emb in documents
    ]
    scored.sort(key=lambda x: x[1], reverse=True)
    return scored[:k]
```

**Q78: Implement a simple in-memory LLM response cache keyed by prompt hash.**
```python
import hashlib

class LLMCache:
    def __init__(self):
        self._cache = {}

    def _hash(self, prompt: str) -> str:
        return hashlib.sha256(prompt.encode()).hexdigest()

    def get(self, prompt):
        return self._cache.get(self._hash(prompt))

    def set(self, prompt, response):
        self._cache[self._hash(prompt)] = response
```

**Q79: Write a function that streams tokens from a generator and prints them as they arrive (simulating LLM streaming).**
```python
def stream_response(token_generator):
    full_response = ""
    for token in token_generator:
        print(token, end="", flush=True)
        full_response += token
    return full_response
```

**Q80: Reverse a linked list (classic warm-up question still commonly asked).**
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverse_list(head):
    prev = None
    current = head
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    return prev
```

---

## Quick Reference: Study Priority for AI Engineer Interviews

| Priority | Topic Area |
|---|---|
| **High** | LLM fundamentals (tokens, context, sampling), RAG architecture, function calling/agents, Python fundamentals |
| **High** | System design for AI apps, prompt engineering, evaluation strategy |
| **Medium** | Transformer architecture internals, fine-tuning vs. RAG tradeoffs, vector DB internals |
| **Medium** | Classic DS&A (still asked at many companies as a baseline filter) |
| **Lower (but know basics)** | Classical ML (bias/variance, regularization), MLOps/deployment specifics |

**Final tip:** For AI Engineer interviews specifically, be ready to whiteboard a RAG or agent architecture end-to-end — this comes up far more often than deep ML theory. Practice explaining trade-offs out loud (why RAG over fine-tuning, why this chunking strategy, how you'd evaluate quality) since reasoning about trade-offs is usually weighted more heavily than reciting definitions.

---

*Good luck with your AI Engineer interview preparation!*
