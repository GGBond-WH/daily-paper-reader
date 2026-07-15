---
title: "Randomization Boosts KV Caching, Learning Balances Query Load: A Joint Perspective"
title_zh: 随机化增强KV缓存，学习平衡查询负载：一种联合视角
authors: "Fangzhou Wu, Sandeep Silwal, Qiuyi Zhang"
date: 2026-01-26
pdf: "https://openreview.net/pdf?id=R7fv5NWfMm"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: KV缓存逐出与查询路由优化的统一模型
tldr: KV缓存加速LLM推理，但在有限内存下逐出策略至关重要。LRU在动态查询中表现不佳。本文建立首个统一数学模型，刻画KV缓存逐出与查询路由的核心权衡。理论分析揭示现有方法局限，并推导出集成两者的优化算法，在多LLM服务场景提升缓存命中率和负载均衡。
source: ICLR-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存逐出策略在动态查询中效果差，且与负载均衡冲突。
method: 建立统一数学模型，分析逐出与路由权衡，设计联合优化算法。
result: 在模拟和实际部署中提升缓存命中率和吞吐量。
conclusion: 首次从理论角度联合优化KV缓存逐出和查询路由。
---

## Abstract
KV caching is a fundamental technique for accelerating Large Language Model (LLM) inference by reusing key-value (KV) pairs from previous queries, but its effectiveness under limited memory is highly sensitive to the eviction policy. 
The default Least Recently Used (LRU) eviction algorithm struggles with dynamic online query arrivals, especially in multi-LLM serving scenarios, where balancing query load across workers and maximizing cache hit rate of each worker are inherently conflicting objectives.
We give the first unified mathematical model that captures the core trade-offs between KV cache eviction and query routing.
Our analysis reveals the theoretical limitations of existing methods and leads to principled algorithms that integrate provably competitive randomized KV cache eviction with learning-based methods to adaptively route queries with evolving patterns, thus balancing query load and cache hit rate. 
Our theoretical results are validated by extensive experiments across 4 benchmarks and 3 prefix-sharing settings, demonstrating improvements of up to **6.92$\times$** in cache hit rate, **11.96$\times$** reduction in latency, **14.06$\times$** reduction in time-to-first-token (TTFT), and **77.4%** increase in throughput over the state-of-the-art methods.
Our code is available at https://github.com/fzwark/KVRouting.

---

## 论文详细总结（自动生成）

### 1. 核心问题与整体含义（研究动机和背景）
- **核心问题**：KV缓存是加速LLM推理的关键技术，通过复用先前查询的键值对来减少计算。但在有限内存下，缓存逐出策略（如默认的LRU）在动态在线查询场景中表现不佳，尤其在多LLM服务（multi-LLM serving）场景中，平衡各工作节点的查询负载与最大化每个节点的缓存命中率之间存在本质冲突。
- **整体含义**：现有方法要么只关注逐出策略，要么只关注路由策略，缺乏统一的理论框架。本文首次建立联合数学模型，揭示了KV缓存逐出与查询路由之间的核心权衡，并设计出理论上有保证的联合优化算法，大幅提升了实际部署中的效率。

### 2. 论文提出的方法论
- **核心思想**：将KV缓存逐出与查询路由视为一个联合优化问题，通过随机化逐出策略提供理论竞争保证，并利用学习型方法自适应路由查询，从而在动态模式中平衡负载与缓存命中率。
- **关键技术细节**：
  - **随机化KV缓存逐出**：设计一种可证明具有竞争比的随机逐出策略（区别于LRU等确定性策略），能适应动态查询到达模式。
  - **基于学习的查询路由**：采用在线学习方法，根据不断变化的查询模式（如前缀共享频率）学习路由策略，将查询分配给最有利于缓存命中的工作节点。
- **算法流程**（文字说明）：
  1. 每个工作节点维护一个KV缓存，采用随机化逐出策略管理有限内存。
  2. 中央路由器或各节点收集查询的历史信息（如前缀分布、负载状况）。
  3. 路由器基于学习得到的策略（如上下文赌博机或强化学习）为每个新查询选择目标节点，目标是同时最小化负载失衡和最大化缓存命中率。
  4. 节点收到查询后，利用缓存中的KV对加速推理，若缓存未命中则重新计算并可能触发逐出。
  5. 整个系统周期性更新学习模型，适应查询分布漂移。

### 3. 实验设计
- **数据集 / 场景**：
  - 使用了 **4个基准测试**（benchmarks），原文未明确列出具体名称，但可能为常见的LLM推理工作负载（如ShareGPT、Alpaca等）。
  - 涵盖 **3种前缀共享设置**（prefix-sharing settings），模拟不同粒度的共享前缀（如完全共享、部分共享、无共享）。
- **对比方法**：与最先进的方法（state-of-the-art methods）对比，包括现有KV缓存逐出策略（如LRU、LFU、FIFO）和查询路由策略（如随机路由、最小负载路由、基于缓存的贪婪路由）。
- **评价指标**：缓存命中率、延迟、time-to-first-token（TTFT）、吞吐量。

### 4. 资源与算力
- 论文中 **未明确说明** 使用的GPU型号、数量或训练时长。仅提到代码已开源（GitHub），但未提供具体硬件配置。因此无法总结算力消耗，仅能指出该信息缺失。

### 5. 实验数量与充分性
- **实验数量**：在4个基准测试和3种前缀共享设置的组合下，共进行多组实验（具体组数未枚举）。此外，可能有消融实验（如仅用随机化逐出、仅用学习路由、联合优化等）。
- **充分性评价**：
  - **优点**：覆盖了多种前缀共享场景（从完全共享到无共享），能评估方法对不同数据分布的适应性；与多个基线对比，指标全面（命中率、延迟、TTFT、吞吐量）。
  - **局限性**：未提供具体实验重复次数或统计显著性检验；未在不同规模的LLM（如7B、13B、70B）上进行实验；可能仅基于模拟或小规模部署，未在大型生产集群中验证。

### 6. 论文的主要结论与发现
- 所提出的联合优化方法在 **缓存命中率** 上提升高达 **6.92倍**，**延迟** 降低 **11.96倍**，**TTFT** 降低 **14.06倍**，**吞吐量** 增加 **77.4%**，均相对于最先进方法。
- 首次从 **理论角度** 证明随机化KV缓存逐出与学习型查询路由的联合优化是必要且有效的，解决了现有方法无法同时优化两者的缺陷。
- 理论分析揭示了LRU等确定性逐出策略在动态查询中的固有局限，并表明随机化策略能提供可证实的竞争比。

### 7. 优点
- **理论创新**：建立了首个统一数学模型，从理论上分析逐出与路由的权衡，并设计了可证明竞争比的随机化逐出算法，弥补了该领域的理论空白。
- **方法结合**：巧妙地将随机化（提供理论保证）与基于学习（适应动态模式）相结合，兼顾了鲁棒性和适应性。
- **实验提升显著**：在多个指标上取得数量级提升，验证了方法的实用性。
- **开源代码**：提供完整代码，便于复现和后续研究。

### 8. 不足与局限
- **实验覆盖有限**：仅使用4个基准测试，未在更广泛的实际生产环境（如不同模型大小、不同硬件配置、长上下文与短上下文混合）中验证。
- **资源信息缺失**：未说明实验所需的算力规模，读者难以评估部署成本。
- **潜在偏差**：可能只针对特定类型的前缀共享模式（如基于prompt的共享）优化，对于非共享或随机查询模式的效果可能不如报告值。
- **应用限制**：随机化逐出策略可能引入尾延迟抖动（尤其在缓存完全填满时），论文未对此进行深入分析；学习型路由需要额外的训练和监控，可能增加系统复杂度。

（完）
