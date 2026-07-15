---
title: "SparseCache: Extreme Sparse Coding for KV Cache Compression"
title_zh: SparseCache：极端稀疏编码用于KV缓存压缩
authors: "Doohyun Cho, Myounghoon Cho, Donghoon Han, Sohyeon Kim, Ji-Hoon Kim, SeungJae Lee"
date: 2025-09-08
pdf: "https://openreview.net/pdf?id=43zTdoRqY4"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 使用K-SVD启发的字典学习进行KV缓存稀疏编码压缩
tldr: LLM推理中KV缓存内存增长成为瓶颈，预计算RAG范式因存储需求过大不实际。本文提出SparseCache，受K-SVD算法启发，通过端到端学习为Key和Value向量分别创建全局共享字典，以重建损失为目标优化，实现稀疏编码压缩。该方法能有效降低KV缓存存储需求，促进高效推理。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: KV缓存内存占用大，预计算RAG存储不可行。
method: 基于K-SVD的交替优化，学习Key和Value的全局共享字典进行稀疏编码。
result: 显著压缩KV缓存，保持推理质量。
conclusion: SparseCache提供了一种高效的KV缓存压缩框架。
---

## Abstract
The growing memory footprint of the Key-Value (KV) cache is a critical bottleneck in Large Language Models (LLMs), significantly hindering inference efficiency. While emerging "precomputed RAG" paradigms promise to reduce latency by precomputing KV caches for entire corpora, their prohibitive storage requirements render them impractical. This paper introduces SparseCache, a novel KV Cache compression framework that addresses this bottleneck. SparseCache employs an end-to-end learning framework, inspired by the K-SVD algorithm's alternating optimization, to create separate globally shared dictionaries for Key and Value vectors across the model. By optimizing these dictionaries directly against a reconstruction loss objective, SparseCache captures fundamental KV Cache redundancies more holistically than prior per-layer methods. Extensive experiments show that SparseCache achieves a state-of-the-art compression ratio of up to 17.7x while preserving model accuracy on challenging long-context benchmarks. Notably, it maintains high performance at over 8x compression, a level where competing techniques struggle. By enabling high-fidelity compression, SparseCache makes the "precomputed RAG" paradigm practical and feasible, leading to reduced Time-To-First-Token (TTFT) and improved overall system throughput.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：大型语言模型（LLM）推理过程中，Key-Value（KV）缓存的内存占用急剧增长，成为效率瓶颈。新兴的“预计算RAG”范式（Precomputed RAG）通过预计算整个语料库的KV缓存来降低延迟，但其存储需求过高，实际不可行。
- **研究动机**：迫切需要一种高压缩比、高保真度的KV缓存压缩方法，使得预计算RAG等依赖大规模KV缓存的技术变得实用，从而减少首Token生成时间（TTFT）并提升系统吞吐量。
- **整体含义**：通过极端稀疏编码实现KV缓存压缩，有望在保持模型精度的前提下显著降低存储开销，推动LLM高效推理的发展。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：受K-SVD算法的交替优化启发，SparseCache采用端到端学习框架，为模型中的Key向量和Value向量分别学习全局共享字典，通过稀疏编码进行压缩，直接优化重建损失。
- **关键技术细节**：
  - **全局共享字典**：不同于以往逐层或逐头的方法，SparseCache为所有层（或跨模型）的Key和Value向量共享同一字典，从而更全局地捕获KV缓存的冗余性。
  - **端到端学习**：将字典学习与稀疏编码嵌入训练流程，以最小化重建误差为目标函数，通过交替优化（类似K-SVD的字典更新和稀疏系数更新）进行训练。
  - **压缩流程**：将原始KV向量表示为稀疏线性组合，仅存储稀疏系数和字典索引，大幅减少存储空间。
  - **算法流程（文字说明）**：
    1. 初始化Key字典和Value字典（随机或基于预训练数据）。
    2. 固定字典，对每个KV向量通过稀疏追踪（如OMP算法）求解稀疏系数。
    3. 固定稀疏系数，利用SVD更新字典原子以最小化重建误差。
    4. 重复步骤2-3直至收敛，得到最终字典。
    5. 推理时，KV缓存存储稀疏系数而非原始向量；解码时通过字典重建。
- **公式**：文中未提供具体公式，但核心为最小化 \(\min_{D, \alpha} \|x - D\alpha\|_2^2 + \lambda \|\alpha\|_0\) 或 \(\|\alpha\|_1\) 形式的稀疏编码问题，其中 \(D\) 为字典，\(\alpha\) 为稀疏系数。

## 3. 实验设计：数据集、场景、Benchmark、对比方法

- **数据集/场景**：在长上下文基准上验证，具体数据集未在摘要中说明（可能包括LongBench、L-Eval等长文本任务）。
- **Benchmark**：挑战性长上下文基准（long-context benchmarks），用于评估压缩后模型精度。
- **对比方法**：未在摘要中明确列出，但提到在8倍压缩下SparseCache优于竞争技术（competting techniques），暗示与现有KV缓存压缩方法（如StreamingLLM、H₂O、Scissorhands等）进行了对比。
- **补充说明**：由于仅提供摘要，实验详细设计（如数据集名称、指标、具体对比方法）缺失。

## 4. 资源与算力

- 摘要和元数据中**未提及**任何GPU型号、数量、训练时长等算力信息。
- 推测可能使用标准LLM训练/微调平台（如A100 GPU），但无法确认。
- **需要指出**：论文未说明训练资源，无法评估计算成本。

## 5. 实验数量与充分性

- 仅提到“大量实验”（extensive experiments）在长上下文基准上达到最高17.7倍压缩比且保持精度。
- 未提及消融实验、不同压缩率对比、字典大小影响等具体实验数量。
- **充分性评估**：信息不足，无法判断实验是否覆盖关键维度（如不同模型大小、不同任务类型、鲁棒性测试）。仅凭“17.7x”单一结论可能不够充分，尤其缺乏与其他方法在多个压缩率下的系统对比。

## 6. 论文的主要结论与发现

- SparseCache实现了最高**17.7倍**的压缩比，同时保持模型在长上下文基准上的准确性。
- 在**8倍以上**压缩水平下仍能保持高性能，而竞争方法在此范围内性能显著下降。
- 该方法使得“预计算RAG”范式变得实用可行，能减少首Token生成时间（TTFT）并提高系统吞吐量。

## 7. 优点：方法或实验设计上的亮点

- **创新性**：将K-SVD引入KV缓存压缩，采用全局共享字典而非逐层压缩，更全面捕获冗余。
- **端到端训练**：直接优化重建损失，避免后处理插值。
- **高压缩比**：17.7x显著优于现有方法（如StreamingLLM等通常仅达2-4x）。
- **保真度高**：在极高压缩下仍保持精度，展示了稀疏编码在KV缓存中的潜力。

## 8. 不足与局限

- **实验细节缺失**：未提供具体数据集、模型大小（如LLaMA-7B/13B/70B）、评估指标（PPL、准确率等），难以复现。
- **计算开销**：字典学习需要训练成本，且稀疏解码可能增加推理延迟，但文中未分析。
- **泛化性未知**：仅测试长上下文基准，未验证在各类下游任务（如QA、摘要、代码生成）中的表现。
- **对比不充分**：未列出竞争方法及其配置，无法确认公平性。
- **资源信息缺失**：无法评估实用性和可扩展性。
- **理论分析不足**：未解释为何全局字典优于逐层字典，以及稀疏度与性能的权衡。

（完）
