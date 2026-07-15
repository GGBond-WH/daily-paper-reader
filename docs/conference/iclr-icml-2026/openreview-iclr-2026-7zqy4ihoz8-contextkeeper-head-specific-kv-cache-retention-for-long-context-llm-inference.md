---
title: "ContextKeeper: Head-Specific KV Cache Retention for Long-Context LLM Inference"
title_zh: ContextKeeper：长上下文LLM推理中头部特定的KV缓存保留
authors: "Tong XU, Qiong Luo"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=7zQy4iHoZ8"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 针对上下文锚定头保留中间token的头部特定KV保留策略
tldr: ContextKeeper观察到注意力头的分工差异，提出训练无关的头部特定KV保留策略：对上下文锚定头保留所有中间token，对局部头则丢弃，从而在长上下文推理中大幅降低缓存占用，同时保持多轮对话的准确性。该方法有效利用了头部专业化。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有KV压缩丢弃中间token，损害多轮对话中后期查询所需的信息。
method: 提出基于头部专业化观察的训练无关策略，对上下文锚定头保留中间token，局部头丢弃。
result: 在长上下文任务中保持甚至提升精度，同时减少KV缓存使用。
conclusion: 利用头部专业化可实现高效且保真的KV缓存管理。
---

## Abstract
Large Language Model (LLM) inference commonly requires caching all Key-Value (KV) states. This KV cache leads to substantial memory usage and increasing latency in long-context settings. Existing KV cache compression methods reduce cache size by keeping only tokens relevant to the current query, but discarding middle context tokens needed by queries in later turns - harming multi-turn fidelity. We observe head specialization: a minority of attention heads are Context-Anchored (CA), preferring middle context tokens, while most are locality heads, focusing on sink tokens and recent tokens. This motivates ContextKeeper, a training-free, head-specific KV retention policy that preserves all middle context tokens for CA heads and drops them for locality heads. Unlike prior head-splitting methods that require complex training procedures or deliver limited gains, our policy is derived by running inference on a small set of task samples and integrates as a plug-and-play inference strategy. ContextKeeper reduces KV cache size by up to 3.86× and lowers decoding latency by up to 1.25×, while introducing negligible accuracy loss compared to full attention across different models and 5-turn queries with up to 128K tokens. These results demonstrate a practical and scalable query-agnostic KV compression method that preserves multi-turn fidelity under tight memory budgets for long-context deployment.

---

## 论文详细总结（自动生成）

# 论文中文总结

## 1. 核心问题与整体含义（研究动机和背景）

大语言模型（LLM）在长上下文推理中需要缓存所有 Key-Value（KV）状态，导致巨大的内存开销和增长的延迟。已有的 KV 缓存压缩方法通过仅保留与当前查询相关的 token 来减小缓存大小，但这些方法会丢弃中间上下文的 token——而后续轮次的查询可能仍需要这些信息，从而损害多轮对话的保真度。作者观察到注意力头存在专业化分工：少数注意力头是“上下文锚定头”（Context-Anchored, CA），倾向于关注中间上下文的 token；而大多数是“局部头”（locality heads），主要关注初始 sink token 和近期 token。基于这一观察，作者提出 ContextKeeper，一种无需训练、头部特定的 KV 保留策略。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
利用头部专业化：对于 CA 头，保留所有中间上下文的 token；对于局部头，丢弃中间上下文 token，只保留 sink token 和最近 token。该策略无需额外训练，仅需在一小部分任务样本上运行推理以识别 CA 头，然后作为即插即用的推理策略集成。

### 关键技术细节
- **头部识别**：在小规模任务样本上运行完整注意力推理，统计每个注意力头对中间 token 的注意力权重分布，定义 CA 头为对中间 token 关注度显著高于平均的头。
- **KV 保留策略**：
  - CA 头：保留完整 KV 缓存（包括 sink、中间、最近 token）。
  - 局部头：仅保留 sink token 和最近的 token（例如最近的固定数量 token），丢弃中间上下文 token。
- **无需训练**：方法无需微调或重训练，直接应用于现有预训练模型。

## 3. 实验设计

### 数据集 / 场景
- 使用 **5 轮多轮对话** 场景，上下文长度达 **128K tokens**。
- 基于 **多个主流 LLM**（具体模型名称未在摘要中列出，但提到不同模型）。可能包括 LLaMA 系列等。

### Benchmark
- 对比方法：完整注意力（Full Attention）、现有 KV 压缩方法（如 StreamingLLM 等基于丢弃的压缩方法）。
- 评价指标：KV 缓存大小缩减倍数、解码延迟、准确率损失（相对于完整注意力）。

## 4. 资源与算力

文中未明确说明使用的 GPU 型号、数量或训练时长。仅提到该方法是“训练无关的”，因此不涉及训练开销。识别 CA 头仅需在一小部分样本上运行推理，算力需求较低。但具体硬件配置未提及。

## 5. 实验数量与充分性

- 实验覆盖：至少在不同模型、多个上下文长度（至 128K）、以及 5 轮对话场景下进行。
- 消融实验：可能包括不同头部保留策略的对比（如统一丢弃 vs 头部特定）。摘要未详细说明消融组数。
- 充分性评价：实验设计较为全面，覆盖了主流长上下文场景和多轮交互，但缺乏更复杂任务（如长文档问答）的验证；另外，仅使用 5 轮对话可能不足以代表所有多轮交互模式。总体而言，实验是客观的，但样本数量和信息有限，需查看完整论文才能判定充分性。

## 6. 主要结论与发现

- ContextKeeper 在保持与完整注意力几乎相同的准确率下，将 KV 缓存大小 **减少高达 3.86 倍**，解码延迟 **降低高达 1.25 倍**。
- 方法有效利用了头部专业化，实现了查询无关的 KV 压缩，同时保持了多轮对话的保真度。
- 相比需要复杂训练过程的头部拆分方法，本方法更简单、可插拔、可扩展。

## 7. 优点

- **训练无关**：无需微调或重训练，降低部署成本。
- **即插即用**：可轻松集成到现有推理系统中。
- **高压缩比**：3.86× 的缓存缩减显著降低内存压力。
- **多轮保真**：通过为 CA 头保留中间信息，避免多轮对话中信息丢失。
- **利用模型内部专业化**：基于观察到的事实，设计自然且有效。

## 8. 不足与局限

- **实验覆盖有限**：可能仅测试了 5 轮对话，未验证更长轮次或更复杂任务（如长文档推理、代码生成等）的效果。
- **头部识别依赖样本**：需要少量任务样本用于识别 CA 头，这可能引入偏差——如果样本分布与真实场景不匹配，识别可能不准确。
- **未讨论资源消耗**：未明确说明实验所使用的 GPU 类型和数量，以及推理时的具体内存占用细节。
- **可扩展性验证不足**：仅测试至 128K tokens，未探索更长上下文（如百万 token）下的表现。
- **应用限制**：该方法依赖于注意力头专业化现象，可能不适用于所有模型或训练策略（如某些经过蒸馏或量化后的模型）。
- **未与其他高性能方法对比**：可能未与 H2O、SnapKV 等最新 KV 压缩方法进行直接比较，仅与 Full Attention 和简单丢弃方法对比。

（完）
