---
title: "Tucker-KV: Provable Tucker Compression of KV Caches with Monotone Refinement and Near-Optimal Budgeting"
title_zh: Tucker-KV：具有单调细化与近似最优预算分配的KV缓存可证明Tucker压缩
authors: Hung-Min Hsu
date: 2025-09-18
pdf: "https://openreview.net/pdf?id=aQ6RiOijal"
tags: ["query:llm-kv-cache"]
score: 10.0
evidence: 使用Tucker分解的KV缓存压缩
tldr: 大语言模型的KV缓存随上下文长度线性增长，现有压缩方法多为矩阵低秩启发式，缺乏多线性保证。本文提出Tucker-KV，基于Tucker分解对KV张量进行可证明的压缩，支持单调细化、分组头并行压缩和贪心预算分配的(1-1/e)保证。实验表明该方法在压缩性能和理论保障上优于现有基线。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 现有KV缓存压缩缺乏多线性结构的理论保证，压缩性能受限。
method: "提出基于Tucker分解的框架，对(L,S,H)维度的KV张量进行压缩，包含HOSVD误差界和HOOI单调细化。"
result: 在多种LLM上验证了压缩效果，理论保证和实际性能均优于现有方法。
conclusion: Tucker-KV提供了首个可证明的KV缓存多线性压缩方案，兼具理论严谨性和实践有效性。
---

## Abstract
Key-Value (KV) caches enable fast Transformer decoding but their memory and compute scale linearly with context length. Prior KV compression works are largely matrix low-rank heuristics, leaving multilinear guarantees underexplored. We present Tucker-KV, a Tucker-based framework with provable properties for compressing KV tensors over (L, S, H). Our analysis establishes: (i) HOSVD-style error upper bounds and monotone refinement via HOOI; (ii) grouped-head separability enabling parallelizable compression; (iii) a (1-1/e) guarantee for greedy budget allocation under mild DR-submodularity; and (iv) robust residual mixing with matrix baselines that never increases error when Tucker fits the residual in least squares. We further characterize the budget regime where Tucker-2 is preferable to full Tucker. On Qwen2.5-7B with RULER at 4k, Tucker-KV matches Full-KV quality (EM/F1 ~ 1.00) while saving 83% KV memory, with perplexity unchanged and favorable prefill throughput. Importantly, Tucker-KV is orthogonal to token-selection methods (sliding/streaming/xKV) and can be stacked with them; our focus is the representation-compression axis with provable monotone refinement and near-optimal budget allocation.

---

## 论文详细总结（自动生成）

# Tucker-KV：具有单调细化与近似最优预算分配的KV缓存可证明Tucker压缩

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：大语言模型（LLM）在解码时依赖 Key-Value（KV）缓存来加速推理，但 KV 缓存的大小随上下文长度线性增长，在长序列场景下导致巨大的内存和计算开销。
- **研究动机**：现有 KV 缓存压缩方法多为基于矩阵低秩的启发式方法，缺乏对 KV 张量多线性结构的理论保证（如误差上界、单调细化、最优预算分配等）。
- **整体含义**：本文旨在提供一种**可证明的、多线性压缩框架**，使得 KV 缓存压缩在理论上严谨，同时在实际部署中保持高效。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程

- **核心思想**：利用 **Tucker 分解**对 KV 张量（维度：序列长度 L、隐藏维度 S、头维度 H）进行压缩，保留多线性结构，避免传统矩阵分解对张量信息的破坏。
- **关键技术细节**：
    - **HOSVD 误差上界**：推导了高阶奇异值分解（HOSVD）压缩后的误差上界，保证压缩误差可控。
    - **HOOI 单调细化**：提出使用高阶正交迭代（HOOI）算法，可在固定秩下单调地降低压缩误差。
    - **分组头并行压缩**：利用注意力头的可分性，将 KV 张量分解为多个子张量分别压缩，支持并行化计算。
    - **近似最优预算分配**：在 DR-子模（DR-submodularity）条件下，贪心算法可实现 `(1 - 1/e)` 的近似比，保证各头之间的秩预算分配接近最优。
    - **鲁棒残差混合**：将 Tucker 分解与矩阵基线（如低秩近似）相结合，当 Tucker 以最小二乘方式拟合残差时，整体误差不会增大。
    - **Tucker-2 与完整 Tucker 的选择**：分析了在什么预算条件下 Tucker-2（仅压缩两个维度）比完整 Tucker 更优。
- **算法流程（文字说明）**：
    1. 将原始 KV 缓存视作一个三阶张量 `(L, S, H)`。
    2. 选择目标秩（各维度压缩后的大小），使用 **HOSVD** 计算初始 Tucker 分解，得到核心张量和三个因子矩阵。
    3. 可选地运行 **HOOI** 迭代优化，单调降低重构误差。
    4. 通过 **贪心预算分配**（结合 DR-子模目标）决定每个注意力头的压缩秩。
    5. 对所有头并行进行 Tucker 分解压缩，存储压缩后的核心张量和因子矩阵。
    6. 解码时，快速重构近似 KV 向量，或直接使用压缩表示进行注意力计算。

## 3. 实验设计：数据集、场景、基准与对比方法

- **数据集/场景**：使用 **RULER** 评估数据集（长上下文评测），在 **4k** 上下文长度下测试。
- **基准模型**：Qwen2.5-7B（7B 参数模型）。
- **对比方法**：与 **Full-KV**（无压缩）以及现有其他压缩方法（如滑动窗口、流式、xKV 等）比较。注意 Tucker-KV 与这些方法正交，可叠加使用。
- **评估指标**：EM（精确匹配）、F1 分数、困惑度（perplexity）、prefill 吞吐量、KV 内存节省比例。

## 4. 资源与算力

- 文中**未明确说明**使用的 GPU 型号、数量或训练时长。仅给出了模型规模（Qwen2.5-7B）和评估上下文长度（4k），但未提及具体的硬件配置或运行时间。需要在论文正文中进一步查找。

## 5. 实验数量与充分性

- **实验数量**：摘要中仅提到在 4k 上下文下对 Qwen2.5-7B 进行了评估，并报告了 EM/F1、困惑度、吞吐量、内存节省。未提及更多数据集或消融实验（如不同上下文长度、不同模型规模、不同秩配置的消融等）。
- **充分性判断**：从摘要看，实验覆盖范围较窄（单一模型、单一长度），缺乏跨模型、跨长度的系统评估。虽然提到了理论分析（DR-子模、误差上界等），但实验验证的充分性**有所不足**。公平性方面，与 Full-KV 比较是客观的，但未列出现有其他压缩方法的详细对比结果（如具体数值），难以判断是否公平。

## 6. 论文的主要结论与发现

- Tucker-KV 在 **4k 上下文中**，以 **83% KV 内存节省** 实现了与 Full-KV 几乎相同的质量（EM/F1 ≈ 1.00），且困惑度不变，prefill 吞吐量有优势。
- 提供了**首个针对 KV 缓存的可证明多线性压缩方案**，具备单调细化、近似最优预算分配等理论保证。
- 与 token 选择类方法（滑动/流式/xKV）正交，可叠加使用，进一步降低内存。

## 7. 优点：方法或实验设计上的亮点

- **理论贡献突出**：给出了 HOSVD 误差上界、HOOI 单调细化、DR-子模贪心预算分配的 `(1 - 1/e)` 保证，以及 Tucker 与矩阵基线混合的不增误差性质。
- **方法设计巧妙**：利用 Tucker 分解保留张量结构，分组并行提高效率，残差混合增强鲁棒性。
- **实用性**：在 83% 内存压缩下不损失质量，且可与现有 token 选择方法叠加，具有工程部署潜力。

## 8. 不足与局限

- **实验覆盖不足**：仅测试了单模型（7B）、单上下文长度（4k），未验证更长上下文（如 32k/128k）或更大模型（如 70B）下的表现。
- **缺少消融研究**：未展示不同秩选择、不同迭代次数（HOOI）、不同头分组策略对性能的影响。
- **对比不充分**：没有与同类张量压缩方法（如 CP 分解、TR 分解）或其他低秩启发式方法（如 KVQuant、SparseAttention 等）进行数值对比。
- **理论假设验证**：DR-子模性在 KV 缓存场景中是否普遍成立，可能依赖于具体注意力模式，需更广泛验证。
- **无硬件资源信息**：缺少实际推理加速和延迟数据，仅提到 prefill 吞吐量，未提供生成阶段 token 延迟。

（完）
