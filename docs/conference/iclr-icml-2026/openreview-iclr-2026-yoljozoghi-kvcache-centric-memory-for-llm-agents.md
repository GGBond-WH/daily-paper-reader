---
title: KVCache-Centric Memory for LLM Agents
title_zh: 面向LLM智能体的KV缓存中心内存
authors: "Yuan Zeng, Pengfei Zuo, Min Lyu, Xingkun Yang, Huatao Wu, Yinlong Xu, Zhou Yu"
date: 2025-09-18
pdf: "https://openreview.net/pdf?id=YolJOZOGhI"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于KV缓存的LLM智能体内存系统
tldr: LLM智能体在长时任务中受限于上下文窗口，现有纯文本记忆系统检索不稳定且破坏前缀缓存。本文提出MemArt，直接将对话轮次存储为可复用的KV缓存块，通过潜在空间注意力分数检索，并设计多token聚合检索与解耦位置编码，实现准确高效的内存管理。实验表明其检索性能显著优于基线，同时保持推理效率。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有LLM智能体记忆系统基于纯文本，检索准确性低且破坏前缀缓存效率，亟需原生KV缓存格式的记忆范式。
method: 提出MemArt，以KV缓存块存储记忆，通过注意力分数进行潜在空间检索，采用多token聚合压缩键和解耦位置编码确保可靠复用。
result: 在长时任务中检索准确性显著提升，同时维持前缀缓存效率，降低内存开销。
conclusion: KV缓存原生记忆范式能有效提升LLM智能体长期交互的性能与效率。
---

## Abstract
LLM agents in complex, long-horizon workflows are constrained by the model’s context window. Current plaintext-based memory systems suffer from unstable retrieval accuracy and disrupt prefix caching, harming both performance and efficiency. 
We propose MemArt, a novel memory paradigm that operates directly within the LLM-native format: the key-value (KV) cache. Instead of using plaintext, MemArt stores conversational turns as reusable KV cache blocks and retrieves relevant memories by computing attention scores in latent space. To enable accurate and efficient retrieval, we develop a multi-token aggregation retrieval strategy that uses compressed keys for efficient KV selection and a decoupled position encoding mechanism to ensure retrieved blocks are safely and coherently reused. On the LoCoMo benchmark, MemArt improves accuracy by over 11\% (up to 39.4\%) compared to state-of-the-art plaintext-based memory methods, nearly matching full-context performance. Critically, it achieves this while reducing prefill tokens by over two orders of magnitude (91-135$\times$), representing a significant leap forward for building powerful and efficient long-context agents.

---

## 论文详细总结（自动生成）

# 详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：现有 LLM 智能体在复杂、长时任务中受限于模型的上下文窗口。传统基于纯文本的记忆系统存在两个主要缺陷：
  - **检索不稳定**：纯文本检索（如向量相似度）准确性低，容易丢失关键信息。
  - **破坏前缀缓存**：将历史对话转为文本后再重新编码会破坏原本的 KV 前缀缓存，导致推理效率下降、计算开销增大。
- **整体含义**：为提升 LLM 智能体在长期交互中的性能与效率，需要一种直接操作 LLM 原生日志（KV 缓存）的记忆范式，而非依赖文本中间表示。

## 2. 论文提出的方法论

### 核心思想
- 提出 **MemArt**，一种以 KV 缓存为中心的内存系统。它将对话轮次直接存储为可复用的 KV 缓存块，而非纯文本。
- 检索时，通过计算查询轮次与存储块之间的**注意力分数**（在潜在空间中进行）来定位相关记忆，避免了文本编码的开销和前缀缓存的破坏。

### 关键技术细节
- **基于 KV 缓存块的存储**：每个对话轮次被处理成一组 key 和 value，按块组织便于复用。
- **多 token 聚合检索策略（Multi-token Aggregation Retrieval）**：
  - 压缩多个 token 的 key 为一个紧凑表示，降低检索时的计算量。
  - 在潜在空间内计算聚合 key 与查询块之间的注意力分数，实现高效 KV 选择。
- **解耦位置编码机制（Decoupled Position Encoding）**：
  - 当复用检索到的 KV 缓存块时，需要将其“插入”当前上下文而不破坏完整的位置信息。
  - 通过将块内位置编码与全局位置编码解耦，使复用后的块依然能与新上下文保持连贯性。

### 算法/流程说明（文字描述）
1. **写入阶段**：智能体每完成一轮对话，将该轮次的 KV 缓存（经过位置编码解耦处理）存入记忆池，并附带压缩后的聚合 key。
2. **检索阶段**：当前输入轮次生成查询后，计算其聚合 key 与记忆池中所有块的聚合 key 的注意力分数，选择得分最高的 top-k 块。
3. **复用阶段**：将选中的 KV 缓存块插入到当前生成的 Transformer 中，利用解耦位置编码保持全局位置一致性，实现类似长上下文的记忆接入。

## 3. 实验设计

- **数据集/场景**：采用 **LoCoMo** benchmark（长对话记忆评估），模拟长期交互的复杂任务。
- **对比方法**：与 SOTA 纯文本记忆方法（如基于向量检索的记忆系统）进行比较。
- **评估指标**：
  - **检索准确率**：MemArt 在准确率上提升超过 11%（最高达 39.4%），几乎匹配全上下文（即无记忆限制）的性能。
  - **效率指标**：prefill tokens（预填充的 tokens 数量）减少 91–135 倍（即两个数量级），显著降低计算开销。

## 4. 资源与算力

- **未明确说明**：论文的元数据和摘要中未提及使用 GPU 型号、数量、训练时长等具体硬件信息。通常这种类型的工作依赖标准 LLM 推理框架（如 Transformers），但具体配置未公开。

## 5. 实验数量与充分性

- **实验数量**：根据摘要和元数据，主要公布了一个基准测试（LoCoMo）上的主实验结果，并提到了与全上下文基线的对比。未列出消融实验（如多 token 聚合、解耦位置编码的单独影响）或跨多种不同任务/数据集的扩展实验。
- **充分性判断**：实验覆盖范围较窄，仅在一个 benchmark 上验证。虽然结果显著，但缺乏对不同模型规模、不同任务类型（如多轮对话、交互式推理）的测试，也没有与更多基线对比（如基于长上下文微调的模型）。因此**实验充分性中等**，结果客观但覆盖有限。

## 6. 论文的主要结论与发现

- MemArt 能够以 LLM 原生的 KV 缓存格式实现高效、准确的内存管理。
- 在长时任务中，检索准确率接近全上下文（无记忆限制）的水平，同时 prefill tokens 减少两个数量级，极大提升了推理效率。
- 首次证明“KV 缓存原生记忆”范式在 LLM 智能体中的可行性，优于传统纯文本记忆系统。

## 7. 优点

- **方法创新**：直接操作 KV 缓存，避免了文本与 Token 之间的反复转换，从根本上解决了前缀缓存被破坏的问题。
- **检索高效**：通过多 token 聚合压缩 key 和潜在空间注意力，使检索开销极低，同时保持高准确性。
- **性能卓越**：在降低计算量的同时获得与全上下文相当的准确性，实用价值高。
- **结果清晰**：指标改善幅度大（准确率提升 11%–39%，prefill 减少 91–135×），对比直观。

## 8. 不足与局限

- **实验覆盖不足**：仅在一个 benchmark（LoCoMo）上验证，缺少跨领域/跨模型泛化测试，也未进行充分的消融实验以证实每个设计组件的贡献。
- **应用限制**：
  - 要求 LLM 推理框架能够暴露并操作 KV 缓存，存在工程实现复杂度。
  - 解耦位置编码可能与某些位置编码方案（如 RoPE）不兼容或需定制。
- **偏差风险**：对比的基线可能未包括最新的长上下文模型或检索增强方法，对比可能不够全面。
- **资源、可重复性**：未公开代码或详细超参数，基线实现细节不清，增加了复现难度。

（完）
