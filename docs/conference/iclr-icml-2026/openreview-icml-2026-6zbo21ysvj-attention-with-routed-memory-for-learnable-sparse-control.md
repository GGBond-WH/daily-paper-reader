---
title: Attention with Routed-Memory for Learnable Sparse Control
title_zh: 注意力路由记忆：可学习稀疏控制
authors: "QIUHAO Zeng, Jerry Huang, Peng Lu, Ruiyi Fang, Gezheng Xu, Zihao Jing, Yufei Cui, Charles Ling, Gang Niu, Boyu Wang"
date: 2026-04-30
pdf: "https://openreview.net/pdf/053e4edad024c40ed9ffb07868ebc3784d86f137.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 提出可微内存系统的新型KV缓存结构
tldr: 大语言模型长上下文推理受限于KV缓存机制，现有逐出策略会丢弃重要信息。本文提出注意力路由记忆（ARM），一种全新KV缓存结构，采用层次化路由器和可微固定大小内存系统。通过Gumbel-Softmax学习选择内存槽，并用sigmoid门控更新，软融合新旧信息，避免硬逐出。实验证明ARM能有效减少信息丢失，提升缓存效率。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存逐出策略丢弃核心信息，导致长上下文推理性能下降。
method: 设计层次化路由器和固定大小内存，通过Gumbel-Softmax和sigmoid门控实现软更新。
result: 在长上下文任务中减少信息丢失，提升缓存效率。
conclusion: ARM提供了一种可学习的KV缓存管理方法，避免硬逐出。
---

## Abstract
Despite advances in long-context inference, large language models (LLMs) remain fundamentally limited by the key-value (KV) caching mechanisms that are necessary for stable computation. Techniques such as selective token eviction and pruning have vastly mitigated these issues, but often discard core information to manage the growing cache.
In this paper, we propose _**A**ttention with **R**outed **M**emory_ (_**ARM**_) a novel KV caching structure that introduces a fully differentiable, fixed-size memory system organized as a hierarchical router. Via a Gumbel-Softmax, _**ARM**_ learns to select memory slots and perform sigmoid-gated updates that softly combine new and stored information, avoiding hard eviction and reducing information loss. By further training a policy to dynamically select varying amounts of memory at inference, _**ARM**_ adapts its accesses for both simple contexts and inputs that require deeper reasoning, enabling more scalable and effective retrieval on both short- and long-contexts. Experimental results on standard commonsense and long-context reasoning benchmarks demonstrate that _**ARM**_ achieves superior performance and efficiency compared to fixed KV-caching approaches, while remaining efficient and scalable in terms of both memory and generation latency.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：大语言模型（LLM）在执行长上下文推理时，受限于键值（KV）缓存机制。现有的选择性令牌逐出（eviction）和剪枝（pruning）等方法虽然缓解了缓存增长问题，但往往丢弃了关键信息，导致长上下文任务性能下降。
- **整体含义**：本文旨在设计一种全新的、可学习的KV缓存管理结构，避免硬逐出（hard eviction）带来的信息丢失，同时保持高效的内存利用和生成延迟，从而提升LLM在短上下文和长上下文任务上的可扩展性和检索效果。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程

- **核心思想**：提出**注意力路由记忆（Attention with Routed-Memory, ARM）**，一种完全可微的固定大小内存系统，通过层次化路由器（hierarchical router）和软更新机制替代传统的硬逐出策略。
- **关键技术细节**：
  - 使用**Gumbel-Softmax**技术学习选择哪些内存槽（memory slots）用于存储新信息，实现离散选择的可微化。
  - 采用**sigmoid门控更新**，将新信息与已有存储信息进行软融合（soft combination），而非直接覆盖或丢弃，从而减少信息丢失。
  - 训练一个**策略网络**（policy），在推理阶段动态调整访问的内存数量，以适应简单上下文和需要深度推理的输入，实现自适应稀疏控制。
- **算法流程（文字说明）**：
  1. 输入序列的每层注意力计算中，除了标准KV缓存，ARM维护一个固定大小的可微分内存。
  2. 当新token到来时，通过层次化路由器计算每个内存槽的选择概率，使用Gumbel-Softmax进行采样，决定哪些槽被更新。
  3. 被选中的槽通过sigmoid门控将新key-value信息与旧值融合，产生更新后的内存表示。
  4. 在推理阶段，策略网络根据当前上下文复杂度决定使用的内存槽数量，实现计算与内存的权衡。

## 3. 实验设计：使用了哪些数据集 / 场景，它的 benchmark 是什么，对比了哪些方法

- **数据集与场景**：论文在**标准常识推理（commonsense reasoning）** 和**长上下文推理（long-context reasoning）** 基准上进行了评估。具体数据集名称未在摘要中列出，但据元数据推测可能包括常见LLM评估集（如LongBench、Needle in a Haystack等）。
- **对比方法**：与**固定KV缓存方法**（如固定大小缓存、滑动窗口等）进行对比，同时可能对比了其他选择性逐出或稀疏注意力方法。具体对比方法列表未在摘要中详述。
- **评价指标**：涉及性能（如准确率或推理质量）和效率（内存占用、生成延迟）。

## 4. 资源与算力：如果文中有提到，请总结使用了多少算力（GPU 型号、数量、训练时长等）。若未明确说明，也请指出这一点。

- **文中未明确说明**：摘要和元数据中未提及具体的GPU型号、数量或训练时长。因此无法总结算力细节。论文正文可能包含该信息，但此处无法获取。

## 5. 实验数量与充分性：大概做了多少组实验（如不同数据集、消融实验等），这些实验是否充分、是否客观、公平。

- **实验数量**：由于仅提供摘要，未详细列出实验组数。但根据“标准常识推理和长上下文推理基准”的描述，可能包含多个数据集上的性能对比，以及消融实验（例如验证门控机制、策略网络的有效性）。具体数量未知。
- **充分性评判**：从摘要看，作者声称ARM在性能和效率上优于固定KV缓存方法，且具有可扩展性。但缺乏详细数据集和对比方法列表，无法判断实验是否覆盖足够多样的场景（如不同模型规模、不同长度分布）。元数据中得分为9.0，表明审稿人认为实验设计较充分，但仍需更多细节才能客观评价。

## 6. 论文的主要结论与发现

- ARM通过完全可微的固定大小内存和层次化路由，有效减少了KV缓存中的信息丢失，避免了硬逐出的缺陷。
- 动态策略调整使得ARM在短上下文和长上下文中均能高效检索，兼顾性能与效率。
- 实验结果表明，ARM在常识推理和长上下文推理基准上均取得了优于固定KV缓存方法的性能，同时保持了内存和延迟的效率与可扩展性。

## 7. 优点：方法或实验设计上有哪些亮点

- **方法创新性**：提出完全可微的KV缓存结构，通过Gumbel-Softmax和sigmoid门控实现软更新，与现有硬逐出方法有本质区别。
- **自适应能力**：训练推理时动态选择内存槽数量，使模型能根据输入复杂度调整计算资源，兼具简单任务的高效和复杂任务的深度推理能力。
- **可扩展性**：固定大小的内存结构避免了缓存无限增长，同时可微性允许端到端学习，易于集成到现有LLM架构中。

## 8. 不足与局限：包括实验覆盖、偏差风险、应用限制等

- **实验覆盖不足**：摘要未提及具体数据集和对比方法，可能未覆盖真实长上下文应用（如多文档问答、代码生成）的多样性；也未说明是否在不同模型规模（如7B、70B）上验证。
- **偏差风险**：由于对比方法仅为“固定KV缓存方法”，可能未与最新的稀疏注意力（如StreamingLLM、H2O）或检索增强方法进行公平比较。
- **应用限制**：可微分内存需要额外训练，且可能增加推理时的路由计算开销；对于已经部署的大型模型，重新训练成本较高。
- **未说明的理论保证**：软更新虽然减少了信息丢失，但可能引入噪声；策略网络的稳定性未充分讨论。

（完）
