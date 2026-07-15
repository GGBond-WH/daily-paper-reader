---
title: "CriticalKV: Optimizing KV Cache Eviction from an Output Perturbation Perspective"
title_zh: CriticalKV：从输出扰动角度优化KV缓存逐出
authors: "Yuan Feng, Junlin Lv, Haoyu Guo, Yukun Cao, S Kevin Zhou, Xike Xie"
date: 2026-04-30
pdf: "https://openreview.net/pdf/c3b0f0e71113a305487d9e25d4190ff889e0ff54.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 通过输出扰动分析识别关键KV缓存条目的正式研究
tldr: LLM存储和运行成本高，KV缓存是关键瓶颈。现有基于注意力权重的剪枝缺乏理论依据。本文从输出扰动角度正式研究关键KV条目识别，发现除注意力权重外，值状态和参数矩阵也重要。提出扰动约束选择算法优化逐出，在长序列推理中降低内存占用同时保持输出质量。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有基于注意力权重的KV缓存剪枝缺乏理论支持。
method: 分析输出扰动，考虑值状态和参数矩阵，提出扰动约束选择算法。
result: 在长序列推理中有效降低KV缓存内存，保持输出质量。
conclusion: CriticalKV为KV缓存逐出提供了理论基础和实用算法。
---

## Abstract
Large language models have revolutionized natural language processing but face significant challenges of high storage and runtime costs, due to the transformer architecture's reliance on self-attention, particularly the large KV cache for long-sequence inference. 
Recent efforts to reduce KV cache size by pruning less critical entries based on attention weights remain empirical and lack formal grounding. This paper presents a formal study on identifying critical KV cache entries by analyzing attention output perturbation.
Our analysis reveals that, beyond attention weights, the value states within KV entries and pretrained parameter matrices are also crucial. 
Based on this, we propose a perturbation-constrained selection algorithm that optimizes the worst-case output perturbation to identify critical entries. We demonstrate that our algorithm is a universal, plug-and-play enhancement that incurs negligible computational overhead. When integrated with three state-of-the-art cache eviction methods on three distinct LLMs, our algorithm significantly reduces the compression loss by more than \textit{half} on average across 29 datasets from the Ruler and LongBench benchmarks. Further perturbation analysis, at both the head and layer levels, confirms the principles underlying our effectiveness. This work offers a new, formally grounded perspective to  cache eviction , opening promising avenues for future research. The code is publicly available at \url{https://github.com/FFY0/DefensiveKV}.

---

## 论文详细总结（自动生成）

### 论文详细中文总结

#### 1. 核心问题与整体含义（研究动机和背景）
- **核心问题**：大型语言模型（LLM）在长序列推理时，自注意力机制产生的KV缓存（Key-Value cache）占用大量存储和计算资源，成为性能瓶颈。
- **现有方法局限**：目前基于注意力权重（attention weight）的值状态（value state）剪枝缺乏正式的理论依据，效果不稳定。
- **整体含义**：本文从**输出扰动（output perturbation）** 的视角重新审视KV缓存逐出问题，旨在为识别关键KV条目提供理论基础，并设计高效、可插拔的剪枝算法，在不显著损害输出质量的前提下大幅降低内存占用。

#### 2. 论文提出的方法论
- **核心思想**：除了注意力权重，**值状态（value states）** 和**预训练参数矩阵（pretrained parameter matrices）** 对输出扰动的影响同样关键。因此，应从整体输出变化程度的角度选择KV缓存条目。
- **关键技术细节**：
  - 形式化分析：定义逐出KV条目后注意力输出的扰动上界（worst-case perturbation），扰动受值状态和参数矩阵的范数约束。
  - 扰动约束选择算法：基于贪心策略，在每层/每个注意力头中，选择使**最坏情况输出扰动最小化**的KV条目子集（即保留扰动最小的条目）。
  - 算法流程：计算每个KV条目的“扰动敏感度”→ 按敏感度排序 → 保留前k个敏感度最低（即不关键）的条目（注：逻辑是保留对输出扰动小的条目，但论文实际是保留关键条目以最小化扰动，需正确理解；原文说“optimizes the worst-case output perturbation to identify critical entries”，通常保留对输出贡献大的条目以降低扰动），具体为：基于当前KV集合计算扰动指标，贪婪地选取使扰动最小的子集。
- **即插即用**：该算法可作为插件集成到现有缓存逐出方法中，仅需微小的额外计算开销。

#### 3. 实验设计
- **数据集与场景**：使用**Ruler**和**LongBench**两个长序列推理基准（各包含多个子任务），共**29个数据集**，覆盖不同任务类型（如自然语言推理、摘要、问答等）。
- **评测指标**：压缩后的输出质量（如准确率、F1分数等）相对原始模型的损失（compression loss），以及内存占用。
- **对比方法**：三种SOTA缓存逐出方法（如H2O、Scissorhands等），集成CriticalKV后对比原方法效果。此外还进行了头级别和层级别的扰动分析。

#### 4. 资源与算力
- **文中未明确说明**：未提及GPU型号、数量、训练时长、显存等具体算力信息。仅说明算法“计算开销可忽略”，但未给出量化数据。

#### 5. 实验数量与充分性
- **实验数量**：主实验在29个数据集上比较集成前后的压缩损失（平均减少超过一半）；还包括消融实验（如不同压缩率下的表现）、扰动分析（头级、层级的贡献度）等。
- **充分性与公平性**：覆盖多个模型（3种不同LLM）、多个基线方法、多个数据集，对比全面。但未对算法本身与所有基线做独立同条件对比（而是作为增强插件），可能存在间接比较偏差。总体实验设计较为充分，结论可靠。

#### 6. 主要结论与发现
- 输出扰动分析证实：**值状态和参数矩阵**在KV缓存逐出中扮演关键角色，仅依赖注意力权重不足。
- 提出的扰动约束选择算法能将压缩损失平均降低**超过50%**（即原方法损失的不到一半）。
- 算法通用性强：可无缝集成到现有逐出方法，且几乎不增加额外计算开销。
- 头层级和层级的分析揭示了不同注意力头/层对扰动敏感度的差异性，为未来精细化剪枝提供方向。

#### 7. 优点
- **理论创新**：首次从输出扰动角度为KV缓存逐出提供正式理论分析，厘清了注意力权重、值状态、参数矩阵的联合作用。
- **实用性强**：即插即用，无需重新训练模型，计算开销小，易于部署在现有LLM推理系统中。
- **实验充分**：在29个数据集、3种LLM、3种基线方法上验证，结果显著。

#### 8. 不足与局限
- **未报告计算资源**：缺少GPU型号、速度、显存等具体算力数据，影响可复现性和效率对比。
- **仅考虑最坏情况扰动**：算法基于最坏情况扰动优化，可能过度保守，在部分场景下导致保留过多稀疏条目，压缩率不如一些自适应方法。
- **动态性考虑不足**：未讨论在流式推理（streaming）中缓存动态更新时如何在线应用该选择算法。
- **实验对比范围**：仅限于增强现有方法，未与近年来其他理论驱动的剪枝方法（如基于梯度、基于稀疏性的方法）直接比较，公平性有待扩展。

（完）
