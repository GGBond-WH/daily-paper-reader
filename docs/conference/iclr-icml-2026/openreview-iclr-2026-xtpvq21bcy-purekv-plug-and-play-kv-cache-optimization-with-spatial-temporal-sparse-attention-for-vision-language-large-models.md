---
title: "PureKV: Plug-and-Play KV Cache Optimization with Spatial-Temporal Sparse Attention for Vision-Language Large Models"
title_zh: PureKV：面向视觉语言大模型的即插即用KV缓存优化与时空稀疏注意力
authors: "Zhonghua Jiang, Kunxi Li, Yiyun Zhou, Sihao Liu, Zhaode Wang, chengfei lv, Shengyu Zhang"
date: 2025-09-20
pdf: "https://openreview.net/pdf?id=XtpVQ21bcY"
tags: ["query:llm-kv-cache"]
score: 8.0
evidence: 利用时空稀疏注意力优化视觉语言大模型的KV缓存
tldr: 视觉语言大模型处理高分辨率输入时，KV缓存增长和注意力二次复杂度成为瓶颈。PureKV提出即插即用的KV缓存优化方法，利用时空稀疏性，与FlashAttention等高效注意力机制兼容。该方法在不显式计算注意力矩阵的情况下压缩KV缓存，适用于多模态推理。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有KV压缩依赖注意力分数，与FlashAttention等高效机制不兼容。
method: 利用时空稀疏注意力，在不计算完整注意力矩阵的情况下直接压缩KV缓存。
result: 在视觉语言任务中，PureKV在保持性能的同时显著降低内存和延迟。
conclusion: PureKV为多模态大模型提供了一种通用、兼容的KV缓存优化方案。
---

## Abstract
Vision-Language Large Models (VLLMs) faces significant efficiency challenges when processing high-resolution inputs. The quadratic complexity in attention and autoregressive generation, as well as the constantly growing key value (KV) cache size, severely hinder the prefilling and decoding stages. Recent efforts have attempted to compress KV cache by identifying and pruning KV cache of less important tokens, but these methods typically rely on attention scores to estimate token importance, making them incompatible with efficient attention mechanisms such as FlashAttention and Sparse Attention, which do not explicitly compute attention matrices. Moreover, existing methods overlook how sparse attention, while accelerating the prefilling stage, alters the information structure of the KV cache—thereby compromising the effectiveness of downstream KV cache compression strategies. To address this issue, we propose PureKV, a plug-and-play framework for joint optimization of sparse attention and KV cache compression. We first introduce a KV cache compression strategy that is fully compatible with efficient attention accelerators. Our method utilizes lower layer attention scores to estimate the importance of high layers' KV cache, enabling active pruning without compromising accuracy. In addition, we have designed a Spatial-Temporal Sparse Attention (ST-SpAttn) module specifically tailored for video KV cache compression algorithms. This module combines spatial and temporal attention sparsity to improve the compression efficiency of KV cache optimization algorithms by purifying spatial noise and temporal redundancy in KV cache. At the same time, ST-SpAttn also accelerated the prefilling stage of VLLMs. Extensive experiments on VLLMs (VideoLLaMA2, Qwen2.5-VL) have shown that PureKV achieves 5.0 × KV cache compression and 3.16 × prefill acceleration, with negligible quality degradation. By seamlessly integrating with sparse attention optimization, our work unlocks scalable deployments for real-time multimodal applications.

---

## 论文详细总结（自动生成）

# PureKV：面向视觉语言大模型的即插即用KV缓存优化与时空稀疏注意力

## 1. 核心问题与整体含义（研究动机和背景）

- **核心问题**：视觉语言大模型（VLLMs）在处理高分辨率输入时，面临两大效率瓶颈：注意力机制的二次复杂度（O(n²)）以及自回归生成中KV缓存（Key-Value Cache）不断增长，导致预填充和解码阶段计算与内存开销巨大。
- **现有方法的不足**：主流KV缓存压缩方法依赖注意力分数来估计标记重要性，但这类方法与FlashAttention、稀疏注意力等高效注意力机制不兼容——这些机制不显式计算完整的注意力矩阵。此外，现有方法忽略了稀疏注意力在加速预填充阶段的同时会改变KV缓存的信息结构，从而破坏后续KV缓存压缩策略的有效性。
- **整体含义**：需要一种即插即用的框架，能够联合优化稀疏注意力与KV缓存压缩，在不牺牲性能的前提下，实现高效的多模态推理，推动实时多模态应用的规模化部署。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：提出PureKV，一个即插即用框架，通过**利用低层注意力分数估计高层KV缓存的重要性**，实现与高效注意力加速器（如FlashAttention）完全兼容的主动KV缓存剪枝；同时设计**时空稀疏注意力（ST-SpAttn）模块**，结合空间和时间稀疏性，净化视频KV缓存中的空间噪声和时间冗余，提升压缩效率并加速预填充阶段。
- **关键技术细节**：
  1. **兼容高效注意力的KV缓存压缩策略**：不直接计算注意力矩阵，而是利用较低层（例如transformer早期层）的注意力分数作为代理，来估计高层KV缓存中每个标记的重要性。这些低层分数已包含足够的语义信息，通过传递即可指导剪枝，从而避免对FlashAttention等机制的修改。
  2. **时空稀疏注意力（ST-SpAttn）模块**：专为视频KV缓存压缩设计。空间维度：对图像/帧内的空间区域进行稀疏采样，减少冗余空间标记；时间维度：对视频帧序列进行稀疏化，去除时间上的高度相似或冗余帧。两者结合，在保留关键信息的同时显著缩小KV缓存大小。
  3. **整体流程**：PureKV作为插件式模块，嵌入现有VLLM架构。预填充时，ST-SpAttn先对输入进行时空稀疏化，然后使用低层注意力分数对剩余KV缓存进行重要性排序并剪枝；解码时，直接利用剪枝后的稀疏KV缓存进行高效自回归生成。
- **兼容性**：方法不依赖于显式注意力矩阵，可与FlashAttention、Sparse Attention等高效机制无缝集成。

## 3. 实验设计：数据集、基准与对比方法

- **使用的模型**：VideoLLaMA2、Qwen2.5-VL。
- **评估场景**：视觉语言任务（具体数据集未在摘要中列出，但涉及视频理解、图像描述等多模态任务）。
- **对比方法**：未明确列出具体名称，但应与现有基于注意力分数的KV缓存压缩方法（如Key-Value pruning based on attention scores）以及无压缩的基线进行对比。
- **Benchmark**：未提及标准benchmark名称，推测使用了常见的视频问答、图像描述等数据集（如ActivityNet、MSVD、MSRVTT等常见视频理解数据集，以及COCO等图像数据集）。

## 4. 资源与算力

- **摘要及元数据中未明确说明使用的GPU型号、数量、训练时长等具体算力信息**。
- 仅提及实验在VLLMs（VideoLLaMA2, Qwen2.5-VL）上进行，具体硬件配置未知。

## 5. 实验数量与充分性

- **实验组数**：摘要中报告了**5.0倍KV缓存压缩**和**3.16倍预填充加速**，以及“negligible quality degradation”（质量下降可忽略）。未提及消融实验的具体数量，但表明进行了多模型、多场景的评估。
- **充分性评价**：实验覆盖了两个代表性VLLM（视频和图像模态），但未提供详细的数据集列表、与多种基线方法的全面对比、以及在不同压缩率下的性能曲线。消融实验可能仅涉及ST-SpAttn有无的对比，缺乏对低层注意力分数代理策略效果的系统验证。实验设计基本合理，但提供的细节较少，需论文全文补充以判断公平性和客观性。

## 6. 论文的主要结论与发现

- PureKV在保持极低性能损失的前提下，实现了**5.0倍KV缓存压缩**和**3.16倍预填充加速**。
- 方法**与FlashAttention、Sparse Attention等高效注意力机制完全兼容**，克服了现有方法在兼容性上的根本缺陷。
- ST-SpAttn模块有效净化了视频KV缓存中的空间噪声和时间冗余，提升了压缩效率，同时加速了预填充阶段。
- 工作为实时多模态应用中的可扩展部署提供了可行方案。

## 7. 优点：方法或实验设计上的亮点

- **即插即用的兼容性**：核心亮点在于无需修改高效注意力机制（如FlashAttention）即可实现KV缓存压缩，极大提升了实用性和通用性。
- **新颖的代理注意力策略**：利用低层注意力分数估计高层重要性，避免了显式计算注意力矩阵，巧妙解决了兼容性问题。
- **时空联合稀疏化**：针对视频模态，同时考虑空间和时间冗余，设计专门模块，体现了对多模态数据特性的深入理解。
- **联合优化框架**：将稀疏注意力加速与KV缓存压缩统一在一个框架内，避免了两者的互相干扰，具有系统级创新。

## 8. 不足与局限

- **实验细节缺失**：未提供具体数据集、对比方法、评估指标（如准确率、BLEU、CIDEr分数）的数值，难以量化“negligible quality degradation”的程度。
- **算力与资源未报告**：缺少GPU型号、数量、训练/推理时间等硬性指标，影响可复现性评估。
- **消融实验不明确**：未详细说明消融实验设计（如单独去掉低层注意力代理、去掉ST-SpAttn等），无法判断各组件贡献。
- **泛化性验证不足**：仅在两个VLLM上测试，未在更多架构（如LLaVA、InternVL）或纯语言模型上验证，可能对视觉模态的任务偏重。
- **潜在偏差风险**：代理注意力分数的有效性可能依赖于模型层间相关性，若模型结构不同（如深层与浅层语义差异大），方法可能失效。
- **应用限制**：主要面向视频和图像多模态任务，对于纯文本或音频等其他模态的适用性未讨论。

---

（完）
