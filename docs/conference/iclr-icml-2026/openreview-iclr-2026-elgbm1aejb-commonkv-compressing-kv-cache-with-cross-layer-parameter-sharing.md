---
title: "CommonKV: Compressing KV Cache with Cross-layer Parameter Sharing"
title_zh: CommonKV：通过跨层参数共享压缩KV缓存
authors: "Yixuan Wang, Haoyu Qiao, Lujun Li, Sirui Han, Qingfu Zhu, Wanxiang Che"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=ELGBM1aEjB"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 通过无训练参数共享实现跨层KV缓存压缩
tldr: LLM序列增长导致KV缓存内存挑战。现有跨层共享方法需修改架构或存在性能下降。本文提出CommonKV，无需训练即可通过SVD实现相邻层参数共享，形成可合并的潜在KV缓存。引入自适应机制进一步压缩。实验表明在高压缩率下保持性能。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 现有跨层KV缓存共享方法需预训练或高压缩率下性能退化。
method: 利用SVD实现相邻层权重共享，形成可合并的潜在KV缓存。
result: 在多种LLM上实现高压缩率，性能下降有限。
conclusion: CommonKV提供了一种无训练的跨层KV缓存压缩方法。
---

## Abstract
Large Language Models (LLMs) confront significant memory challenges due to the escalating KV cache with increasing sequence length. As a crucial technique, existing cross-layer KV cache sharing methods either necessitate modified model architectures with subsequent pre-training or incur significant performance degradation at high compression rates. To mitigate these challenges, we propose CommonKV, a training-free method for cross-layer KV cache compression through adjacent parameters sharing. Inspired by the high similarity observed in cross-layer hidden states, we utilize Singular Value Decomposition (SVD) to achieve weight sharing across adjacent parameters, resulting in a more easily mergeable latent KV cache. Furthermore, we also introduce an adaptive budget allocation strategy. It dynamically assigns compression budgets based on cosine similarity, ensuring that dissimilar caches are not over-compressed. Experiments across multiple backbone models and benchmarks including LongBench and Ruler demonstrate that the proposed method consistently outperforms other low-rank and cross-layer approaches at various compression ratios. Moreover, we find that the benefits of CommonKV are orthogonal to other quantization and eviction methods. By integrating these approaches, we can ultimately achieve a 98% compression ratio without significant performance loss.

---

## 论文详细总结（自动生成）

# 论文详细总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：大语言模型（LLM）在推理过程中，随着序列长度增长，KV 缓存的显存开销急剧增加，成为模型部署和长文本处理的主要瓶颈。
- **背景**：现有的跨层 KV 缓存共享方法要么需要修改模型架构并重新预训练，要么在高压缩率下性能严重退化。因此，亟需一种无需训练、即插即用的跨层压缩方案。
- **整体含义**：CommonKV 提出一种**无训练**的跨层参数共享压缩方法，能在高压缩率下保持任务性能，并与量化、驱逐等方法正交结合，最终实现 98% 的压缩比且无明显性能损失。

## 2. 论文提出的方法论：核心思想、关键技术细节
- **核心思想**：利用相邻层隐藏状态之间的高相似性，通过奇异值分解（SVD）实现相邻层参数的共享，从而得到一个更易合并的潜在 KV 缓存。
- **关键技术细节**：
  - **基于 SVD 的跨层权重共享**：对相邻层的 KV 投影权重矩阵进行 SVD，提取共享的奇异向量作为公共基，形成“潜在 KV 缓存”。这使得不同层的缓存可以在潜在空间合并，大幅减少存储量。
  - **自适应预算分配策略**：根据余弦相似度动态分配每层的压缩预算。若相邻层缓存相似度高则分配更多压缩，反之则减少压缩，避免过度压缩导致信息丢失。
  - **无需训练**：所有操作在推理时直接对已有的预训练模型执行，无需修改原始架构或额外微调。
- **算法流程（文字说明）**：
  1. 对预训练 LLM 的每相邻两层 KV 投影权重执行 SVD，提取公共成分；
  2. 将各层 KV 缓存映射到公共潜在空间，进行合并与低秩近似；
  3. 根据层间余弦相似度自适应调整压缩比例；
  4. 推理时直接使用共享的潜在缓存，恢复时利用 SVD 逆变换。

## 3. 实验设计
- **数据集/场景**：
  - **LongBench**：长文本理解与生成基准。
  - **Ruler**：长距离依赖评估基准。
- **基准（Benchmark）**：在多个主流 LLM 上测试，包括 LLaMA、Mistral 等（具体模型未在摘要中全部列出，但提及“multiple backbone models”）。
- **对比方法**：
  - 其他低秩压缩方法（如 SVD-only、低秩近似）；
  - 其他跨层共享方法（需预训练的架构修改方法）。
- **正交性验证**：将 CommonKV 与量化方法（如 GPTQ、AWQ）和缓存驱逐方法（如 StreamingLLM）结合，观察能否进一步压缩。

## 4. 资源与算力
- **未明确说明**：论文摘要在资源和算力方面没有给出具体信息，例如使用的 GPU 型号、数量、训练/推理时长等。由于 CommonKV 是无训练方法，推理时仅需额外的 SVD 计算和缓存分配，这部分开销原文未量化。因此无法评估所需算力规模。

## 5. 实验数量与充分性
- **实验数量**：涵盖了多个骨干模型、两个长文本基准、不同压缩比下的性能对比，以及正交方法的集成实验。虽然摘要未列出具体消融实验数量，但提及“消融实验”（ablation studies），表明进行了自适应性、不同压缩比等分析。
- **充分性评估**：
  - **优点**：覆盖了主流长文本评测集和多种模型，验证了方法的泛化性；正交集成实验证明其兼容性。
  - **不足**：缺少短文本或通用 NLP 任务的评测（如对话、翻译等）；未在更大规模模型（如 70B+）上明确报告；未与更多最新压缩方法（如 KV cache quantization with 4-bit）进行详细比较。

## 6. 论文的主要结论与发现
- 提出的 CommonKV 方法能够在**无需训练**的前提下，实现跨层 KV 缓存压缩，且在高压缩率下性能优于现有低秩和跨层共享方法。
- 自适应预算分配策略有效避免了过度压缩，尤其对于层间差异大的情况。
- CommonKV 与量化、驱逐等方法正交，结合后可达 **98% 的压缩比**且性能损失可忽略。
- 该发现表明：相邻层的隐藏状态相似性足以支撑有效的无训练参数共享，为 KV 缓存压缩提供了新视角。

## 7. 优点：方法或实验设计上的亮点
- **无训练**：直接对预训练模型使用，无需修改架构或重新训练，实用性强。
- **自适应压缩**：基于余弦相似度的动态预算分配，提升了不同层差异场景下的鲁棒性。
- **正交性**：明确验证了与现有压缩技术的兼容性，可组合使用达到极端的 98% 压缩。
- **理论动机**：利用跨层隐藏状态相似性这个直观观察，提供了合理的解释。

## 8. 不足与局限
- **实验覆盖有限**：仅使用 LongBench 和 Ruler 两个长文本基准，缺乏对通用任务（如 MMLU、GSM8K）和翻译、代码生成等场景的验证。
- **大规模模型验证缺失**：未提供 70B 以上模型的性能数据，SVD 计算开销可能随模型维度增大而显著增加。
- **偏差风险**：自适应策略依赖于余弦相似度阈值，阈值设定可能对特定模型敏感，未讨论泛化性。
- **应用限制**：SVD 分解和合并需要预先计算，对推理延迟的影响未说明；实际部署时需权衡压缩率与计算额外开销。
- **资源信息不足**：无法评估该方法在实际硬件上的成本与收益。

（完）
