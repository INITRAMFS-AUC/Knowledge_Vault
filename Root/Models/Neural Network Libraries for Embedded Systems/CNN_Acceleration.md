# Accelerating Convolutional Neural Networks (CNNs) on TinyML SoCs

*A concise, implementation-oriented summary of what to accelerate and how — written for hardware & ML co-design.*

---

## Introduction

CNNs are a natural fit for TinyML (e.g., keyword spotting) because most work is **embarrassingly parallel** across output pixels and channels. The design goal is to (1) keep MAC units busy on critical kernels (3×3, depthwise 3×3, pointwise 1×1), and (2) **minimize data movement**, which typically dominates energy. Line-buffered streaming and row/weight-stationary dataflows are the standard playbook [2],[3].

---

## CNN Variants in Practice (and why they matter on tiny SoCs)

* **Standard $k\times k$ convolution.**
  For input $X\in\mathbb{R}^{H\times W\times C_{\text{in}}}$ and kernel $K\in\mathbb{R}^{k\times k\times C_{\text{in}}\times C_{\text{out}}}$, the output $Y\in\mathbb{R}^{H_o\times W_o\times C_{\text{out}}}$ has $\text{MACs}=H_o W_o, C_{\text{out}}, (k^2 C_{\text{in}})$. Parallelism is across $H_o$, $W_o$, $C_{\text{out}}$ with reductions over $k^2 C_{\text{in}}$.

* **Depthwise-separable convolution (DS-CNN).**
  Factorize dense conv into **depthwise** $k\times k$ (per-channel spatial) followed by **pointwise** $1\times 1$ (cross-channel mixing): $\text{MACs}=H_o W_o,(k^2 C_{\text{in}} + C_{\text{in}} C_{\text{out}})$. For $k=3$ and moderate $C_{\text{out}}$, this is typically **≈8–9× fewer MACs** than a dense $3\times 3$ conv with similar capacity; **MobileNet** popularized this trade-off for embedded use [1].

* **Winograd for $3\times 3$.**
  Reduces multiplications via transforms (e.g., F(2×2, 3×3)); helpful at larger channel counts but adds transform cost and can be fragile under aggressive INT8 quantization—profile end-to-end energy–delay and accuracy before committing [4],[8],[9].

> **Bottom line:** For always-on TinyML budgets, **DS-CNN** (3×3 depthwise + 1×1 pointwise) gives an excellent accuracy/compute/parallelism trade-off [1],[2].

---

## Key Operations & Hardware Acceleration Targets

### 1) Convolutions (where the cycles go)

* **3×3 dense conv:** common in early layers; high reuse if tiled well.
* **3×3 depthwise (DW):** trivially channel-parallel; loves line buffers; small compute per channel.
* **1×1 pointwise (PW):** effectively a **GEMM** over channels and often the **dominant** MAC hotspot; tune the MAC array and dataflow for this case. Modern libraries map conv to **implicit GEMM** (forming tiles on the fly rather than explicit `im2col`) [6],[7].

### 2) Nonlinearities & Elementwise Ops

ReLU/Clamp are elementwise and cheap; keep them **fused** with conv. BatchNorm at inference is an **affine** transform; **fold BN into conv weights/bias** (Conv+BN+ReLU fusion) to avoid extra memory traffic and layers [5],[10],[11].

### 3) Pooling / Downsampling

Favor **stride-2 conv** instead of separate pooling to enable fusion and reduce feature-map traffic.

---

## Stateless vs Stateful Processes (parallelism & scheduling)

**Stateless (per inference sample):**
Dense/DW/PW conv, ReLU, folded BN, stride/pooling — all **parallel across pixels and channels**.

**Stateful (implementation-level, not algorithmic recurrences):**

* **Line buffers** retain $(k-1)$ previous rows per channel to feed a sliding window.
* **Partial-sum buffers** hold accumulations across $k^2 C_{\text{in}}$ (or groups) before store.
  These are small, predictable states that enable single-pass streaming without external memory round-trips. Data movement dominates energy; the **row-stationary (RS)** dataflow explains why [2],[3].

---

## Dataflow & Memory: what to keep stationary (to save energy)

Choose a dataflow that maximizes reuse of the most expensive operand (often IFMAP or weights):

* **Row-stationary (RS):** co-optimizes reuse of inputs, weights, and partial sums across a PE array; a robust default.
* **Weight-stationary:** keep filter tiles resident; stream IFMAPs; accumulate locally.
* **Output-stationary:** keep partial sums local until final write-back.

**Streaming** via line buffers avoids **im2col** blow-ups on tiny SRAM. **Fuse layers** as $[\text{Conv (BN folded)}\rightarrow\text{ReLU}\rightarrow\text{(optional) stride-2}]$ to minimize spills between SRAM levels [2],[3],[5].

---

## Operation–Technique Map

| CNN Component      | Mathematical Operation                        | Computational Intensity | Acceleration Priority | Primary Techniques                                  |
| ------------------ | --------------------------------------------- | ----------------------: | :-------------------: | --------------------------------------------------- |
| **3×3 Conv**       | Sliding dot-products over $k^2 C_{\text{in}}$ |                    High |        **High**       | RS/weight-stationary tiling, PE array, line buffers |
| **3×3 Depthwise**  | Per-channel spatial conv                      |                  Medium |        **High**       | Channel-parallel lanes; tiny line/window buffers    |
| **1×1 Pointwise**  | Cross-channel GEMM                            |           **Very High** |     **Very High**     | GEMM-tuned MAC array, implicit GEMM mapping         |
| **Activations**    | Elementwise (ReLU/Clamp)                      |                     Low |          Med          | Fuse with conv; SIMD/bit-wise units                 |
| **BN (inference)** | Affine                                        |                     Low |          Low          | **Fold** BN into conv weights/bias                  |
| **Pooling/Stride** | Local reduce / subsample                      |                     Low |          Low          | Prefer stride-2 conv (fusable)                      |

---

## Algorithm-Level Optimizations (hardware-friendly)

1. **Integer-only inference (INT8), per-channel where possible.**
   Use int32 accumulators + requantization; per-channel weight scales markedly help 1×1 convs. TFLite’s integer-only scheme is a good template [12],[13].

2. **Exploit DS-CNN structure.**
   $3\times 3$ DW + $1\times 1$ PW often yields **order-of-magnitude** MAC savings vs dense $3\times 3$ with minimal accuracy loss on KWS [1].

3. **Prune for energy, not just parameters.**
   Prefer **energy-aware pruning** that reduces real data movement/compute in layers that dominate energy (not only FLOPs) [14],[15],[16].

4. **Use transforms cautiously.**
   Winograd can reduce mults for $3\times 3$ but adds transform overhead and can hurt INT8 accuracy; profile end-to-end EDP + accuracy [4],[8],[9].

5. **Avoid explicit `im2col` on tiny SRAM.**
   Use direct or **implicit-GEMM** methods and stream tiles from line buffers to prevent activation-matrix blow-ups [6],[7],[17].

---

## Feature Extraction & Statefulness (KWS context)

If the network consumes spectrograms (log-mel/MFCC), the CNN itself is **stateless per frame**, but the front-end is lightly **stateful**:

* **Framing with overlap** (ring buffer over audio samples).
* **Per-Channel Energy Normalization (PCEN)**: an AGC-like, IIR smoother per frequency bin; robust vs. loudness/noise and implementable in fixed-point [18],[19],[20].

---

## Micro-Architecture Recommendations (what to build/tune)

* **Compute:** PE/MAC array sized to a sweet spot for **1×1 PW GEMM** (e.g., 8×8…16×16), with a mode for channel-parallel **3×3 DW**.
* **On-chip memory:**

  * **Weight SRAM** for filter tiles (favor weight-stationary on PW),
  * **IFMAP scratch + line buffers**: $(k_h-1),W_{\text{tile}},C_{\text{in}}$ bytes (INT8) plus a $k_h k_w C_{\text{in}}$ window,
  * **Partial-sum buffer**: $H_{\text{tile}} W_{\text{tile}} C_{\text{out}}$ in int32.
* **Dataflow:** Row- or weight-stationary with **fused Conv(+BN)+ReLU** passes.
* **Kernels:** Align with **CMSIS-NN-style INT8** (per-channel scales, saturating SIMD) for bring-up and predictable performance [21].

---

## Risks & Knobs

* **Transform precision:** Winograd on INT8 may degrade accuracy; keep a direct kernel fallback [4],[8].
* **Memory pressure:** Explicit `im2col` can exceed SRAM; prefer streaming/direct or implicit-GEMM [6],[7],[17].
* **"Smaller model" != "lower energy":** use **energy-aware** criteria when pruning or shrinking [14],[15],[16].

---

## Concrete Verdict (design choice)

**Choose:** **INT8 DS-CNN** (3×3 depthwise + 1×1 pointwise), **per-channel quantization**, **Conv+BN+ReLU fusion**, **stride-2 in early blocks**, scheduled with **row/weight-stationary** dataflow and **line-buffered streaming**.
**If audio/KWS:** feed with **log-mel/MFCC + PCEN** in fixed-point.
This maximizes parallelism, minimizes data movement, and maps cleanly to MCU/ASIC-class kernels.

---

## Appendix — Quick formulas your hardware team will ask for

* **Dense $k\times k$ conv MACs:** $H_o W_o, C_{\text{out}}, (k^2 C_{\text{in}})$.
* **DS-conv MACs:** $H_o W_o, (k^2 C_{\text{in}} + C_{\text{in}} C_{\text{out}})$.
* **Reduction factor (vs dense 3×3):** $\approx \tfrac{1}{k^2} + \tfrac{1}{C_{\text{out}}}$ → typically **≈8–9× fewer MACs** at $k=3$.
* **Line buffer size (INT8):** $(k_h-1), W_{\text{tile}}, C_{\text{in}}$ bytes.
* **Window buffer (INT8):** $k_h k_w C_{\text{in}}$ bytes.

---

## At-a-Glance Takeaways

* Prioritize **1×1 PW (GEMM)** throughput and **3×3 DW** channel parallelism.
* **Keep data local:** row/weight-stationary, line buffers, fused passes — **data movement is the enemy**.
* **Quantize** (INT8, per-channel), **fold BN**, **use DS-CNN**, and **prune with energy models**, not just FLOPs.

---

## References

* [1] Howard et al., **MobileNets: Efficient CNNs for Mobile Vision** (2017). [https://arxiv.org/abs/1704.04861](https://arxiv.org/abs/1704.04861)
* [2] Chen, Emer, Sze et al., **Eyeriss: A Spatial Architecture for Energy-Efficient Dataflow for CNNs** (ISCA 2016). PDF: [https://eems.mit.edu/wp-content/uploads/2016/04/eyeriss_isca_2016.pdf](https://eems.mit.edu/wp-content/uploads/2016/04/eyeriss_isca_2016.pdf)
* [3] Chen et al., **Eyeriss (overview/RS dataflow)** — ACM DL abstract: [https://dl.acm.org/doi/10.1145/3007787.3001177](https://dl.acm.org/doi/10.1145/3007787.3001177)
* [4] Lavin & Gray, **Fast Algorithms for CNNs (Winograd)** (CVPR 2016). [https://openaccess.thecvf.com/content_cvpr_2016/papers/Lavin_Fast_Algorithms_for_CVPR_2016_paper.pdf](https://openaccess.thecvf.com/content_cvpr_2016/papers/Lavin_Fast_Algorithms_for_CVPR_2016_paper.pdf)
* [5] PyTorch docs, **Conv/BN/Activation fusion & BN folding**. [https://pytorch.org/docs/stable/generated/torch.ao.quantization.fuse_modules.html](https://pytorch.org/docs/stable/generated/torch.ao.quantization.fuse_modules.html)
* [6] NVIDIA CUTLASS docs, **Implicit GEMM Convolution**. [https://docs.nvidia.com/cutlass/media/docs/cpp/implicit_gemm_convolution.html](https://docs.nvidia.com/cutlass/media/docs/cpp/implicit_gemm_convolution.html)
* [7] NVIDIA developer blog, **CUTLASS & implicit GEMM mapping**. [https://developer.nvidia.com/blog/cutlass-linear-algebra-cuda/](https://developer.nvidia.com/blog/cutlass-linear-algebra-cuda/)
* [8] Lavin (arXiv), **Winograd for small filters**. [https://arxiv.org/abs/1509.09308](https://arxiv.org/abs/1509.09308)
* [9] Survey/note on Winograd performance & sensitivity. [https://www.semanticscholar.org/paper/Fast-Algorithms-for-Convolutional-Neural-Networks-Lavin-Gray/1121ff5cdeaa470521b8dff084ba1424dd613cc1](https://www.semanticscholar.org/paper/Fast-Algorithms-for-Convolutional-Neural-Networks-Lavin-Gray/1121ff5cdeaa470521b8dff084ba1424dd613cc1)
* [10] PyTorch tutorial, **Conv–BN fusion mechanics**. [https://docs.pytorch.org/tutorials/intermediate/custom_function_conv_bn_tutorial.html](https://docs.pytorch.org/tutorials/intermediate/custom_function_conv_bn_tutorial.html)
* [11] PyTorch AO intrinsic modules (e.g., ConvBnReLU2d). [https://pytorch.org/docs/stable/generated/torch.ao.nn.intrinsic.ConvBnReLU2d.html](https://pytorch.org/docs/stable/generated/torch.ao.nn.intrinsic.ConvBnReLU2d.html)
* [12] Jacob et al., **Quantization and Training of Neural Networks for Efficient Integer-Arith-Only Inference** (CVPR 2018). [https://openaccess.thecvf.com/content_cvpr_2018/html/Jacob_Quantization_and_Training_CVPR_2018_paper.html](https://openaccess.thecvf.com/content_cvpr_2018/html/Jacob_Quantization_and_Training_CVPR_2018_paper.html)
* [13] Jacob et al., arXiv version. [https://arxiv.org/abs/1712.05877](https://arxiv.org/abs/1712.05877)
* [14] Yang, Chen, Sze, **Designing Energy-Efficient CNNs via Energy-Aware Pruning** (CVPR 2017). [https://openaccess.thecvf.com/content_cvpr_2017/papers/Yang_Designing_Energy-Efficient_Convolutional_CVPR_2017_paper.pdf](https://openaccess.thecvf.com/content_cvpr_2017/papers/Yang_Designing_Energy-Efficient_Convolutional_CVPR_2017_paper.pdf)
* [15] Yang, Chen, Sze, arXiv version. [https://arxiv.org/abs/1611.05128](https://arxiv.org/abs/1611.05128)
* [16] Sze et al., **Efficient Processing of DNNs: A Tutorial and Survey** (Proc. IEEE 2017). [https://people.csail.mit.edu/emer/media/papers/2018.02.sysml.energy-efficient-dnn.pdf](https://people.csail.mit.edu/emer/media/papers/2018.02.sysml.energy-efficient-dnn.pdf)
* [17] Zhang et al., **Characterizing the Implicit Convolution (im2col/implicit)** (2021). [https://ar5iv.labs.arxiv.org/html/2110.03901](https://ar5iv.labs.arxiv.org/html/2110.03901)
* [18] Wang et al., **Trainable Frontend for Robust Far-Field KWS (PCEN)** (2016). [https://arxiv.org/abs/1607.05666](https://arxiv.org/abs/1607.05666)
* [19] Lostanlen et al., **PCEN: Why and How** (IEEE SPL 2018). [https://www.justinsalamon.com/uploads/4/3/9/4/4394963/lostanlen_pcen_spl2018.pdf](https://www.justinsalamon.com/uploads/4/3/9/4/4394963/lostanlen_pcen_spl2018.pdf)
* [20] Lostanlen et al., **Long-distance Detection with PCEN** (2019). [https://arxiv.org/abs/1911.00417](https://arxiv.org/abs/1911.00417)
* [21] Lai et al., **CMSIS-NN: Efficient NN Kernels for Arm Cortex-M** (2018). [https://arxiv.org/abs/1801.06601](https://arxiv.org/abs/1801.06601)
