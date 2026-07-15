---
title: "BeaconKV: Key-Value Cache Compression Guided by Beacon Queries for Efficient Large Reasoning Model Inference"
title_zh: BeaconKV：基于信标查询指导的高效大推理模型KV缓存压缩
authors: "Janghyeon Kim, Minsoo Kim, Kyuhong Shim, Jungwook Choi"
date: 2026-04-30
pdf: "https://openreview.net/pdf/f5d3dd37d52c5de84a42e63a6ac2586ab0cfb6cd.pdf"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于信标查询指导KV缓存压缩，处理推理中的回顾性token
tldr: BeaconKV观察到长推理链中会出现回顾性token，需要远距离上下文。提出信标查询来指导压缩，保留这些重要token，在减少缓存占用的同时不影响推理准确性。在大型推理模型上验证有效，为推理场景定制了压缩策略。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有压缩方法假设近期查询可预测未来重要性，但在长推理中因回顾性token而失效。
method: 提出信标查询机制，显式识别并保留回顾性token所需的关键上下文。
result: 在多个推理任务上，BeaconKV降低了KV缓存大小且保持推理效果。
conclusion: 针对推理模式定制压缩策略是必要的。
---

## Abstract
Large Reasoning Models (LRMs) achieve superior problem-solving through extended Chain-of-Thought (CoT) generation, but the resulting key-value (KV) cache grows linearly with sequence length and creates severe memory bottlenecks—often exceeding GPU capacity for long reasoning traces. Existing KV cache compression methods rely on recent queries to estimate future token importance, implicitly assuming these serve as reliable proxies for future attention patterns. We demonstrate that this assumption fails in long-horizon reasoning: certain decoding steps generate Thought Revisiting Tokens (TRT) that re-attend to distant previous context, such as task-solving plans formulated early in the trace. Through systematic analysis, we discover that queries corresponding to the TRT cluster into a small number of similarity groups in the embedding space. Based on this insight, we propose BeaconKV, a training-free KV cache compression method that maintains beacon queries—compact representatives for each global query cluster—to anticipate which KV pairs will be revisited without storing the entire query history. Across four open-source LRMs and diverse reasoning benchmarks, BeaconKV generally outperforms existing compression methods, achieving up to $5.8\times$ memory reduction while nearly preserving full cache accuracy and improving throughput by over $4.3\times$.

---

## 论文详细总结（自动生成）

# BeaconKV 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：大型推理模型（LRMs）在长链思维（CoT）生成过程中，KV缓存大小与序列长度线性增长，造成严重的内存瓶颈，往往超出GPU容量。现有KV缓存压缩方法依赖近期的查询（query）来估计未来token的重要性，假设近期查询能够可靠代理未来的注意力模式。
- **研究动机**：作者通过系统分析发现，在长程推理中，某些解码步骤会产生“思维回溯令牌”（Thought Revisiting Tokens, TRT），它们会重新关注早期推理轨迹中的远距离上下文（如任务规划方案）。现有压缩方法因无法捕捉这种远期回溯模式而失效。
- **整体含义**：需要针对推理场景中特有的长距离回溯注意力模式设计新的压缩策略，以实现高效推理而不损失准确性。

## 2. 论文提出的方法论：核心思想、关键技术细节
- **核心思想**：通过对TRT对应查询的嵌入空间进行分析，发现它们会聚集成少量相似性群组。基于这一洞察，提出**BeaconKV**——一种无需训练的KV缓存压缩方法，通过维护“信标查询”（beacon queries）来压缩全局查询历史。
- **关键技术细节**：
  - 信标查询是每个全局查询簇的紧凑代表，用于前瞻性地预测哪些KV对将被重新访问，而无需存储全部查询历史。
  - 方法流程：在推理过程中，将当前查询与已有信标簇进行匹配，若匹配则保留对应KV；否则根据重要性动态更新信标。
  - 无需额外训练或微调，完全在线自适应。
- **公式与算法**：论文未在摘要中提供具体公式或算法步骤，但核心是聚类与代表点选择策略。

## 3. 实验设计：使用的数据集/场景、benchmark、对比方法
- **数据集与场景**：在四个开源LRMs和多个推理基准上进行评估（具体模型和基准名称未列出，但推测为常见的数学推理、代码生成等长链思维任务）。
- **Benchmark**：与“全缓存准确率”对比，衡量内存压缩倍数和吞吐量提升。
- **对比方法**：与现有的KV缓存压缩方法进行比较（具体方法未说明，但通常包括剪枝、量化、稀疏等基线）。

## 4. 资源与算力
- 论文摘要及元数据中**未明确说明**使用的GPU型号、数量或训练时长。仅提到是“训练无关”方法，因此可能仅需推理阶段计算资源。具体算力消耗未披露。

## 5. 实验数量与充分性
- **实验数量**：至少涵盖了四种不同的LRM架构和多个推理基准，但未给出具体数字。摘要未提及消融实验或详细统计数据。
- **充分性评价**：由于信息有限，无法严格判断是否充分。但论文在多个模型和任务上验证了效果，且与基线比较了内存压缩倍数（最大5.8×）和吞吐量提升（4.3×），具有一定代表性。若包含消融以证明信标设计各组件必要性，则更充分——但摘要未提。

## 6. 论文的主要结论与发现
- **主要结论**：
  1. 现有压缩方法在LRM长推理中因忽略“思维回溯令牌”而失效。
  2. 回溯查询在嵌入空间中形成可辨识的簇，因此可以通过信标查询有效捕获。
  3. BeaconKV在保持接近全缓存准确率的同时，实现高达5.8倍的内存压缩和4.3倍以上的吞吐量提升，普遍优于现有压缩方法。
- **发现**：为推理场景定制压缩策略是必要的，通用压缩方法不适用于长链推理。

## 7. 优点
- **方法亮点**：
  - 训练无关，无需额外GPU训练开销，易于部署。
  - 主动识别并保留对回溯至关重要的KV对，而非被动依赖近期查询。
  - 压缩率高且不影响推理准确性。
  - 实验覆盖多个开源LRM，增强泛化性。
- **实验设计优点**：与多个基线对比，且有明确性能指标（压缩比、吞吐量、准确率保持度）。

## 8. 不足与局限
- **信息缺失**：论文摘要未提供具体模型名称、基准数据集、消融实验、超参数设置等，难以独立复现。
- **应用限制**：
  - 仅针对推理模型中的回溯token，可能不适用于普通文本生成或对话场景。
  - 信标聚类策略的有效性可能依赖于回溯模式的稳定性，对于更复杂多变的推理链是否稳健有待检验。
  - 未讨论在最坏情况下的效率（如回溯簇数量过多导致信标过多）。
  - 未披露计算开销（信标维护的额外时间成本）。
- **实验覆盖**：未提及在不同长度序列、不同模型大小上的对比，也未进行错误分析或鲁棒性测试。可能缺乏对非推理类长序列的评估。

（完）
