# The Roofline Model in LLM Systems: Arithmetic Intensity, Hardware Bottlenecks, and FlashAttention

This reference document provides a comprehensive breakdown of the **Roofline Model**, a foundational performance analysis framework for understanding how Large Language Models (LLMs) interact with GPU hardware. It covers the core metrics of compute and memory, step-by-step mathematical derivations of **Arithmetic Intensity** for both the Decode and Prefill phases, the geometry of the Roofline graph (Slanted Roof, Ridge Point, and Flat Roof), hardware trade-offs between HBM and SRAM, and how **FlashAttention** shifts workloads into the compute-bound regime.

---

## Table of Contents
1. [Core Hardware Metrics: Compute vs. Memory](#1-core-hardware-metrics-compute-vs-memory)
2. [Deep Dive: What is Arithmetic Intensity?](#2-deep-dive-what-is-arithmetic-intensity)
3. [Step-by-Step Math: Calculating FLOPs per Byte](#3-step-by-step-math-calculating-flops-per-byte)
4. [Anatomy of the Roofline Model Graph](#4-anatomy-of-the-roofline-model-graph)
5. [Hardware & Software Trade-offs: HBM vs. SRAM](#5-hardware--software-trade-offs-hbm-vs-sram)
6. [FlashAttention and Why Prefill is Compute-Bound](#6-flashattention-and-why-prefill-is-compute-bound)

---

## 1. Core Hardware Metrics: Compute vs. Memory

When evaluating GPU performance for LLM training and inference, two hardware subsystems compete to avoid becoming the bottleneck: **Compute Units (Tensor Cores)** and **Memory Subsystems (HBM and SRAM)**.

### 1.1 The Three Fundamental Definitions
1. **Computational Throughput (FLOPS — Floating-Point Operations Per Second):**
   * **What it measures:** The maximum speed at which the GPU's on-chip compute units can execute mathematical operations (multiplications and additions).
   * **Units:** TeraFLOPS ($	ext{TFLOPS}$, $10^{12}$ ops/sec) or PetaFLOPS ($	ext{PFLOPS}$, $10^{15}$ ops/sec).
   * **Dependency:** Highly dependent on numerical precision (data type). For example, on an NVIDIA H200 GPU, peak compute throughput doubles when moving from 16-bit precision (`FP16`/`BF16` at $\sim 989	ext{ TFLOPS}$) to 8-bit precision (`FP8` at $\sim 1,979	ext{ TFLOPS}$).

2. **Memory Bandwidth (Bytes/second):**
   * **What it measures:** The rate at which data can be transferred across the memory bus between off-chip **High Bandwidth Memory (HBM)** and on-chip **SRAM / Compute Units**.
   * **Units:** Gigabytes per second ($	ext{GB/s}$) or Terabytes per second ($	ext{TB/s}$). On an NVIDIA H200, HBM bandwidth is roughly $4.8	ext{ TB/s}$.

3. **Arithmetic Intensity (FLOPs/Byte):**
   * **What it measures:** The ratio of total floating-point operations performed to total bytes of data transferred across the memory bus:
     $$	ext{Arithmetic Intensity} = rac{	ext{Total Compute Operations (FLOPs)}}{	ext{Total Memory Moved (Bytes)}}$$
   * **Units:** $	ext{FLOPs/byte}$.

---

## 2. Deep Dive: What is Arithmetic Intensity?

**Arithmetic Intensity** quantifies **operational efficiency**—specifically, how much mathematical work you extract from every single byte of data you pay the cost to move between slow off-chip HBM and fast on-chip SRAM.

### 2.1 How "Bytes Transferred" Are Physically Counted on a GPU
To understand Arithmetic Intensity, we must look at the physical layout of a GPU:

```text
┌───────────────────────────────────────────────────────────────────┐
│                         GPU PACKAGE                               │
│                                                                   │
│  ┌──────────────────────┐              ┌───────────────────────┐  │
│  │   Off-Chip HBM       │  Memory Bus  │   On-Chip SRAM        │  │
│  │ (High Bandwidth Mem) │ ───────────► │ (L1/L2 Cache & Regs)  │  │
│  │                      │ ◄─────────── │                       │  │
│  │ Capacity: ~80-141 GB │  ~1-4.8 TB/s │ Capacity: ~20-50 MB   │  │
│  │ Speed: "Slow"        │              │ Speed: ~20-30+ TB/s   │  │
│  └──────────────────────┘              └───────────┬───────────┘  │
│                                                    │              │
│                                                    ▼              │
│                                        ┌───────────────────────┐  │
│                                        │  On-Chip Compute      │  │
│                                        │  (Tensor Cores / SMs) │  │
│                                        │  Speed: ~1000+ TFLOPS │  │
│                                        └───────────────────────┘  │
└───────────────────────────────────────────────────────────────────┘
```

1. **Where data lives:** All model weights ($W$), input activations ($X$), and KV caches reside in **off-chip HBM**.
2. **Where math happens:** Matrix multiplications happen inside the **on-chip Compute Units**, which can only operate on data currently loaded into **on-chip SRAM**.
3. **Counting the Bytes:** Every time a layer executes, the GPU must:
   * **Read** input tensors and weight matrices from HBM into SRAM.
   * **Compute** the floating-point operations inside the Tensor Cores.
   * **Write** the resulting output tensor from SRAM back to HBM.
   
Therefore, the denominator of Arithmetic Intensity (**Bytes Transferred**) is the exact sum of all **HBM Reads + HBM Writes** required to complete that operation.

---

## 3. Step-by-Step Math: Calculating FLOPs per Byte

Let us calculate the exact Arithmetic Intensity for a linear layer multiplying an input activation by a weight matrix $W$ of shape $[K 	imes M]$, stored in **FP16 precision** ($16	ext{ bits} = 2	ext{ bytes}$ per parameter).

### Case A: Decoding 1 Token at a Time (Matrix-Vector Multiplication / GEMV)
During the **Autoregressive Decode Phase**, the model generates tokens one by one. At each step, we feed a single token vector $x$ of shape $[1 	imes K]$ and multiply it by the weight matrix $W$ of shape $[K 	imes M]$ to produce an output vector $y$ of shape $[1 	imes M]$.

1. **Total Math Operations (FLOPs):**
   * Each element of the output vector $y$ (length $M$) is a dot product of length $K$ ($K$ multiplications and $K$ additions = $2K$ FLOPs).
   * Across all $M$ output elements:
     $$	ext{Total FLOPs} = 2 	imes M 	imes K = 2MK	ext{ FLOPs}$$

2. **Total Memory Moved (Bytes):**
   * **Read Weight Matrix $W$ ($K 	imes M$):** $2MK	ext{ bytes}$ (at 2 bytes per FP16 parameter).
   * **Read Input Vector $x$ ($1 	imes K$):** $2K	ext{ bytes}$.
   * **Write Output Vector $y$ ($1 	imes M$):** $2M	ext{ bytes}$.
   * Total Bytes: $2MK + 2K + 2M	ext{ bytes}$. Because $M$ and $K$ are large in LLMs (e.g., $4096$ or $8192$), the weight matrix term $2MK$ completely dominates $2K + 2M$:
     $$	ext{Total Bytes} pprox 2MK	ext{ bytes}$$

3. **Arithmetic Intensity (Case A):**
   $$	ext{Arithmetic Intensity}_{	ext{decode}} = rac{2MK	ext{ FLOPs}}{2MK + 2K + 2M	ext{ bytes}} pprox rac{2MK}{2MK} = \mathbf{1	ext{ FLOP/byte}}$$

* **Takeaway:** During single-token decoding, you transfer **2 bytes** of weight data from slow HBM just to perform **2 FLOPs** of math. Arithmetic Intensity is rock-bottom ($pprox 1	ext{ FLOP/byte}$), making the Decode phase severely **Memory-Bandwidth Bound**.

---

### Case B: Prefilling $B$ Tokens in Parallel (Matrix-Matrix Multiplication / GEMM)
During the **Prefill Phase** (or batched processing), all $B$ tokens (e.g., a $B = 1,000$ token prompt) are known upfront. The input is now a matrix $X$ of shape $[B 	imes K]$ multiplied by the **exact same** weight matrix $W$ of shape $[K 	imes M]$ to produce an output matrix $Y$ of shape $[B 	imes M]$.

1. **Total Math Operations (FLOPs):**
   * We perform the $2MK$ matrix operations for all $B$ tokens in parallel:
     $$	ext{Total FLOPs} = 2 	imes B 	imes M 	imes K = 2BMK	ext{ FLOPs}$$

2. **Total Memory Moved (Bytes):**
   * **Read Weight Matrix $W$ ($K 	imes M$) ONCE:** $2MK	ext{ bytes}$. *(Because all $B$ tokens are processed together, tiles of $W$ are loaded into SRAM once and reused across all $B$ tokens!)*
   * **Read Input Matrix $X$ ($B 	imes K$):** $2BK	ext{ bytes}$.
   * **Write Output Matrix $Y$ ($B 	imes M$):** $2BM	ext{ bytes}$.
   * Total Bytes: $2MK + 2BK + 2BM	ext{ bytes}$.

3. **Arithmetic Intensity (Case B):**
   $$	ext{Arithmetic Intensity}_{	ext{prefill}} = rac{2BMK}{2MK + 2BK + 2BM}$$
   When model dimensions $M, K$ are large relative to $B$, $2MK$ dominates the denominator, and the expression simplifies to:
   $$	ext{Arithmetic Intensity}_{	ext{prefill}} pprox rac{2BMK}{2MK} = \mathbf{B	ext{ FLOPs/byte}}$$

* **Takeaway:** By processing $B = 1,000$ tokens in parallel, Arithmetic Intensity scales linearly with $B$—jumping from **$1	ext{ FLOP/byte}$** up to hundreds of **$	ext{FLOPs/byte}$**!

---

## 4. Anatomy of the Roofline Model Graph

The **Roofline Model** plots **Arithmetic Intensity ($	ext{FLOPs/byte}$)** on the horizontal X-axis against **Achievable Computational Throughput ($	ext{FLOPS}$)** on the vertical Y-axis (typically on a log-log scale).

```text
Computational Throughput (FLOPS - Operations/sec)
  ▲
  │                        Ridge Point
  │                (Neither Memory nor Compute Bound)
  │                             │
  │   Peak GPU Compute Limit    ▼____________________________________
  │   (Max Hardware TFLOPS) ────┼───────────────────────────────────/ ◄── FLAT ROOF (Green/Orange Zone)
  │                            /│                                         Compute-Bound Region
  │                           / │                                         (Prefill Phase / FlashAttention)
  │                          /  │
  │       SLANTED ROOF      /   │
  │      (Purple Zone)     /    │
  │   Memory-Bound Region /     │
  │   Slope = HBM BW     /      │
  │   (Decode Phase)    /       │
  │                    /        │
  └───────────────────┴─────────┴───────────────────────────────────► Arithmetic Intensity (FLOPs/byte)
                      0      AI_ridge
```

The upper boundary forms the shape of a roof and is governed by the fundamental equation:

$$	ext{Achievable Throughput (FLOPS)} = \min \left( 	ext{Peak Hardware FLOPS}, \ 	ext{Memory Bandwidth} 	imes 	ext{Arithmetic Intensity} 
ight)$$

### 4.1 The Slanted Line (Left of the Vertical Line — Memory-Bound Region)
* **Location:** Left of the dashed vertical line (Low Arithmetic Intensity, e.g., Decode Phase at $pprox 1	ext{ FLOP/byte}$).
* **Why it is slanted:** In this region, the GPU's Tensor Cores finish their math almost instantaneously and sit idle waiting for bytes to arrive from HBM. Therefore, your throughput is strictly proportional to your Arithmetic Intensity multiplied by HBM Bandwidth:
  $$	ext{Throughput} = 	ext{Bandwidth}_{	ext{HBM}} 	imes 	ext{Arithmetic Intensity}$$
* **How to climb the slope:** Moving right along the X-axis (by batching more tokens or fusing operations to reduce HBM traffic) increases throughput linearly up the slanted roof.

### 4.2 The Ridge Point (The Dashed Vertical Line)
* **Definition:** The exact transition point (labeled `Reach Point` / `Ridge Point`) where the slanted memory roof intersects the flat compute ceiling. At this exact point, the workload is **neither memory-bound nor compute-bound**—both the HBM memory bus and the Tensor Cores are running at 100% utilization simultaneously.
* **Formula for the Ridge Point:**
  At the intersection, $	ext{Bandwidth}_{	ext{HBM}} 	imes 	ext{Arithmetic Intensity}_{	ext{ridge}} = 	ext{Peak Computational Throughput}$. Solving for Arithmetic Intensity gives:
  $$	ext{Arithmetic Intensity}_{	ext{ridge}} = rac{	ext{Peak Computational Throughput (Max FLOPS)}}{	ext{Peak Memory Bandwidth (Max Bytes/sec)}}$$
* **Real Hardware Example (NVIDIA H200 in FP16):**
  * Peak FP16 Compute = $989	ext{ TFLOPS}$
  * Peak HBM3e Bandwidth = $4.8	ext{ TB/s}$
  * $	ext{Ridge Point} = rac{989	ext{ TFLOPS}}{4.8	ext{ TB/s}} pprox \mathbf{206	ext{ FLOPs/byte}}$.
  * Any operation with $< 206	ext{ FLOPs/byte}$ is **Memory-Bound**; any operation with $> 206	ext{ FLOPs/byte}$ is **Compute-Bound**.

### 4.3 The Flat Horizontal Line (Right of the Vertical Line — Compute-Bound Region)
* **Location:** Right of the dashed vertical line (High Arithmetic Intensity, e.g., Prefill Phase with large prompt length $N$).
* **Why it is flat:** Once Arithmetic Intensity exceeds the Ridge Point, memory transfer is fast enough to keep the compute units fed continuously. You hit the **physical compute ceiling** of the GPU's Tensor Cores ($	ext{Peak Hardware FLOPS}$).
* **What happens if you increase Arithmetic Intensity further?** Moving further right along the X-axis past the Ridge Point **does NOT increase throughput (FLOPS)**—you stay on the flat horizontal line. The only way to raise the flat roof higher is to use a GPU with faster/more Tensor Cores or switch to a lower-precision data type (such as moving from `FP16` to `FP8`).

---

## 5. Hardware & Software Trade-offs: HBM vs. SRAM

A crucial systems question is: **"Can we increase Arithmetic Intensity by simply increasing the size of HBM?"**

### 5.1 Why Increasing HBM Capacity Does NOT Directly Increase Arithmetic Intensity
* **HBM Capacity vs. HBM Traffic:** HBM is the off-chip storage pool (e.g., $141	ext{ GB}$). The denominator of Arithmetic Intensity is **Bytes Transferred between HBM and on-chip SRAM**.
* If you double HBM capacity from $141	ext{ GB}$ to $282	ext{ GB}$ without changing the algorithm or batch size, the GPU still has to transfer the exact same number of weight bytes across the bus from HBM into SRAM to compute a layer. Thus, $	ext{FLOPs/Byte}$ remains unchanged.
* **The Indirect Benefit of Larger HBM:** A larger HBM capacity allows you to store a larger KV Cache and therefore run a **larger Batch Size ($B$)** of concurrent user requests without running out of memory. Increasing the batch size $B$ *does* increase Arithmetic Intensity because weights read from HBM are reused across more parallel tokens in SRAM.

### 5.2 Why On-Chip SRAM Size is the True Hardware Lever (and Why SRAM is Small)
* If hardware engineers could make on-chip **SRAM** massive (e.g., hundreds of gigabytes), matrices would never have to spill back out to slow HBM, and Arithmetic Intensity would skyrocket.
* **The Physical Silicon Trade-off:** On a silicon die, SRAM cells (6-transistor static RAM) take up vastly more physical surface area per bit than HBM DRAM cells. Making SRAM huge would leave no physical room on the die for Compute Units (Tensor Cores) and would drive chip manufacturing costs and power consumption to impractical levels. Thus, GPUs settle on a small, ultra-fast SRAM ($\sim 20	ext{–}50	ext{ MB}$) paired with a large, slower off-chip HBM ($\sim 80	ext{–}141	ext{ GB}$).

---

## 6. FlashAttention and Why Prefill is Compute-Bound

Even during the **Prefill Phase** (or training), naive self-attention can accidentally drag a workload back into the **Memory-Bound** region if we are not careful with HBM reads and writes.

### 6.1 Why Naive Self-Attention is Memory-Bound
Standard self-attention computes:
$$	ext{Attention}(Q, K, V) = 	ext{Softmax}\left(rac{QK^T}{\sqrt{d_k}}
ight)V$$

For a sequence length of $N = 10,000$ tokens, the intermediate attention score matrix $S = QK^T$ has dimensions $N 	imes N = 10,000 	imes 10,000 = 100,000,000$ elements (far too large to fit inside on-chip SRAM).
Because a naive $	ext{Softmax}$ requires scanning the entire row of $N$ elements to compute the denominator sum $\sum_{j=1}^N e^{s_j}$ before dividing:
1. The GPU computes $S = QK^T$ in SRAM and **writes the entire $N 	imes N$ matrix out to slow HBM**.
2. It **reads the $N 	imes N$ matrix back from HBM** into SRAM to compute $	ext{Softmax}(S) = P$, and **writes $P$ ($N 	imes N$) back out to HBM**.
3. It **reads $P$ ($N 	imes N$) from HBM again** to multiply by $V$.

This massive $O(N^2)$ HBM read/write traffic inflates the denominator ($	ext{Bytes Transferred}$), collapsing Arithmetic Intensity and making naive attention **Memory-Bound**.

### 6.2 How FlashAttention Restores High Arithmetic Intensity
**FlashAttention** restructures the math using **Tiling** and **Online Softmax** so the $N 	imes N$ attention matrix is **never materialized in HBM**:

1. **SRAM Tiling:** It loads small blocks (tiles) of $Q$, $K$, and $V$ from HBM into fast on-chip **SRAM**.
2. **Fused Iterative Numerator & Denominator:** Instead of waiting for the full row to finish before applying Softmax and multiplying by $V$, FlashAttention maintains a running numerator $N_t$ and running denominator $D_t$ directly inside SRAM:
   $$N_t = e^{m_{t-1} - m_t} N_{t-1} + e^{s_t - m_t} v_t$$
   $$D_t = e^{m_{t-1} - m_t} D_{t-1} + e^{s_t - m_t}$$
   where $m_t = \max(m_{t-1}, s_t)$ tracks the running maximum score to prevent exponential overflow ($e^{s_t}$) in FP16.
3. **Zero Intermediate HBM Traffic:** Because all intermediate attention scores and Softmax rescales stay 100% inside on-chip SRAM, the GPU only reads $Q, K, V$ from HBM and writes the final compact output $O$ back to HBM.

### 6.3 Summary: Why Prefill + FlashAttention Sits in the Compute-Bound Region
When processing a prompt at **Prefill Time** with **FlashAttention**:
1. **Linear Layers (`GEMM`):** Model weights ($W_Q, W_K, W_V, W_{	ext{FFN}}$) are read from HBM once and multiplied across all $N$ input tokens in parallel, yielding high FLOPs per byte.
2. **Attention Layers (FlashAttention):** Tiling and operator fusion eliminate the $O(N^2)$ HBM memory transfers, keeping $	ext{Bytes Transferred}$ minimal while the Tensor Cores execute all $O(N^2 d)$ attention FLOPs at full speed.

Together, **high parallel FLOPs** (numerator) and **minimized HBM byte transfers** (denominator) push Arithmetic Intensity far to the right of the Ridge Point—placing the Prefill phase squarely on the **flat, Compute-Bound roof** of the Roofline Model.
