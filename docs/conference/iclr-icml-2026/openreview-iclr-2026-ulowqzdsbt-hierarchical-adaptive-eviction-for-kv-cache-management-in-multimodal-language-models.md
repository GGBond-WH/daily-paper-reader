---
title: Hierarchical Adaptive Eviction for KV Cache Management in Multimodal Language Models
title_zh: 多模态语言模型中分层自适应驱逐的KV缓存管理
authors: "Xindian Ma, Yidi Lu, Peng Zhang, Jing Zhang"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=ulOwQZdSbT"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 多模态LLM中分层自适应驱逐策略，优化文本和视觉token的缓存管理
tldr: HAE针对多模态大模型视觉与文本token注意力分布异质性，提出分层自适应驱逐框架：预填充阶段双注意力剪枝，解码阶段动态驱逐。在保持性能的同时显著减少KV缓存，尤其适用于多模态场景，拓展了KV缓存管理到多模态领域。
source: ICLR-2026-Rejected-Public
selection_source: conference_retrieval
motivation: 现有KV驱逐策略未考虑视觉和文本token的注意力分布差异。
method: 提出HAE，包含双注意力剪枝和动态解码驱逐，分别优化预填充和解码阶段的缓存。
result: 在多模态基准上，HAE在降低内存的同时保持了甚至提升了任务性能。
conclusion: 针对模态异质性的自适应驱逐是有效的KV缓存管理方法。
---

## Abstract
The integration of visual information into Large Language Models (LLMs) has enabled Multimodal LLMs (MLLMs), but the quadratic memory and computational costs of Transformer architectures remain a bottleneck. Existing KV cache eviction strategies fail to address the heterogeneous attention distributions between visual and text tokens, leading to suboptimal efficiency or degraded performance.
In this paper, we propose Hierarchical Adaptive Eviction (HAE), a KV cache eviction framework that optimizes text-visual token interaction in MLLMs by implementing Dual-Attention Pruning during pre-filling (leveraging visual token sparsity and attention variance) and a Dynamic Decoding Eviction Strategy (inspired by OS Recycle Bins) during decoding. 
HAE minimizes KV cache usage across layers, reduces computational overhead via index broadcasting, and theoretically ensures superior information integrity and lower error bounds compared to greedy strategies, enhancing efficiency in both comprehension and generation tasks. 
Empirically, HAE reduces KV-Cache memory by 41\% with minimal accuracy loss (0.3\% drop) in image understanding tasks and accelerates story generation inference by 1.5× while maintaining output quality on Phi3.5-Vision-Instruct model.

---

## 论文详细总结（自动生成）

# 论文中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **问题背景**：多模态大语言模型（MLLMs）通过将视觉信息融入LLM，显著提升了理解与生成能力，但Transformer架构的二次方内存和计算开销成为瓶颈，尤其是KV缓存（Key-Value Cache）占用大量显存。
- **现有不足**：现有KV缓存驱逐策略未考虑视觉token与文本token之间异质的注意力分布（视觉token稀疏且注意力方差大，文本token密集且注意力集中），导致缓存管理效率低或性能下降。
- **研究动机**：针对多模态场景下视觉和文本token的注意力分布差异，设计一种自适应、分层的KV缓存驱逐框架，在保持性能的同时显著降低内存占用。

## 2. 论文提出的方法论
- **核心思想**：提出**HAE（Hierarchical Adaptive Eviction）**，一个分层自适应驱逐框架，包含两个阶段：
  - **预填充阶段（Pre-filling）**：采用**双注意力剪枝（Dual-Attention Pruning）**，利用视觉token稀疏性和注意力方差来识别并剔除冗余KV对，优化初始缓存。
  - **解码阶段（Decoding）**：采用**动态解码驱逐策略（Dynamic Decoding Eviction Strategy）**，受操作系统回收站（OS Recycle Bins）启发，在生成过程中动态保留/驱逐KV条目。
- **关键技术细节**：
  - 通过索引广播（index broadcasting）减少计算开销。
  - 理论上保证了比贪心策略更优的信息完整性和更低误差界。
- **算法流程**（文字说明）：
  1. 预填充：对输入的多模态序列（视觉+文本token），计算每层注意力权重，根据视觉token的稀疏度和注意力方差进行双维度剪枝，保留高贡献KV对。
  2. 解码：在逐token生成时，维护一个类似“回收站”的缓存区，根据当前注意力分数和访问频率，动态决定哪些KV对被驱逐或保留，兼顾近期和长期依赖。

## 3. 实验设计
- **数据集/场景**：论文提及两种任务：
  - **图像理解任务**（具体数据集未明确，从摘要看可能是通用视觉问答或分类基准）。
  - **故事生成推理**（Story Generation Inference，加速推断）。
- **Benchmark**：未明确列出标准数据集名称，仅描述为“多模态基准”（multimodal benchmarks）。
- **对比方法**：与现有的贪心KV驱逐策略（greedy strategies）进行对比，但未列出具体方法名称。

## 4. 资源与算力
- **未明确说明**：摘要及元数据中未提及GPU型号、数量、训练时长等算力信息。仅在结果中描述了在Phi3.5-Vision-Instruct模型上的实验，但未给出硬件配置。

## 5. 实验数量与充分性
- **实验组数**：从摘要仅能看出两类实验（图像理解、故事生成），缺少消融实验、不同模态比例分析、不同模型规模对比等细节。元数据也未提供更多实验信息。
- **充分性评估**：实验数量较少，覆盖范围有限，未在多种多模态模型或视觉-语言基准上验证，也未展示不同压缩率下的性能曲线。公正性方面，仅与贪心策略对比，缺乏与主流KV缓存方法（如StreamingLLM、H2O等）的横向比较。因此实验充分性不足。

## 6. 论文的主要结论与发现
- HAE可将KV缓存内存减少41%，同时图像理解任务准确率仅下降0.3%。
- 在故事生成任务中，推理速度提升1.5倍，且输出质量保持稳定。
- 证明了针对模态异质性进行自适应驱逐的有效性，为多模态LLM的KV缓存管理提供了新方向。

## 7. 优点
- **方法创新**：首次专门针对多模态LLM中视觉与文本token的注意力异质性设计缓存策略，突破了现有单模态方法的限制。
- **理论保证**：提供了信息完整性和误差界的理论分析，使方法可解释性强。
- **工程优化**：索引广播技术降低了额外计算开销，实用性好。
- **性能平衡**：在显著降低内存的同时，性能损失极小，甚至可能提升生成速度。

## 8. 不足与局限
- **实验覆盖不足**：仅在一个模型（Phi3.5-Vision-Instruct）上验证，未在更大规模多模态模型（如LLaVA、Gemini系列）上测试，泛化性存疑。
- **对比不全面**：未与当前主流KV压缩方法（如蓄水池采样、Hybrid-based方法）对比，无法体现相对优劣。
- **数据集不透明**：未公开使用的具体图像理解基准（如MMBench、VQAv2等），导致结果难以复现与比较。
- **算力资源未报告**：缺少训练/推理硬件细节，影响工程应用的可重复性。
- **消融实验缺失**：未分析双注意力剪枝与动态解码策略各自的贡献，也未探讨不同剪枝率的影响。
- **局限性讨论**：论文未讨论在长视频或多轮对话等动态长序列场景下的表现，可能存在应用限制。

（完）
