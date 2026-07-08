---
title: "Judge Q: Trainable Queries for Optimized Information Retention in KV Cache Eviction"
title_zh: Judge Q：可训练查询优化KV缓存驱逐中的信息保持
authors: "Yijun Liu, Yixuan Wang, Yuzhuang Xu, Shiyu Ji, Yang Xu, Qingfu Zhu, Wanxiang Che"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40497/44458"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 可训练查询优化KV缓存驱逐以降低内存
tldr: 当前KV缓存驱逐方法使用预填充阶段最后窗口作为查询，过于关注局部导致全局信息丢失。本文提出Judge Q，通过可训练的软令牌列表优化查询，仅调整嵌入层参数。该方法使模型自适应学习如何选择要保留的KV对，实验表明在长文本任务中性能优于基于窗口的驱逐策略，且有效减少内存占用。Judge Q为KV缓存驱逐提供了一种轻量级训练范式。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV驱逐仅用局部窗口查询，忽略全局重要性，导致重要信息被误删。
method: 引入软令牌列表作为可训练查询，仅微调嵌入层，学习全局重要性评分以指导驱逐。
result: 在长文本语言建模和问答任务上，Judge Q在同等缓存预算下困惑度更低，信息召回更全面。
conclusion: 通过可训练查询驱逐KV缓存能更有效保留全局信息，是一种前景广阔的轻量级优化方案。
---

## Abstract
Large language models (LLMs) utilize key-value (KV) cache to store historical information during sequence processing. The size of KV cache grows linearly as the length of the sequence extends, which seriously affects memory usage and decoding efficiency. Current methods for KV cache eviction typically utilize the last window from the pre-filling phase as queries to compute the KV importance scores for eviction. Although this scheme is simple to implement, it tends to overly focus on local information, potentially leading to the neglect or omission of crucial global information. To mitigate this issue, we propose **Judge Q**, a novel training method which incorporates a soft token list. This method only tunes the model’s embedding layer at a low training cost. By concatenating the soft token list at the end of the input sequence, we train these tokens' attention map to the original input sequence to align with that of the actual decoded tokens. In this way, the queries corresponding to the soft tokens can effectively capture global information and better evaluate the importance of the keys and values within the KV cache, thus maintaining decoding quality when KV cache is evicted. Under the same eviction budget, our method exhibits less performance degradation compared to existing eviction approaches. We validate our approach through experiments conducted on models such as Llama-3.1-8B-Instruct and Mistral-7B-Instruct-v0.3, using benchmarks including LongBench, RULER, and Needle-in-a-Haystack. Results indicate an improvement of approximately 1 point on the LongBench and over 3 points on RULER. This proposed methodology can be seamlessly integrated into existing open-source models with minimal training overhead, thereby enhancing performance in KV cache eviction scenarios.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：大语言模型（LLM）在长序列处理中使用键值缓存（KV Cache）存储历史信息，但序列越长，KV Cache的线性增长会导致严重的内存占用和推理效率下降。现有的KV Cache驱逐方法（如H2O、SnapKV、PyramidKV等）通常直接使用预填充阶段最后窗口的查询（query）来计算键值对的重要性分数，这种方法虽然简单，但倾向于过度聚焦局部信息，忽略全局重要信息，尤其在问题不位于输入末尾时性能退化明显。
- **关键发现**：作者通过实验发现，如果直接使用实际解码的token作为查询来引导KV Cache驱逐（即“理论最优上限”），性能显著优于现有方法。但实际解码时无法预知未来token，因此需要一种**近似的方法**，在预填充阶段就能识别出与解码token重要性分布相近的关键KV对。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- **可训练软令牌（soft tokens）**：在模型词汇表中添加一组少量的、可学习的软令牌（数量n=32）。训练时将其拼接在输入序列末尾，仅微调这些软令牌对应的嵌入层参数（其余模型参数冻结），使得软令牌的注意力分布与真实解码token的注意力分布对齐。推理时利用这些软令牌产生的注意力分数来指导KV Cache驱逐。

### 关键技术细节
- **训练目标**：最小化软令牌与真实解码token对原始输入序列的平均注意力图的均方误差（MSE）。
  - 定义软令牌序列 `Soft` 和响应序列 `Resp`，分别拼接在Prompt后形成 `Input_soft` 和 `Input_resp`。
  - 计算平均注意力图：  
    `A_soft = (1/n) * sum_i Attention(Prompt, Soft_i)`  
    `A_resp = (1/m) * sum_j Attention(Prompt, Resp_j)`
  - 损失 `L = MSE(A_soft, A_resp)`
- **推理过程**：在预填充阶段，同样将n个软令牌拼接在输入末尾，计算它们的注意力分数作为键值的重要性分数，保留Top-k的KV对，然后移除软令牌，后续解码使用剪枝后的KV Cache。
- **训练数据与配置**：使用ShareGPT数据集的5万条样本（45k通用域+5k计算机领域），模型仅训练嵌入层对应的参数，其余冻结。实验发现n=32时效果与效率平衡最佳。

## 3. 实验设计

### 数据集/场景
- **训练数据**：ShareGPT子集（5万条，包含prompt和response）。
- **测试基准**：
  - **LongBench**：多任务长文本理解基准，包括Single-Doc QA、Multi-Doc QA、Summarization、Few-shot、Synthesis、Code等。
  - **RULER**：长上下文理解基准，测试序列长度8192和32768。
  - **Needle-in-a-Haystack**：检索测试，最小长度4000，最大长度32000，文档深度间隔10%。
- **其他评测**：MATH-500和AIME24（数学推理任务）上的文本延续任务，以及针对Prompt调整（问题放在开头）的退化比例测试。

### 对比方法
- StreamingLLM（仅保留开头和局部窗口）
- H2O（累积注意力分数动态驱逐）
- SnapKV（局部窗口计算分数+池化）
- PyramidKV（基于SnapKV的分层动态预算分配）

所有基线在同一设置下测试（局部窗口大小=32，池化配置相同）。

### 评估指标
- 任务准确率（F1/EM等）
- 关键KV命中率（HitRate）：与真实解码token选择的KV对重叠比例
- 性能退化百分比

## 4. 资源与算力

论文明确提到：
- 训练使用的模型为Llama-3.1-8B-Instruct和Mistral-7B-Instruct-v0.3。
- 训练数据为ShareGPT的5万条样本。
- 训练过程仅调整嵌入层参数，其余冻结，因此训练成本很低。
- **未明确说明**使用的具体GPU型号、数量、训练时长等详细算力信息。仅提到“minimal training overhead”（最小训练开销）。

## 5. 实验数量与充分性

### 实验数量
- **主实验**：在LongBench上，对两种模型（Llama-3.1-8B-Instruct和Mistral-7B-Instruct-v0.3）分别在3个KV Cache预算（128、256、512）下与4个基线对比，共24组（2模型×3预算×4基线）加上自身结果（2模型×3预算×1方法）以及Full KV结果（2模型）。
- **RULER**：2种序列长度（8192、32768）×3种预算（256、512、1024）×2模型×5方法 + Full KV，共30组。
- **Needle-in-a-Haystack**：多深度多长度下的可视化对比。
- **消融与分析实验**：
  - 关键KV命中率对比（Judge Q vs SnapKV，3个预算）。
  - Prompt调整（问题前置）下的性能退化比例（与基线对比）。
  - 文本延续任务（MATH-500, AIME24）上Judge Q vs SnapKV（1个预算）。
  - 数据质量影响：对比ShareGPT、C4、Wikipedia、GitHub等训练数据来源。
  - 软令牌数量影响（n=8,16,32,64等），验证损失和可训练参数量。
- 总实验组数超过60组，覆盖多种任务、预算、模型和消融维度。

### 充分性与公平性
- **充分性**：实验覆盖了主流长文本基准、不同模型家族（Llama、Mistral）、多种预算（高、中、低）、多种任务类型，并包括消融研究，较为充分。
- **公平性**：所有基线在同一代码框架、相同局部窗口大小、相同池化配置下测试。训练数据不涉及测试集（ShareGPT独立），且方法可无缝集成到开源模型。

## 6. 主要结论与发现

1. **Judge Q在几乎所有任务和预算下均优于现有方法**：在LongBench上平均提升约1分，在RULER上提升超过3分（最高近10分），在Needle-in-a-Haystack上显著优于基线。
2. **低预算下优势更明显**：当KV Cache预算很小时（如128），Judge Q的性能退化远小于基线，说明其能更有效地保留关键信息。
3. **关键KV命中率高**：Judge Q与理论最优（使用真实解码token）的命中率比SnapKV高约8个百分点。
4. **对问题位置变化鲁棒**：当问题从末尾移到开头时，Judge Q的性能退化小于7%，而基线约10%。
5. **训练数据质量重要**：使用模型自身生成的响应（而非数据集原始响应）效果更好，且内容覆盖全面的ShareGPT优于单一分布的数据（如Wikipedia或GitHub）。
6. **软令牌数量32为最优平衡点**：在训练成本与泛化性能之间达到良好折中。

## 7. 优点

- **方法创新性**：针对现有方法局部信息偏置的缺陷，提出可训练软令牌来近似真实解码token的注意力分布，本质上是一种“学习如何查询”的范式，而非手工设计规则。
- **轻量级训练**：仅微调嵌入层（极少量参数），模型其余部分冻结，训练成本极低，可轻松适配任何开源模型。
- **易于集成**：无需修改模型架构，仅需在预填充阶段额外处理软令牌，推理时移除，对推理延迟影响小。
- **理论支撑明确**：通过对比理论最优（真实解码token），验证了方法的有效性，且引入关键KV命中率这一合理指标。
- **实验全面**：覆盖多种模型、多种基准、多种预算，并做了充分消融，结论可信。

## 8. 不足与局限

- **训练数据依赖**：方法依赖于高质量的、包含prompt-response对的训练数据（ShareGPT），且需要模型自身生成响应（增加一次前向推理）。对于无法获取此类数据的场景（如闭源模型或私有数据），可能难以应用。
- **软令牌数量固定**：论文实验仅探索了32个软令牌的效果，未充分讨论在不同任务或不同序列长度下是否需要自适应调整数量。
- **流式KV Cache驱逐未实现**：当前方法仅在预填充阶段进行一次驱逐，未实现解码过程中的动态驱逐。论文在“未来工作”中提及此点。
- **算力细节不透明**：未报告训练所需的GPU型号、时间、显存消耗，难以复现或估计部署成本。
- **实验偏差风险**：虽然对比基线按相同设置实施，但训练数据仅来自ShareGPT（对话类），对于其他领域（如代码、学术）的泛化能力可能有限。另外，在Needle-in-a-Haystack上未提供数值结果（仅可视化），不够量化。
- **文本延续任务实验较简单**：在MATH-500/AIME24上仅与SnapKV对比，且预算固定（25%输出长度），未与其他基线横向比较。

（完）
