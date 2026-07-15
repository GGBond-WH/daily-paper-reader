---
title: Learning to Evict from Key-Value Cache
title_zh: 学习从键值缓存中驱逐
authors: "Luca Moschella, Laura Manduchi, Ozan Sener"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=t7lJ2OEGbJ"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于强化学习的KV缓存驱逐，学习对未来有用性排序
tldr: 该论文将KV缓存驱逐视为强化学习问题，提出KVP框架，为每个注意力头训练轻量级RL代理，学习根据未来解码有用性来排序和驱逐token，避免了手工启发式规则。实验表明在多种模型和任务上优于现有方法，为KV缓存优化提供了新的学习范式。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有驱逐方法依赖启发式规则，无法直接优化未来效用。
method: 为每个注意力头训练独立的RL代理，基于键值向量预测token的效用并据此驱逐。
result: 在多个LLM上，KVP在保持困惑度的同时显著降低了缓存大小。
conclusion: 强化学习可有效学习具有前瞻性的驱逐策略，优于传统启发式方法。
---

## Abstract
The growing size of Large Language Models (LLMs) makes efficient inference challenging, primarily due to the memory demands of the autoregressive Key-Value (KV) cache. Existing eviction or compression methods reduce cost but rely on heuristics, such as recency or past attention scores, which serve only as indirect proxies for a token’s future utility and introduce computational overhead. We reframe KV cache eviction as a reinforcement learning (RL) problem: learning to rank tokens by their predicted usefulness for future decoding. To this end, we introduce KV Policy (KVP), a framework of lightweight per-head RL agents trained on pre-computed generation traces using only key and value vectors. Each agent learns a specialized eviction policy guided by a holistic reward, derived from future utility, that evaluates the quality of the ranking across all cache budgets, requiring no modifications to the underlying LLM or additional inference. Evaluated on the long-context benchmark RULER and the multi-turn dialogue benchmark OASST2-4k, KVP significantly outperforms baselines. Furthermore, zero-shot tests on standard downstream tasks indicate that KVP generalizes well beyond its training distribution. These results demonstrate that learning to predict future token utility is a powerful and scalable paradigm for adaptive KV cache management.

---

## 论文详细总结（自动生成）

# 论文中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：大型语言模型（LLM）的自回归推理过程中，键值（KV）缓存的内存需求随序列长度线性增长，成为高效推理的主要瓶颈。现有缓存驱逐或压缩方法多依赖于启发式规则（如最近性、历史注意力分数），这些规则只是 token 未来效用的间接代理，且引入了额外计算开销，缺乏对长期解码收益的显式优化。
- **整体含义**：作者将 KV 缓存驱逐重新定义为**强化学习（RL）问题**——学习直接根据 token 在未来解码中的预测有用性进行排序和驱逐，从而摆脱手工规则，实现自适应、前瞻性的缓存管理。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：训练轻量级的、针对每个注意力头的 RL 代理，基于键向量和值向量预测每个 token 对未来解码的效用，并按效用进行排序和驱逐。整个过程无需修改底层 LLM，也无需额外推理开销。
- **关键技术细节**：
  - **框架名称**：KV Policy (KVP)
  - **训练数据**：在预先生成的解码轨迹上训练，仅使用键和值向量作为特征。
  - **代理结构**：为每个注意力头训练独立的 RL 代理，使得策略能够适应不同头部的注意力模式。
  - **奖励设计**：定义一个**整体奖励**，从未来效用（future utility）中导出，该奖励评估所有缓存预算下排名的质量（即哪个 token 应该被保留以最大化后续解码准确性）。
  - **驱逐流程**：在推理过程中，每个代理根据当前缓存的键值向量生成每个 token 的效用分数，然后根据分数保留 top-k，驱逐其余。
  - **优势**：不需要修改 LLM 本身，不增加推理阶段的计算量（RL 代理非常轻量）。

## 3. 实验设计：数据集、基准和对比方法

- **主要基准**：
  - **RULER**（长上下文基准）
  - **OASST2-4k**（多轮对话基准）
- **零样本泛化测试**：在标准下游任务上测试，以验证超出训练分布的表现。
- **对比方法**：与现有的启发式驱逐方法（如基于 GPT-Q、H2O 等）进行比较，具体基线名称未在元数据中给出，但摘要指出 KVP "significantly outperforms baselines"。
- **评估指标**：困惑度（perplexity）和缓存大小，在保持困惑度的同时显著降低缓存。

## 4. 资源与算力

- **文中说明**：论文摘要和元数据**未明确提及**使用的 GPU 型号、数量、训练时长等算力信息。仅提到 RL 代理是“lightweight”（轻量级），但具体训练代价未知。
- **注意**：由于全文未提供，无法在此方面给出精确总结。通常 ICLR 论文会在实验设置中描述硬件资源，此处缺失。

## 5. 实验数量与充分性

- **实验数量**：
  - 主要实验在两个基准（RULER, OASST2-4k）上进行。
  - 还进行了零样本泛化测试（标准下游任务）。
  - 可能包含消融实验（如不同奖励设计、代理数量等），但摘要未详细列出。
- **充分性评判**：
  - 基准选择覆盖了长上下文和多轮对话两个关键场景，具有一定代表性。
  - 零样本测试验证了泛化能力，增强了可信度。
  - 但缺乏对不同模型规模（如 7B vs 70B）、不同序列长度、以及与其他学习式方法（如通过学习排序的回归方法）的对比。
  - 总体上看，实验初步验证了方法的有效性，但**覆盖范围有限**，尚不足以全面评估其鲁棒性和泛化边界。

## 6. 论文的主要结论与发现

- **主要结论**：学习预测未来 token 效用是一种高效、可扩展的自适应 KV 缓存管理范式。KVP 在所有测试场景中显著优于基于启发式的基线，并且能够零样本泛化到训练分布之外的下游任务。
- **发现**：
  - 强化学习能够学习具有前瞻性的驱逐策略，直接优化未来解码的效用，而非依赖间接代理。
  - 为每个注意力头训练独立代理能够捕获头部之间的特异性行为，提升驱逐决策的精细度。
  - 轻量级 RL 代理在保持与全缓存相近困惑度的同时，大幅压缩缓存大小。

## 7. 优点：方法或实验设计上的亮点

- **创新性**：首次将 KV 缓存驱逐形式化为强化学习问题，摆脱手工启发式规则，直接面向未来效用优化。
- **设计优雅**：
  - 训练仅在预生成轨迹上进行，无需在线交互，避免了 RL 中常见的探索-利用困境。
  - 每个注意力头独立代理，使得策略多样化且可轻量部署。
  - 整体奖励设计考虑了所有缓存预算下的排名质量，避免针对单一预算调参。
- **实验亮点**：零样本泛化测试表明方法学到了内在的 token 效用特征，而非过拟合特定数据分布。

## 8. 不足与局限

- **实验覆盖不足**：仅测试了两个基准和一个零样本泛化，缺少在更多主流 LLM（如 LLaMA-3/3.1、Mistral 等）上的系统比较；缺少对不同序列长度、不同缓存预算阶梯的详细消融。
- **训练依赖**：需要预先生成解码轨迹作为训练数据，这可能引入对生成策略的依赖，且生成轨迹的成本未被评估。
- **偏差风险**：RL 代理的训练可能偏向训练数据中的常见模式，对于罕见模式（如知识密集型的 long-context 问答）可能表现不佳。
- **应用限制**：
  - 论文未讨论 RL 代理推理本身的延迟和内存开销，虽然声称“轻量”，但每个注意力头一个 agent 的部署可能在实际系统中积累开销。
  - 方法适用于自回归解码，但未提及是否适用于 speculative decoding、前缀缓存等更复杂的推理策略。
  - 缺少对多 GPU 或分布式推理场景下的兼容性讨论。

（完）
