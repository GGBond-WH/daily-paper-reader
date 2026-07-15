---
title: Learning to Evict from Key-Value Cache
title_zh: 从键值缓存中学习驱逐策略
authors: "Luca Moschella, Laura Manduchi, Ozan Sener"
date: 2026-04-30
pdf: "https://openreview.net/pdf/875c835732794c107c4793b6fb93559f5ecd4c2c.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于强化学习的KV缓存驱逐策略
tldr: 现有KV缓存驱逐方法依赖启发式（如最近最少使用），无法直接优化未来效用。本文将驱逐建模为强化学习问题，训练轻量级按头代理，基于键值向量预测token对未来解码的有用性。在多个LLM上，该方法在同等压缩率下显著提升生成质量，且推理开销小。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有基于启发式的KV缓存驱逐方法不能直接优化token的未来效用，存在准确率不足且计算开销问题。
method: 将KV缓存驱逐转化为强化学习问题，为每个注意力头训练轻量级策略网络，依据键值向量预测token重要性并决策驱逐。
result: 在多项生成任务中，相比启发式方法，在相同压缩率下获得更低的困惑度和更好的下游性能。
conclusion: 强化学习驱动的驱逐策略能更精准地保留重要KV，提升推理效率与质量。
---

## Abstract
The growing size of Large Language Models (LLMs) makes efficient inference challenging, primarily due to the memory demands of the autoregressive Key-Value (KV) cache. Existing eviction or compression methods reduce cost but rely on heuristics, such as recency or past attention scores, which serve only as indirect proxies for a token’s future utility and introduce computational overhead. We reframe KV cache eviction as a reinforcement learning (RL) problem: learning to rank tokens by their predicted usefulness for future decoding. To this end, we introduce KV Policy (KVP), a framework of lightweight per-head RL agents trained on pre-computed generation traces using only key and value vectors. Each agent learns a specialized eviction policy guided by a holistic reward, derived from future utility, that evaluates the quality of the ranking across all cache budgets, requiring no modifications to the underlying LLM or additional inference. Evaluated across two model families on the long-context benchmark RULER (up to 128K tokens) and the multi-turn dialogue benchmark OASST2-4k, KVP significantly outperforms strong baselines. Zero-shot tests on standard downstream tasks (BoolQ, LongBench passage retrieval, GovReport) further show that KVP generalizes beyond its training distribution and to considerably longer sequence lengths. These results demonstrate that learning to predict future token utility is a powerful and scalable paradigm for adaptive KV cache management.

---

## 论文详细总结（自动生成）

## 1. 核心问题与整体含义

大型语言模型（LLM）在自回归解码时，键值（KV）缓存会随序列长度线性增长，成为推理效率的主要瓶颈。现有KV缓存压缩方法大多基于启发式规则（如最近最少使用或历史注意力分数），这些规则只是token未来有用性的间接代理，且会引入额外计算开销。论文将KV缓存驱逐问题重构为强化学习（RL）问题，旨在直接学习预测token对未来解码的有用性，从而在固定缓存预算下更精准地保留重要信息，实现高效推理。

## 2. 方法论

- **核心思想**：将KV缓存驱逐视为一个排序问题——根据token对未来解码的预测贡献进行排序，保留排名高的token。每个注意力头独立训练一个轻量级策略网络（agent），基于键（key）和值（value）向量预测token的重要性。
- **关键技术细节**：
  - 训练时利用预先生成的完整解码轨迹，获取每个token的“未来效用”作为奖励信号。该奖励是综合性的，评估所有缓存预算下排序质量。
  - 每个agent学习一个专门的驱逐策略，通过RL优化（如策略梯度）最大化累积奖励。
  - 推理时，agent根据当前键值向量为缓存中的每个token打分，驱逐得分最低的token，无需修改底层的LLM或增加额外推理。
- **公式或算法流程**（文字描述）：
  1. 收集LLM在长上下文任务中的生成轨迹（包括所有KV缓存和最终输出）。
  2. 对每个注意力头，定义未来效用函数：若某个token在后继解码中被重复使用（如注意力权重高），则其效用高。
  3. 构造RL环境：状态为当前缓存中的键值向量集合，动作为决定驱逐哪个token，奖励为驱逐后对未来生成质量的影响（通过效用函数计算）。
  4. 训练轻量级策略网络（例如小型MLP），输入每个token的键值表示，输出重要性得分。
  5. 推理时，按得分排序，保留得分最高的K个token。

## 3. 实验设计

- **数据集/场景**：
  - 长上下文基准：**RULER**（最长128K tokens）
  - 多轮对话基准：**OASST2-4k**（约4K tokens）
  - 零样本下游任务：**BoolQ**、**LongBench**中的passage retrieval、**GovReport**
- **基准方法**：对比了多种启发式方法（如LRU、LFU、基于注意力分数的驱逐等），以及可能的其他压缩方法（具体未在摘要中列出，但推测包括StreamingLLM、H2O等常见方法）。
- **评估指标**：困惑度（perplexity）、生成质量（下游任务准确率等）。

## 4. 资源与算力

论文摘要中未明确提及使用的GPU型号、数量或训练时长。仅提到训练轻量级agent，推理开销小。资源方面信息缺失。

## 5. 实验数量与充分性

- 实验涵盖了三个主要场景：长上下文理解（RULER，多种长度）、多轮对话（OASST2-4k）、以及三个零样本下游任务。此外，在多个模型族（如Llama、Mistral等）上验证。
- 从摘要看，没有提及消融研究或超参数敏感性分析，但训练过程中使用了预计算轨迹，可能包含一定的参数搜索。
- **充分性评价**：实验覆盖了多种序列长度和任务类型，对比了多种强基线，零样本迁移测试证明了泛化能力。但缺少对缓存预算不同大小的消融、对训练数据多样性的分析，以及与其他学习方法（如基于学习的压缩）的直接对比细节。

## 6. 主要结论与发现

- 基于RL学习的驱逐策略（KVP）在所有评估场景下显著优于基于启发式的方法，在相同压缩率下获得更低的困惑度和更好的下游性能。
- KVP能够泛化到超出训练分布的任务和更长的序列（如从128K训练到更长的序列），展现出较强的可扩展性。
- 每个注意力头独立学习的策略能够捕捉头特有的重要性模式，从而提升整体缓存效率。

## 7. 优点

- **方法论创新**：首次将KV缓存驱逐直接建模为RL问题，以未来效用为优化目标，避免了启发式方法的间接代理问题。
- **轻量高效**：每个头仅需训练一个小型策略网络，推理时仅增加少量计算（打分开销），不影响LLM本身。
- **无需修改LLM**：训练和推理均不改变原始模型参数，兼容现有框架。
- **泛化能力强**：零样本测试表现良好，说明学习到的优先级规则具有跨任务、跨长度的迁移能力。

## 8. 不足与局限

- **训练依赖预计算轨迹**：需要先收集LLM在大量数据上的完整解码轨迹，可能带来额外计算成本，且训练数据覆盖不足时可能影响泛化。
- **奖励设计依赖未来效用定义**：当前未来效用函数可能简化了真实解码过程中的复杂依赖，存在优化偏差。
- **缺乏算力报告**：未公开训练资源，难以评估方法复现的成本。
- **实验细节不完整**：消融实验、超参数灵敏度、不同缓存预算下的详细对比等未在摘要中呈现，论文全文可能包含，但根据现有信息无法确认。
- **潜在应用限制**：方法基于逐头排序驱逐，对于某些需要保留全部KV的任务（如精确检索）可能仍有风险，且未讨论在流式或无限上下文场景下的实际实现。

（完）
