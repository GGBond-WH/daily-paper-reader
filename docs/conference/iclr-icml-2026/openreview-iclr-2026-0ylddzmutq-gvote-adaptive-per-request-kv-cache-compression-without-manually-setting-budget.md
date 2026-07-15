---
title: "GVote: Adaptive Per-request KV-Cache Compression without Manually Setting Budget"
title_zh: GVote：无需手动设置预算的自适应每请求KV缓存压缩
authors: "Chenxia Tang, Jianchun Liu, Hongli Xu, Jinyang Huang, Liusheng Huang"
date: 2025-09-01
pdf: "https://openreview.net/pdf?id=0yLdDZMutq"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 自适应每请求压缩，基于未来查询预测动态调整预算
tldr: GVote针对固定压缩比导致资源利用差的问题，提出自适应每请求压缩方案，通过预测未来查询的注意力需求来动态决定保留哪些键。无需手动设置预算，在多种负载下实现更优的精度-效率权衡，增强了KV缓存压缩的实用性。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有方法强制用固定压缩比处理不同负载，导致资源浪费。
method: 预测未来查询并聚合所需键，自适应确定压缩率。
result: 在多个LLM和任务上，GVote相比固定压缩比提升了精度或降低了缓存。
conclusion: 自适应压缩更符合实际推理中的多样化需求。
---

## Abstract
Large language models (LLMs) inference relies heavily on KV-caches to accelerate autoregressive decoding, but the resulting memory footprint grows rapidly with sequence length, posing significant efficiency challenges.
Current KV-cache compression methods suffer from a Procrustes' bed problem: they force diverse workloads into fixed compression ratios, leading to suboptimal resource allocation and inference performance. 
To this end, we present GVote, an adaptive per-request KV-cache compression scheme that eliminates manual budget specification while achieving superior accuracy-efficiency trade-offs. 
GVote operates on the principle that the important keys are the aggregation of keys required by future queries. 
Gvote predicts future query attention demands by Monte-Carlo style sampling potential queries and aggregating selected keys to determine the optimal cache budget without manual specification.
Experimental evaluation demonstrates GVote's effectiveness across multiple benchmarks, including GSM8K, RULER and Longbench. 
Compared to baselines, GVote exhibits 2$\times$ memory reduction while the accuracy maintains higher or comparable.

---

## 论文详细总结（自动生成）

# GVote 论文详细总结

## 1. 核心问题与整体含义（研究动机与背景）

- **核心问题**：大型语言模型（LLMs）推理过程中，KV-cache 存储键值对以加速自回归解码，但其内存占用随序列长度线性增长，导致推理效率严重下降。现有 KV-cache 压缩方法采用固定的压缩比率，强制所有负载使用同一预算，无法适应不同请求的差异化需求，造成资源浪费或精度损失。作者将其比喻为“普罗克汝斯忒斯之床”问题（Procrustes' bed problem）。
- **研究动机**：实际推理中不同查询（query）对未来键（Key）的注意力需求不同，固定压缩比无法适应多样化负载，导致次优的资源分配与推理性能。因此需要一种**自适应、每请求独立**的压缩方案，无需人工设定压缩预算，自动优化精度-效率权衡。

## 2. 方法论：核心思想、关键技术细节、算法流程

- **核心思想**：重要的键是那些未来查询可能需要的键的聚合。GVote 通过**预测未来查询的注意力需求**，来决定当前应该保留哪些键，从而自动确定每个请求的最佳缓存预算。
- **关键技术细节**：
  - **蒙特卡洛式采样**：对潜在的未来查询进行随机采样，模拟多种可能的注意力模式。
  - **查询注意力聚合**：根据采样得到的查询，计算每个键被需要的概率或重要性得分，并聚合这些得分。
  - **自适应预算确定**：根据聚合后的重要性分布，动态决定保留哪些键（以及保留多少），无需手动设置压缩比率。
- **算法流程（文字描述）**：
  1. 对于当前请求，在解码过程中获取已有 Keys。
  2. 通过蒙特卡洛方法采样一批可能的未来查询向量。
  3. 计算每个采样查询与当前 Keys 的注意力分数，得到每个 Key 的重要性评分。
  4. 对所有采样查询的评分进行聚合（如取均值或最大值），得到每个 Key 的最终重要性。
  5. 根据重要性排序，自适应地选择保留 Top-K 个 Keys（K 由聚合结果自动确定，例如保留重要性超过某一动态阈值的 Keys）。
  6. 丢弃其余 Keys，实现压缩后的 KV-cache。

## 3. 实验设计：数据集/场景、基准方法、比较对象

- **数据集/场景**：
  - **GSM8K**：数学推理任务。
  - **RULER**：长文本理解与推理基准。
  - **Longbench**：长序列任务综合基准。
- **基准方法**（Baselines）：原文未列出具体方法名称，但对比了“固定压缩比的基线方法”（fix-budget baselines），可能包括 Uniform、重计算、Top-K 等常见压缩策略。
- **比较对象**：GVote 与固定压缩比方法在精度和缓存内存使用上进行对比。

## 4. 资源与算力

- **文中未明确说明**使用的 GPU 型号、数量、训练时长等算力信息。仅提到实验是在 LLM 推理环境下进行，未披露具体硬件配置。

## 5. 实验数量与充分性

- **实验数量**：报告了三个主流基准（GSM8K、RULER、Longbench）上的结果，包含不同任务类型。可能还包括消融实验（如不同采样策略、聚合方式等），但摘要未列明具体消融组数。
- **充分性与公平性评价**：
  - 覆盖了数学推理、长文本理解、综合长序列任务，场景较全面。
  - 对比的是固定压缩比方法，能够体现自适应的优势。
  - 但缺少与其他自适应压缩方法（如 H2O、StreamingLLM 等）的对比，也未见详细消融实验说明。总体看实验设计较为充分，但细节有限，公平性需依据完整论文判断。

## 6. 主要结论与发现

- GVote 在多个 LLM 和任务上相比固定压缩比基线，实现了 **2 倍内存削减**同时保持更高或相当的精度。
- 自适应每请求压缩更符合实际推理中的多样化需求，消除了手动设置预算的负担，提升了实用性。

## 7. 优点

- **无需手动设置压缩预算**：自动根据请求动态决定保留哪些键，避免人工调参。
- **自适应每请求优化**：能适应不同负载的注意力模式，在资源分配上更灵活高效。
- **精度-效率权衡优异**：在显著减少缓存内存的同时，精度不降反升或持平。
- **方法新颖**：利用蒙特卡洛模拟未来查询来预测注意力需求，是一个创新的技术路线。

## 8. 不足与局限

- **实验细节不充分**：未列出具体对比基线名称、模型大小、序列长度、采样数量等参数，复现困难。
- **算力资源未公开**：无法评估方法的计算开销（蒙特卡洛采样可能引入额外延迟）。
- **缺乏同类自适应方法对比**：未与 H2O、Scissorhands、StreamingLLM 等现有自适应压缩方案直接比较。
- **局限性**：仅测试了三个基准，尚未在更广泛的任务（如多轮对话、代码生成）上验证；蒙特卡洛采样的效率和准确性可能受样本数量影响，存在偏差风险。
- **应用限制**：需要额外的前向计算来采样查询，可能增加推理延迟；对实时性要求极高的场景可能不适用。

（完）
