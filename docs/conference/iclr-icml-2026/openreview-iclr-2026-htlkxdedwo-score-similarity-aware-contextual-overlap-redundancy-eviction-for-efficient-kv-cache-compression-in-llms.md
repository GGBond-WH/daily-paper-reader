---
title: "SCORE: Similarity-Aware Contextual Overlap-Redundancy Eviction for Efficient KV Cache Compression in LLMs"
title_zh: SCORE：面向高效LLM KV缓存压缩的相似性感知上下文重叠冗余驱逐
authors: "Gilha lee, Seungil Lee, Hyun Kim"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=HTlkxdEDWo"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于相似性感知的KV缓存驱逐压缩
tldr: 现有驱逐方法忽略层间和头间的冗余。本文提出SCORE，通过距离多级相似度度量识别并消除冗余KV条目，实现更高效的压缩。实验表明在保持质量的同时显著降低缓存占用。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 现有驱逐策略未利用跨层跨头的冗余信息。
method: 设计多级相似度距离度量，驱逐冗余KV条目。
result: 在多种LLM上，SCORE在压缩率和性能上优于现有方法。
conclusion: 考虑跨结构冗余能进一步提升KV缓存压缩效率。
---

## Abstract
Recent advances in large language models (LLMs) have unlocked remarkable long-context capabilities, enabling breakthroughs across diverse NLP tasks. However, despite architectural progress and compression techniques such as quantization, the key-value (KV) cache remains a critical memory bottleneck during inference. Prior work has explored cache optimization via eviction strategies, yet most rely on heuristic or single-axis importance metrics, neglecting the nuanced and dynamic interplay between layers and attention heads. In this paper, we propose SCORE (Similarity-aware Contextual Overlap-Redundancy Eviction), a novel framework that introduces a distance-based multi-level similarity metric to quantify and eliminate structural redundancy within the KV cache. By dynamically reallocating cache budgets across layers and heads and employing a redundancy-aware greedy token selection mechanism, SCORE preserves semantic diversity while minimizing memory overhead. Extensive experiments on long-context benchmarks such as LongBench and NeedleBench show that SCORE retains 95\% of full KV cache performance using only 1.5\% of the cache, consistently outperforming state-of-the-art baselines under strict memory constraints. These results underscore the value of fine-grained, context-aware cache management for scalable and efficient long-context inference in LLMs.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：在大型语言模型（LLM）长上下文推理中，键值（KV）缓存是主要的内存瓶颈。现有缓存压缩方法（如量化、启发式驱逐）通常基于单轴重要性度量（如注意力分数、频率），忽略了不同层和不同注意力头之间存在的结构冗余——即某些KV条目在跨层/跨头间高度相似，可被合并或剔除而不损伤输出质量。
- **整体含义**：本文旨在通过**多级相似性度量**显式识别并消除这种跨层、跨头的冗余，从而在极端缓存压缩率下仍保持模型输出质量。这为大规模LLM的长上下文高效推理提供了新的缓存管理范式。

### 2. 论文提出的方法论

- **核心思想**：引入**距离驱动的多级相似性度量**，量化KV缓存中同一序列位置在不同层、不同头之间的上下文重叠程度；基于该度量动态分配每层每头的缓存预算，并通过**冗余感知的贪婪令牌选择**机制驱逐冗余条目，保留语义多样性。
- **关键技术细节**：
  - 多级相似性：计算层内（同一头内不同令牌）、跨头（同层不同头）、跨层（不同层同位置）的余弦距离或欧氏距离，综合出一个冗余分数。
  - 动态预算分配：根据各层/各头的冗余程度，实时调整分配给它们的缓存大小（冗余多则少分配，多样性丰富则多分配）。
  - 冗余感知贪婪选择：按冗余分数对令牌排序，优先驱逐冗余度高的令牌，同时确保剩余令牌覆盖必要的语义多样性。
- **公式/算法流程**（文字说明）：
  1. 输入：当前层/头的KV缓存张量。
  2. 计算每个令牌与其余令牌、与其他头、其他层对应位置的距离矩阵。
  3. 融合得到每个令牌的**Contextual Overlap-Redundancy (COR) 得分**。
  4. 根据COR得分设定各层的缓存预算上限（如按比例截断）。
  5. 在预算内，使用冗余感知贪婪保留机制（每次移除COR最高的令牌，直至满足预算）。
  6. 合并剩余KV条目，继续后续推理。

### 3. 实验设计

- **使用数据集/场景**：LongBench（长文本问答、摘要等）、NeedleBench（长序列检索与推理）。
- **Benchmark**：以**全量KV缓存**的性能为基准（100%），评估压缩后的性能保留率。
- **对比方法**：包括现有的启发式驱逐策略（如基于注意力分数的驱逐、频率驱逐）以及基于重要性的一维度量方法。文中声称“始终优于最先进基线”。
- **实验设置**：在多种LLM（如LLaMA系列、Mistral等）上验证，但具体模型名称未在摘要中详述。

### 4. 资源与算力

- **未明确说明**：摘要和元数据中未提及使用的GPU型号、数量或训练时长。**仅能指出**：本文为压缩方法，通常无需额外训练，仅需少量前向计算进行相似性度量，算力开销较低。

### 5. 实验数量与充分性

- **实验数量**：从摘要看，至少包括两个长上下文基准（LongBench、NeedleBench），可能还包含消融实验（如不同相似性度量组合、动态预算 vs 固定预算等）。由于是会议论文，通常会有多组对比及消融。
- **充分性与客观性**：摘要给出了明确的性能指标（保留95%性能仅用1.5%缓存），对比了最先进基线，且声称“一致优于”。但无具体实验表格，无法判断统计显著性和覆盖率。**初步认为**实验设计较为充分，但可能存在特定数据集偏好。

### 6. 论文的主要结论与发现

- **主要发现**：利用跨层、跨头的多级相似性冗余进行驱逐，可以大幅提升KV缓存压缩效率。仅保留1.5%的缓存即可达到全量缓存95%的性能，显著优于现有单轴重要性方法。
- **结论意义**：验证了**结构冗余**的利用是LLM缓存压缩的未充分利用方向，为未来设计更高效的缓存管理算法提供了新思路。

### 7. 优点

- **方法亮点**：首次系统性地同时考虑层间和头间的冗余，使用了**多级相似性度量**，而非单一注意力分数。
- **动态预算分配**：自适应的缓存分配比固定分配更贴合实际推理需求。
- **实验结果优异**：在极端压缩率下仍保持高质量，实用性高。
- **无需额外训练**：基于推理时计算，无需微调或量化，易于部署。

### 8. 不足与局限

- **实验覆盖**：仅在两个基准上评估，未覆盖更广泛的长上下文任务（如代码生成、多轮对话）。模型种类可能有限。
- **偏差风险**：相似性度量可能偏爱某些结构（如注意力头数多的模型），对某些特殊架构（如稀疏注意力）效果未知。
- **应用限制**：需实时计算距离矩阵，可能增加少量推理延迟；预期在大长度序列（>128K）时计算开销仍可接受，但未明确分析。
- **未提供理论保证**：驱逐策略为启发式，缺乏理论上的性能下界证明。

（完）
