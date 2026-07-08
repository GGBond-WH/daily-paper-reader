---
title: "Q Cache: Visual Attention Is Valuable in Less than Half of Decode Layers for Multimodal Large Language Model"
title_zh: "Q Cache: 多模态大语言模型中视觉注意力在不到一半解码层中有价值"
authors: "Jiedong Zhuang, Lu Lu, Ming Dai, Rui Hu, Jian Chen, Qiang Liu, Haoji Hu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38414/42376"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 通过识别有价值层减少KV缓存
tldr: Q Cache针对多模态大语言模型中视觉token导致KV缓存瓶颈的问题，深入分析注意力机制发现大部分解码层的视觉注意力价值低。据此提出方法仅保留少数有价值层的KV缓存，显著降低内存占用且不损害长文本生成。为多模态模型KV缓存优化提供了新视角。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 多模态大语言模型视觉token冗余导致KV缓存瓶颈。
method: 通过分析注意力模式识别出有价值的解码层，仅保留这些层的KV缓存。
result: 大幅减少KV缓存占用，不影响长文本生成质量。
conclusion: 利用注意力价值差异可高效压缩多模态模型的KV缓存。
---

## Abstract
Multimodal large language models (MLLMs) are plagued by exorbitant inference costs attributable to the profusion of visual tokens within the vision encoder. The redundant visual tokens engenders a substantial computational load and key-value (KV) cache footprint bottleneck. Existing approaches focus on token-wise optimization, leveraging diverse intricate token pruning techniques to eliminate non-crucial visual tokens. Nevertheless, these methods often unavoidably undermine the integrity of the KV cache, resulting in failures in long-text generation tasks. To this end, we conduct an in-depth investigation towards the attention mechanism of the model from a new perspective, and discern that attention within more than half of all decode layers are semantic similar. Upon this finding, we contend that the attention in certain layers can be streamlined by inheriting the attention from their preceding layers. Consequently, we propose Lazy Attention, an efficient attention mechanism that enables cross-layer sharing of similar attention patterns.
It ingeniously reduces layer-wise redundant computation in attention. In Lazy Attention, we develop a novel layer-shared cache, Q Cache, tailored for MLLMs, which facilitates the reuse of queries across adjacent layers. In particular, Q Cache is lightweight and fully compatible with existing inference frameworks, including Flash Attention and KV cache. Additionally, our method is highly flexible as it is orthogonal to existing token-wise techniques and can be deployed independently or combined with token pruning approaches. Empirical evaluations on multiple benchmarks demonstrate that our method can reduce KV cache usage by over 35% and achieve 1.5x throughput improvement, while sacrificing only approximately 1% of performance on various MLLMs. Compared with SOTA token-wise methods, our technique achieves superior accuracy preservation.

---

## 论文详细总结（自动生成）

## 论文详细中文总结

### 1. 核心问题与整体含义（研究动机和背景）

- **核心问题**：多模态大语言模型（MLLMs）在推理过程中，由于视觉编码器生成大量视觉 token，导致注意力计算复杂度呈二次增长，并且 KV cache 的内存占用极大，成为推理效率的瓶颈。现有基于 token 剪枝的方法虽然能在预填充阶段减少冗余 token，但会破坏 KV cache 的完整性，在长文本生成（如图像描述）任务中效果不佳。
- **整体含义**：本文从层间注意力冗余的新视角出发，发现 MLLM 中超过一半的解码层的注意力模式高度相似（甚至重复），因此提出通过跨层共享注意力来减少冗余计算和 KV cache 开销，而无需丢弃任何视觉 token，从而在保持 KV cache 完整性的同时实现高效推理。

### 2. 方法论：核心思想、关键技术细节

- **核心思想**：将具有高度注意力相似性的相邻解码层划分为“懒块”（Lazy Block），块内除了第一层外，后续的“懒注意力层”（Lazy Attention Layer）直接复用前一层的查询（Q）和部分键（K），避免重复计算。为此设计了一个轻量级的层间共享缓存——**Q Cache**，用于在块内传递查询。
- **关键技术细节**：
  1. **注意力相似性度量**：使用 Jensen–Shannon (JS) 散度计算相邻层最后一行的注意力分布（最后一个 token 的注意力）的相似度。通过设定阈值 ε 将连续相似层聚合为一个 Lazy Block。
  2. **Lazy Attention 的两种模式**：
     - **全局懒注意力（GLA）**：块内所有层共享全部 Q 和 K，仅独立计算 V。
     - **视觉懒注意力（VLA）**：仅共享视觉 token 的 Q 和 K，文本 token 的 Q/K 仍独立计算。理由是视觉 token 数量远多于文本 token，且视觉注意力在层间相似性更高，而文本注意力（尤其无意义 token）变化较大。
  3. **与现有框架兼容**：Q Cache 与 KV Cache、Flash Attention 完全兼容，无需额外模块，可即插即用。同时与 token 剪枝方法正交，可联合部署。
- **公式与算法流程**（文字说明）：
  - 预填充阶段：块内首层正常计算 Q, K, V 并存入 Q Cache 和 K Cache；后续懒注意力层直接从 Q Cache 读取 Q，从 K Cache 读取 K，仅计算 V。
  - 解码阶段：首层更新 Q 和 K；后续懒注意力层复用首层的 Q 和 K，仅更新 V（对于 VLA，只复用视觉部分的 Q 和 K，文本部分独立计算）。

### 3. 实验设计

- **使用数据集/场景**：涵盖多项视觉语言任务：
  - 视觉问答：GQA、VQA v2、TextVQA、VizWiz、AI2D、ScienceQA-IMG、MMMU、MMBench、POPE
  - 图像描述：COCO Captions、NoCaps、Flickr30K
- **Benchmark**：以 LLaVA-v1.5-7B/13B、LLaVA-NEXT-7B/13B 为基座模型，在 12 个数据集上评估准确率（如 GQA、VQA、TextVQA 等）和效率指标（FLOPs、延迟、KV cache 大小、吞吐量）。
- **对比方法**：
  - 现有 token 剪枝方法：FastV (ECCV24)、VTW (AAAI25)、HiRED (AAAI25)、PruMerge+ (ICCV25)、SparseVLM (ICML25)
  - 自身消融：GLA vs VLA；随机分组 vs 相似性分组；与 FastV 联合使用；在纯 LLM（Vicuna）上直接进行懒指令微调

### 4. 资源与算力

- 论文**未明确说明**实验所用的 GPU 型号、数量及训练时长。仅提到效率测试在单张 80GB A100 GPU 上进行（用于测量吞吐量和最大 batch size）。由于本方法主要在已训练好的模型上应用（后训练或微调），训练算力需求较低，但具体数值未给出。

### 5. 实验数量与充分性

- **实验数量**：较为充分。核心结果表（Table 1）在两种模型尺寸（7B/13B）和两个版本（v1.5/NEXT）上对比了 6 种基线方法，覆盖 12 个数据集；另有联合兼容性实验（Table 2）、效率测试（Table 3、Figure 5）、随机分组对比（Figure 6）、注意力量化分析（Figure 7、8）以及懒指令微调对比（Table 4）。
- **充分性评估**：实验设计较全面，既包含性能对比也包含效率分析，还进行了消融和可解释性分析。但存在以下不足：
  - 仅使用 LLaVA 系列模型，未在更多 MLLM（如 Qwen-VL、InternVL）上验证泛化性。
  - 未考察在更长上下文（如视频理解）或更复杂的多模态场景下的表现。
  - 未报告多次运行的方差或统计显著性检验。

### 6. 论文的主要结论与发现

- **主要发现**：超过 50% 的解码层注意力模式高度相似（JS 散度接近 0），且该现象在 LLM 中已存在，MLLM 继承自其基座 LLM。
- **方法有效性**：VLA 模式可在仅损失约 1% 性能的前提下，减少 35%+ 的 KV cache 内存，实现 1.5× 吞吐提升。显著优于现有 token 剪枝方法（尤其在图像描述任务上避免长文本生成失败）。
- **GLA vs VLA**：GLA 性能下降更明显（尤其有意义的文本注意力变化大），VLA 更稳定且与光速注意力兼容更好。
- **与 token 剪枝正交**：结合 FastV 仅额外带来 1–2% 的精度损失，表明层间冗余缩减与 token 级剪枝可叠加使用。

### 7. 优点

- **视角新颖**：从层间（而非 token 级）冗余入手，提出跨层共享注意力，避免 KV cache 完整性受损，适合长文本生成。
- **轻量兼容**：Q Cache 无需额外参数，与 Flash Attention、KV Cache 等主流框架无缝集成，即插即用。
- **正交灵活**：可独立使用或与现有 token 剪枝/量化方法结合，形成混合加速方案。
- **效率显著**：KV cache 减少超过三分之一，吞吐提升达 1.5×，且性能损失极小。
- **深入分析**：对注意力的层间相似性进行量化（JS 散度），并分解视觉/文本注意力，解释 GLA 与 VLA 差异的原因，提供可解释性。

### 8. 不足与局限

- **模型覆盖不足**：仅测试了 LLaVA 系列，未在 Qwen-VL、InternVL、LLaMA-Adapter 等主流 MLLM 上验证，泛化性存疑。
- **上下文长度限制**：实验输入长度固定为 2048，未测试更长序列（如视频帧或高密度图文混合）下的表现，可能无法反映极端资源场景。
- **缺乏统计显著性**：未报告多次运行结果的标准差或置信区间，难以判断性能差异的稳定性。
- **仅评估后训练/指令微调**：未探索在预训练阶段直接添加 Q Cache 的影响，也未评估对模型收敛性的影响。
- **内存减少比理想化**：论文计算 KV cache 节省率时假设文本 token 可忽略（VLA 中文本仍保留 K/V），实际节省率低于 n/2N 理想值（因 W_i 等固定开销未计入）。

（完）
