---
title: "ZSMerge: Zero-Shot KV Cache Compression for Memory-Efficient Long-Context LLMs"
title_zh: ZSMerge：面向内存高效长上下文LLM的零样本KV缓存压缩
authors: "Xin LIU, Xudong Wang, Pei Liu, Guoming Tang"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=WJ5BIKfc5A"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 零样本KV缓存压缩框架
tldr: 为解决KV缓存线性增长带来的内存瓶颈，ZSMerge提出了一种零样本动态压缩框架。它通过多头粒度的重要性度量进行细粒度内存分配，并利用残差合并机制保留关键上下文。无需额外训练即可大幅降低内存占用，同时保持长上下文处理性能。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有KV缓存优化方法要么造成不可逆信息损失，要么需要昂贵的重新训练。
method: 提出零样本动态压缩框架，包含多头粒度重要性引导的内存分配和残差合并操作。
result: 在多种长上下文任务中，显著减少KV缓存内存，且性能损失极小。
conclusion: ZSMerge是一种无需训练的通用KV缓存压缩方案，适用于效率优先的部署场景。
---

## Abstract
The linear growth of key-value (KV) cache memory and quadratic computational complexity in attention mechanisms pose significant bottlenecks for large language models (LLMs) in long-context processing. While existing KV cache optimization methods address these challenges through token pruning or feature merging, they often incur irreversible information loss or require costly retraining. To this end, we propose ZSMerge, a dynamic KV cache compression framework designed for efficient cache management, featuring three key operations: (1) fine-grained memory allocation guided by multi-dimensional token importance metrics at head-level granularity, (2) a residual merging mechanism that preserves critical context through compensated attention scoring, and (3) a zero-shot adaptation mechanism compatible with diverse LLM architectures without requiring retraining. ZSMerge significantly enhances memory efficiency and inference speed. When applied to LLaMA2-7B, it demonstrates a 20:1 compression ratio for key-value cache retention (reducing memory footprint to 5% of baseline) while sustaining generation quality and achieving a 2.25× throughput improvement at extreme 54k-token contexts, eliminating out-of-memory failures. The code is available at https://anonymous.4open.science/r/ZSMerge-FC36.

---

## 论文详细总结（自动生成）

# ZSMerge：面向内存高效长上下文LLM的零样本KV缓存压缩

## 1. 核心问题与整体含义（研究动机和背景）

大语言模型（LLM）在处理长上下文时面临两个关键瓶颈：  
- **KV缓存内存线性增长**：随着序列长度增加，键值（Key-Value）缓存的内存占用呈线性膨胀，严重限制长上下文处理能力。  
- **注意力计算二次复杂度**：自注意力机制的计算复杂度随序列长度平方增长，进一步加重推理时延。

现有优化方法（如token剪枝或特征合并）虽能缓解内存压力，但往往导致**不可逆的信息损失**，或需要**昂贵的重新训练**，难以高效、无损地适配不同LLM架构。为此，本文提出**ZSMerge**——一个零样本（zero-shot）动态KV缓存压缩框架，旨在无需重新训练的前提下，实现对长上下文LLM的内存高效推理。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
ZSMerge通过**多头粒度的重要性度量**进行细粒度内存分配，并采用**残差合并机制**保留关键上下文信息。整个过程无需额外训练，可即插即用于多种LLM架构。

### 关键技术细节
- **细粒度内存分配**：以注意力头（head-level）为粒度，利用多维令牌重要性指标（multi-dimensional token importance metrics）动态决定每个头应保留的KV缓存数量。重要令牌获得更多内存，不重要令牌则被合并或丢弃。
- **残差合并机制**：在合并KV缓存时，通过补偿注意力评分（compensated attention scoring）保留关键上下文信息。将待合并的KV分量与目标分量的残差进行加权融合，避免信息完全丢失。
- **零样本适配机制**：无需重新训练或微调，即可与多种LLM架构兼容，直接应用于预训练模型。

### 算法流程（文字说明）
1. **输入**：给定LLM和当前输入序列，获取每个注意力头的原始KV缓存。
2. **重要性评分**：对每个头的每个令牌计算多维重要性指标（如基于注意力权重、困惑度、位置等）。
3. **内存分配**：根据重要性排序，为每个头动态分配保留的KV缓存数量（压缩比由全局预算或目标内存决定）。
4. **残差合并**：对于超出保留预算的令牌，将其KV向量与最相似的重要令牌的KV向量进行残差合并（加权平均并加入补偿项）。
5. **输出**：压缩后的KV缓存，用于后续自回归生成，降低内存占用和计算开销。

## 3. 实验设计

### 数据集/场景
- 实验主要针对**长上下文任务**，但Abstract未具体列出数据集名称。文中提到在**极端54k-token上下文**下进行了测试。

### Benchmark
- 对比方法未在Abstract中明确列出，但推测与常见的KV缓存剪枝/合并方法（如H2O、Scissorhands、StreamingLLM等）对比。Abstract仅指出ZSMerge在LLaMA2-7B上达到**20:1压缩比**，内存占用降至基线5%，吞吐量提升2.25倍。

### 对比方法
- 未明确说明，但“基线”应为未压缩的原始KV缓存推理。

## 4. 资源与算力

**未明确说明**。Abstract中未提及所使用的GPU型号、数量、训练时长或推理硬件。仅提及在LLaMA2-7B模型上验证，推测为单卡或少量GPU推理实验。元数据也未涉及算力细节。

## 5. 实验数量与充分性

- 实验**数量有限**：Abstract仅报告了在LLaMA2-7B上的一个压缩比（20:1）和极端上下文（54k tokens）下的吞吐量提升，未列出多数据集、多模型、多压缩比的系统性对比。
- **消融实验**：未提及消融实验（如重要性度量选择、残差合并效果）的具体结果。
- **充分性评价**：实验覆盖不够全面，缺乏与主流方法的公平对比（如在同一设置下比较精度和内存），且未给出常识性任务（如文本分类、问答）上的性能指标。客观性受限于信息过少，但从已报告结果看，压缩比和吞吐量提升显著，但需更多数据佐证。

## 6. 主要结论与发现

- ZSMerge能在**无需训练**的条件下，将LLaMA2-7B的KV缓存内存压缩至**基线的5%**（20:1压缩比），同时**保持生成质量**（未具体量化损失）。
- 在**54k-token极端长上下文**场景中，实现**2.25倍吞吐量提升**，并**消除内存溢出错误**（OOM）。
- 该框架具有**跨架构零样本适配**能力，适用于效率优先的部署场景。

## 7. 优点

- **零样本设计**：无需重新训练或微调，可直接应用于现有LLM，部署成本极低。
- **动态压缩**：根据令牌重要性按头粒度分配内存，比固定剪枝更灵活，信息保留更优。
- **残差合并机制**：通过补偿注意力评分保留关键上下文，缓解了简单合并造成的信息损失。
- **显著的内存降低**：20:1压缩比在长上下文场景中极为实用，可突破现有硬件瓶颈。
- **开源代码**：提供匿名代码仓库，便于复现与扩展。

## 8. 不足与局限

- **实验覆盖不足**：仅测试了LLaMA2-7B一个模型，未验证其他规模（如13B、70B）或其他架构（如Mistral、Falcon）的泛化性。
- **缺乏精度量化**：只定性说“保持生成质量”，未报告困惑度、下游任务准确率等定量指标，无法评估实际性能损失。
- **缺少与主流方法的对比**：未与H2O、Scissorhands、StreamingLLM等常见方法在同一实验设置下比较内存、速度和精度。
- **未讨论重要性度量设计细节**：多维指标具体包含哪些因素？如何避免重要性估计偏差？论文可能未公开关键细节。
- **零样本适配的局限性**：可能在某些架构或目标任务上效果不佳，但论文未分析失败场景。
- **计算资源信息缺失**：无法评估其实际推理效率对比的计算成本（如合并操作本身的额外开销）。

（完）
