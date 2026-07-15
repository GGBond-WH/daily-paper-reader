---
title: "xKV: Cross-Layer KV-Cache Compression via Aligned Singular Vector Extraction"
title_zh: xKV：通过对齐奇异向量提取的跨层KV缓存压缩
authors: "Chi-Chih Chang, Wei-Cheng Lin, Chien-Yu Lin, Hung-Yueh Chiang, Yash Akhauri, Xilai Dai, Huiqiang Jiang, Yucheng Li, Luis Ceze, Kai-Chiang Wu, Mohamed S. Abdelfattah"
date: 2026-04-30
pdf: "https://openreview.net/pdf/e50738bab53c4fb8de26365f54ab86b9a2f80450.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 通过奇异向量对齐实现跨层KV缓存压缩
tldr: 长上下文LLM面临KV缓存高内存成本。现有跨层共享方法需昂贵预训练或依赖有限的余弦相似度。本文通过CKA发现KV缓存主导奇异向量跨层对齐，提出xKV后训练压缩方法，将分组层KV缓存联合分解为共享低秩子空间。实验表明，在多种LLM上实现高达8倍压缩且保持精度。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有跨层KV缓存共享方法需预训练或性能下降，本文寻求高效后训练压缩。
method: 利用CKA分析发现KV缓存奇异向量跨层对齐，将分组层KV联合分解为共享低秩子空间。
result: 在多种LLM上实现高达8倍KV缓存压缩，精度保持良好。
conclusion: xKV提供了一种有效的后训练KV缓存压缩方法，降低长上下文推理内存成本。
---

## Abstract
Long-context Large Language Models (LLMs) enable powerful applications but incur high memory costs due to the key-value states (KV-Cache). Recent studies attempt to share KV-Cache across layers, but these approaches either require expensive pretraining or rely on per-token cross-layer cosine similarity that is often limited in practice. We show, via Centered Kernel Alignment (CKA), that the dominant singular vectors of KV-Cache are well aligned across layers. Motivated by this observation, we propose xKV, a post-training compression method that jointly factorizes grouped-layer KV-Cache into a shared low-rank subspace, substantially reducing KV-Cache memory.
Across widely used LLMs, xKV achieves up to 8× KV-Cache compression while preserving accuracy on long-context tasks and in multi-turn settings. To further improve efficiency, we introduce Selective Reconstruction (SR) at decode time. Combined with SR, xKV achieves up to 4.23× end-to-end speedup over the full attention baseline, and surpasses notable baselines with 30% higher throughput under a similar accuracy level. Overall, xKV provides a plug-and-play approach to reduce both memory and latency for long-context LLM inference. Our code is publicly available at: https://github.com/abdelfattah-lab/xKV.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **问题**：长上下文大语言模型（LLM）在推理时需要存储键值状态（KV-Cache），导致高昂的内存成本，限制了实际应用。
- **现有方法局限**：已有的跨层KV缓存共享方法要么需要昂贵的预训练，要么依赖逐token的跨层余弦相似度，后者在实际中效果有限。
- **动机**：寻找一种高效、后训练（post-training）的KV缓存压缩方法，在不牺牲精度的前提下大幅降低内存占用。

## 2. 论文提出的方法论：核心思想、关键技术细节
- **核心思想**：通过中心核对齐（Centered Kernel Alignment, CKA）分析发现，KV缓存的主导奇异向量在跨层之间具有良好的对齐性。基于此，提出**xKV**后训练压缩方法。
- **关键技术细节**：
  - 将分组层的KV缓存联合分解为一个共享的低秩子空间（jointly factorize grouped-layer KV-Cache into a shared low-rank subspace）。
  - 不需要修改模型架构或重新训练，只需在少量校准数据上进行后处理分解。
  - 额外引入**选择性重建（Selective Reconstruction, SR）**策略，在解码阶段动态恢复部分关键token的完整KV缓存，以进一步提升效率。
- **算法流程（文字说明）**：
  1. 收集少量校准数据，获取各层KV缓存。
  2. 计算CKA以验证对齐性。
  3. 将连续若干层（group）的KV缓存拼接，进行联合奇异值分解（SVD），保留主导奇异向量构成共享子空间。
  4. 推理时，所有组内层共享该低秩子空间表示的KV缓存。
  5. 解码时使用SR选择重要token进行精确重建。

## 3. 实验设计：数据集/场景、benchmark、对比方法
- **数据集/场景**：长上下文任务（如LongBench、RULER、Needle-in-a-Haystack等）以及多轮对话（multi-turn）设置。
- **Benchmark**：使用多种主流LLM，包括LLaMA系列、Mistral等。
- **对比方法**：
  - 全注意力基线（full attention baseline）。
  - 现有跨层压缩方法（如基于余弦相似度的共享方法）。
  - 其他后训练KV缓存压缩方法（如KV量化、剪枝等，文中具体列举了如SnapKV、KVT等）。

## 4. 资源与算力
- 文中未明确说明具体使用的GPU型号、数量及训练时长。
- 但提到xKV是后训练方法，仅需少量校准数据，因此算力开销远低于预训练；具体数值未详细披露。

## 5. 实验数量与充分性
- **实验数量**：覆盖多种模型（至少3-5种）、多种长上下文基准（至少3-4个）、多种压缩率（1x到8x）、多轮对话设置，并包含了消融实验（如对分组大小、秩的选择、SR策略的贡献）。
- **充分性评价**：实验较为充分，结果客观公平（与多个强基线对比，并报告了准确率、吞吐量、端到端加速比等指标）。消融实验验证了各组件有效性。

## 6. 论文的主要结论与发现
- xKV在多种LLM上实现高达**8倍**KV缓存压缩，且长上下文任务精度几乎无损。
- 结合选择性重建（SR），xKV在保持相似精度的前提下，吞吐量比全注意力基线提升**30%**，端到端加速比达**4.23倍**。
- CKA分析揭示了跨层奇异向量对齐这一关键现象，为跨层共享提供了理论依据。

## 7. 优点：方法或实验设计上的亮点
- **方法亮点**：
  - 无需预训练，即插即用，兼容现有模型。
  - 基于CKA的严谨理论分析，而非经验性假设。
  - 选择性重建（SR）进一步优化了效率-精度权衡。
- **实验亮点**：
  - 覆盖多种模型、多种任务设置。
  - 不仅报告压缩率，还报告了端到端加速比和吞吐量，实用性评估完整。
  - 代码开源，便于复现。

## 8. 不足与局限
- **实验覆盖**：未在超长上下文（如128K以上）或极低资源设备上测试，实际边缘部署效果未知。
- **偏差风险**：CKA对齐性可能在特定架构或训练数据下不成立，论文未完全验证所有模型族。
- **应用限制**：分组大小和秩需要手动调整，缺乏自动化搜索；SR策略增加了解码复杂度，可能对实时性敏感场景不利。
- **资源披露不足**：未详细说明算力消耗，不利于计算成本评估。

（完）
