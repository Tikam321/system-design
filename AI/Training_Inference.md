# Comprehensive Guide to LLM Systems: Hardware Parallelism, Backpropagation Calculus, and Inference Optimization

This repository document provides an exhaustive, first-principles breakdown of how Large Language Models (LLMs) are trained and served at scale. It covers the physical hardware constraints, the mathematical foundations of tensors and distributed parallelism, step-by-step numerical walkthroughs of the forward and backward passes (Loss, Calculus, Chain Rule, and Gradient Descent), and real-time inference mechanics.

---

## Table of Contents
1. [Foundational Concepts: What is a Tensor?](#1-foundational-concepts-what-is-a-tensor)
2. [Hardware Architecture & Memory Bottlenecks](#2-hardware-architecture--memory-bottlenecks)
3. [Distributed Parallelism Strategies (5D Parallelism)](#3-distributed-parallelism-strategies-5d-parallelism)
4. [Training vs. Inference: The Fundamental Divide](#4-training-vs-inference-the-fundamental-divide)
5. [Deep Dive: The Forward Pass & Cross-Entropy Loss](#5-deep-dive-the-forward-pass--cross-entropy-loss)
6. [Deep Dive: The Backward Pass, Calculus, and the Chain Rule](#6-deep-dive-the-backward-pass-calculus-and-the-chain-rule)
7. [Complete Numerical Walkthrough: Forward & Backward Pass](#7-complete-numerical-walkthrough-forward--backward-pass)
8. [Single-GPU Optimization: FlashAttention](#8-single-gpu-optimization-flashattention)
9. [Inference-Time Systems & Optimization](#9-inference-time-systems--optimization)

---

## 1. Foundational Concepts: What is a Tensor?

In deep learning and linear algebra, a **Tensor** is a generalized mathematical container for storing numbers across $N$ dimensions. Every piece of information in an LLM—input tokens, contextual embeddings, attention weights, feed-forward matrices, and gradients—is stored and manipulated as a tensor on the GPU.

### The Hierarchy of Tensors
* **0D Tensor (Scalar):** A single number (e.g., the final Loss value `L = 2.45` or learning rate `α = 0.001`).
* **1D Tensor (Vector):** A 1-dimensional array of numbers (e.g., `[0.12, -0.45, 0.88]`). A single token's embedding of size $d_{\text{model}} = 4096$ is a 1D vector.
* **2D Tensor (Matrix):** A 2-dimensional grid of rows and columns, shaped `[Rows, Columns]`. All linear projection weights in a Transformer (such as Query $W_Q$, Key $W_K$, Value $W_V$, and Feed-Forward network weights $W_1, W_2$) are 2D matrices.
* **3D Tensor:** A cube of numbers, typically shaped `[Batch_Size, Sequence_Length, Hidden_Dimension]`. When a batch of sentences flows through a Transformer layer, the activations are represented as a 3D tensor.
* **4D Tensor:** Used in Multi-Head Attention, shaped `[Batch_Size, Num_Heads, Sequence_Length, Head_Dimension]`.

**Why the term "Tensor" matters for Systems Engineering:**
Because GPUs are specialized parallel matrix-multiplication engines (via Tensor Cores), scaling an LLM means slicing, routing, concatenating, and accumulating these massive multi-dimensional arrays across memory hierarchies and physical devices.

---

## 2. Hardware Architecture & Memory Bottlenecks

### 2.1 Compute vs. Memory Metrics
* **FLOPs (floating-point operations - lowercase 's'):** Measures the total **compute budget** or volume of mathematical operations required for a task.
* **FLOPS (Floating-Point Operations Per Second - uppercase 'S'):** Measures **compute speed**—how fast a piece of hardware can execute operations (e.g., TeraFLOPS / TFLOPS).
* **Memory Bandwidth:** The speed at which data can be transferred between memory storage and compute units, measured in **Terabytes per second (TB/s)**.

### 2.2 Inside a Modern GPU (HBM vs. SRAM)
A flagship GPU (such as the NVIDIA H100 or H200) consists of two primary memory tiers and on-chip compute units:

| Component | Location | Typical Capacity | Bandwidth (Transfer Speed) | Role in LLM Execution |
| :--- | :--- | :--- | :--- | :--- |
| **HBM (High Bandwidth Memory)** | Off-chip (VRAM) | ~80 GB – 141 GB | ~1 – 4.8 TB/s | Stores model weights, optimizer states, gradients, and the KV cache. Large capacity, but relatively slow to read/write. |
| **SRAM (Static RAM)** | On-chip (L1/L2 Cache & Registers) | ~20 MB – 50 MB | ~19 – 30+ TB/s | Ultra-fast working memory right next to the Tensor Cores. Holds tiny tiles of matrices currently being multiplied. |
| **Compute Units (SMs / Tensor Cores)** | On-chip | N/A | ~1,000 – 4,000 TFLOPS (FP16/FP8) | Executes massive parallel dot products and matrix multiplications. |

### 2.3 The Napkin Math of LLM Memory (Why One GPU is Not Enough)
Suppose we have a **100-Billion Parameter ($100 \times 10^9$) Model** stored in 16-bit precision (**FP16** or **BF16**, where $16 \text{ bits} = 2 \text{ bytes}$):

1. **Model Weights (Parameters):**
   $$100 \text{B} \times 2 \text{ bytes} = 200 \text{ GB}$$
   *(This already exceeds the 141 GB HBM limit of a single NVIDIA H200 GPU!)*
2. **Gradients (Backward Pass):**
   Every parameter requires a matching gradient to record its error slope:
   $$100 \text{B} \times 2 \text{ bytes} = 200 \text{ GB}$$
3. **Optimizer States (AdamW):**
   Standard AdamW maintains FP32 (4-byte) master weights, first moments (momentum $m$), and second moments (variance $v$):
   $$100 \text{B} \times (4 + 4 + 4) \text{ bytes} = 1,200 \text{ GB}$$
4. **Forward Activations:**
   Intermediate layer outputs saved during the forward pass so the backward pass can compute derivatives via the Chain Rule. Scales dynamically with `Batch_Size × Sequence_Length × Layers × Hidden_Dim`.

**Total Training Memory:** Well over **1.6 Terabytes (TB)**. Therefore, multi-GPU distributed strategies are mandatory.

---

## 3. Distributed Parallelism Strategies (5D Parallelism)

To fit models into memory and accelerate training/inference, engineers divide workloads across two axes: **Data** and **Model**.

### 3.1 Data Parallelism (DP) & ZeRO
In **Data Parallelism**, the training dataset batch is partitioned across multiple GPUs, while each GPU processes its own mini-batch through the model layers.

* **Naive Data Parallelism:**
  * Every GPU holds a **complete, identical copy** of the entire model's weights, gradients, and optimizer states.
  * If you have a global batch of 64 sequences and 4 GPUs, GPU 1 processes sequences 1–16, GPU 2 processes 17–32, etc.
  * During the backward pass, all GPUs communicate via an **All-Reduce** operation to average their calculated gradients, ensuring every GPU updates its weights identically.
  * *Limitation:* Fails if the model itself is too large to fit on a single GPU, and wastes immense memory duplicating identical states across devices.

* **ZeRO (Zero Redundancy Optimizer / FSDP):**
  Eliminates redundant copies in Data Parallelism by **sharding (partitioning)** the model states across the data-parallel GPUs:
  * **ZeRO-1:** Shards only the **Optimizer States** across GPUs (reduces memory by ~4x).
  * **ZeRO-2:** Shards **Optimizer States + Gradients** across GPUs (reduces memory by ~8x).
  * **ZeRO-3:** Shards **Optimizer States + Gradients + Model Parameters** across GPUs.
  * *Why ZeRO-3 is still Data Parallelism (and not Model Parallelism):* Even though parameters are sharded, each GPU is still responsible for executing the full forward and backward computation for its own distinct sub-batch of data. When GPU 1 reaches Layer $k$ and needs parameters stored on GPU 2, it fetches them on-the-fly (`All-Gather`), runs the full computation locally on its data batch, and discards the borrowed parameters immediately after.

### 3.2 Model Parallelism (MP)
Instead of each GPU running all model computations on a slice of data, **Model Parallelism splits the model's computational graph across devices**. Each GPU only computes a specific subset of the model's operations.

#### A. Tensor Parallelism (TP) — Intra-Layer (Horizontal Slicing)
**Tensor Parallelism** slices the individual weight matrices (**tensors**) *inside a single layer* across multiple GPUs so they compute parts of the **exact same layer simultaneously**.

* **Feed-Forward Network Example (Column Parallelism):**
  Suppose a linear layer computes $Y = XW$, where input $X$ has shape `[Batch, d]` and weight matrix $W$ has shape `[d, 2d]`. We slice $W$ vertically along its columns into two sub-matrices: $W = [W_1 \mid W_2]$.
  * **GPU 1** holds $W_1$ and computes $Y_1 = XW_1$.
  * **GPU 2** holds $W_2$ and computes $Y_2 = XW_2$.
  * Both GPUs execute millions of matrix multiplications in parallel. They then exchange their partial results and concatenate them: $Y = [Y_1 \mid Y_2]$.
* **Multi-Head Attention Example:**
  If a Transformer layer has 32 attention heads, GPU 1 holds the $W_Q, W_K, W_V$ tensors for Heads 1–16, and GPU 2 holds Heads 17–32. Both compute attention in parallel, project their outputs, and sum/concatenate them via an `All-Reduce` operation.
* **Hardware Requirement (NVIDIA NVLink):** Because GPUs must synchronize and merge partial tensors *twice per Transformer layer* (once after Attention, once after the FFN), Tensor Parallelism requires ultra-high-bandwidth, low-latency interconnects like **NVIDIA NVLink** (up to 900 GB/s – 1.8 TB/s) within a **single multi-GPU server node**.

#### B. Pipeline Parallelism (PP) — Inter-Layer (Vertical Slicing)
Splits the model sequentially by **layers** across different GPUs or server nodes.
* **Mechanism:** In a 32-layer LLM across 4 GPUs:
  * GPU 0 holds Layers 1–8
  * GPU 1 holds Layers 9–16
  * GPU 2 holds Layers 17–24
  * GPU 3 holds Layers 25–32
* GPU 0 computes Layers 1–8 on the input and sends the final activation tensor to GPU 1.
* **The "Pipeline Bubble" Problem:** While GPU 0 waits for GPUs 1, 2, and 3 to finish the forward pass AND send gradients back during the backward pass, GPU 0 sits completely idle. Engineers mitigate this idle "bubble" by splitting batches into smaller micro-batches (e.g., GPipe, 1F1B schedules) so multiple micro-batches are in flight simultaneously.

#### C. Context Parallelism (CP) / Ring Attention
Splits the input and activations along the **Sequence Length ($N$) dimension**.
* When training on ultra-long contexts (e.g., 128K to 1M tokens), a single GPU cannot hold the activations for all tokens.
* Device 1 holds tokens $1 \dots N/2$ (computing $Q_1, K_1, V_1$) and Device 2 holds tokens $N/2+1 \dots N$ (computing $Q_2, K_2, V_2$).
* Devices keep their Queries fixed and pass their Key ($K$) and Value ($V$) blocks around in a ring topology so every query attends to every key across the entire cluster.

#### D. Expert Parallelism (EP)
Used specifically in **Mixture-of-Experts (MoE)** models (such as Mixtral or DeepSeek-V3), where the dense Feed-Forward Network is replaced by multiple distinct "Expert" FFN sub-networks.
* Different experts are placed on different GPUs.
* A lightweight gating router inspects each token's representation after self-attention and dispatches the token (via an `All-to-All` network communication) only to the specific GPU hosting its assigned expert.

### 3.3 Summary Comparison of Parallelism Methods

| Parallelism Type | What is Partitioned? | Communication Overhead | Typical Hardware Scope |
| :--- | :--- | :--- | :--- |
| **Data Parallelism (DP / ZeRO)** | Training Data Batch (and sharded states in ZeRO) | Moderate (Gradient `All-Reduce` or parameter `All-Gather`) | Across multiple nodes |
| **Tensor Parallelism (TP)** | Intra-layer weight matrices & attention heads | **Very High** (Sync required inside every single layer) | Within a single server node via **NVLink** |
| **Pipeline Parallelism (PP)** | Sequential blocks of Transformer layers | Low/Moderate (Only boundary activations passed between stages) | Across multiple server nodes |
| **Context Parallelism (CP)** | Token sequence length ($N$) | Moderate/High ($K, V$ tensors passed in a ring) | Within or across high-speed nodes |
| **Expert Parallelism (EP)** | MoE Feed-Forward Expert sub-networks | High (`All-to-All` token routing per layer) | Across multi-GPU clusters |

---

## 4. Training vs. Inference: The Fundamental Divide

A frequent point of confusion is how the model knows the "True Target" probability and when weight updates actually occur. The behavior of an LLM is strictly divided into two distinct lifecycle phases:

| Characteristic | Phase 1: Training Time | Phase 2: Inference Time (Real-Time Serving) |
| :--- | :--- | :--- |
| **Primary Objective** | Learn language patterns by adjusting model weights to minimize prediction error. | Generate helpful text responses rapidly for users using learned knowledge. |
| **State of Model Weights** | **Dynamic / Mutable:** Updated continuously after every batch. | **Frozen / Read-Only:** Locked permanently; never modified during user chats. |
| **Do we have a "True Target"?** | **YES.** Uses Self-Supervised Learning on existing text (books, web, code). The very next word in the training document is the known 100% Ground Truth. | **NO.** The model is generating brand-new text in real time; there is no ground-truth answer sheet to compare against. |
| **Passes Executed** | **Forward Pass + Loss Calculation + Backward Pass + Optimizer Step.** | **Forward Pass ONLY.** |
| **Memory Footprint** | Massive (Weights + Gradients + Optimizer States + Forward Activations). | Much smaller (Weights + Dynamic KV Cache). |

---

## 5. Deep Dive: The Forward Pass & Cross-Entropy Loss

During **Training**, we play a massive game of "fill-in-the-blank" (Autoregressive Next-Token Prediction).

### 5.1 How the "True Target" is Created (Self-Supervised Learning)
Suppose our training dataset contains the sentence:
> `"The cat sits on the mat"`

We slice this sentence so the model sees the prefix and must predict the immediate next token:
* **Input Sequence ($X$):** `["The", "cat", "sits", "on", "the"]`
* **True Target Token ($y$):** `"mat"`

Because `"mat"` is literally the next word in our existing training file, we know with **100% certainty ($1.0$ probability)** that `"mat"` is the target for this context.

Suppose our simplified vocabulary contains only $V = 4$ words:
`Vocab = ["dog", "mat", "chair", "floor"]` (where `"mat"` is at index `1`).

We represent the **True Target ($y$)** as a **One-Hot Encoded Vector**:
$$y = [0.0, \ 1.0, \ 0.0, \ 0.0]$$
* `"dog"` = $0.0$
* `"mat"` = $1.0$ (100% Ground Truth)
* `"chair"` = $0.0$
* `"floor"` = $0.0$

### 5.2 Forward Pass: From Input to Softmax Probabilities
1. **Embedding & Transformer Layers:** The input tokens flow through the Masked Multi-Head Attention and Feed-Forward layers, producing a final hidden state vector $h$ for the last token (`"the"`).
2. **Unembedding / Logits ($z$):** The hidden state $h$ is multiplied by the output projection matrix $W_{\text{vocab}}$ to produce raw, unnormalized scores called **Logits ($z$)**:
   $$z = [1.0, \ 2.0, \ 0.5, \ -1.0]$$
3. **Softmax Conversion ($\hat{y}$):** To turn raw logits into a valid probability distribution (where all values are between $0$ and $1$ and sum to $1.0$), we apply the **Softmax function**:
   $$\hat{y}_i = \text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{V} e^{z_j}}$$
   Let's compute the exponentials:
   * $e^{1.0} \approx 2.718$
   * $e^{2.0} \approx 7.389$ (for `"mat"`)
   * $e^{0.5} \approx 1.649$
   * $e^{-1.0} \approx 0.368$
   * **Sum of exponentials:** $2.718 + 7.389 + 1.649 + 0.368 = 12.124$

   Dividing each by the sum gives the model's **Predicted Probability Distribution ($\hat{y}$)**:
   * $\hat{y}_0 (\text{"dog"}) = 2.718 / 12.124 \approx 0.224$ ($22.4\%$)
   * $\hat{y}_1 (\text{"mat"}) = 7.389 / 12.124 \approx 0.609$ ($60.9\%$)
   * $\hat{y}_2 (\text{"chair"}) = 1.649 / 12.124 \approx 0.136$ ($13.6\%$)
   * $\hat{y}_3 (\text{"floor"}) = 0.368 / 12.124 \approx 0.030$ ($3.0\%$)

### 5.3 Calculating Cross-Entropy Loss
Now we compare the **True Target $y = [0, 1, 0, 0]$** with the **Prediction $\hat{y} = [0.224, 0.609, 0.136, 0.030]$** using **Categorical Cross-Entropy Loss**:

$$L = - \sum_{i=0}^{V-1} y_i \ln(\hat{y}_i)$$

Because $y_i = 0$ for every wrong word and $y_1 = 1.0$ for the true word (`"mat"`), all zero terms cancel out, leaving only the negative natural log of the predicted probability for the true target token:

$$L = - (1.0 \times \ln(0.609)) = -(-0.496) = 0.496$$

* **Intuition:**
  * If the model had predicted $\hat{y}_{\text{mat}} = 0.99$, then $L = -\ln(0.99) = 0.01$ (almost zero penalty).
  * If the model had predicted $\hat{y}_{\text{mat}} = 0.01$, then $L = -\ln(0.01) = 4.605$ (massive penalty).

---

## 6. Deep Dive: The Backward Pass, Calculus, and the Chain Rule

Once we have the Loss $L$, how do we adjust a weight $W_{\text{attn}}$ buried deep inside Layer 1's Attention block, or a weight $W_{\text{ffn}}$ inside Layer 15's Feed-Forward Network?

### 6.1 What is a Derivative / Gradient in Deep Learning?
In calculus, a **derivative** measures sensitivity—the **slope** or rate of change of one quantity with respect to another.
For any specific weight $w$ in our neural network, the partial derivative $\frac{\partial L}{\partial w}$ answers the exact question:
> *"If I nudge this specific weight $w$ upward by a microscopic amount $\Delta w$, how much and in what direction does the final Loss $L$ change?"*

* **If $\frac{\partial L}{\partial w} > 0$ (Positive Slope):** Increasing $w$ makes the Loss go **UP** (worse). Therefore, we must **decrease** $w$.
* **If $\frac{\partial L}{\partial w} < 0$ (Negative Slope):** Increasing $w$ makes the Loss go **DOWN** (better). Therefore, we must **increase** $w$.
* **If $\frac{\partial L}{\partial w} = 0$ (Flat Slope):** We have reached a local minimum (the bottom of the valley).

A **Gradient ($\nabla_W L$)** is simply a tensor collecting the partial derivatives for every individual number inside a weight matrix $W$.

### 6.2 Why We Need the Chain Rule of Calculus
A neural network is a nested composition of functions (a computational chain):
1. Attention Layer computes $a_1 = f_1(x, W_{\text{attn}})$
2. Feed-Forward Layer computes $a_2 = f_2(a_1, W_{\text{ffn}})$
3. Output Layer computes prediction $\hat{y} = f_3(a_2, W_{\text{out}})$
4. Loss function computes error $L = \text{Loss}(\hat{y}, y)$

Because $W_{\text{attn}}$ inside the Attention layer does not directly touch $L$, we cannot take its derivative in one jump. Instead, calculus provides the **Chain Rule**: if variable $L$ depends on $\hat{y}$, which depends on $a_2$, which depends on $a_1$, which depends on $W_{\text{attn}}$, we multiply the local step-by-step derivatives backward along the chain:

$$\frac{\partial L}{\partial W_{\text{attn}}} = \underbrace{\frac{\partial L}{\partial \hat{y}}}_{\text{Loss slope}} \cdot \underbrace{\frac{\partial \hat{y}}{\partial a_2}}_{\text{Output layer slope}} \cdot \underbrace{\frac{\partial a_2}{\partial a_1}}_{\text{FFN layer slope}} \cdot \underbrace{\frac{\partial a_1}{\partial W_{\text{attn}}}}_{\text{Attention weight slope}}$$

### 6.3 How Attention vs. Feed-Forward Layers Get Their Gradients
During the **Backward Pass (Backpropagation)**:
1. **At the Output Layer:** We first compute the derivative of the Cross-Entropy Loss with respect to the logits $z$. Remarkably, the calculus of Softmax + Cross-Entropy simplifies to a clean subtraction:
   $$\frac{\partial L}{\partial z} = \hat{y} - y$$
   *(In our `"mat"` example: $[0.224, 0.609, 0.136, 0.030] - [0, 1, 0, 0] = [+0.224, -0.391, +0.136, +0.030]$. Notice the negative gradient $-0.391$ on `"mat"`—telling the model to push `"mat"` higher—and positive gradients on `"dog"`, `"chair"`, and `"floor"`—telling the model to push them lower!)*
2. **At the Feed-Forward Network (FFN):** The error signal flows backward into the FFN. To find the gradient for $W_{\text{ffn}}$, the GPU multiplies the incoming upstream gradient by the transpose of the FFN's input activation (which was saved in GPU memory during the forward pass). It also passes the error signal further backward toward the Attention layer.
3. **At the Attention Layer:** The error signal reaches the Attention output and splits backward through the Value ($W_V$), Key ($W_K$), and Query ($W_Q$) matrices using the exact same Chain Rule multiplication.

---

## 7. Complete Numerical Walkthrough: Forward & Backward Pass

To make the calculus 100% concrete, let's trace a mini 2-stage neural network (representing an Attention score feeding into a Feed-Forward layer) with real numbers.

### 7.1 Network Setup
Suppose we have:
* **Input token value ($x$):** $2.0$
* **Layer 1 Weight ($w_1$ — e.g., Attention weight):** $1.5$
* **Layer 2 Weight ($w_2$ — e.g., FFN weight):** $3.0$
* **Bias ($b$):** $1.0$
* **True Target ($y$):** $20.0$
* **Learning Rate ($\alpha$):** $0.01$

### 7.2 Step 1: Forward Pass (Computing Activations and Loss)
1. **Layer 1 (Attention computation):**
   $$a_1 = w_1 \times x = 1.5 \times 2.0 = 3.0$$
   *(Note: The GPU saves $x = 2.0$ and $a_1 = 3.0$ in memory for the backward pass!)*
2. **Layer 2 (FFN computation):**
   $$\hat{y} = (w_2 \times a_1) + b = (3.0 \times 3.0) + 1.0 = 9.0 + 1.0 = 10.0$$
3. **Loss Calculation (Squared Error):**
   $$L = (\hat{y} - y)^2 = (10.0 - 20.0)^2 = (-10.0)^2 = 100.0$$

Our initial **Loss is $100.0$**. Now we must reduce it.

### 7.3 Step 2: Backward Pass (Computing Gradients via the Chain Rule)

#### A. Gradient of Loss with respect to Prediction ($\hat{y}$):
Using the power rule of calculus on $L = (\hat{y} - y)^2$:
$$\frac{\partial L}{\partial \hat{y}} = 2(\hat{y} - y) = 2(10.0 - 20.0) = -20.0$$

#### B. Gradient for Layer 2 Weight ($w_2$ — FFN Weight):
Since $\hat{y} = w_2 \cdot a_1 + b$, the local derivative of $\hat{y}$ with respect to $w_2$ is simply its input activation $a_1$:
$$\frac{\partial \hat{y}}{\partial w_2} = a_1 = 3.0$$

Applying the **Chain Rule**:
$$\frac{\partial L}{\partial w_2} = \frac{\partial L}{\partial \hat{y}} \times \frac{\partial \hat{y}}{\partial w_2} = (-20.0) \times 3.0 = -60.0$$

#### C. Gradient for Layer 1 Weight ($w_1$ — Attention Weight):
To reach $w_1$, we must chain one step deeper through $a_1$:
* Local derivative of $\hat{y}$ with respect to $a_1$:
  $$\frac{\partial \hat{y}}{\partial a_1} = w_2 = 3.0$$
* Local derivative of $a_1 = w_1 \cdot x$ with respect to $w_1$:
  $$\frac{\partial a_1}{\partial w_1} = x = 2.0$$

Applying the **3-part Chain Rule**:
$$\frac{\partial L}{\partial w_1} = \frac{\partial L}{\partial \hat{y}} \times \frac{\partial \hat{y}}{\partial a_1} \times \frac{\partial a_1}{\partial w_1} = (-20.0) \times 3.0 \times 2.0 = -120.0$$

### 7.4 Step 3: Gradient Descent (Updating the Weights)
Now we apply the Gradient Descent update rule:
$$w_{\text{new}} = w_{\text{old}} - \left(\alpha \times \frac{\partial L}{\partial w}\right)$$

1. **Update Layer 2 Weight ($w_2$):**
   $$w_{2,\text{new}} = 3.0 - (0.01 \times -60.0) = 3.0 - (-0.60) = 3.60$$
2. **Update Layer 1 Weight ($w_1$):**
   $$w_{1,\text{new}} = 1.5 - (0.01 \times -120.0) = 1.5 - (-1.20) = 2.70$$

### 7.5 Step 4: Verification (Next Forward Pass)
Let's run the exact same input $x = 2.0$ through our newly updated weights:
1. $a_1 = w_{1,\text{new}} \times x = 2.70 \times 2.0 = 5.40$
2. $\hat{y}_{\text{new}} = (w_{2,\text{new}} \times a_1) + b = (3.60 \times 5.40) + 1.0 = 19.44 + 1.0 = 20.44$
3. $L_{\text{new}} = (20.44 - 20.0)^2 = (0.44)^2 = 0.1936$

**Result:** In a single step of calculus and gradient descent, our prediction moved from **$10.0 \rightarrow 20.44$** (target $20.0$), and our Loss plummeted from **$100.0 \rightarrow 0.1936$**!

---

## 8. Single-GPU Optimization: FlashAttention

In standard Self-Attention, we compute:
$$\text{Attention}(Q, K, V) = \text{Softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

### 8.1 The Memory Bottleneck of Standard Attention
If sequence length is $N$ (e.g., $N = 32,768$), the matrix $S = QK^T$ has dimensions $N \times N$ ($>1$ billion entries).
In naive attention:
1. $Q$ and $K$ are read from slow HBM into SRAM, multiplied to form $S$, and $S$ ($N \times N$) is written back to slow HBM.
2. $S$ is read from HBM into SRAM to compute $\text{Softmax}(S) = P$, and $P$ ($N \times N$) is written back to HBM.
3. $P$ and $V$ are read from HBM into SRAM to compute the output $O = PV$, which is written to HBM.

Because standard Softmax requires summing the exponentials across the **entire row** before dividing, naive implementations materialize the full $N \times N$ matrix in slow HBM.

### 8.2 How FlashAttention Solves It (Online Softmax & Tiling)
**FlashAttention** is an **exact** algorithm (zero approximation) that never materializes the $N \times N$ matrix in HBM:
1. **Iterative Numerator & Denominator Fusion:** It breaks $Q, K, V$ into small tiles that fit entirely inside fast on-chip **SRAM** and maintains running sums for the Softmax numerator ($N_t$) and denominator ($D_t$) multiplied directly by $V_t$.
2. **Numerical Stability via Safe Rescaling ($m_t$):** Because $e^{s_i}$ overflows in FP16 when $s_i$ is large, FlashAttention tracks the running maximum score encountered so far ($m_t = \max(m_{t-1}, s_t)$). Whenever a new block has a larger maximum $m_t$, it dynamically rescales the previous running sums by multiplying them by the correction factor $e^{m_{t-1} - m_t}$:
   $$N_t = e^{m_{t-1} - m_t} N_{t-1} + e^{s_t - m_t} v_t$$
   $$D_t = e^{m_{t-1} - m_t} D_{t-1} + e^{s_t - m_t}$$
3. **Backward Pass Recomputation:** Instead of saving the huge $N \times N$ attention matrix in HBM during the forward pass for the backward pass, FlashAttention discards it and **recomputes it on-the-fly in SRAM** during the backward pass. Even though it performs *more FLOPs*, avoiding slow HBM reads/writes makes total training runtime significantly faster!

---

## 9. Inference-Time Systems & Optimization

At inference time, weights are frozen and generation splits into two distinct phases:

### 9.1 Prefill vs. Decode & Latency Metrics
$$T_{\text{request}} = T_{\text{queue}} + T_{\text{prefill}} + (O - 1) \times T_{\text{per\_output\_token}}$$

1. **Prefill Phase (Prompt Processing):**
   * All input tokens in the user's prompt are processed **in parallel** in a single forward pass.
   * **Compute-Bound** on the Roofline Model (high Arithmetic Intensity: many FLOPs per byte of memory moved).
   * Determines **TTFT (Time To First Token)**.
2. **Decode Phase (Autoregressive Token Generation):**
   * Tokens are generated sequentially **one by one** ($O$ output tokens), because token $t$ depends on token $t-1$.
   * **Memory-Bandwidth Bound** on the Roofline Model (low Arithmetic Intensity: the GPU must load 100% of the model weights from HBM into SRAM just to generate 1 single token).
   * Determines **TPOT (Time Per Output Token)**.

### 9.2 KV Caching & Memory Compression
To avoid recomputing Keys ($K$) and Values ($V$) for all preceding tokens at every decode step, the GPU stores past $K$ and $V$ vectors in HBM (**KV Cache**). To prevent the KV Cache from exhausting GPU memory, modern systems use three key innovations:

1. **PagedAttention (vLLM):**
   Eliminates memory fragmentation. Instead of pre-allocating a massive contiguous chunk of HBM for the maximum possible sequence length of every request, PagedAttention divides the KV Cache into fixed-size non-contiguous blocks (e.g., $B = 16$ tokens per block) tracked via a virtual memory lookup table.
2. **GQA (Grouped-Query Attention):**
   Instead of maintaining $H$ separate Key and Value heads to match $H$ Query heads (Multi-Head Attention), GQA groups queries so that $H$ Query heads share only $G$ Key/Value heads (where $1 < G < H$). For example, with $H=24$ and $G=2$, KV Cache memory drops by $1 - \frac{2}{24} = 91.6\%$!
3. **MLA (Multi-head Latent Attention):**
   Used in architectures like DeepSeek. Compresses all Key and Value heads for a token into a **single low-rank shared latent vector** via down-projection before caching, reconstructing the full $K$ and $V$ heads on-the-fly during computation.

### 9.3 Speculative Decoding (Accelerating the Decode Phase)
Because verifying $K$ tokens in parallel takes nearly the exact same time on a GPU as generating $1$ token (due to the Decode phase being memory-bandwidth bound), **Speculative Decoding** uses two models:
1. **Draft Step:** A tiny, fast **Draft Model** ($D$) rapidly generates $K$ candidate tokens autoregressively ($d_1, d_2, \dots, d_K$).
2. **Verification Step:** The massive **Target Model** ($T$) processes all $K$ draft tokens **in a single parallel forward pass** to compute its exact probabilities $T(x)$.
3. **Rejection Sampling (Exact Distribution Preservation):**
   For each drafted token $x$:
   * If $T(x) \ge D(x)$ (Target is at least as confident as Draft), **accept** the token immediately (probability $1$).
   * If $T(x) < D(x)$, accept the token with probability $\frac{T(x)}{D(x)}$.
   * If rejected, discard all subsequent draft tokens and sample a replacement token directly from the **residual distribution**:
     $$P_{\text{residual}}(x) = \frac{\max(0, \ T(x) - D(x))}{\sum_{x'} \max(0, \ T(x') - D(x'))}$$
   * By the Law of Total Probability:
     $$P(X = x) = \min(D(x), T(x)) + \max(0, T(x) - D(x)) = T(x)$$
   This mathematically guarantees **zero loss in output quality**—the final tokens are sampled from the exact probability distribution of the massive Target Model, while achieving a $2\times - 3\times$ wall-clock speedup!
