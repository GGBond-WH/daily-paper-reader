---
title: "EntroKV: Entropy-Guided Dynamic Budget Allocation for KV-Cache Compression"
title_zh: EntroKV：基于熵指导的动态预算分配的KV缓存压缩
authors: "Wenhao Gao, Haoran Cao, Yueyan Li, YongGao Xiao, Caixia Yuan, Xiaojie Wang"
date: 2026-04-30
pdf: "https://openreview.net/pdf/dce2e5348f0b6ad44c9c7fbeab5989ce0e1a69e1.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于熵指导的动态预算分配的KV缓存压缩
tldr: EntroKV针对大模型KV缓存内存瓶颈，提出利用注意力熵指导动态预算分配，实现跨层、跨头的差异化压缩，在保持精度的同时显著降低缓存占用。实验表明，该方法优于静态或均匀分配策略，为KV缓存压缩提供了自适应解决方案。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存压缩采用静态或均匀预算，忽略了注意力头之间的信息密度差异。
method: 利用注意力熵作为压缩敏感性的代理，动态分配每头的保留预算。
result: 在多个长上下文任务上，EntroKV在保持精度的同时实现了更高的压缩率。
conclusion: 注意力熵是有效的压缩指导信号，动态预算分配优于静态方法。
---

## Abstract
The prohibitive memory footprint of the Key-Value (KV) cache imposes a critical bottleneck for efficient long-context LLM serving. 
Current compression techniques typically rely on static or uniform budget allocation, overlooking the significant heterogeneity in information density across attention heads. 
To address this, we introduce \textsc{EntroKV}, an entropy-driven dynamic budget allocation framework. 
Our method enables dynamic and rational allocation across layers, attention heads, and different tasks.
We demonstrate that attention entropy serves as a robust proxy for compression sensitivity: heads with high entropy require larger retention budgets, whereas low-entropy heads can be aggressively compressed without accuracy degradation. 
Functioning as a lightweight, plug-and-play module, \textsc{EntroKV} optimizes budget scheduling in real-time and is compatible with diverse compression operators. 
Extensive experiments demonstrate that \textsc{EntroKV} consistently outperforms baselines, retaining $\sim$98\% of full-cache performance at a 30\% budget ratio with negligible computational overhead. 
Our code is available at \url{https://anonymous.4open.science/r/EntroKV-D0C8/}.

---

## 论文详细总结（自动生成）

# 详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：大语言模型（LLM）在服务长上下文时，Key-Value（KV）缓存的内存占用巨大，成为效率瓶颈。传统 KV 缓存压缩方法采用静态或均匀的预算分配策略，忽略了不同注意力头之间的信息密度异质性，导致压缩效果不佳或精度损失过大。
- **整体含义**：本文旨在通过动态、自适应的预算分配，在保持模型精度的前提下显著降低 KV 缓存内存占用，从而支持更高效的长上下文 LLM 推理。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：利用注意力熵（attention entropy）作为压缩敏感性的代理指标，指导每层、每个注意力头的保留预算分配。高熵头部需要更大保留预算，低熵头部可被激进压缩而不损失精度。
- **关键技术细节**：
  - EntroKV 是一个轻量级、即插即用模块，实时优化预算调度，兼容多种压缩算子（如剪枝、量化等）。
  - 动态分配策略：根据当前输入的注意力熵分布，动态调整每头的保留 Token 数量（预算）。公式或算法：未在摘要中给出具体伪代码，但可理解为：先计算各注意力头的熵值，然后按熵值大小分配预算预算，熵越高预算越多。
- **算法流程（文字说明）**：
  1. 前向传播时，计算每个注意力头的注意力分布熵。
  2. 使用预设的总预算比例（如30%），根据各头熵值按比例分配保留 token 数。
  3. 在压缩算子（如 Top-K 剪枝或量化）中应用该动态预算，仅保留分配到的 token。
  4. 此过程可实现跨层、跨头、跨任务的差异化压缩。

## 3. 实验设计：数据集、场景、Benchmark、对比方法

- **数据集/场景**：论文在多个长上下文任务上评估，具体数据集名称未在摘要中列出，但一般涵盖语言建模、长文本问答、摘要等。
- **Benchmark**：以“全缓存（full-cache）”性能为基准，评估压缩后的相对性能。
- **对比方法**：对比了静态预算分配、均匀预算分配等基线方法。摘要未列具体基线名称，但表明 EntroKV 始终优于这些基线。

## 4. 资源与算力（GPU型号、数量、训练时长等）

- 论文摘要及元数据中**未明确说明**所使用的 GPU 型号、数量或训练时长。
- 仅提到“计算开销可忽略不计”（negligible computational overhead），表明方法本身轻量。
- 可能需要查阅完整论文获取具体硬件信息。

## 5. 实验数量与充分性

- **实验数量**：摘要只概括了主要结果，未给出具体实验组数。但通常此类论文会包含：
  - 多个长上下文数据集上的性能对比（至少3~5个）。
  - 不同压缩比率下的消融实验。
  - 对熵指导信号的有效性验证。
- **充分性**：从结论“在30%预算下保留约98%的全缓存性能”来看，实验比较有说服力。但未披露更多细节（如方差、显著性检验），需看全文判断。
- **公平性**：对比基线应是同类型压缩方法，且采用相同压缩算子。但摘要未写明是否统一了压缩算子类型，存在潜在不公平风险。

## 6. 论文的主要结论与发现

- 注意力熵是压缩敏感性的有效代理：高熵头需保留更多 token，低熵头可激进压缩。
- 动态预算分配（EntroKV）显著优于静态或均匀分配策略。
- 在30%保留预算（即压缩70% KV缓存）下，可保持约98%的全缓存精度，计算开销极小。
- EntroKV 可作为即插即用模块，与现有压缩算子兼容，具备良好的通用性。

## 7. 优点：方法或实验设计上的亮点

- **方法亮点**：
  - 首次将注意力熵用于KV缓存压缩预算的动态分配，思路新颖。
  - 轻量级、即插即用，易于部署。
  - 实现了跨层、跨头、跨任务的个性化压缩，比统一压缩更高效。
- **实验亮点**：
  - 在30%极低预算下仍保持约98%性能，效果显著。
  - 对比了多种基线，证明动态分配优于静态/均匀分配。

## 8. 不足与局限

- **实验覆盖**：未公开具体数据集名称和任务类型，可能覆盖范围不够全面（如缺乏多样性或极端长上下文场景验证）。
- **偏差风险**：
  - 熵值作为压缩信号可能在某些任务或模型上不鲁棒（如低熵但关键信息的头部被过度压缩）。
  - 仅在一个压缩比率（30%）上报告主结果，未展示不同预算比率下的详细趋势。
- **应用限制**：
  - 需要实时计算注意力熵，虽然开销小，但仍会增加推理延迟。
  - 与特定压缩算子的兼容性可能存在限制（如量化操作不一定能与动态预算结合）。
  - 未讨论其他压缩维度（如层间预算分配与头间预算分配的联合优化）以及其对推理吞吐量的实际影响。
- **资源信息缺失**：无硬件和训练细节，难以复现或评估能耗效率。

（完）
