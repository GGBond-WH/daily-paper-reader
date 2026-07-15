---
title: Reconstructing KV Caches with Cross-Layer Fusion for Enhanced Transformers
title_zh: 跨层融合重构KV缓存用于增强变换器
authors: "Hongzhan Lin, ZhiqiBai, Xinmiao Zhang, Siran Yang, Jiamang Wang, Yunlong Xu, Jiaheng Liu, Yongchi Zhao, Xiang Li, Yuchi Xu, Wenbo Su, Bo Zheng"
date: 2026-01-26
pdf: "https://openreview.net/pdf?id=4pivvEJiCl"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 跨层KV缓存融合，通过从较低层重构高层缓存来减少内存
tldr: FusedKV通过分析键和值的信息流，发现高层值主要来自底层，键来自底中层。据此学习融合底中层信息来重构高层KV缓存，在保持性能的同时降低内存占用。在长序列任务中优于跨层共享方法，证明了层间信息利用的有效性。
source: ICLR-2026-Accepted
selection_source: conference_retrieval
motivation: 跨层KV缓存共享性能通常低于层内方法，原因在于信息流分布不均衡。
method: 将底中层最有信息的键值进行可学习的融合以生成高层KV缓存。
result: 在多个基准上，FusedKV相比跨层共享方法提升了性能，同时降低了内存。
conclusion: 合理利用层间信息流可实现更高效的KV缓存压缩。
---

## Abstract
Transformer decoders have achieved strong results across tasks, but the memory required for the KV cache becomes prohibitive at long sequence lengths.
Although Cross-layer KV Cache sharing (e.g., YOCO, CLA) offers a path to mitigate KV Cache bottleneck, it typically underperforms within-layer methods like GQA.
To understand the root cause, we investigate the information flow of keys and values of the top-layers.
Our preliminary reveals a clear distribution: values are predominantly derived from the bottom layer, while keys draw more information from both bottom and middle layers.
Building upon this, we propose FusedKV, whose top-layer KV caches are a learnable fusion of the most informative ones from the bottom and middle layers. 
This fusion operates directly on post-RoPE keys, preserving relative positional information without the computational cost of re-applying rotary embeddings.
To further improve efficiency, we propose FusedKV-Lite, an cross-layer sharing approach, where top-layer KV caches are directly derived from the bottom-layer values and the middle-layer keys.
Compared to FusedKV, FusedKV-Lite reduces I/O overhead at the cost of a slight increase in perplexity.
In experiments on LLMs ranging from 332M to 4B parameters, our proposed method reduce 50\% cache memory while achieving lower validation perplexity than the standard Transformer decoder, establishing it as a memory-efficient, high-performance architectural alternative. We have made our Triton implementation available.

---

## 论文详细总结（自动生成）

好的，以下是对给定论文的详细中文总结。

---

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **研究动机**：Transformer 解码器在长序列任务中，KV 缓存（Key-Value Cache）的内存占用会急剧增长，成为性能瓶颈。
- **现有问题**：虽然跨层 KV 缓存共享（如 YOCO、CLA）被提出以减小内存，但其性能通常低于层内方法（如 GQA），原因尚未明确。
- **背景揭示**：作者通过分析高层（top-layers）键（keys）和值（values）的信息流发现：值主要由底层（bottom layer）信息构建，而键则同时依赖底层和中间层信息。
- **研究目标**：基于这一发现，设计一种新型跨层融合机制，在保持模型性能的同时大幅降低 KV 缓存的内存占用。

## 2. 论文提出的方法论

### 核心思想
- **FusedKV**：不再直接复制或简单共享底层缓存，而是**学习性地融合底层和中间层中最有信息量的 KV 缓存**，以重构高层的 KV 缓存。

### 关键技术细节
- **融合对象**：直接作用于**经过 RoPE（旋转位置编码）后的 keys**，从而保留相对位置信息，避免重新计算旋转嵌入带来的额外计算开销。
- **学习方式**：使用可学习的融合模块（具体结构文中未给出公式细节，推测为线性或轻量注意力），从底层和中间层提取关键的键值信息，合成高层的 key 和 value。
- **FusedKV-Lite（简化版）**：直接使用**底层 values** 和 **中间层 keys** 作为高层 KV 缓存，不进行额外学习，以减少 I/O 开销，但会略微增加困惑度。

### 算法流程（文字说明）
1. 输入序列经过前几层（底层+中间层）Transformer 计算，得到对应的 KV 缓存。
2. 对于高层 Transformer 层，不自行计算 KV 缓存，而是：
   - 从底层获取 value 信息（对于 key 则从底层和中间层获取）。
   - 通过可学习的融合模块（FusedKV）或直接选取（FusedKV-Lite）生成高层的 key 与 value。
3. 高层解码时直接使用融合生成的 KV 缓存，无需进一步计算。

## 3. 实验设计

- **数据集/场景**：文中仅提到在**长序列任务**上评估，未明确列举具体数据集名称（如 WikiText、LongBench 等）。
- **Benchmark**：以**验证困惑度（validation perplexity）**作为主要性能指标。
- **对比方法**：
  - 标准 Transformer 解码器（无 KV 缓存压缩）
  - 层内方法（如 GQA）
  - 跨层共享方法（如 YOCO、CLA）
- **模型规模**：从 332M 到 4B 参数的 LLM。

## 4. 资源与算力

- **文中未明确说明**使用的 GPU 型号、数量、训练时长以及推理时的具体硬件配置。
- **唯一提及的技术实现**：作者公开了 Triton 实现（一种 GPU 编程语言），但未提供算力细节。

## 5. 实验数量与充分性

- **实验数量**：从中提取的信息有限——主要涵盖不同模型规模（332M、1B？、4B）下的困惑度对比，以及消融实验（对比 FusedKV 与 FusedKV-Lite）。
- **充分性评判**：
  - **积极方面**：覆盖了多个参数量级，并比较了主流跨层共享方法和层内方法。
  - **不足**：缺乏在**具体下游任务（如生成、问答、摘要）**上的性能评估；未报告推理吞吐量或延迟；未在更大模型（>10B）上验证；未说明 GPU 内存节省的实际数值（仅说“减少50%缓存内存”，但未给出绝对数值或对比基线）。

## 6. 论文的主要结论与发现

- **核心发现**：高层 KV 缓存的信息来源存在不对称分布——值主要来自底层，键来自底层+中间层。
- **方法有效性**：FusedKV 在**减少 50% 缓存内存**的同时，能实现**低于标准 Transformer 的验证困惑度**，性能优于现有的跨层共享方法（如 YOCO、CLA）。
- **FusedKV-Lite 作为轻量替代**：通过牺牲微小困惑度换取更低的 I/O 开销，在效率敏感场景下具有吸引力。

## 7. 优点

- **理论洞察深刻**：通过信息流分析揭示了跨层 KV 共享性能不佳的根因，为设计提供了科学依据。
- **方法创新**：提出的可学习融合机制直接作用于 post-RoPE 键，避免重复计算，设计巧妙。
- **兼顾效率与性能**：同时提供了全精度版本（FusedKV）和轻量版本（FusedKV-Lite），满足不同需求。
- **开源实现**：提供了 Triton 实现，便于复现和扩展。
- **实验规模合理**：从 332M 到 4B 参数覆盖了多个主流模型大小。

## 8. 不足与局限

- **实验覆盖不完整**：
  - 未在长序列推理等具体任务（如 LongBench、RULER）上评估。
  - 未报告推理延迟（Latency）和吞吐量（Throughput）。
  - 未对大模型（>10B 参数）进行验证。
- **消融实验有限**：仅对比了 FusedKV 与 FusedKV-Lite，未深入分析不同融合层数、不同融合权重的影响。
- **偏差风险**：仅在困惑度上比较，未涉及生成质量（如 BLEU、ROUGE、人类评估），可能掩盖模型真实生成能力差异。
- **资源细节缺失**：未提供训练和推理所需的 GPU 数、时长，难以评估实际部署成本。
- **基线选择**：与 YOCO、CLA 比较，但未与最新的 KV 缓存量化或剪枝方法（如 KIVI、CacheGen）对比，可能不够全面。

---

（完）
