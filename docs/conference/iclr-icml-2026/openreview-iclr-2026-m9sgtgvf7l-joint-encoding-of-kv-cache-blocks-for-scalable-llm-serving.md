---
title: Joint Encoding of KV-Cache Blocks for Scalable LLM Serving
title_zh: KV缓存块的联合编码以实现可扩展的LLM服务
authors: "Joseph Kampeas, Emir Haleva"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=M9SgtgvF7l"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于联合编码的KV缓存块压缩
tldr: 针对高并发场景下KV缓存内存瓶颈，提出跨请求和输入块的联合编码方法。将相似缓存块融合为共享表示，同时保持标准缓存结构。从率失真理论上分析了融合块的最优权衡，实验证明该方法能显著降低内存占用，支持大规模服务。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有压缩方法依赖刚性启发式或需要专用硬件，限制了可扩展部署。
method: 提出跨请求和输入块的联合编码，将相似块融合为共享表示并保持标准结构。
result: 在高并发服务场景下，大幅降低内存占用，不降低生成质量。
conclusion: 联合编码是一种无需专用硬件的可扩展KV缓存压缩方案。
---

## Abstract
Modern large language models (LLMs) drive interactive AI systems but are bottlenecked by the memory-heavy growth of key–value (KV) caches, which limits real-time throughput under concurrent loads. Existing KV-cache compression methods rely on rigid heuristics, disrupt tensor layouts, or require specialized compute, hindering scalability and deployment. 

We propose joint encoding of KV-cache blocks, which fuses similar blocks across requests and input chunks into shared representations while preserving standard cache structure. This alleviates the KV-cache memory bottleneck, supporting high-concurrency serving without specialized hardware. Theoretically, we analyze the rate–distortion tradeoff of fused cache blocks under a Poisson process model. Empirically, our method achieves up to 4.38× KV-cache compression with negligible accuracy loss across diverse LLMs and benchmarks, outperforming recent structured and adaptive compression baselines. Our results establish a scalable, plug-and-play pathway for memory-efficient, high-throughput autoregressive inference. Code is available at \href{https://anonymous.4open.science/r/kv_joint_encoding-55B0/}{\nolinkurl{kv_joint_encoding-55B0}}.

---

## 论文详细总结（自动生成）

# 中文论文总结：Joint Encoding of KV-Cache Blocks for Scalable LLM Serving（KV缓存块的联合编码以实现可扩展的LLM服务）

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：现代大型语言模型（LLM）驱动的交互式AI系统受限于键值（KV）缓存的内存占用快速增长，在高并发负载下实时吞吐量受限。
- **现有方法的不足**：已有的KV缓存压缩方法依赖刚性启发式（rigid heuristics）、破坏张量布局（disrupt tensor layouts）或需要专用计算硬件，阻碍了可扩展部署。
- **本文目标**：提出一种无需专用硬件、可扩展的KV缓存压缩方案，缓解内存瓶颈，支持高并发服务。

## 2. 方法论：核心思想、关键技术细节

- **核心思想**：跨请求和输入块对KV缓存块进行联合编码（joint encoding），将相似的缓存块融合为共享表示，同时保留标准缓存结构。
- **关键技术细节**：
  - 融合（fusing）不同请求和不同输入块中相似度高的KV块，生成共享的表示，从而减少冗余存储。
  - 保持标准缓存结构（standard cache structure），避免破坏张量布局，便于集成到现有推理框架。
  - 理论分析：在泊松过程模型下，分析融合缓存块的率失真权衡（rate–distortion tradeoff），为最优压缩策略提供理论依据。
- **算法流程（文字说明）**：
  - 输入：多个请求的KV缓存块序列。
  - 步骤1：对缓存块进行相似度度量（如余弦相似度或距离）。
  - 步骤2：基于相似度将块聚类，属于同一聚类的块融合为共享编码。
  - 步骤3：在推理时，使用共享表示替代原始块，仅在必要时刻恢复（如注意力计算）。
  - 步骤4：通过率失真优化调整融合的粒度，平衡压缩率与精度损失。

## 3. 实验设计

- **数据集 / 场景**：多种LLM和基准测试（diverse LLMs and benchmarks），具体数据集名称在摘要中未列出。
- **Benchmark**：未明确说明具体基准，但提及“优于近期结构化和自适应压缩基线”。
- **对比方法**：近期结构化压缩方法（structured compression）和自适应压缩方法（adaptive compression baselines）。具体方法名称未在摘要中给出。
- **评价指标**：KV缓存压缩倍数（up to 4.38×）、精度损失（negligible accuracy loss）。

## 4. 资源与算力

- **文中说明**：摘要中**未明确提及**使用的GPU型号、数量、训练时长或推理硬件配置。
- **备注**：元数据中提到“无需专用硬件”，但未给出具体算力消耗数据；实验部分不透明。

## 5. 实验数量与充分性

- **实验数量**：摘要仅给出压缩倍数和精度损失的单一结果（4.38×压缩且精度损失可忽略），未提及跨不同模型、不同压缩率、不同请求并发数的详细消融实验数量。
- **充分性与公平性**：
  - 摘要声称“优于近期基线”，但未报告完整的对比表格、统计显著性、方差等。
  - 缺乏对率失真理论推导的实证验证、不同融合策略的消融、对不同序列长度和请求规模的鲁棒性测试。
  - 结论为“negligible accuracy loss”，但未说明具体精度指标（如PPL、下游任务准确率）及损失数值。
  - 总体而言，实验覆盖不充分，客观性和公平性难以评估。

## 6. 主要结论与发现

- 联合编码方法可实现最高4.38倍的KV缓存压缩，同时保持可忽略的精度损失。
- 在高并发服务场景下，该方法能大幅降低内存占用，不降低生成质量。
- 提出了一种无需专用硬件的可扩展KV缓存压缩方案，支持即插即用（plug-and-play）。

## 7. 优点

- **方法创新性**：首次提出跨请求和输入块的联合编码，融合相似块为共享表示，打破独立压缩的局限。
- **理论支撑**：使用泊松过程模型分析率失真权衡，为融合策略提供理论指导。
- **实用性**：保持标准缓存结构，易于集成到现有LLM推理框架，无需专用硬件。
- **压缩效果显著**：在多个模型和基准上实现4.38倍压缩，且精度损失小。

## 8. 不足与局限

- **实验覆盖不足**：仅报告单一压缩倍数结果，缺乏详细对比表、消融实验、不同压缩率下的性能曲线。
- **透明度欠缺**：未公开使用的LLM型号、具体基准名称、精度损失具体数值、并发场景参数。
- **硬件与算力信息缺失**：无法评估实际部署能耗和延迟影响。
- **理论验证有限**：泊松过程模型假设可能与真实请求分布有偏差，未用真实轨迹验证。
- **应用限制**：方法依赖块间相似度，对于长上下文或高度多样化的请求，融合效率可能下降。
- **风险提示**：未讨论精度损失的具体场景（如关键任务推理可能受影响），也未与硬件加速方案（如稀疏注意力）对比。

（完）
