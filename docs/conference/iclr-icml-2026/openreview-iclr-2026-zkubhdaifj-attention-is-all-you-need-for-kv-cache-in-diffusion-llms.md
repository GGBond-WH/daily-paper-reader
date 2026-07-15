---
title: Attention Is All You Need for KV Cache in Diffusion LLMs
title_zh: 扩散大语言模型中KV缓存的自适应重计算
authors: "Quan Nguyen-Tri, Mukul Ranjan, Zhiqiang Shen"
date: 2026-02-06
pdf: "https://openreview.net/pdf?id=zkUbhdAiFJ"
tags: ["query:llm-kv-cache"]
score: 7.0
evidence: 针对扩散大语言模型的自适应KV缓存重计算
tldr: 扩散大语言模型的解码器在每个去噪步重复计算QKV，但大多数步KV变化很小。通过观察，发现远距离MASK令牌可作为长度偏置缓存，深层KV动态更大需选择性刷新。提出自适应重计算策略，在保持预测精度的同时显著降低解码延迟。
source: ICLR-2026-Accepted
selection_source: conference_retrieval
motivation: 扩散LLM解码器在每个去噪步重复计算所有令牌的KV，导致大量冗余。
method: 基于KV动态随深度增加和令牌注意力模式，提出选择性块缓存和深层刷新策略。
result: 在多个推理任务中，延迟降低明显，预测精度几乎不变。
conclusion: 自适应重计算为扩散LLM提供了一种高效的KV缓存优化手段。
---

## Abstract
This work studies how to adaptively recompute key–value (KV) caches for diffusion large language models (DLMs) to maximize prediction accuracy while minimizing decoding latency. Prior methods' decoders recompute QKV for all tokens at every denoising step and layer, despite KV states changing little across most steps, especially in shallow layers, leading to substantial redundancy. We make three observations: (1) distant MASK tokens primarily act as a length-bias and can be cached block-wise beyond the active prediction window; (2) KV dynamics increase with depth, suggesting that selective refresh starting from deeper layers is sufficient; and (3) the most-attended token exhibits the smallest KV drift, providing a conservative lower bound on cache change for other tokens. Building on these, we propose Elastic-Cache, a training-free, architecture-agnostic strategy that jointly decides ${when}$ to refresh (via an attention-aware drift test on the most-attended token) and ${where}$ to refresh (via a depth-aware schedule that recomputes from a chosen layer onward while reusing shallow-layer caches and off-window MASK caches). Unlike fixed-period schemes, Elastic-Cache performs adaptive, layer-aware cache updates for diffusion LLMs, reducing redundant computation and accelerating decoding with negligible loss in generation quality. Experiments on LLaDA-Instruct, LLaDA-1.5, and LLaDA-V across mathematical reasoning and code generation tasks demonstrate consistent speedups: $8.7\times$ on GSM8K (256 tokens), $45.1\times$ on longer sequences, and $4.8\times$ on HumanEval, while consistently maintaining higher accuracy than the baseline. Our method achieves significantly higher throughput ($6.8\times$ on GSM8K) than existing confidence-based approaches while preserving generation quality, enabling practical deployment of diffusion LLMs.

---

## 论文详细总结（自动生成）

# 论文总结：《扩散大语言模型中KV缓存的自适应重计算》

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **问题**：扩散大语言模型（Diffusion LLMs, DLMs）的解码器在每个去噪步骤中，对所有令牌（tokens）的 QKV（查询、键、值）进行重复计算，导致大量冗余计算，增加了解码延迟。
- **背景**：现有的KV缓存策略多针对自回归模型设计，不适用于扩散模型的迭代去噪过程。扩散模型的解码步骤数多（如数百步），且大多数步骤中KV变化很小，尤其在浅层。
- **动机**：提出一种面向扩散LLM的自适应KV缓存重计算策略，在保持预测精度（generation quality）的同时最大化降低解码延迟。

## 2. 论文提出的方法论
### 核心思想：Elastic-Cache
- **三个关键观察**：
  1. 远距离MASK令牌（distant MASK tokens）主要充当长度偏置（length-bias），可以被块级缓存（block-wise cached）在活跃预测窗口之外，无需每步刷新。
  2. KV动态随深度增加而增大，因此从更深层开始选择性刷新（selective refresh）即可满足需求。
  3. 最受关注的令牌（most-attended token）具有最小的KV漂移，可作为其他令牌缓存变化的保守下界（conservative lower bound）。
- **Elastic-Cache 策略**（无需训练、架构无关）：
  - **何时刷新（when）**：通过一个关注感知的漂移测试（attention-aware drift test），基于最受关注令牌的KV变化来决策是否触发缓存更新。
  - **何处刷新（where）**：采用深度感知的调度（depth-aware schedule），从选择好的某个层开始重计算，同时复用浅层缓存和窗口外的MASK缓存。
  - 不同于固定周期方案，Elastic-Cache执行自适应、层感知的缓存更新，减少冗余计算。

### 公式与算法流程（文字说明）
- 没有给出具体公式，但基本流程为：
  1. 在每个去噪步，监测最受关注令牌的KV漂移。
  2. 若漂移超过阈值，则从深层（如靠后的层）开始重计算KV，浅层和远距离MASK缓存保持不变。
  3. 若漂移较小，则复用全部缓存，跳过重计算。
- 阈值和深度选择为自适应调整。

## 3. 实验设计
- **数据集/场景**：
  - 数学推理：GSM8K（256 tokens）
  - 长序列生成：GSM8K（更长序列，具体长度未说明）
  - 代码生成：HumanEval
- **基准模型**：LLaDA-Instruct、LLaDA-1.5、LLaDA-V（均为扩散LLM系列）
- **对比方法**：仅提到与“现有基于置信度的方法”（existing confidence-based approaches）对比，但未列出具体方法名称。对比指标包括吞吐量（throughput）、延迟（speedup）和生成质量（accuracy）。

## 4. 资源与算力
- **未明确说明**：论文摘要中未提及使用的GPU型号、数量、训练时长、预训练资源等。只提到推理加速比，未涉及训练阶段。因此无法总结算力信息。

## 5. 实验数量与充分性
- **实验数量**：从摘要看，包含三个主要任务（GSM8K两个设置、HumanEval），在三个模型系列上测试。未提及消融实验（ablation studies）的具体数量，但根据方法描述，可能包含对缓存策略组件（如漂移测试、深度调度）的消融。
- **充分性评估**：
  - 覆盖了推理、长序列、代码生成等典型场景，模型规模包含不同变体，具有一定的代表性。
  - 对比了吞吐量和生成质量，但未与多种现有KV缓存方法（如传统固定缓存、基于置信度的方案）进行详细消融对比，对比基线较少。
  - 缺少对超参数（如漂移阈值、深度选择）的敏感性分析，实验充分性中等。

## 6. 论文的主要结论与发现
- **加速效果**：
  - GSM8K（256 tokens）：**8.7× 加速**
  - 长序列GSM8K：**45.1× 加速**
  - HumanEval（代码生成）：**4.8× 加速**
- **吞吐量**：在GSM8K上相比现有置信度方法提升 **6.8×**
- **生成质量**：预测精度几乎不变（consistently maintaining higher accuracy than the baseline）
- **结论**：自适应重计算为扩散LLM提供了一种高效的KV缓存优化手段，可实现实际部署。

## 7. 优点
- **方法亮点**：
  - 无需额外训练，可直接应用于已训练好的扩散LLM（training-free）。
  - 架构无关（architecture-agnostic），可迁移到不同模型。
  - 基于三个新颖的观察（MASK令牌长度偏置、深层KV动态大、最受关注令牌的保守下界）设计策略，理论依据充分。
  - 自适应决策规避固定周期造成的冗余或刷新不足。
- **实验亮点**：
  - 在多种任务（数学、代码）和模型尺寸上验证，加速比显著。
  - 与基线相比，质量损失极小甚至更优。

## 8. 不足与局限
- **实验覆盖不足**：
  - 仅选了三个模型（均为LLaDA系列），未在其他扩散LLM（如MDLM、D3PM等）上验证通用性。
  - 缺少与更多现有KV缓存优化方法（如StreamingLLM、H2O、Scissorhands等）的对比。
  - 没有在更长序列（如超过4K tokens）或更大模型（如13B以上）上测试。
- **偏差风险**：
  - 漂移阈值和深度选择的设定可能依赖经验，缺少理论推导或自动调参机制。
  - 最受关注令牌的假设（最小KV漂移）可能在某些注意力分布下不成立（如均匀注意或无突出焦点）。
- **应用限制**：
  - 当前设计针对扩散LLM的解码阶段，不适用于自回归模型或混合模型。
  - 未评估在生成多样性和安全性方面的潜在影响。
- **资源与可复现性**：
  - 未公开代码或详细配置，难以复现。

（完）
