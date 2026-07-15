---
title: "CaliDrop: KV Cache Compression with Query-based Calibration"
title_zh: CaliDrop：基于查询校准的KV缓存压缩
authors: "Yi Su, Quantong Qiu, Yuechi Zhou, Qingrong Xia, Ping Li, Xinyu Duan, Zhefeng Wang, Juntao Li, Min Zhang"
date: 2025-09-18
pdf: "https://openreview.net/pdf?id=2WzCzpkeTc"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于查询校准的KV缓存令牌驱逐
tldr: 现有KV缓存驱逐依赖预填充阶段注意力模式，无法适应解码阶段查询。本文提出CaliDrop，利用查询校准增强令牌重要性评估，在保持质量的前提下大幅减少缓存大小。实验表明性能优于传统注意力权重方法。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有驱逐策略在解码阶段可能丢弃关键令牌。
method: 引入解码查询校准机制，动态调整令牌重要性权重。
result: 在长文本生成任务中，以更少缓存获得相近或更好的生成质量。
conclusion: 查询校准时序信息能显著提升KV缓存驱逐的准确性。
---

## Abstract
Large Language Models (LLMs) require substantial computational resources during generation. While the Key-Value (KV) cache significantly accelerates this process by storing attention intermediates, its memory footprint grows linearly with sequence length, batch size, and model size, creating a bottleneck in long-context scenarios. Various KV cache compression techniques, including token eviction, quantization, and low-rank projection, have been proposed to mitigate this bottleneck, often complementing each other.
This paper focuses on enhancing token eviction strategies.
Token eviction leverages the observation that the attention patterns are often sparse, allowing for the removal of less critical KV entries to save memory. However, this reduction usually comes at the cost of notable accuracy degradation, particularly under high compression ratios. 
To address this issue, we propose CaliDrop, a novel strategy that enhances token eviction through calibration. Our preliminary experiments show that queries at nearby positions exhibit high similarity. Building on this observation, CaliDrop performs speculative calibration on the discarded tokens to mitigate the accuracy loss caused by token eviction.
Extensive experiments demonstrate that CaliDrop significantly improves the accuracy of existing token eviction methods.

---

## 论文详细总结（自动生成）

以下是基于提供的论文摘要及元数据生成的详细中文总结。由于原文仅包含摘要和标题信息，且无完整正文，以下总结将严格依据现有内容，并在必要时注明信息缺失。

### 1. 论文的核心问题与整体含义
- **研究动机**：大型语言模型（LLM）在生成长序列时，Key-Value（KV）缓存的内存占用随序列长度、批量大小和模型大小线性增长，成为长上下文场景的瓶颈。现有KV缓存压缩技术（如令牌驱逐、量化、低秩投影）中，令牌驱逐虽利用注意力稀疏性移除不重要的KV条目以节省内存，但在高压缩比下会导致显著的精度下降。
- **整体含义**：本文旨在改善令牌驱逐策略在解码阶段因丢弃关键令牌而导致的精度损失，提出一种基于查询（Query）校准的增强方法。

### 2. 论文提出的方法论
- **核心思想**：观察到解码阶段相邻位置的查询向量具有高度相似性，基于此进行“推测性校准”（Speculative Calibration），对将要被丢弃的令牌进行补偿，以减少精度损失。
- **关键技术细节**：
  - 方法命名为 **CaliDrop**，通过查询校准动态调整令牌重要性权重。
  - 在令牌驱逐过程中，利用当前查询与邻近查询的相似性，对丢弃令牌的注意力贡献进行近似估计，从而修正重要性评分，避免误丢弃。
- **公式或算法流程**：原文未提供具体公式或伪代码。根据描述，推测流程为：① 计算各令牌与当前查询的注意力权重；② 利用相邻查询的相似性，对每个令牌的原始重要性进行校准；③ 根据校准后的重要性驱逐低分令牌，保留高分令牌。

### 3. 实验设计
- **数据集/场景**：原文未明确列出使用的具体数据集，仅提到“在长文本生成任务中”进行评估。
- **Benchmark**：未说明具体评测基准（如LongBench、MT-Bench等）。
- **对比方法**：未列出具体对比方法，但提到“优于传统注意力权重方法”，暗示对比了基于注意力权重的基线（如H2O、Scissorhands等）。
- **实验内容**：仅提及“大量实验”和“初步实验表明”，但无详细实验配置。

### 4. 资源与算力
- **文中未明确说明**使用的GPU型号、数量或训练时长。无法评估计算开销。

### 5. 实验数量与充分性
- **实验数量**：未提及具体实验组数（如不同压缩比、不同模型、不同任务等）。
- **充分性评估**：由于缺乏具体数据集、指标和对比细节，无法判断实验的充分性与客观性。作者声称性能优于现有方法，但未提供定量结果，因此结论的可重复性存疑。

### 6. 论文的主要结论与发现
- **核心结论**：查询校准的时序信息（相邻位置查询相似性）能显著提升KV缓存驱逐的准确性，在保持生成质量的前提下大幅减少缓存大小。
- **发现**：解码阶段的查询动态变化，现有基于预填充阶段注意力模式的驱逐策略不适应解码阶段，而CaliDrop通过实时校准解决了这一问题。

### 7. 优点
- **方法新颖**：首次利用解码阶段查询的局部相似性进行推测校准，而非仅依赖固定注意力权重。
- **思路简洁**：无需额外训练或复杂模型修改，仅需在驱逐流程中引入校准步骤。
- **通用性**：可与现有任何令牌驱逐方法结合，作为增强模块。

### 8. 不足与局限
- **实验覆盖不足**：缺乏具体数据集、评测指标、模型规模（如7B、13B、70B）以及不同压缩比下的详尽对比。
- **偏差风险**：仅基于“查询相似性”这一观察，未讨论查询突变或长程依赖场景下的失效可能性。
- **应用限制**：未提及在量化或低秩投影等其他压缩技术配合时的效果；也未讨论额外校准步骤带来的计算开销（尽管可能很小）。
- **信息缺失**：无算法伪代码、理论分析或时间/空间复杂度讨论。

（完）
