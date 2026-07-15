---
title: "On 10X Better Scalability: KV Stores Scale Up KV Cache"
title_zh: 论10倍可扩展性：KV存储扩展KV缓存
authors: "Weiping Yu, Ye Jiarui, He Mengke, Junfeng Liu, Siqiang Luo"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=QqpbjtPbIc"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于LSM树的KV缓存存储系统
tldr: 现有磁盘KV缓存系统因文件元数据开销和I/O不高效而扩展性差。本文提出SGLANG-LSM，采用日志结构合并树架构，通过前缀保持存储、键值分离和自适应配置，实现了10倍以上的扩展性提升，支持更长序列推理。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 磁盘KV缓存系统面临文件系统元数据开销和I/O瓶颈。
method: 采用LSM-tree架构设计分层存储引擎，包含前缀保持存储和自适应控制器。
result: 在长上下文推理场景中，延迟和吞吐量显著优于现有系统。
conclusion: 数据库启发的方法能有效解决KV缓存存储的可扩展性问题。
---

## Abstract
Large language models (LLMs) rely on Key-Value (KV) cache to reduce time-
to-first-token (TTFT) latency, but existing disk-based KV cache systems using
file-per-object layouts suffer from severe scalability bottlenecks due to file system
metadata overhead, I/O inefficiency, and poor spatial locality. This paper presents
SGLANG-LSM, a database-inspired system that leverages Log-Structured Merge-
tree (LSM-tree) architectures for scalable KV cache management. SGLANG-LSM
implements a layered system design with three coordinated components: (1) a
prefix-preserving storage engine that maintains token sequence locality while
efficiently storing large KV cache tensors through key-value separation, (2) an
adaptive controller that dynamically optimizes LSM-tree configurations based on
shifting workload characteristics, and (3) runtime services including batch opera-
tions and automatic resource management for production deployment. Evaluation
on large-scale dynamic workloads demonstrates that SSGLANG-LSM significantly
improves cache hits by up to 143% and reduces TTFT by up to 24% compared to
state-of-the-art systems, representing the first systematic application of database
storage architectures to large-scale LLM cache management.

---

## 论文详细总结（自动生成）

## 1. 论文的核心问题与整体含义

- **研究动机**：大语言模型（LLM）依赖 KV 缓存降低首 Token 延迟（TTFT），但现有基于磁盘的 KV 缓存系统普遍采用“每对象文件”布局（file-per-object），导致严重可扩展性瓶颈——包括文件系统元数据开销巨大、I/O 效率低下、空间局部性差。
- **整体含义**：本文首次系统性地将数据库存储架构（LSM-tree）引入大规模 LLM 缓存管理，旨在解决磁盘 KV 缓存的扩展性难题。提出系统 SGLANG-LSM，通过日志结构合并树设计实现 10 倍以上扩展性提升，支持更长序列推理。

## 2. 论文提出的方法论

- **核心思想**：受数据库启发，采用 Log-Structured Merge-tree（LSM-tree）架构替代传统文件布局，实现高效、可扩展的 KV 缓存存储。
- **关键技术细节**：
  - **三层协同组件**：
    1. **前缀保持存储引擎**：维护 Token 序列的局部性，并通过键值分离（key-value separation）高效存储大型 KV 缓存张量。
    2. **自适应控制器**：根据运行时工作负载特征动态优化 LSM-tree 配置（如层大小、合并策略等）。
    3. **运行时服务**：提供批处理操作、自动资源管理等生产级部署功能。
  - **算法/流程**：未给出显式公式，但推理流程大致为：KV 缓存数据首先按前缀（token 序列位置）组织写入内存表，随后通过 LSM-tree 的分层合并（compaction）逐步持久化到磁盘，查询时按层级顺序检索，结合 Bloom filter 等优化减少 I/O。
- **关键创新**：将 LSM-tree 的写优化特征应用于 KV 缓存场景，同时利用键值分离避免大张量导致的写放大。

## 3. 实验设计

- **使用的数据集/场景**：大规模动态工作负载，具体数据集名称未在摘要中提及（推测可能采用长上下文推理轨迹或合成请求）。
- **Benchmark**：对比“state-of-the-art systems”（具体系统名称未列出，可能包括基于文件布局的缓存系统如 vLLM、PagedAttention 的磁盘扩展版本）。
- **对比方法**：与现有磁盘 KV 缓存系统进行对比，评测指标包括**缓存命中率（cache hits）** 和**首 Token 延迟（TTFT）**。结果显示 SGLANG-LSM 提升缓存命中达 143%，降低 TTFT 达 24%。

## 4. 资源与算力

- 论文摘要及元数据中**未明确说明**使用的 GPU 型号、数量、训练时长或推理集群配置。
- 推断：作为存储系统研究，实验可能以存储性能（延迟、吞吐、命中率）为主，算力细节并非重点，但缺乏算力信息降低了结果复现性。

## 5. 实验数量与充分性

- **实验数量**：摘要仅给出一个整体结果（缓存命中 +143%，TTFT -24%），未报告多组实验（如不同序列长度、不同工作负载模式、消融实验等）。
- **充分性评估**：
  - **优点**：动态工作负载覆盖了实际部署场景。
  - **不足**：缺乏与多种基线系统的详细对比、未进行消融研究（如单独验证前缀保持、键值分离、自适应控制器各自的贡献）、未给出统计扰动（如方差、置信区间）。因此实验充分性较低，客观性受限。

## 6. 论文的主要结论与发现

- 数据库启发的 LSM-tree 架构能有效解决 KV 缓存存储的可扩展性问题。
- SGLANG-LSM 相比现有系统，在长上下文推理中**显著提升缓存命中率（最高 143%）**，并**降低首 Token 延迟（最高 24%）**。
- 这是首个系统性地将数据库存储架构应用于大规模 LLM 缓存管理的研究，证明了跨领域方法融合的有效性。

## 7. 优点

- **方法创新**：首次将成熟数据库存储引擎（LSM-tree）应用于 LLM 缓存管理，解决了传统文件布局的元数据和 I/O 瓶颈。
- **系统设计完整性**：包含前缀保持、键值分离、自适应配置和运行时服务，覆盖存储、优化、部署三层面。
- **问题定位清晰**：直击现有磁盘 KV 缓存的根本缺陷（文件系统元数据开销、空间局部性差），解决方案具有理论依据。
- **性能提升显著**：缓存命中率翻倍以上，TTFT 降低近 1/4，效果直观且有实际意义。

## 8. 不足与局限

- **实验细节缺乏**：未公开具体数据集、对比系统名称、超参数设置、硬件环境，导致结果难以复现和公平比较。
- **消融实验缺失**：未验证各组件单独贡献，无法判断性能提升主要来自前缀保持、键值分离还是自适应控制器。
- **应用限制**：仅针对磁盘 KV 缓存场景，未讨论与 GPU 显存缓存的协同；LSM-tree 的写放大和合并开销在极端写密集场景下可能依然存在。
- **偏差风险**：可能选择对 LSM-tree 有利的 workload（如顺序访问或写多读少），未在随机读写或只读场景下验证。
- **论文状态**：被 ICLR 2026 拒稿，说明可能存在方法论或实验上的短板未被同行评委认可。

（完）
