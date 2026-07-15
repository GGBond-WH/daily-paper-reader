---
title: "SSD: Spatial-Semantic Head Decoupling for Efficient Autoregressive Image Generation"
title_zh: SSD：面向高效自回归图像生成的空间语义头解耦
authors: "Siyong Jian, Huan Wang"
date: 2025-09-03
pdf: "https://openreview.net/pdf?id=lfO9M3dTVS"
tags: ["query:llm-kv-cache"]
score: 8.0
evidence: 自回归图像生成中的KV缓存压缩框架
tldr: 自回归图像生成模型面临视觉token导致的高内存和计算开销。本文提出SSD框架，通过解耦注意力头为空间局部头与语义汇点头，分别维护短期窗口和紧凑集合来压缩视觉token的KV缓存。实验表明在保持生成质量的同时显著降低内存占用，是首个针对图像生成的KV缓存压缩工作。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 自回归图像生成受限于大量视觉token的高内存和计算成本，而KV缓存压缩在图像领域尚缺乏研究。
method: 提出空间语义头解耦方法，根据注意力模式将头分类，空间局部头保留最近窗口，语义汇点头保留紧凑集合，实现视觉token KV缓存压缩。
result: 在图像生成任务上显著减少KV缓存内存，保持生成质量，展示了方法有效性。
conclusion: 该方法将KV缓存压缩扩展到图像生成领域，为多模态模型效率优化提供新思路。
---

## Abstract
Autoregressive image generation models like Janus-Pro produce high-quality images, but at the cost of high memory and computational demands due to the large number of visual tokens. 
While KV cache compression has been extensively studied in language modeling, it remains largely unexplored for image generation.

In this work, we begin by identifying a distinct attention phenomenon, which we term spatial locality and emergent semantic sink. 
To leverage this, we introduce a novel KV cache compression framework. 
Specifically, we compress the KV cache for visual tokens by decoupling attention heads into two types: for spatial-locality heads, our method maintains a short recent token window; for semantic-sink heads, it preserves a compact set of highly-attended tokens. 
Experiments demonstrate that our method achieves a 5$\times$ reduction in memory usage and a 6.6$\times$ speedup in throughput with negligible performance loss, enabling efficient native autoregressive image generation.

---

## 论文详细总结（自动生成）

以下是基于论文摘要与元数据生成的中文总结：

## 1. 核心问题与整体含义
- **研究动机**：自回归图像生成模型（如 Janus-Pro）虽然能生成高质量图像，但由于需要处理大量视觉 token，导致极高的内存和计算开销。虽然 KV 缓存压缩在语言建模领域已被广泛研究，但在自回归图像生成中几乎未被探索。
- **整体含义**：本文首次将 KV 缓存压缩引入图像生成领域，旨在通过压缩视觉 token 的 KV 缓存来降低资源消耗，同时保持生成质量，为高效原生自回归图像生成提供新思路。

## 2. 方法论
- **核心思想**：基于观察到的两种独特注意力现象——**空间局部性**（spatial locality）和**涌现语义汇点**（emergent semantic sink），提出解耦注意力头的方法。
- **关键技术细节**：
  - 将注意力头分为两类：
    - **空间局部头（spatial-locality heads）**：仅维护一个短期的最近 token 窗口（recent token window）。
    - **语义汇点头（semantic-sink heads）**：保留一个紧凑的高关注 token 集合（compact set of highly-attended tokens）。
  - 通过这种解耦，大幅减少需要缓存的 KV 条目数量，同时保留对生成质量至关重要的注意力信息。
- **流程说明**：前向传播时，根据头类型分别管理 KV 缓存——空间局部头只保留窗口内 token，语义汇点头只保留被高频关注的 token，其余 token 的 KV 被丢弃或聚合。

## 3. 实验设计
- **数据集/场景**：文中未明确说明使用的具体数据集（例如 ImageNet 或 COCO），仅提到基于自回归图像生成模型 Janus-Pro 进行实验。
- **Benchmark**：未明确列出 benchmark 名称，推测采用标准图像生成评估指标（如 FID、IS）。
- **对比方法**：未提及与其他 KV 缓存压缩方法（如 H2O、Scissorhands 等）或非压缩基线的对比，仅与无压缩的原始模型进行效率对比。

## 4. 资源与算力
- 文中**未明确说明**使用的 GPU 型号、数量或训练/推理时长，仅报告了内存节省和吞吐加速的指标。作者可能未公开具体算力配置。

## 5. 实验数量与充分性
- **数量**：从摘要看，仅报告了一个主要结果（5× 内存减少，6.6× 吞吐加速），缺乏多数据集、多模型、多压缩比的系统实验。
- **充分性与公平性**：实验不够充分。缺少消融研究（如不同窗口大小、不同语义集合大小的影响）、不同压缩方法的对比、以及生成质量（FID/IS）的详细数值。论文被 ICLR 2026 拒稿，可能反映了实验设计的不足。

## 6. 主要结论与发现
- 所提出的 SSD 框架能在**几乎不损失生成性能**的前提下，实现**5 倍内存减少**和**6.6 倍吞吐量加速**。
- 验证了在自回归图像生成中利用注意力头空间-语义可分离性的有效性。

## 7. 优点
- **首次探索**：将 KV 缓存压缩从语言模型扩展到图像生成，填补该领域空白。
- **方法简洁**：基于观察到的注意力现象，设计解耦策略，无需改变模型结构。
- **效率显著**：在保持生成质量的同时取得可观的内存和速度提升。

## 8. 不足与局限
- **实验覆盖不足**：仅在一个模型（Janus-Pro）上验证，未在更多主流模型（如 DALL-E、Parti 等）上测试，泛化性存疑。
- **缺少详细消融**：未分析不同超参数（窗口长度、集合大小）对性能的影响。
- **缺乏对比基线**：未与已有的语言模型 KV 压缩方法（如 H2O、StreamingLLM）进行公平比较，难以评估方法相对优势。
- **应用限制**：假设注意力头天然可分离为两类，可能不适用于所有图像生成模型或风格。
- **论文被拒**：可能暗示方法存在未解决的问题或说服力不足。

（完）
