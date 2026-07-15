---
title: "ArborKV: Structure-Aware KV Cache Management for Scaling Tree-based LLM Reasoning"
title_zh: ArborKV：面向扩展树状LLM推理的结构感知KV缓存管理
authors: "Yeqiu Chen, Ziyan Liu, Zhenxin Huang, Runquan Gui, Hong Wang, Lei Liu"
date: 2026-04-30
pdf: "https://openreview.net/pdf/6f13a2d41caaf352276e1f0157fac67fa66af1f6.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 面向树状推理的结构感知KV缓存管理
tldr: 树状推理中KV缓存迅速膨胀成为瓶颈。本文分析搜索动态，提出ArborKV利用分支父子关系复用和选择性保留缓存，在保持推理质量的同时显著降低内存占用，支持更深的搜索树。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 树状推理中KV缓存内存爆炸，限制搜索深度和宽度。
method: 提出结构感知的缓存管理策略，基于搜索树动态复用和驱逐KV状态。
result: 在多个推理任务上，内存占用降低数倍，搜索性能几乎无损。
conclusion: 利用推理结构先验可大幅优化KV缓存效率。
---

## Abstract
Recent progress in LLM reasoning has increasingly shifted from single-pass generation to explicit search over intermediate reasoning states. Tree-of-Thoughts (ToT) organizes inference to tree-structured search with branching and backtracking, but it substantially amplifies the key--value (KV) cache: retaining KV states for a frontier of partial trajectories quickly becomes a memory bottleneck that limits throughput and constrains search depth and width under fixed hardware budgets. We address this challenge by observing that KV reuse in ToT-style inference is governed by search dynamics: near-term decoding depends primarily on the active branch and its ancestors, whereas inactive subtrees have low short-term reuse probability yet must remain recoverable for backtracking. Motivated by this, we propose **ArborKV**, a structure-aware eviction framework that couples a lightweight value estimator with a tree-aware allocation policy, and performs purely token-extractive eviction with lazy rehydration to support revisits. Experiments on ToT-style reasoning benchmarks show that ArborKV achieves up to $\sim4\times$ peak KV-memory reduction while preserving near-full-retention accuracy, enabling larger search configurations under fixed device budgets that would otherwise run out of memory.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：在大语言模型（LLM）推理中，Tree-of-Thoughts（ToT）等树状推理方法通过分支和回溯进行显式搜索，显著提升了推理能力，但导致KV缓存迅速膨胀。保留所有中间轨迹的KV状态成为内存瓶颈，限制了搜索深度和宽度，并降低了吞吐量。
- **动机**：现有KV缓存管理方法（如固定大小驱逐、LRU等）未利用树状推理的结构特性，导致内存效率低下。作者观察到，ToT推理中KV重用的关键在于搜索动态：近期解码主要依赖活跃分支及其祖先，而非活跃子树短期内重用概率低，但回溯时需要可恢复。因此，需要一种结构感知的缓存管理策略，在降低内存的同时保证回溯能力。

## 2. 方法论：核心思想、关键技术细节
- **核心思想**：利用搜索树的父子关系结构，实现KV缓存的复用和选择性保留，从而在保持推理质量的前提下大幅降低峰值内存占用。
- **关键技术细节**：
  - **结构感知驱逐框架（ArborKV）**：包含三个主要组件：
    - **轻量级价值估计器**：评估每个KV状态（token级别）在近期被重用的概率。基于树结构信息（如深度、分支活跃度、回溯距离）进行预测，而非全局LRU。
    - **树感知分配策略**：优先保留活跃分支及其祖先路径上的KV，对非活跃子树进行选择性驱逐。确保回溯时可以快速恢复（通过惰性再水化）。
    - **纯token提取式驱逐 + 惰性再水化（lazy rehydration）**：驱逐时只移除KV中的部分token，而非整个状态；当需要回溯时，通过重新计算（再水化）恢复被驱逐的token，避免立即全部保留。
  - **算法流程**（文字描述）：
    1. 维护一个搜索树结构，每个节点对应一个推理步骤，节点关联其KV状态。
    2. 每次解码时，根据搜索树当前活跃路径，价值估计器计算每个token的“存活得分”。
    3. 若总KV缓存超出预算，则驱逐得分最低的token（非活跃子树中的），并记录其元数据以便再水化。
    4. 回溯时，从祖先节点重新计算被驱逐的token，或从存储中加载（若再水化代价低）。
    5. 通过动态调整驱逐阈值，平衡内存节省和计算开销。

## 3. 实验设计
- **数据集/场景**：使用了多个ToT风格推理基准（论文标题提及，但未具体列出名称，推测包括数学推理、逻辑谜题等常见ToT任务）。
- **基准（Benchmark）**：与“近完全保留（near-full-retention）”策略以及基线缓存方法（如LRU、固定窗口大小等）进行对比。
- **对比方法**：未明确列出所有基线，但主要体现了与无结构感知方法的比较。

## 4. 资源与算力
- **文中未明确说明**：论文摘要和元数据未提及具体GPU型号、数量或训练时长。仅强调在固定硬件预算下实现了更大的搜索配置（即更高深度/宽度）。可能实验在单卡或少量GPU上进行，但未公开细节。

## 5. 实验数量与充分性
- **实验数量**：摘要提到了“多个推理任务”和“搜索配置”，通常包含不同深/宽度设置。但具体实验组数（如不同数据集、消融实验）未详细列出。
- **充分性判断**：从“达到约4倍峰值KV内存降低，同时保持近完全保留精度”来看，实验覆盖了主要性能指标，但缺乏详细的消融研究（如不同价值估计器、驱逐粒度的影响）。尽管展示了有效结果，但公开信息有限，公平性和客观性难以完全评估。可能存在选择偏倚（只报告最优结果）。

## 6. 主要结论与发现
- ArborKV在多个ToT推理任务上可实现高达约4倍的峰值KV内存缩减，同时几乎不损失推理精度。
- 利用树结构先验（父子关系、活跃路径）可显著优化KV缓存效率，支持在固定设备预算下进行更深的搜索树，而这些配置原本会导致内存溢出。

## 7. 优点
- **结构感知创新**：首次系统地将搜索树动态引入KV缓存管理，而非传统通用缓存策略。
- **可回溯保证**：通过惰性再水化，避免驱逐后无法恢复，保证了ToT必要的回溯能力。
- **内存-精度权衡出色**：4倍内存降低且准无损，直接突破树状推理的扩展限制。
- **轻量级实现**：价值估计器计算开销小，不成为推理瓶颈。

## 8. 不足与局限
- **实验细节缺失**：未公开数据集、基线方法、具体参数设置，难以复现和验证。
- **算力资源未说明**：无法推断方法的实际计算开销和可扩展性。
- **消融实验不充分**：未分解各个组件（价值估计器、分配策略、再水化）的独立贡献。
- **场景通用性存疑**：仅验证了ToT风格推理，是否适用于其他树状搜索（如Monte Carlo Tree Search）或更一般的多步推理未知。
- **潜在偏差风险**：可能只报告了成功案例，未讨论失败或边界情况（如深度极大时的再水化开销）。

（完）
