# Accelerating Recurrent Neural Networks (RNNs) on TinyML SoCs
*A concise, implementation‑oriented summary of what to accelerate and how.*

---

## Introduction
Recurrent Neural Networks (RNNs) such as LSTMs and GRUs have achieved remarkable success in sequence tasks (language, speech, etc.), but their sequential nature poses challenges for hardware acceleration. Unlike CNNs, RNNs have **intra‑timestep** (within a single cell) and **inter‑timestep** (temporal) dependencies, which limit parallelism.

## RNN Variants in Practice
- **LSTM**: employs four interacting gate signals—the forget $f$, input $i$, cell‑state candidate $\tilde{c}$, and output $o$—to regulate the flow of information across time steps.
- **GRU**: simplifies the structure by merging the input and forget gates into a single **update** gate and adding a **reset** gate, resulting in a lighter model with fewer parameters.

For clarity, the common LSTM update is:
$$
\begin{aligned}
i_t &= \sigma(W_{xi} x_t + W_{hi} h_{t-1} + b_i),\\
f_t &= \sigma(W_{xf} x_t + W_{hf} h_{t-1} + b_f),\\
o_t &= \sigma(W_{xo} x_t + W_{ho} h_{t-1} + b_o),\\
\tilde{c}_t &= \tanh(W_{xc} x_t + W_{hc} h_{t-1} + b_c),\\[4pt]
c_t &= f_t \odot c_{t-1} + i_t \odot \tilde{c}_t,\\
h_t &= o_t \odot \tanh(c_t).
\end{aligned}
$$

---

## Key Operations and Hardware Acceleration Targets

### 1) Matrix–Vector Multiplications (MVMs)
Each timestep multiplies the input vector and previous hidden state by gate/candidate weight matrices. These dense MVMs dominate the compute—**$\sim 90\%$ of RNN work is MACs**—so accelerators prioritize wide MAC arrays (e.g., systolic), pipelined dataflow, and aggressive weight reuse in on‑chip SRAM to minimize bandwidth.

### 2) Nonlinear Activation Functions
LSTM/GRU gates use **sigmoid** ($\sigma$) and **tanh** ($\tanh$), which are expensive if implemented with exponentials/divisions. Hardware typically uses LUTs, piecewise‑linear (PLA) or CORDIC approximations to keep latency and area low while staying on the recurrence critical path.

### 3) Gating Mechanisms (Elementwise Ops)
Hadamard products and vector adds (e.g., $i_t \odot \tilde{c}_t$, $f_t \odot c_{t-1}$) update the cell/hidden states. Though lighter than MVMs, they benefit from **vector ALUs/SIMD**, fixed‑point arithmetic, and keeping state in **local scratchpads** to avoid memory bottlenecks.

### 4) Recurrent State Handling
Because each timestep depends on the previous state, designs use **pipelining across timesteps**, **double‑buffered hidden/cell state**, and **broadcast** of $h_{t-1}$ to multiple multipliers so all gates can be computed concurrently. Specialized SRAM/register files and careful scheduling help maintain throughput despite data dependencies.

---

## Operation–Technique Map

| RNN Component                               | Mathematical Operation                   | Computational Intensity                           | Hardware Acceleration Priority          | Primary Hardware Techniques                                   |
|---|---|---|---|---|
| Gate updates ($i_t, f_t, \tilde{c}_t, o_t$) | Matrix–Vector Multiplication (MVM)       | High (compute‑bound, $\mathcal{O}(L_h^2)$)       | **High** (tackles recurrence bottleneck) | Systolic/parallel MACs, pipelined dataflow, on‑chip weight reuse |
| Gating functions ($\sigma, \tanh$)         | Non‑linear activation (exp/div)          | Medium (resource/latency cost)                    | **Critical** (on critical path)          | LUT / PLA / CORDIC, fused activation units                    |
| State update ($c_t, h_t$)                    | Elementwise mul/add (Hadamard, sum)      | Low                                              | Low–Medium (memory‑sensitive)           | Fixed‑point SIMD/Vector ops, local scratchpad for states       |

> **Notation.** $L_h$: hidden dimension size; $\odot$: elementwise product.

---

## Algorithm‑Level Optimizations (Hardware‑Friendly)

- **Low‑Precision Quantization.**  
  Use INT8 (or lower) weights/activations to speed up MACs and shrink model/activation memory. Many RNNs retain accuracy with 8‑bit; mixed/dynamic precision can allocate higher bits to volatile timesteps and fewer bits to stable ones.

- **Pruning & Sparsity.**  
  Remove insignificant weights to create sparse matrices so hardware can **skip multiplies**. Structured sparsity (bank‑balanced, block/circulant) improves memory coalescing and parallelism compared to unstructured sparsity.

- **Structured / Low‑Rank Matrices.**  
  Constrain weights (e.g., block‑circulant, Toeplitz) or factorize into low‑rank products to reduce MACs and enable FFT‑style acceleration or smaller dense multiplies.

### The Synergy for TinyML (“Deep Compression”)
For TinyML deployments, the most effective recipe combines:
1. **Quantization** as the foundation to meet SRAM/flash and fixed‑point arithmetic needs.  
2. **Pruning/Sparsity** to cut MACs and off‑chip traffic.  
3. (Optionally) **Structured matrices/low‑rank** to further reduce compute and enable specialized datapaths.  

Together, these methods routinely yield **~90% size reductions** while preserving accuracy in small KWS/sequence models, with proportional gains in latency and energy efficiency when the hardware exploits the structure.

---

## Reference
- *A Survey on Hardware Accelerators and Optimization Techniques for RNNs* — Comprehensive methods and design patterns for accelerating RNNs in hardware (dense, sparse, and structured).  
  Link: https://www.researchgate.net/publication/343006357_A_Survey_on_Hardware_Accelerators_and_Optimization_Techniques_for_RNNs

---

### At‑a‑Glance Takeaways
- Prioritize **MVM throughput** (wider MAC arrays, systolic/weight‑stationary dataflows, on‑chip weight reuse).  
- Keep activations fast and local: **LUT/PLA/CORDIC** for $\sigma$, $\tanh$.  
- Minimize memory movement: **scratchpads** for state, **tiling** to keep working sets on‑chip.  
- Stack algorithmic tricks: **INT8 + structured sparsity + low‑rank** ⇒ TinyML‑ready models with large speed/energy wins.
