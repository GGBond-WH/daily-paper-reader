---
title: "HitKV: Activation Frequency Knows Which Tokens Are Important"
title_zh: HitKV：激活频率揭示哪些令牌重要
authors: "Sanle Zhao, Yujuan Tan, Yu Jing, Zhuoxin Bai, Yue Niu, Jiayi Guo, Zongjie Wang, Ao Ren"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40105/44066"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于激活频率的KV缓存压缩方法
tldr: 针对大语言模型长上下文处理中KV缓存线性增长导致的显存瓶颈问题，本文提出HitKV方法。该方法在固定显存预算下，利用令牌在注意力中的激活频率（即达到top-k得分的次数）来识别重要令牌，克服了传统仅依赖单次注意力得分进行驱逐的局限。实验表明，HitKV在保持模型精度的同时显著降低了缓存占用，提升了长序列推理效率。该工作为KV缓存压缩提供了基于访问频率的新视角。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存压缩方法仅关注单步注意力得分，忽略了令牌被频繁激活的重要性，导致关键信息被误删。
method: 提出基于令牌激活频率的KV驱逐策略，动态保留高频令牌的KV对，并结合固定预算进行压缩。
result: 在多个长文本任务上，HitKV在相同预算下相比现有方法提升了困惑度或下游任务准确率。
conclusion: 激活频率是衡量令牌重要性的有效指标，可显著改进KV缓存压缩效果。
---

## Abstract
The demand for long-context processing in large language models (LLMs) continues to escalate alongside rapid advancements in their capabilities. However, the intermediate attention keys and values (KV cache) employed to avoid re-computations, also grow linearly with sequence length, far exceeding the memory capacity of consumer-grade GPUs. Consequently, many studies have proposed KV cache compression methods that evict unimportant tokens based on variant attention scoring strategies. These methods typically retain the KV pairs of the top-k scoring tokens under a fixed memory budget. However, they still face several limitations. First, they disregard the activation frequency of tokens, specifically the count of times tokens achieve top-k scores in the attention distribution of following tokens. The methods based on variant attention scores may incorrectly evict some high-activation-frequency yet low final-scoring tokens. Second, the activation frequency exhibits different distribution patterns across layers and tasks. Neglecting these differences negatively impacts model performance and task adaptability. Our analysis of the actual token activation frequency and its unique characteristics across layers and task types reveals potential opportunities to address these issues. In this paper, we propose HitKV, which employs hit rates to directly characterize token activation frequencies, enabling adaptive layer-aware and task-aware KV cache eviction under the uniform memory allocation strategies. Also, HitKV can be easily integrated into layer-specific memory allocation methods. Experimental results demonstrate that HitKV maintains model performance with preserving only 3% of the KV cache, achieves high-quality generation outputs in long-text generation tasks, and delivers 4× throughput improvement over baselines.

---

## 论文详细总结（自动生成）

# HitKV: Activation Frequency Knows Which Tokens Are Important 论文总结

## 1. 核心问题与整体含义（研究动机和背景）

大型语言模型（LLM）在长上下文处理中，为避免重复计算而存储的中间注意力键值对（KV cache）会随序列长度线性增长，远超消费级GPU显存容量。现有KV缓存压缩方法通常基于注意力得分（如累积得分、平均得分或最后token得分）保留top-k令牌，但忽略了令牌的**激活频率**——即该令牌在后续令牌的注意力分布中进入top-k的次数。这导致一些高激活频率但单次得分不高的令牌被错误驱逐，损害生成质量和长上下文理解能力。此外，激活频率在不同层和不同任务（长提示任务 vs. 长文本生成任务）中呈现显著差异，现有方法缺乏自适应性。因此，本文提出基于激活频率（命中率）的自适应KV缓存驱逐策略，以更准确地保留关键令牌。

## 2. 方法论：核心思想、关键技术细节

- **核心思想**：用 **hit rate（命中率）** 量化令牌的激活频率，作为重要性指标。命中率定义为：在最近窗口（recent window）中，该令牌在后续每个token的注意力分布中进入top-K（称一次“Hit”）的次数占窗口总token数的比例。
- **关键技术细节**：
  - **Hit定义**：对于给定令牌i，若其在最近窗口某令牌j的注意力得分分布中排名前K，则记为一次Hit。
  - **命中率计算**：\( \text{Thr}(t_i) = \frac{\sum_{j=0}^{|L_w|} \text{Hit}_{ji}}{|L_w|} \)，其中 \(L_w\) 为最近窗口令牌集合。
  - **驱逐策略**：保留命中率 ≥ θ 的KV对（θ=50%），驱逐命中率低的KV对。为补齐每头注意力头的均匀长度，额外添加具有最高最近累积注意力得分的KV对。
  - **参数配置**：通过实验发现，设置 θ=50% 且 hit coefficient K = 预算大小 (budget)，可使保留数量与预算高度一致，无需复杂调参。
  - **可扩展性**：HitKV可轻松集成到其他逐层内存分配策略（如PyramidKV），仅需替换其踢出指标为命中率，形成P-HitKV。

## 3. 实验设计

- **数据集/场景**：
  - **LongBench**：16个数据集，涵盖单/多文档QA、摘要、少样本学习、合成任务、代码补全，评估长上下文理解。
  - **Needle-in-a-Haystack**：评估关键信息抽取能力。
  - **PG19**：包含100本书，平均长度70K tokens，评估长文本生成质量（困惑度）。
- **基准方法**：对比 StreamingLLM、H2O、SnapKV、PyramidKV，以及集成HitKV后的P-HitKV。
- **模型**：Llama3-8B-Instruct、Mistral-v0.2-Instruct（支持4K-128K上下文）。因硬件限制仅测试7B-16B模型。
- **指标**：LongBench上各子任务准确率、Needle平均准确率、PG19上累积平均负对数似然（NLL/困惑度）、解码延迟。

## 4. 资源与算力

论文明确提到：所有实验在 **NVIDIA A6000 48GB GPU** 上运行。未说明GPU数量、训练时长或总耗能。实验仅涉及推理阶段（压缩KV缓存），未涉及训练。

## 5. 实验数量与充分性

- **实验组数量**：
  - LongBench：在两种模型、两种内存预算（128和1024 tokens）下对比，每个模型有16个数据集结果（表1），并展示了平均得分折线图（图6）。
  - Needle：两个模型、5种预算（64~1024 tokens），共10组结果（表2）。
  - PG19：仅展示Mistral-7B上NLL曲线（图7），未给出多个预算或模型。
  - 吞吐量：解码延迟对比（图8），仅展示一种预算（128 tokens）。
  - 参数分析：图4展示了不同K和θ下保留数量与预算对齐情况。
- **消融实验**：仅对比了基于累积得分、平均得分、最近token得分等驱逐指标的高频令牌丢失比例及其影响（图2），以及P-HitKV（集成HitKV到PyramidKV）的对比。
- **充分性与公平性**：
  - 充分性：覆盖了长上下文理解、信息检索、长文本生成、性能开销等多个维度，数据点较多。
  - 公平性：各方法在相同预算下对比，且使用相同模型和评估流程。但缺少与更多近期方法（如CAKE、D2O）的直接对比（仅提到可集成）；PG19实验仅一个预算、一个模型，不够全面。

## 6. 主要结论与发现

- 基于激活频率（命中率）的KV驱逐策略显著优于传统基于注意力得分的方法：在LongBench上平均准确率更高，在Needle中达到接近全缓存的检索性能，在PG19上获得更低困惑度。
- HitKV能自适应层和任务特性，即使统一内存预算，其保留的KV对数量在不同层自动变化，适应任务需求。
- 仅保留3%的KV缓存即可维持模型性能，解码延迟降低4倍（24k输入长度时）。
- 集成HitKV到逐层分配策略（P-HitKV）进一步提升性能，证明其可扩展性。

## 7. 优点

- **创新性**：首次系统性地将令牌激活频率作为KV缓存驱逐的核心指标，类比操作系统LFU策略，视角新颖。
- **实用性**：参数配置简单（θ=50%，K=budget），无需复杂调优；可轻松集成到现有逐层分配方法。
- **性能优势**：在相同预算下，多项指标全面超越现有方法，尤其在低预算下（64-128 tokens）提升显著。
- **实验设计合理**：覆盖多种任务类型和模型，对比公平，且提供了参数分析和可扩展性验证。

## 8. 不足与局限

- **实验覆盖有限**：
  - 仅测试7B-8B参数模型，未在更大模型（如13B、70B）或更多架构上验证。
  - PG19实验仅一个预算（128 tokens），未展示不同预算下的困惑度趋势。
  - 未与最新方法（如CAKE、D2O）直接比较，仅通过集成形式间接对比。
- **潜在偏差风险**：
  - 命中率计算依赖于最近窗口，可能对窗口大小敏感（论文未探讨窗口大小影响）。
  - 假设“高激活频率=重要性”，但可能存在极少数低频但关键的异常token，虽通过补充累积得分进行补救，但未充分分析其影响。
- **资源信息不透明**：未说明GPU数量、训练/推理总时间，缺少能耗或成本分析。
- **理论解释不足**：为何θ=50%和K=budget能有效对齐预算，论文仅称“经验规律”，缺乏理论证明。
- **应用限制**：仅适用于自回归解码场景，且需在预填充阶段确定命中率，可能不适合流式或无限上下文场景。

（完）
