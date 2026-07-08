---
title: "AMS-KV: Adaptive KV Caching in Multi-Scale Visual Autoregressive Transformers"
title_zh: "AMS-KV: 多尺度视觉自回归Transformer中的自适应KV缓存"
authors: "Boxun Xu, Yu Wang, Zihu Wang, Peng Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39936/43897"
tags: ["query:llm-kv-cache"]
score: 7.0
evidence: 多尺度视觉自回归Transformer自适应KV缓存
tldr: 视觉自回归模型通过下一尺度预测生成图像，但KV缓存随尺度增加急剧膨胀。本文提出AMS-KV，分析发现局部尺度注意力对质量贡献大，粗尺度可少量分配内存。据此设计自适应KV缓存策略，为不同尺度分配差异化缓存容量。实验表明在保持生成质量的同时，KV内存占用大幅降低。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 视觉自回归模型多尺度生成导致KV缓存内存过度增长，限制可扩展性。
method: 基于注意力头功能分类，为局部尺度分配更多缓存，粗尺度使用压缩缓存。
result: 在保持生成质量前提下显著减少KV缓存内存占用。
conclusion: 为视觉自回归模型提供了高效的KV缓存管理方案，可推广至类似多尺度架构。
---

## Abstract
Visual autoregressive modeling (VAR) via next-scale prediction has emerged as a scalable image generation paradigm. While Key and Value (KV) caching in large language models (LLMs) has been extensively studied, next-scale prediction presents unique challenges, and KV caching design for next-scale based VAR transformers remains largely unexplored. A major bottleneck is the excessive KV memory growth with the increasing number of scales—severely limiting scalability. Our systematic investigation reveals that: (1) Attending to tokens from local scales significantly contributes to generation quality (2) Allocating a small amount of memory for the coarsest scales, termed as condensed scales, stabilizes multi-scale image generation (3) Strong KV similarity across finer scales is predominantly observed in cache-efficient layers, whereas cache-demanding layers exhibit weaker inter-scale similarity. Based on the observations, we introduce AMS-KV, a scale-adaptive KV caching policy for next-scale prediction in VAR models. AMS-KV prioritizes storing KVs from condensed and local scales, preserving the most relevant tokens to maintain generation quality. It further optimizes KV cache utilization and computational efficiency identifying cache-demanding layers through inter-scale similarity analysis. Compared to the vanilla next-scale prediction-based VAR models, AMS-KV reduces KV cache usage by up to 84.83% and self-attention latency by 60.48%. Moreover, when the baseline VAR-d30 model encounters out-of-memory failures at a batch size of 128, AMS-KV enables stable scaling to a batch size of 256 with improved throughput.

---

## 论文详细总结（自动生成）

# 论文结构化总结：AMS-KV: 自适应多尺度视觉自回归Transformer中的KV缓存

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：视觉自回归模型（VAR）通过“下一尺度预测”（next-scale prediction）实现高质量图像生成，但多尺度生成导致KV缓存（Key-Value cache）随尺度数量急剧增长。例如，VAR-d30模型在256×256分辨率下，默认KV缓存占用22.41GB；VAR-d36在512×512分辨率下甚至达到117GB，超出单GPU内存。这种内存瓶颈严重限制了模型的批量处理和可扩展性，阻碍了实际部署。
- **整体含义**：本文旨在解决VAR模型中的KV缓存效率问题，通过自适应策略减少内存占用，同时保持生成质量，从而提升AR视觉模型的可扩展性和实际应用能力。

## 2. 方法论：核心思想、关键技术细节

- **核心思想**：基于两项关键观察设计自适应缓存策略：
  1. **尺度重要性不均**：Condensed scales（最粗的两个尺度，如1×1和2×2）和Local scales（最近几个细尺度）对生成质量贡献最大，Intermediate scales（中间尺度）冗余较多。
  2. **层层缓存需求差异**：低层（早期）缓存需求高（cache-demanding layers），高层（后期）缓存效率高（cache-efficient layers）。通过相邻尺度键的相似性（负ℓ2距离）可区分两类层：相似性低的层需要更大缓存。

- **关键技术细节**：
  - **AMS-KV缓存策略**：
    - 每个层独立管理缓存，包含三个超参数：`C_min`（缓存高效层容量）、`C_max`（缓存需求层容量）、`C_cds`（保留的condensed scales数量）。
    - 初始化时每层分配`C_min`容量。生成过程中，优先保留condensed scales（锁定不淘汰），动态添加local scales，淘汰中间尺度（采用改进的CLRU策略——Condensed Least Recently Used，即FIFO但保留condensed scales）。
    - 当当前层的缓存预算`C_bgt`（初始为`C_min`）用尽，且当前层被判定为cache-demanding层（相邻尺度键相似度低于阈值θ），则将预算扩展为`C_max`。
  - **相似度计算**：将第`i-1`尺度的键通过2D插值缩放到第`i`尺度的分辨率，计算负ℓ2距离。
  - **算法流程**：参见论文Algorithm 1，每条刻度依次处理；超过最大容量则跳过；当当前缓存大小超过预算时，触发扩展或淘汰机制。

## 3. 实验设计

- **数据集与场景**：
  - 无条件/条件图像生成：ImageNet-1K 256×256和512×512类条件生成。
  - 文本到图像任务：使用Infinity-2B模型在GenEval框架上评估。
- **Benchmark**：
  - 指标：FID（Fr´echet Inception Distance）、IS（Inception Score）、Precision、Recall。
  - 基线模型：VAR-d30、VAR-d36（来自Tian et al. 2024），以及Infinity-2B（Han et al. 2024）。
- **对比方法**：
  - 与三种LLM KV缓存压缩方法比较：SWA（滑动窗口注意力）、H2O（重击者丢弃）、STA（注意力沉溺）。均在固定75%压缩率下对比FID/IS。
  - 与两种缓存分配策略对比：均匀分配（S1）与相似性感知分配（S2）。
  - 与基线VAR模型直接对比KV缓存大小和生成质量。

## 4. 资源与算力

- 文中未明确说明训练时长或训练所用的GPU数量。仅提及：
  - 使用**单一NVIDIA A100-80G GPU**进行推理和KV缓存内存测量。
  - 实验环境：PyTorch CUDA profiling工具，FlashAttention-2作为默认注意力实现。
- 未报告训练阶段算力消耗（所有实验基于预训练模型进行推理优化）。

## 5. 实验数量与充分性

- **主要实验**：
  - **端到端对比**（表3）：VAR-d30和VAR-d36在不同分辨率、不同超参数设置下对比基线 vs AMS-KV，报告FID/IS/Precision/Recall及KV缓存缩小比例（71.29%~78.72%）。
  - **批量扩展实验**（表6）：从batch size 16到256，记录KV内存和吞吐量，展示AMS-KV支持4×批量（256）而基线在128时OOM。
  - **鲁棒性消融**（图7）：扫描`C_min`和`C_max`的多种组合，展示FID与注意力延迟、IS与KV内存的trade-off曲线。
  - **策略对比**（表2）：均匀分配（S1）vs 相似性感知分配（S2），证明S2在相同总缓存预算下FID更低（3.37 vs 6.65）。
  - **尺度重要性消融**（表1和图5）：分别移除Condensed/Local/Intermediate尺度，量化对FID/IS的影响，并展示生成图像质量退化。
  - **跨模型泛化**（表4）：在Infinity-2B文本到图像任务上验证，内存降低35.6%，吞吐量提升。
  - **与其他压缩方法对比**（表5）：在固定75%压缩时，AMS-KV的FID=2.09远优于SWA（11.61）、H2O（8.80）、STA（2.81）。
- **充分性评价**：实验数量丰富，覆盖多种模型、分辨率、任务、消融和对比，分析维度全面。所有对比均在相同总缓存预算下进行（表2、表5），保证了公平性。但缺少对其他VAE或不同架构VAR模型（如其他backbone）的测试，可能限制泛化结论。

## 6. 主要结论与发现

1. **内存节省显著**：AMS-KV在保持FID波动±0.02~+0.06的前提下，减少KV缓存用量高达84.83% (VAR-d30) 至71.29% (VAR-d36)，实现3.48×~4.70×压缩。
2. **延迟改善**：自注意力延迟降低60.48%。
3. **可扩展性增强**：支持batch size从128（基线OOM）扩展至256，吞吐量从不可用提升至23.88 img/s。
4. **策略有效性**：优先保留condensed scales和local scales、基于层间相似性动态调整容量，比均匀分配或简单丢弃策略（SWA/H2O/STA）质量高得多。
5. **鲁棒性**：多种缓存容量配置下均能维持高质量，展示出对超参数不敏感。

## 7. 优点

- **创新性**：首次系统研究VAR多尺度KV缓存冗余，提出scale-wise重要性和layer-wise缓存偏好，设计专用缓存策略。
- **高效实用**：无需重新训练或调参（tuning-free），可直接应用于预训练模型，且兼容FlashAttention。
- **全面验证**：跨模型（VAR-d30/36、Infinity-2B）、跨任务（无条件生成、条件生成、文本到图像）、多指标（FID/IS/Precision/Recall/内存/延迟/吞吐）充分评估。
- **消融细致**：通过移除不同尺度组、不同分配策略、不同组合容量，清晰揭示冗余本质和策略有效性。

## 8. 不足与局限

- **实验覆盖有限**：
  - 仅测试了VAR-d30和VAR-d36两个尺度的模型，未覆盖更小或更大变体（如VAR-d16）。
  - 缺少对其他视觉自回归模型（如LlamaGen、MaskGIT等）的泛化验证。
  - 文本到图像实验仅使用Infinity-2B，未与其他SOTA文本到图像VAR模型对比。
- **偏差风险**：
  - 评估仅基于ImageNet-1K，数据集单一，未覆盖其他复杂场景（如自然景观、人脸、医学图像）。
  - 所有实验使用A100-80G单一硬件，未验证在不同架构（如H100、4090）上的可复现性。
- **应用限制**：
  - 策略中阈值θ和容量C_min/C_max需人工设定，虽然鲁棒，但最佳值可能依赖模型和分辨率。
  - 未讨论推理时动态阈值调整或自动搜索方法。
  - CLRU策略简单（FIFO+锁定），可能无法充分适应长序列或动态重要性变化。
- **算力资源未报告**：训练阶段计算开销、预训练模型本身耗时等缺乏信息，限制了对整体效率的全面评估。

（完）
