# LLM Inference Mechanics: Prefill vs. Decode, KV Caching, and Latency Analysis

This document provides a comprehensive, step-by-step reference on how Large Language Models (LLMs) process input queries and generate output responses at inference time. It explains why processing a large input prompt is significantly faster than generating a large output response, and includes architectural block diagrams illustrating the exact data flow through the Transformer layers and the GPU KV Cache.

---

## Table of Contents
1. [The Inference Speed Paradox: 10k Input vs. 10k Output Tokens](#1-the-inference-speed-paradox-10k-input-vs-10k-output-tokens)
2. [The Latency Equation & Pricing Economics](#2-the-latency-equation--pricing-economics)
3. [Phase 1: The Prefill Stage (Processing the Input Query)](#3-phase-1-the-prefill-stage-processing-the-input-query)
4. [Phase 2: The Decode Stage (Autoregressive Output Generation)](#4-phase-2-the-decode-stage-autoregressive-output-generation)
5. [Summary Comparison: Prefill vs. Decode](#5-summary-comparison-prefill-vs-decode)

---

## 1. The Inference Speed Paradox: 10k Input vs. 10k Output Tokens

Consider two inference scenarios that process the exact same total number of tokens ($10,010$ tokens):
* **Case 1:** $10$ input tokens $\rightarrow$ $10,000$ output tokens (e.g., *"Generate a story for my teddy bear to fall asleep"*).
* **Case 2:** $10,000$ input tokens $\rightarrow$ $10$ output tokens (e.g., summarizing a 10,000-token document into a 10-token sentence).

**Verdict:** **Case 2 ($10,000$ input tokens $\rightarrow$ $10$ output tokens) is drastically faster** than Case 1.

### 1.1 Why Input Tokens Are Fast (The Prefill Phase)
* **Massive Parallelism:** When a user submits $10,000$ input tokens in Case 2, all $10,000$ tokens are already known upfront. The GPU processes all $10,000$ tokens **simultaneously in a single parallel forward pass** using dense matrix-matrix multiplications (`GEMM`).
* **Compute-Bound Efficiency:** The GPU loads the frozen model weights from off-chip High Bandwidth Memory (HBM) into fast on-chip SRAM **just once** per layer and applies them across all $10,000$ tokens simultaneously. This achieves high **Arithmetic Intensity** (many FLOPs executed per byte of memory transferred) and saturates the GPU's parallel Tensor Cores.

### 1.2 Why Output Tokens Are Slow (The Autoregressive Decode Phase)
* **Sequential Dependency (One-by-One):** When generating $10,000$ output tokens in Case 1, the model operates **autoregressively**. It cannot generate token #2 until it has finished generating token #1, and it cannot generate token #3 until it has token #2. Consequently, the GPU must execute **10,000 separate, sequential forward passes** one after another.
* **Memory-Bandwidth Bottleneck:** For every single output token generated, the GPU must stream 100% of the model's billions of parameters from slow off-chip HBM into on-chip SRAM just to compute **one** token (matrix-vector multiplication, `GEMV`). The GPU compute cores sit mostly idle, waiting for weights to arrive over the memory bus.

| Metric / Property | Case 1: 10 Input $\rightarrow$ 10,000 Output | Case 2: 10,000 Input $\rightarrow$ 10 Output |
| :--- | :--- | :--- |
| **Prefill (Parallel Forward Passes)** | 1 parallel pass (processing 10 tokens) | 1 parallel pass (processing 10,000 tokens) |
| **Decode (Sequential Forward Passes)** | ~10,000 sequential passes (1 token each) | ~10 sequential passes (1 token each) |
| **Roofline Hardware Bottleneck** | **Memory-Bandwidth Bound** (loads all weights 10,000 times) | **Compute-Bound** (loads all weights once for prompt, plus ~10 times for output) |
| **Overall Wall-Clock Speed** | **Very Slow** | **Much Faster** |

---

## 2. The Latency Equation & Pricing Economics

Total end-to-end latency for an LLM inference request is governed by three components:

$$T_{\text{request}} = T_{\text{queue}} + T_{\text{prefill}} + (O - 1) \times T_{\text{per\_output\_token}}$$

Where:
* **$T{\text{queue}}$:** Time spent waiting in the server's scheduling/batching queue.
* **$T{\text{prefill}}$:** Time required to process all input prompt tokens in parallel and generate the very first output token.
* **$O$:** Total number of generated output tokens.
* **$T{\text{per\_output\_token}}$:** Time required to generate each subsequent output token sequentially.

### Key Infrastructure Metrics
1. **TTFT (Time To First Token):**
   $$\text{TTFT} = T_{\text{queue}} + T_{\text{prefill}}$$
   Measures how quickly the user sees the model begin responding.
2. **TPOT (Time Per Output Token):**
   $$\text{TPOT} = T_{\text{per\_output\_token}}$$
   Measures the streaming speed of each generated token after the first token appears.
3. **TTLT (Time To Last Token):**
   $$\text{TTLT} = \text{TTFT} + (O - 1) \times \text{TPOT}$$
   Total time for the complete response to finish.

**Why Output Tokens Cost ~5x More Than Input Tokens:**
Because generating a single output token ties up the entire GPU's memory bandwidth to load all model weights for just one token, output generation consumes far more GPU time per token than parallel input processing. Commercial LLM providers reflect this hardware reality by pricing output tokens roughly $5\times$ higher than input tokens (and offering up to $10\times$ discounts on cached input prompts).

---

## 3. Phase 1: The Prefill Stage (Processing the Input Query)

A common question is: *"During the input phase, does the model only compute Keys and Values ($K, V$), or does it run the full Attention and Feed-Forward Network (FFN) layers?"*

**Answer:** The input tokens pass through **every single Attention and Feed-Forward Network (FFN) layer** in the model—just like output tokens do. However, all input tokens are processed **together in parallel**, accomplishing two critical goals simultaneously:
1. **Populating the KV Cache** across every layer so we never have to recompute the input prompt again.
2. **Predicting the 1st Output Token** from the final hidden state of the last input token.

### Step-by-Step Walkthrough of Phase 1 (Prefill)
Suppose the user submits a 3-token input prompt: $t_1 = \text{"The"}$, $t_2 = \text{"cat"}$, $t_3 = \text{"sits"}$.

1. **Tokenization & Embedding Lookup ($W_E$):**
   All 3 tokens look up their pre-trained embedding vectors in parallel inside the frozen Embedding Matrix $W_E$, producing initial hidden vectors $[x_1, x_2, x_3]$.
2. **Q, K, V Linear Projections (Inside Layers $1 \dots L$):**
   The input matrix $X = [x_1, x_2, x_3]$ is multiplied in parallel by the pre-trained weight matrices $W_Q, W_K, W_V$:
   * Queries: $Q = X W_Q \rightarrow [q_1, q_2, q_3]$
   * Keys: $K = X W_K \rightarrow [k_1, k_2, k_3]$
   * Values: $V = X W_V \rightarrow [v_1, v_2, v_3]$
3. **Saving to the GPU KV Cache:**
   The computed Keys $[k_1, k_2, k_3]$ and Values $[v_1, v_2, v_3]$ for that layer are immediately written to the **GPU KV Cache** in HBM. Future output tokens will reuse these exact vectors without ever recalculating them.
4. **Masked Multi-Head Self-Attention (Parallel):**
   Using a causal triangular mask (so earlier tokens cannot peek at future tokens), all 3 queries attend to their valid keys and values in parallel:
   * $q_1$ (for `"The"`) attends to $[k_1, v_1]$.
   * $q_2$ (for `"cat"`) attends to $[k_1, k_2]$ and $[v_1, v_2]$.
   * $q_3$ (for `"sits"`) attends to $[k_1, k_2, k_3]$ and $[v_1, v_2, v_3]$.
5. **Feed-Forward Network (FFN) (Parallel):**
   The attention outputs for all 3 tokens pass through the layer's Feed-Forward Network (and residual connections + LayerNorm) in parallel. Their updated representations then feed into Layer 2, continuing all the way to the final Layer $L$ to produce $[h_1, h_2, h_3]$.
   *(Why do $t_1$ and $t_2$ need to go through the FFN in Layer 1? Because in Layer 2, $k_1, v_1, k_2, v_2$ are computed from the Layer 1 FFN outputs of $t_1$ and $t_2$!)*
6. **Generating the 1st Output Token:**
   After exiting the final Layer $L$, only the final hidden vector of the **last input token** ($h_3$, corresponding to `"sits"`) is needed, because $h_3$ has already aggregated the full context of `"The cat sits"`. Passing $h_3$ through the Unembedding Linear Layer and Softmax yields the **1st generated output token**: $t_4 = \text{"on"}$.

### Architectural Block Diagram: Phase 1 (Prefill Stage)

```text
INPUT QUERY:       "The" (t1)          "cat" (t2)          "sits" (t3)
                       │                   │                   │
                       ▼                   ▼                   ▼
[Embedding W_E]       x1                  x2                  x3
                       │                   │                   │
               ┌───────┴───────────────────┴───────────────────┴───────┐
               │             TRANSFORMER LAYER (1 to L)                │
               │                                                       │
               │   1. Multiply by Pre-trained W_Q, W_K, W_V:           │
               │      [q1, k1, v1]        [q2, k2, v2]    [q3, k3, v3] │
               │           │                   │               │       │
               │           └─────────┬─────────┴───────────────┼───────┼──► [SAVE k1..k3, v1..v3]
               │                     ▼                         ▼       │     INTO GPU KV CACHE!
               │   2. Masked Self-Attention (Computed in Parallel):    │
               │      (t1 sees t1)   (t2 sees t1..t2)  (t3 sees t1..t3)│
               │           │                   │               │       │
               │           ▼                   ▼               ▼       │
               │   3. Feed-Forward Network (FFN) (Computed in Parallel)│
               │           │                   │               │       │
               └───────────┼───────────────────┼───────────────┼───────┘
                           ▼                   ▼               ▼
Final Hidden States:      h1                  h2              h3
                      (ignored)           (ignored)            │
                                                               ▼
                                                    [Linear + Softmax]
                                                               │
                                                               ▼
1st GENERATED OUTPUT TOKEN:                                "on" (t4)
```

---

## 4. Phase 2: The Decode Stage (Autoregressive Output Generation)

Once the Prefill stage produces the first output token ($t_4 = \text{"on"}$), the system transitions into the **Decode Stage** to generate the remaining tokens ($t_5 = \text{"the"}$, $t_6 = \text{"mat"}$, etc.) **one token at a time**.

Because $[k_1, k_2, k_3]$ and $[v_1, v_2, v_3]$ are already safely stored inside the **GPU KV Cache** for every layer, **we never feed `"The cat sits"` into the model again.**

### Step-by-Step Walkthrough of Phase 2 (Decode)
1. **Single-Token Input:**
   Only the newly generated token $t_4 = \text{"on"}$ is fed into the model and embedded into vector $x_4$.
2. **Compute Q, K, V for $t_4$ Only:**
   Inside each layer, we multiply the single vector $x_4$ by $W_Q, W_K, W_V$ to produce **only** $q_4, k_4, v_4$.
3. **Append to the KV Cache:**
   We append $k_4$ and $v_4$ to the layer's existing KV Cache in GPU memory. The cache now holds $[k_1, k_2, k_3, k_4]$ and $[v_1, v_2, v_3, v_4]$.
4. **Self-Attention Against the KV Cache:**
   To let `"on"` attend to all previous tokens (`"The cat sits on"`), we compute the dot product of the single query vector $q_4$ with all cached keys $[k_1, k_2, k_3, k_4]$, apply Softmax, and compute the weighted sum over all cached values $[v_1, v_2, v_3, v_4]$:
   $$\text{Attention}(q_4) = \text{Softmax}\left(\frac{q_4 \cdot [k_1, k_2, k_3, k_4]^T}{\sqrt{d_k}}\right) [v_1, v_2, v_3, v_4]$$
5. **Feed-Forward Network (FFN) & Next Token Prediction:**
   The single attention output vector for $t_4$ passes through the layer's FFN. After exiting the final Layer $L$ as hidden state $h_4$, it passes through the Linear + Softmax head to sample the **2nd generated output token**: $t_5 = \text{"the"}$.
6. **Autoregressive Loop:**
   Next, $t_5 = \text{"the"}$ is fed in alone as the input, computes $q_5, k_5, v_5$, appends $k_5, v_5$ to the KV Cache, attends to $[k_1 \dots k_5]$, passes through the FFN, and outputs $t_6 = \text{"mat"}$. This loop repeats until an `<EOS>` (End-of-Sequence) token is generated.

### Architectural Block Diagram: Phase 2 (Decode Stage)

```text
NEW TOKEN ONLY:                      "on" (t4)
                                         │
                                         ▼
[Embedding W_E]                         x4
                                         │
               ┌─────────────────────────┴─────────────────────────┐
               │             TRANSFORMER LAYER (1 to L)            │
               │                                                   │
               │   1. Compute Q, K, V for t4 ONLY:                 │
               │                  [q4, k4, v4]                     │
               │                    │      │                       │
               │                    │      ▼                       │
               │                    │  ┌────────────────────────┐  │
               │                    │  │      GPU KV CACHE      │  │
               │                    │  │ Cached: k1,k2,k3,v1..v3│  │
               │                    │  │ + Append: k4, v4       │  │
               │                    │  └───────────┬────────────┘  │
               │                    ▼              ▼               │
               │   2. Self-Attention:                              │
               │      q4 attends to ALL [k1, k2, k3, k4]           │
               │      and weights ALL   [v1, v2, v3, v4]           │
               │                           │                       │
               │                           ▼                       │
               │   3. Feed-Forward Network (FFN) (for t4 only)     │
               └───────────────────────────┼───────────────────────┘
                                           │
                                           ▼
Final Hidden State:                       h4
                                           │
                                           ▼
                                [Linear + Softmax]
                                           │
                                           ▼
2nd GENERATED OUTPUT TOKEN:            "the" (t5) ───► (Loops back as input to generate "mat"!)
```

---

## 5. Summary Comparison: Prefill vs. Decode

| Feature | Phase 1: Prefill Stage (Input Processing) | Phase 2: Decode Stage (Output Generation) |
| :--- | :--- | :--- |
| **Input to the Model** | All $N$ prompt tokens simultaneously (`[t1, t2, ..., tN]`) | Only the $1$ most recently generated token (`[t_current]`) |
| **Layers Executed** | Embedding $\rightarrow$ Attention $\rightarrow$ FFN across all $L$ layers | Embedding $\rightarrow$ Attention $\rightarrow$ FFN across all $L$ layers |
| **Q, K, V Computed For** | All $N$ input tokens in parallel | Only the $1$ new token (`q_t, k_t, v_t`) |
| **KV Cache Operation** | **Populates (Writes)** initial $K$ and $V$ tensors for all $N$ input tokens | **Appends** new $k_t, v_t$ and **Reads** all past cached $[k_1 \dots k_t]$ and $[v_1 \dots v_t]$ |
| **Matrix Operation Type** | Matrix-Matrix Multiplication (`GEMM`) | Matrix-Vector Multiplication (`GEMV`) |
| **Hardware Bottleneck** | **Compute-Bound** (High Arithmetic Intensity) | **Memory-Bandwidth Bound** (Low Arithmetic Intensity) |
| **Primary Latency Metric** | **TTFT** (Time To First Token) | **TPOT** (Time Per Output Token) |
| **Token Output** | Generates the **1st output token** (from $h_N$) | Generates tokens **$2, 3, \dots, O$** sequentially |
