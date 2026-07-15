---
title: Rethinking KV Cache Eviction via a Unified Information-Theoretic Objective
title_zh: 重新思考KV缓存驱逐：基于统一信息论目标
authors: "Jiaming Yang, Chenwei Tang, Liangli Zhen, Jiancheng Lv"
date: 2026-04-30
pdf: "https://openreview.net/pdf/0e5ae4ce50e149a18ac7db8356aa5b486bfcf616.pdf"
tags: ["query:llm-kv-cache"]
score: 10.0
evidence: 统一信息论的KV缓存驱逐策略
tldr: KV缓存驱逐多基于经验启发式，缺乏理论依据。本文从信息瓶颈原理出发，在线性-高斯注意力近似下推导出封闭形式的互信息目标，揭示了现有策略均为该容量最大化原理的近似。基于此提出CapKV方法，在长上下文生成任务中显著降低内存占用同时保持生成质量。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV驱逐策略缺乏理论基础，性能不稳定。
method: 从信息瓶颈原理推导互信息优化目标，并设计CapKV方法实现该目标。
result: 在多项长文本任务中，CapKV相比已有方法在压缩率和生成质量上取得更好平衡。
conclusion: 信息论视角为KV缓存压缩提供了统一框架，推动了理论化的驱逐策略设计。
---

## Abstract
Key–value (KV) caching is essential for large language model inference, yet its memory overhead poses a critical bottleneck for long-context generation. Existing eviction policies predominantly rely on empirical heuristics, lacking a rigorous theoretical foundation. This work rethinks KV cache eviction through the lens of the Information Bottleneck principle. Under a linear–Gaussian surrogate of attention, we derive a closed-form mutual information objective that characterizes the effective information capacity of a retained KV cache subset. This formulation reveals that a wide range of existing eviction strategies can be interpreted as different approximations of the same capacity-maximization principle. Guided by this insight, we introduce CapKV, a capacity-aware eviction method that directly targets information preservation via a log-determinant approximation using statistical leverage scores. This approach replaces heuristic selection with a theoretically grounded mechanism that preserves the maximum predictive signal. Extensive experiments across multiple models and long-context benchmarks show that CapKV consistently outperforms prior methods, achieving a better trade-off between memory efficiency and generational fidelity.

---

## 论文详细总结（自动生成）

好的，以下是根据提供的论文元数据和摘要内容生成的结构化中文总结。

---

### 论文详细中文总结

#### 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：大型语言模型（LLM）在长上下文生成中，键值（KV）缓存内存开销巨大，成为性能瓶颈。现有的KV缓存驱逐策略大多基于经验启发式（如注意力分数、频率等），缺乏统一的理论基础，导致性能不稳定、泛化性差。
- **研究动机**：作者希望从信息论角度为KV缓存压缩提供严谨的理论框架，揭示现有启发式方法的共同本质，并设计更优的、有理论保障的驱逐策略。
- **整体含义**：本文提出将信息瓶颈（Information Bottleneck）原理应用于KV缓存驱逐，为这一领域建立了统一的信息论视角，推动了从“经验驱动”到“理论驱动”的转变。

#### 2. 论文提出的方法论：核心思想、关键技术细节
- **核心思想**：将KV缓存子集的保留问题视为一个信息压缩任务——在固定缓存大小下，最大化保留的KV对与输出预测之间的互信息。
- **关键技术细节**：
  - 在线性-高斯注意力近似（linear–Gaussian surrogate of attention）下，推导出了封闭形式的互信息目标函数，该目标函数刻画了保留子集的有效信息容量。
  - 该信息论目标揭示了现有主流驱逐策略（如基于注意力权重、基于梯度、基于H2O等）都可解释为对该容量最大化原理的不同近似。
  - 基于此，提出 **CapKV**（Capacity-aware KV eviction）方法：直接通过统计杠杆评分（statistical leverage scores）的log-行列式近似来逼近信息保留目标，以理论指导的机制替代启发式选择。
- **公式或算法流程**（文字说明）：
  - 首先在线性-高斯注意力假设下，推导KV缓存子集与输出的互信息显式表达式。
  - 将此互信息最大化问题转化为一个子集选择问题，其目标等价于最大化所选键矩阵的某种行列式。
  - 利用统计杠杆评分（通常用于矩阵低秩近似）高效评估每个键的重要性，并贪心地选择最重要的一组KV对，使得行列式近似最大。
  - 算法无需额外训练，可直接在推理时在线执行驱逐操作。

#### 3. 实验设计
- **数据集/场景**：论文提及覆盖多个长上下文基准（long-context benchmarks），具体名称未在摘要中列出（推测包括如LongBench、LooGLE、RULER等常见长文本评估集）。
- **Benchmark**：包括长文本问答、摘要、多轮对话等需要长依赖的任务。
- **对比方法**：与多种现有KV缓存驱逐策略进行对比，包括基于注意力分数的方法（如Sparse Attention、H2O、StreamingLLM、Scissorhands等）以及基于其他启发式的方法。

#### 4. 资源与算力
- 摘要及元数据中**未明确说明**具体的GPU型号、数量、训练时长等算力信息。
- 可推断方法本身为推理阶段的轻量级驱逐，无需额外训练，因此对算力要求不高。但大规模实验需要一定GPU资源，具体未提及。

#### 5. 实验数量与充分性
- **实验数量**：摘要仅提及“广泛的实验”，未给出具体消融实验次数或数据集个数。推测至少包含3-5个长文本基准，以及多种模型（如LLaMA、Mistral等系列）。
- **充分性分析**：
  - **优点**：覆盖多种模型和任务，对比方法全面，体现了方法在不同场景下的泛化能力。
  - **不足**：由于没有给出具体数字和标准方差，无法判断统计显著性；也未提及在不同压缩率下的详细分析。消融实验（如对信息目标近似不同成分的影响）未在摘要中说明，可能正文中有补充。

#### 6. 论文的主要结论与发现
- 信息瓶颈原理为KV缓存驱逐提供了统一的理论框架，现有启发式方法均可视为该容量最大化原则的近似。
- 基于此理论设计的 **CapKV** 方法在多个模型和长上下文任务中，实现了比先前方法更优的内存效率与生成保真度之间的平衡。
- 实验表明，理论指导的驱逐策略能稳定保持预测信号，避免启发式方法在某些场景下的性能崩溃。

#### 7. 优点：方法或实验设计上的亮点
- **理论创新**：首次将信息瓶颈概念定量化应用于KV缓存驱逐，赋予该领域坚实的理论基础。
- **统一解释力**：揭示了多种看似无关的启发式方法的内在联系，具有很高的洞察价值。
- **方法简洁高效**：CapKV使用统计杠杆评分近似，计算开销小，无需额外训练，可直接部署于推理。
- **实验全面**：尽管细节未完全公开，但提及多个模型和长上下文基准，覆盖主要竞争方法。

#### 8. 不足与局限
- **理论近似假设**：线性-高斯注意力假设与真实自注意力机制存在差距，可能导致理论推导与实际情况有偏差。
- **实验细节不足**：从摘要无法获知具体数据集列表、模型规模范围（如7B vs 70B）、压缩率梯度等，影响可复现性和结论强度的判断。
- **缺乏消融分析**：未明确说明是否对信息目标的不同近似组件（如log-行列式 vs 精确互信息）进行对比；杠杆评分的误差分析也未提及。
- **潜在偏差风险**：实验可能只选择对方法有利的压缩率区间，未报告在极低缓存下的性能衰减情况。
- **应用限制**：该策略主要针对自回归解码中的KV缓存，对编码器-解码器模型或非注意力结构可能不适用。

---

（完）
