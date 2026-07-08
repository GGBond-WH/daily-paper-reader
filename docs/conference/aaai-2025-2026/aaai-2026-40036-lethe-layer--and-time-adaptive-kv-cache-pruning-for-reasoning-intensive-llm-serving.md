---
title: "Lethe: Layer- and Time-Adaptive KV Cache Pruning for Reasoning-Intensive LLM Serving"
title_zh: "Lethe: 面向推理密集型LLM服务的层和时间自适应KV缓存剪枝"
authors: "Hui Zeng, Daming Zhao, Pengfei Yang, WenXuan Hou, Tianyang Zheng, Hui Li, Weiye Ji, Jidong Zhai"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40036/43997"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 面向推理任务的层和时间自适应KV缓存剪枝
tldr: LLM在推理任务中生成长序列，KV缓存内存和延迟开销大。现有压缩方法主要针对长输入预填充阶段，忽视长生成动态和层敏感性。本文提出Lethe，在空间维度根据层注意力冗余分配剪枝预算，在时间维度自适应调整剪枝策略。实验表明在保持推理质量的同时，KV缓存内存减少数倍，解码延迟显著降低。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 长生成推理中KV缓存动态累积，现有压缩方法未考虑层差异和时间变化。
method: 基于层注意力冗余度分配剪枝预算，并结合时间维度自适应调整。
result: 显著降低KV缓存内存和延迟，保持推理质量。
conclusion: Lethe为长文本推理提供了高效的KV缓存管理框架。
---

## Abstract
Generative reasoning with large language models (LLMs) often involves long decoding sequences, leading to substantial memory and latency overheads from accumulating key-value (KV) caches. While existing KV compression methods primarily focus on reducing prefill memory from long input sequences, they fall short in addressing the dynamic and layer-sensitive nature of long-form generation, which is central to reasoning tasks. We propose Lethe, a dynamic KV cache management framework that introduces adaptivity along both the spatial and temporal dimensions of decoding. Along the spatial dimension, Lethe performs layerwise sparsity-aware allocation, assigning token pruning budgets to each transformer layer based on estimated attention redundancy. Along the temporal dimension, Lethe conducts multi-round token pruning during generation, driven by a Recency-Aware Selective Retention (RASR) mechanism. RASR extends traditional recency-based heuristics by also considering token relevance derived from evolving attention patterns, enabling informed decisions about which tokens to retain or evict. Empirical results demonstrate that Lethe achieves a favorable balance between efficiency and generation quality across diverse models and tasks, increases throughput by up to 2.56×.

---

## 论文详细总结（自动生成）

# Lethe: 面向推理密集型LLM服务的层和时间自适应KV缓存剪枝 — 论文详细总结

## 1. 论文的核心问题与整体含义
- **研究背景**：大语言模型（LLM）在推理任务（如Chain-of-Thought）中产生长解码序列，KV缓存持续累积导致严重的内存和延迟瓶颈。
- **核心问题**：现有KV压缩方法主要针对预填充阶段的长输入，忽视了生成阶段中KV缓存的动态增长和层间注意力差异性。尤其在推理任务中，token重要性随时间变化且层间异构，静态压缩策略（如均匀预算或金字塔式预算）效果不佳。
- **整体含义**：需要一种同时考虑空间（层间）和时间（解码步骤）自适应性的KV缓存管理方案，以在保持推理质量的同时大幅降低内存和延迟。

## 2. 论文提出的方法论
### 2.1 核心思想
- 双维度自适应KV缓存剪枝：
  - **空间维度**：基于每层注意力稀疏度动态分配token保留预算。
  - **时间维度**：利用Recency-Aware Selective Retention (RASR)机制，在解码过程中多轮剪枝，结合注意力历史与最近活跃度决定token保留。
- 框架名为Lethe，在解码时监听缓存大小，超过阈值时触发剪枝。

### 2.2 关键技术细节
- **层注意力稀疏度估计**：使用Hoyer稀疏度度量（公式1）量化每层注意力分布的集中程度，值域[0,1]，越高表示注意力越集中（越稀疏）。
- **空间剪枝（算法1）**：
  1. 对每层，聚合所有头、批次、查询的注意力权重得到token重要性向量s(l)。
  2. 将token按重要性排序，划分为D个段。
  3. 寻找第一个段内最大注意力比值≤阈值τ的位置，保留该段及之前的所有token（称为显著token），加上前slen个sink token和最近r个token。
  4. 若无断点（即所有段内比值均>τ），则保守地延迟剪枝。
- **时间管理（RASR）**：
  - 维护每个token的累积分数s_t，通过指数衰减结合当前注意力更新（公式5）：s_t = γ·s_{t-1} + 当前层注意力和。
  - 定期根据s_t和token年龄排名，低于动态阈值的token被驱逐，保留最近和边界的token。

### 2.3 公式与算法流程说明
- **Hoyer稀疏度**：Sparsity(a) = (√n - ||a||₁/||a||₂) / (√n - 1)
- **重要性聚合**：s(l) = Σ_b Σ_h Σ_q A^{(l)}_{b,h,q,:}
- **RASR分数更新**：s_t = γ·s_{t-1} + Σ_h Σ_q Σ_j A^{(t)}_h(i,j)
- **算法流程**：预填充→解码循环→监视缓存大小→触发剪枝（每层独立执行算法1）→继续解码。

## 3. 实验设计
### 3.1 数据集与场景
- **Math500**：500道复杂数学题，测试符号推理。
- **MMLU子集**：8个科目（抽象代数、解剖学、天文学、商业伦理、临床知识、大学生物、大学化学、大学计算机科学），测试事实理解。
- **场景**：
  - 单批解码：生成长度从1.5k到20k token。
  - 多批解码：批量大小1~32，模拟并发服务。

### 3.2 基准模型
- **测试模型**：DeepSeek-R1-Distill系列：Qwen-7B、Qwen-32B、LLaMA-8B、LLaMA-70B。
- **对比方法**：
  - FullKV（无剪枝）
  - H2O（基于Top-K注意力保留）
  - StreamingLLM（固定大小滑动窗口）
  - PyramidKV（静态层间预算分配）
  - 排除SnapKV（设计用于预填充，不适合增量解码）

### 3.3 评估指标
- **精度**：准确率（Accuracy）
- **效率**：解码延迟、峰值GPU内存、token吞吐量（tokens/s）

## 4. 资源与算力
- **硬件**：NVIDIA A100 80GB GPU
  - 70B模型使用3路模型并行（tensor parallelism）
  - 其他模型单卡
- **训练/推理**：论文仅涉及推理优化，无额外训练，未报告训练时长或算力开销。剪枝过程本身计算开销很小（仅需聚合注意力分数与分段搜索）。

## 5. 实验数量与充分性
- **精度实验**：对4个模型×2个基准（Math500 + MMLU 8科），共12个模型-基准组合的完整结果（表1），对比4种方法+FullKV。统计充分。
- **效率实验**：
  - 延迟/内存/吞吐量趋势图（图4）：4个模型在不同生成长度下的曲线。
  - 批量内存与吞吐量表格（表2、表3）：4个模型×5个batch size（1,4,8,16,32），显示OOM情况。
- **消融实验**：
  - 稀疏率（sparse ratio）从100到800变化，分析对准确率和压缩率的影响。
  - 最近率（recent ratio）从0.1到0.5变化，分析保留最新token比例的影响。
- **公平性**：所有基线在统一框架下重新实现，确保一致。排除了不适用方法（SnapKV）并解释原因。
- **充分性评价**：实验覆盖了不同规模模型、不同任务类型、不同负载场景，消融实验分析了关键超参数，整体较为充分、客观。

## 6. 论文的主要结论与发现
- **精度保持**：Lethe在Math500和MMLU上基本匹配或接近FullKV准确率，部分科目甚至略超FullKV（如Qwen-32B的抽象代数、商业伦理），说明适度剪枝可去除噪声。
- **效率提升**：
  - 在LLaMA-70B上，batch=32时FullKV导致OOM，Lethe实现89.5 tok/s的吞吐量。
  - 内存减少高达91.7%（相比FullKV），延迟降低20-40%。
  - 吞吐量提升最高达2.56×（LLaMA-8B batch=16时316.4 vs 123.4 tok/s）。
- **空间与时间自适应有效**：对比PyramidKV（静态预算），Lethe避免了非单调注意力模式下的性能下降（如LLaMA-70B Math500：FullKV 88.7%，PyramidKV 80.8%，Lethe 88.1%）。
- **超参数影响**：稀疏率400左右取得最佳平衡；最近率0.3时精度与压缩最优。

## 7. 优点
- **双维度自适应**：同时考虑层间稀疏性和时间动态性，优于仅关注一维的方法（如H2O、StreamingLLM）。
- **非单调层模式支持**：揭示并利用了CoT推理中层注意力非单调变化，纠正了金字塔假设的局限。
- **RASR机制**：将传统LRU与注意力分数结合，兼顾时间衰退与语义重要性，避免简单丢弃近期大量有用token。
- **轻量计算**：剪枝过程仅需聚合已有注意力分数，无额外前向或重计算，开销很小。
- **广泛验证**：覆盖从7B到70B四个模型、多种batch和长度，结果稳健。

## 8. 不足与局限
- **任务覆盖有限**：仅评估CoT推理任务（Math500和MMLU），未涉及其他长文本场景（如RAG、文档摘要、多轮对话），可能限制泛化性。
- **未与量化方法联合测试**：虽然提到可互补（Lethe减少token数，量化减少每token位数），但未提供联合实验结果，缺乏复合压缩效果数据。
- **超参数敏感**：稀疏率、最近率、分段数D等需要针对模型和任务调优，自动化调整机制未探讨。
- **模型范围受限**：未在更大训练模型（如DeepSeek-V3 671B）上实验，因硬件限制使用蒸馏版本，可能遗漏原始模型上的行为差异。
- **注意力假设风险**：方法依赖注意力权重反映token重要性，但某些场景下注意力可能不精确（如过度关注特殊标记），可能导致错误剪枝。
- **基线对比**：未包含SnapKV、Keyformer等最新方法（虽解释原因），但读者可能希望看到对比。此外StreamingLLM表现较差，部分原因可能是窗口不适合长推理。

（完）
