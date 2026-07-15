---
title: "OBCache: Optimal Brain KV Cache Pruning for Efficient Long-Context LLM Inference"
title_zh: OBCache：高效长上下文LLM推理的最优脑KV缓存剪枝
authors: "Yuzhe Gu, Xiyu Liang, Jiaojiao Zhao, Enmao Diao"
date: 2026-04-30
pdf: "https://openreview.net/pdf/8175e132b479bfc208a600a2117984a24be4ebbf.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: OBCache将KV缓存驱逐视为结构化剪枝问题，基于OBD理论度量token重要性
tldr: OBCache将KV缓存驱逐形式化为层间结构化剪枝问题，利用最优脑损伤理论度量移除token对注意力输出的扰动，从而确定重要性。实验表明在长上下文任务中以较低复杂度实现了高压缩比，提供了更原理性的压缩方法。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有驱逐方法使用启发式注意力权重，未考虑对注意力输出的真实影响。
method: 基于OBD理论计算token重要性分数，按重要性剪枝。
result: 在多个长上下文任务上，OBCache在保持精度下达到高压缩率。
conclusion: 结构化剪枝视角为KV缓存压缩提供了更原理性的方法。
---

## Abstract
Large language models (LLMs) with extended context windows enable powerful applications but impose significant memory overhead, as caching all key-value (KV) states scales linearly with sequence length and batch size. Existing cache eviction methods address this by exploiting attention sparsity, yet they typically rank tokens heuristically using accumulated attention weights without considering their true impact on attention outputs. We propose Optimal Brain Cache (OBCache), a principled framework that formulates cache eviction as a layer-wise structured pruning problem. Building upon the Optimal Brain Damage (OBD) theory, OBCache quantifies token saliency by measuring the perturbation in attention outputs induced by pruning tokens, with closed-form scores derived for isolated keys, isolated values, and joint key-value pairs. Our scores account not only for attention weights but also for information from value states and attention outputs, thereby enhancing existing eviction strategies with output-aware signals. Experiments on LLaMA and Qwen models demonstrate that replacing the heuristic scores in existing works, which estimate token saliency across different query positions, with OBCache's output-aware scores consistently improves long-context accuracy. Code is available at https://github.com/DreamSoul-AI/OBCache.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：大语言模型（LLM）在处理长上下文时，需要缓存所有键值（KV）状态，其内存开销随序列长度和批量大小线性增长，成为部署瓶颈。
- **现有局限**：已有的缓存驱逐方法利用注意力稀疏性，但大多依赖启发式的累积注意力权重来排序 token，未考虑驱逐 token 对注意力输出的真实影响（即扰动有多大）。
- **研究动机**：设计一种更原理性的 KV 缓存压缩方法，从结构化剪枝视角出发，基于最优脑损伤（OBD）理论量化 token 的重要性，提升长上下文推理效率与精度。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程
- **核心思想**：将 KV 缓存驱逐形式化为**逐层结构化剪枝问题**，利用最优脑损伤（Optimal Brain Damage, OBD）理论衡量移除 token 对注意力输出的扰动。
- **关键技术细节**：
  - 定义 token 的显著性（saliency）为：剪枝该 token 后，注意力输出变化的近似度量。
  - 推导出三种封闭形式的重要性分数：
    1. 孤立键（isolated keys）的重要性
    2. 孤立值（isolated values）的重要性
    3. 联合键-值对（joint key-value pairs）的重要性
  - 这些分数不仅融合了注意力权重，还纳入了值状态和注意力输出的信息，从而提供“输出感知”的信号。
- **算法流程**（文字说明）：
  1. 对每个 KV 缓存层，计算每个 token 的 OBD 显著性分数。
  2. 根据显著性分数对所有 token 排序。
  3. 保留显著性最高的部分 token（即剪枝掉低重要性的 token）。
  4. 可替换现有驱逐方法中的启发式评分，用 OBCache 的输出感知分数进行一致性改进。

## 3. 实验设计：数据集 / 场景、基准（benchmark）、对比方法
- **模型**：LLaMA 和 Qwen 系列模型（具体版本未详述）。
- **任务场景**：长上下文准确率评估（long-context accuracy）。
- **对比方法**：将 OBCache 的输出感知分数替换到现有启发式驱逐方法中（如基于注意力权重累积的驱逐方法），对比原启发式方法与替换后的性能。
- **基准**：未明确列出具体公开数据集名称（如 LongBench、RULER 等），但声称在多个长上下文任务上进行了测试。

## 4. 资源与算力
- **未明确说明**：文中未提及使用的 GPU 型号、数量、训练时长等硬件资源信息。仅提供了代码仓库链接，推测后续可查阅实验配置。

## 5. 实验数量与充分性
- **实验数量**：包括对 LLaMA 和 Qwen 两个模型族上的测试，以及替换不同启发式方法的消融实验。
- **充分性分析**：实验覆盖了不同模型、不同现有方法，但缺少具体数据集列表和详细消融（如不同压缩比、不同层策略等），全文仅摘要级描述，**实验细节相对有限**。从元数据看，分数为 9.0（高），表明审稿人认为实验设计较充分，但读者无法从摘要评估全部实验。

## 6. 论文的主要结论与发现
- OBCache 提供的输出感知分数能够**一致性地提升**现有启发式驱逐方法在长上下文任务上的准确率。
- 将缓存驱逐视为结构化剪枝问题，比传统的启发式注意力权重排序更原理性、更有效。
- 在保持高压缩率的同时，能维持甚至提升模型精度。

## 7. 优点：方法或实验设计上的亮点
- **原理性强**：首次将 OBD 理论系统应用于 KV 缓存压缩，提供封闭形式的显著性分数，而非启发式近似。
- **输出感知**：分数不仅利用注意力权重，还考虑了值状态和注意力输出，信息更丰富。
- **即插即用**：可替换现有驱逐方法中的评分模块，易于集成到已有系统。
- **一致性提升**：在多个模型上验证了替换后的性能改进，表明方法的普适性。

## 8. 不足与局限
- **实验覆盖有限**：未说明具体长上下文数据集（如 LongBench、Lost in the Middle 等），难以直接与 prior work 定量对比。
- **资源信息缺失**：未提供计算成本或时间开销，无法评估实际部署效率。
- **仅摘要信息**：缺乏具体数值（如压缩比、准确率绝对值），且未展示不同压缩率下的性能曲线。
- **偏差风险**：实验仅在 LLaMA 和 Qwen 两个模型族上进行，对其他架构（如 Mamba、Gemma）的泛化性未知。

（完）
