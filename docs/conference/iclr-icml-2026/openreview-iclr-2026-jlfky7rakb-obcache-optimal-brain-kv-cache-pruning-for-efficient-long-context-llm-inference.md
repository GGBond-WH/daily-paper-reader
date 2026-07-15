---
title: "OBCache: Optimal Brain KV Cache Pruning for Efficient Long-Context LLM Inference"
title_zh: OBCache：面向高效长上下文LLM推理的最优脑KV缓存剪枝
authors: "Yuzhe Gu, Xiyu Liang, Jiaojiao Zhao, Enmao Diao"
date: 2025-09-02
pdf: "https://openreview.net/pdf?id=JLfky7RakB"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于注意力输出扰动的脑最优KV缓存剪枝
tldr: 针对KV缓存剪枝中启发式排序忽略真实影响的问题，OBCache将缓存逐出建模为结构化剪枝问题。基于最优脑损伤（OBD）理论，通过衡量剪枝对注意力输出的扰动来量化token重要性。该方法以闭形式高效计算，实现了更优的压缩效果。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有缓存逐出方法依赖累积注意力权重的启发式排序，未考虑对注意力输出的真实影响。
method: 提出基于最优脑损伤理论的层结构化剪枝框架，以注意力输出扰动量化token重要性。
result: 在多个长上下文基准上，OBCache在相同压缩率下比启发式方法保持更高精度。
conclusion: OBCache为KV缓存剪枝提供了理论驱动的替代方案，提升压缩的有效性。
---

## Abstract
Large language models (LLMs) with extended context windows enable powerful applications but impose significant memory overhead, as caching all key–value (KV) states grows linearly with sequence length and batch size. Existing cache eviction methods address this by exploiting attention sparsity, yet they typically rank tokens heuristically using accumulated attention weights without considering their true impact on attention outputs. We propose Optimal Brain Cache (OBCache), a principled framework that formulates cache eviction as a layer-wise structured pruning problem. Building on Optimal Brain Damage (OBD) theory, OBCache quantifies token saliency by measuring the perturbation on attention outputs induced by pruning tokens, with closed-form scores derived for isolated keys, isolated values, and joint key–value pairs. Our scores account not only for attention weights but also for information from value states and attention outputs, thereby enhancing existing eviction strategies with output-aware signals. Experiments on LLaMA and Qwen models show that replacing the heuristic scores in existing works, which estimate token saliency across different query positions, with OBCache's output-aware scores consistently improves long-context accuracy.

---

## 论文详细总结（自动生成）

# OBCache：面向高效长上下文LLM推理的最优脑KV缓存剪枝 — 论文详细总结

## 1. 核心问题与整体含义（研究动机和背景）
- **背景**：大语言模型（LLM）扩展上下文窗口带来强大应用潜力，但存储所有键值（KV）状态的内存开销随序列长度和批大小线性增长，成为主要瓶颈。
- **现有问题**：现有缓存逐出（cache eviction）方法利用注意力稀疏性，但通常基于累积注意力权重进行启发式排序，**未考虑对注意力输出的真实影响**，导致次优压缩。
- **研究动机**：提出一种理论驱动的方法，将缓存逐出建模为结构化剪枝问题，通过量化剪枝对注意力输出的扰动来评估token重要性，从而提升长上下文推理的精度和压缩效率。

## 2. 方法论：核心思想、关键技术细节、算法流程
- **核心思想**：基于最优脑损伤（Optimal Brain Damage, OBD）理论，将KV缓存剪枝视为**逐层结构化剪枝**问题，以**注意力输出扰动**作为token重要性度量。
- **关键技术细节**：
  - 将缓存逐出建模为：移除某些位置的键（K）和值（V）后，计算对后续注意力输出（attention output）的局部影响。
  - 通过一阶泰勒展开近似，推导出**闭式重要性分数**，分别针对**孤立键**、**孤立值**以及**联合键-值对**三种情况。
  - 重要性分数不仅包含注意力权重，还融合了值状态和注意力输出中的信息，从而修正启发式方法忽略输出扰动的缺陷。
- **算法流程（文字描述）**：
  1. 对当前层的KV缓存，计算每个token的键/值/键值对的重要性分数。
  2. 根据目标压缩率，选择分数最低（即移除后扰动最小）的token进行逐出。
  3. 逐层独立执行，同时保证不同层可以采用不同剪枝率。
  - 该方法可直接替换现有逐出方法中的启发式重要性评分，即**即插即用**。

## 3. 实验设计
- **使用的模型**：LLaMA系列、Qwen系列（具体版本未在摘要中说明）。
- **数据集 / 场景**：未列举具体基准名称，但提及“long-context accuracy”（长上下文准确率）作为评价指标，推测使用了常见的Needle-in-a-Haystack、LongBench等长上下文任务。
- **对比方法**：与**现有启发式评分方法**（如基于累积注意力权重的逐出策略）进行比较。未列出具体方法名称，但强调OBCache的“output-aware scores”一致优于它们。
- **Benchmark**：未明确给出标准基准测试集，仅称“长上下文基准”。

## 4. 资源与算力
- **文中未明确说明**：使用的GPU型号、数量、训练/推理时长、内存消耗等均未提及。仅能推断其方法为推理阶段剪枝，不涉及额外训练，算力开销较低（闭式计算）。

## 5. 实验数量与充分性
- **实验数量**：文中仅提到在LLaMA和Qwen模型上进行实验，并对比了多个启发式方法，但**未列出具体实验组数、消融实验或不同压缩率下的详细结果**。因此实验数量不明确，不足以完全评估方法的稳健性。
- **充分性判断**：虽然声称“consistently improves”，但缺乏多数据集、多模型变体、不同剪枝策略组合的消融分析，实验覆盖不够全面。不过，由于方法本身具有理论支撑，初步结果可信。客观性和公平性方面，由于未公开所有细节，难以完全判断。

## 6. 主要结论与发现
- OBCache将KV缓存剪枝问题重新定义为基于OBD理论的结构化剪枝，证明了以注意力输出扰动为度量的有效性。
- 闭式重要性分数综合了注意力权重、值状态和输出信息，优于仅依赖注意力权重的启发式分数。
- 在LLaMA和Qwen模型上，替换现有逐出方法中的启发式分数后，**长上下文准确率一致提升**。

## 7. 优点
- **理论驱动**：首次将最优脑损伤理论引入KV缓存剪枝，提供了数学严谨的重要性度量，而非经验启发式。
- **即插即用**：重要性分数可无缝替换现有方法中的启发式分数，无需修改模型结构或额外训练。
- **高效性**：闭式求解，计算开销低，适合在线推理。
- **考虑深层影响**：同时考虑键、值及联合效应，比仅基于注意力权重的评分更全面。

## 8. 不足与局限
- **实验细节缺失**：未列出具体数据集、基准指标、对比方法名称、剪枝率范围等，无法独立复现或评估泛化性。
- **未讨论效率对比**：虽然方法高效，但未比较OBCache计算重要性分数的额外开销与启发式方法的差异。
- **未考虑跨层差异**：逐层独立剪枝可能破坏层间协同，未分析不同层剪枝率分配的最优策略。
- **应用限制**：依赖于注意力输出梯度的近似，可能在高压缩率下误差累积；仅适用于Transformer解码器的KV缓存，不适用于其他剪枝场景。
- **文献支撑不足**：仅引用OBD理论，未与其他前沿结构化剪枝方法（如SparseGPT、Wanda等）进行对比，也未讨论与局部敏感哈希（LSH）等方法的关联。

（完）
