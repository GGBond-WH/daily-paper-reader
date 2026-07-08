---
title: "Sharp Eyes and Memory for VideoLLMs: Information-Aware Visual Token Pruning for Efficient and Reliable VideoLLM Reasoning"
title_zh: "Sharp Eyes and Memory for VideoLLMs: 信息感知的视觉Token剪枝实现高效可靠的视频LLM推理"
authors: "Jialong Qin, Xin Zou, Di Lu, Yibo Yan, Xuming Hu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/37805/41767"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 针对视频大模型的KV缓存剪枝
tldr: SharpV针对视频大模型视觉token冗余导致的KV缓存扩展问题，提出信息感知的自适应剪枝方法。根据时空信息动态调整剪枝比例，在KV缓存剪枝阶段还考虑视觉信息退化。实验表明偶尔超过密集模型性能，为自适应剪枝提供了新范式。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 视频大模型视觉token冗余导致KV缓存扩展和计算开销大。
method: 根据时空信息自适应剪枝视觉token和KV缓存。
result: 自适应剪枝有时实现性能提升，超越密集模型。
conclusion: 自适应信息剪枝可优化视频LLM的KV缓存效率。
---

## Abstract
Current Video Large Language Models (VideoLLMs) suffer from quadratic computational complexity and key-value cache scaling, due to their reliance on processing excessive redundant visual tokens. To address this problem, we propose SharpV, a minimalist and efficient method for adaptive pruning of visual tokens and KV cache. Different from most uniform compression approaches, SharpV dynamically adjusts pruning ratios based on spatial-temporal information. Remarkably, this adaptive mechanism occasionally achieves performance gains over dense models, offering a novel paradigm for adaptive pruning. During the KV cache pruning stage, based on observations of visual information degradation, SharpV prunes degraded visual features via a self-calibration manner, guided by similarity to original visual features. In this way, SharpV achieves hierarchical cache pruning from the perspective of information bottleneck, offering a new insight into VideoLLMs' information flow. Experiments on multiple public benchmarks demonstrate the superiority of SharpV. Moreover, to the best of our knowledge, SharpV is notably the first two-stage pruning framework that operates without requiring access to exposed attention scores, ensuring full compatibility with hardware acceleration techniques like Flash Attention.

---

## 论文详细总结（自动生成）

# 论文总结：Sharp Eyes and Memory for VideoLLMs

## 1. 核心问题与整体含义（研究动机和背景）

视频大语言模型（VideoLLMs）在处理长视频时，需要输入大量的视觉 token，导致二次计算复杂度和 KV cache 的持续增长，严重制约了推理效率和内存使用。现有方法大多采用固定的剪枝比率，不能根据视频内容信息量自适应调整，且许多方法依赖暴露的注意力分数，与 Flash Attention 等硬件加速技术不兼容。本文提出 SharpV，旨在通过**信息感知的自适应剪枝**同时减少视觉 token 和 KV cache，在保持或提升性能的前提下实现高效推理，甚至偶尔超越密集模型，为自适应剪枝提供了新范式。

## 2. 方法论：核心思想、关键技术细节

SharpV 是一个两阶段、无需训练、即插即用的剪枝框架，包含：

### 2.1 预LLM阶段：Visual SharpV（视觉空间剪枝）
- **核心思想**：根据**时空重要性**自适应地确定每帧的 token 保留比率。
- **关键技术细节**：
  - **相异度计算模块**：采用 L2 归一化后的欧氏距离（Dissim）替代余弦相似度，避免高维空间中余弦相似度接近零的问题。
  - **空间重要性 \(S\)**：每个 token 与该帧平均特征的 Dissim，反映该 token 在该帧中的独特性。
  - **时间重要性 \(T\)**：相邻帧对应 token 之间的 Dissim，反映运动信息。
  - **综合重要性 \(I = T + w \cdot S\)**，其中 \(w\) 为超参数。
  - **自适应阈值**：基于帧间时间重要性的 L2 范数计算出每帧的保留比例（第一帧使用空间重要性代替），实现帧级动态剪枝。
- **复杂度**：\(O(n \cdot d)\)，线性复杂度。

### 2.2 内LLM阶段：Memory SharpV（记忆空间剪枝）
- **核心思想**：发现视觉 token 在各层中的信息随深度增加而退化（Visual Information Degradation），类似人类记忆曲线。
- **关键技术细节**：
  - 计算每层视觉 token 与原始视觉特征的余弦相似度。
  - 当相似度低于阈值 \(M\) 时，丢弃该层的 KV cache。
  - 该方法无需暴露注意力分数，完全兼容 Flash Attention。
- **复杂度**：从 \(O(n^2 \cdot d)\) 降至 \(O(n \cdot d)\)。

## 3. 实验设计

### 3.1 数据集与 Benchmark
- **视频理解**: **MVBench** (20个子任务), **VideoMME** (短/中/长视频)
- **视频问答**: **NExT-QA**, **ActNet-QA**

### 3.2 对比方法
- 基线：无剪枝的密集模型
- 预LLM阶段：PruMerge
- 内LLM阶段：FastV, DyCoke
- 此外还比较了随机剪枝变体（V-Random†、V-Random∗）

### 3.3 模型与实现
- **模型**: LLaVA-OneVision (0.5B, 7B), PLLaVA (7B)
- **框架**: 基于 lmms-eval 和 PLLaVA 官方实现
- **硬件**: NVIDIA A800 (80GB) GPU

## 4. 资源与算力
论文仅说明实验在**NVIDIA A800 (80GB) GPU**上运行，未提供具体 GPU 数量、训练时长或总计算量。由于 SharpV 是无需训练的方法，主要开销为推理过程中的剪枝计算，作者强调其轻量级设计（O(n·d)复杂度），但未量化具体资源消耗。

## 5. 实验数量与充分性

- **主要性能对比**：在 **4 个 benchmark** 上对比了 **3 种 SOTA 方法**（FastV, PruMerge, DyCoke），并报告了 Accuracy、FLOPs、TTFT、GPU memory 等指标。
- **消融实验**：
  - Visual SharpV 与两种随机剪枝策略对比，验证了时空重要性与自适应阈值的有效性。
  - Memory SharpV：不同帧尺寸下的 KV cache 和 TPOT 分析。
  - 超参数分析：\(w\)（空间权重）、\(M\)（退化阈值）、\(K\)（手动阈值）的影响。
- 实验设计较为充分，覆盖了多个模型规模（0.5B/7B）、多种视频类型（短/中/长）、多维度评估（性能+效率）。对比方法均采用官方实现或标准超参数，公平性较好。

## 6. 主要结论与发现

1. **自适应剪枝优于固定比率**：SharpV 在相似或更低的 token budget（约12%~19%）下，在多个 benchmark 上达到 SOTA 性能，并且**偶尔超过密集模型 1%~2%**，得益于去除视觉噪声。
2. **兼容 Flash Attention**：首次提出无需注意力分数的内LLM剪枝，实现线性复杂度，显著降低内存和延迟（TTFT 加速 1.56~1.65×，TPOT 同步降低）。
3. **信息退化假设成立**：视觉 token 的余弦相似度随层数加深迅速下降，利用该规律进行 KV cache 剪枝是有效且稳定的。
4. **有效性验证**：消融实验表明时空重要性评分和帧级自适应阈值均优于随机选择，且参数选择具有鲁棒性。

## 7. 优点

- **创新性**：提出信息感知的自适应剪枝范式，首次将信息瓶颈原理应用于 VideoLLM 的 KV cache 剪枝，且不依赖注意力分数。
- **实用性**：两阶段方法均保持 O(n·d) 复杂度，完全兼容 Flash Attention，便于实际部署。
- **效果突出**：在性能持平甚至提升的同时实现显著加速，展示了剪枝不仅降低计算量还能提升推理质量。
- **实验全面**：覆盖多种模型、多种视频类型、多个 benchmark，并进行了充分的消融和超参分析。

## 8. 不足与局限

- **潜在信息损失**：对于需要细粒度视觉细节（如微表情）的任务，时空剪枝可能仍会丢失少量关键信息。
- **理论深度不足**：视觉信息退化假设仅通过余弦相似度经验验证，缺少严格的数学证明或更深入的理论解释。
- **超参依赖**：虽然自适应阈值减少了手动调参，但仍需设置 \(w\) 和 \(M\)，且 \(M\) 在 <0.2 时效果稳定，但超过 0.2 性能下降。
- **实验规模有限**：仅使用了两个模型系列（LLaVA-OneVision, PLLaVA）和 4 个 benchmark，未在更大规模模型（如 13B/34B）或更多样化的视频数据集（如 Ego4D）上验证。
- **算力报告缺失**：未提供训练或剪枝过程的详细能耗、GPU 数量、运行时间等，不利于复现和公平对比。
- **基准对比不够全面**：未与最新方法（如 VTW, FrameFusion, PruneVid）进行直接比较（尽管在表格中提及了这些方法的属性，但缺少数值结果）。

（完）
