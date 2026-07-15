---
title: "Bottlenecked Transformers: Periodic KV Cache Consolidation for Generalised Reasoning"
title_zh: 瓶颈变换器：用于广义推理的周期KV缓存合并
authors: "Adnan Oomerjee, Zafeirios Fountas, Haitham Bou Ammar, Jun Wang"
date: 2026-01-26
pdf: "https://openreview.net/pdf?id=fWgKnl4itC"
tags: ["query:llm-kv-cache"]
score: 8.0
evidence: 通过周期KV缓存合并优化推理，用于广义推理
tldr: 该论文受大脑记忆巩固和再巩固过程启发，提出在Transformer推理过程中周期性地合并KV缓存，减少冗余存储并加速后续推理。初步实验表明在保持推理能力的同时，内存需求显著降低，为推理优化提供了新思路。
source: ICLR-2026-Accepted
selection_source: conference_retrieval
motivation: 现有推理优化方法多局限于token空间，缺乏对缓存合并的探索。
method: 引入周期性的KV缓存合并与再巩固操作，模拟大脑记忆稳定过程。
result: 在推理任务上，周期合并降低了内存开销且保持了推理准确性。
conclusion: 缓存合并在Transformer推理中是一种有前景的优化方向。
---

## Abstract
Transformer LLMs have been shown to exhibit strong reasoning ability that scales with inference-time compute, most prominently through token-space “thinking” (i.e., chains of thought). A growing line of work pushes this extra computation into the model’s latent space (adjacent to standard decoding) which we term Auxiliary Latent-Space Computation (ALSC). Existing ALSC methods largely fall into three buckets: (i) token-mediated latent or special-token rollouts, (ii) residual/activation steering, and (iii) memory compression via cache pruning, merging, or summarization. An underexplored alternative is memory consolidation and reconsolidation, two processes in the brain that are responsible for stabilising newly formed memory traces, and, upon recall, transiently rendering established traces plastic such they can integrate new contextual information before restabilising. In a Transformer LLM, this can be seen as analogous to performing in-place global rewrites of incoming KV segments, and rewrites of past segments conditioned on newly observed tokens. In this work, we give a theoretical justification as to why memory (re)consolidation via KV cache rewrites is beneficial for improved reasoning. We do this through the lens of Information Bottleneck (IB) theory, which posits that model generalisation emerges from an optimal balance between input information compression and retention of predictive information in latent representations. We prove using IB theory that Vanilla decoder-only Transformers are inherently constrained in their ability to form task-optimal sequence representations. We then introduce the Bottlenecked Transformer, which augments a decoder-only backbone LLM with a lightweight Cache Processor, an auxiliary Transformer that performs periodic, non-causal, in-place KV rewrites at newline-delimited reasoning step boundaries. The processor consolidates recently written KV entries and reconsolidates a small, top-$k$ attention-selected set of prior entries, conditioned on recent context. We evaluate our Bottlenecked Transformer architecture on seven mathematical reasoning benchmarks, with four backbone LLMs. Our model sees consistent performance gains over vanilla Transformers and pause-token augmented Transformer baselines, with gains of up to +6.6pp for selected tasks and backbones.

---

## 论文详细总结（自动生成）

### 论文核心问题与整体含义（研究动机和背景）

- **问题**：现有Transformer大语言模型（LLM）的推理能力虽然可以通过增加测试时计算（如思维链CoT）来提升，但这类计算通常局限在词元空间（token-space）。另一类方法将额外计算引入潜在空间（即辅助潜在空间计算，ALSC），但现有ALSC方法主要分为三类：词元介导的潜变量/特殊词元展开、残差/激活引导、以及通过缓存剪枝、合并或摘要的内存压缩。一个未被充分探索的替代方案是**记忆巩固与再巩固**——受大脑记忆稳定和重新整合过程的启发。
- **整体含义**：本文旨在将记忆巩固/再巩固机制引入Transformer解码过程，通过周期性地对KV缓存进行非因果、原地改写，以改进模型在复杂推理任务中的泛化能力。理论支撑来自信息瓶颈（Information Bottleneck, IB）理论，证明原始Transformer在形成任务最优序列表示时存在固有局限。

### 论文提出的方法论

- **核心思想**：受大脑记忆巩固（将新形成的记忆痕迹稳定化）和再巩固（在回忆时临时使已稳定痕迹变得可塑，以便整合新上下文信息后重新稳定）启发，本文提出在Transformer推理中，对KV缓存进行周期性的非因果原地改写。这相当于对输入KV段进行全局重写，并根据新观察到的词元对先前段进行条件性重写。
- **关键技术细节**：
  - 提出**瓶颈变换器（Bottlenecked Transformer）**：在标准仅解码器骨干LLM基础上，添加一个轻量级**缓存处理器（Cache Processor）**，它是一个辅助Transformer，在换行分隔的推理步骤边界处执行周期性的非因果原地KV改写。
  - 处理器对最近写入的KV条目进行**巩固**，并对一小部分通过top-k注意力选择的历史条目进行**再巩固**，条件依赖于最近的上下文。
- **理论公式**：利用信息瓶颈理论，证明原始仅解码器Transformer在形成任务最优序列表示时受到约束（文中给出了理论证明，但未提供具体公式）；而周期性缓存合并有助于打破这种约束，实现更好的泛化。

### 实验设计

- **数据集/场景**：在7个数学推理基准上评估，包括多个标准数学题数据集（如GSM8K、MATH等，具体名称未在摘要中列出，但提及“seven mathematical reasoning benchmarks”）。
- **Benchmark**：对比方法包括：
  - 普通Transformer（vanilla decoder-only LLM）作为基线。
  - 停顿词元增强Transformer（pause-token augmented Transformer，一种常见的ALSC方法）。
- **对比方法**：未提及对比其他ALSC方法（如缓存剪枝、压缩等），但涵盖了最相关的基线。
- **评估指标**：准确率（accuracy，以百分点pp表示）。

### 资源与算力

- **文中未明确说明**：摘要和元数据中没有提到使用的GPU型号、数量、训练时长、参数量等信息。因此无法总结具体算力，但可以指出这一点缺失。

### 实验数量与充分性

- **实验数量**：在7个数学推理基准上，使用4种骨干LLM（backbone LLMs，如不同尺寸的LLaMA等，具体未列出）。对比了两种基线（vanilla和pause-token），因此主要实验结果数量较多（7×4×3=84组结果，但可能部分组合未全部报告）。另外可能包含消融实验（如缓存处理器的不同设计选择），摘要未提及，但通常此类工作会包含。
- **充分性与公平性**：从摘要看，实验覆盖了多个数据集和多种骨干模型，与最相关的基线进行了对比，并报告了统计显著的性能提升（最高+6.6pp）。但缺乏对超参数、缓存处理器大小、合并频率等的详细消融，以及与其他ALSC方法的全面对比。因此公平性较好但仍有提升空间。

### 论文的主要结论与发现

- 瓶颈变换器在7个数学推理基准上，针对4种骨干LLM，均取得了比普通Transformer和停顿词元增强Transformer更一致的性能提升，在某些任务和骨干上提升高达+6.6个百分点。
- 理论分析证明了原始Transformer在任务最优表示上的局限，而周期性KV缓存合并能够缓解此问题，从而改善推理泛化。
- 缓存合并是一种有前景且高效的推理优化方向，内存开销降低且保持了推理准确性（内存效率在摘要中提及但未量化）。

### 优点

- **理论创新**：首次从信息瓶颈理论角度解释KV缓存合并对推理的益处，为方法提供了坚实的理论依据。
- **方法创新**：模拟大脑记忆巩固/再巩固机制，提出轻量级缓存处理器，在不改变主干模型结构的情况下实现非因果KV改写，计算开销小。
- **实验覆盖较广**：在多个数据集和多种骨干模型上验证，且与直接相关的基线对比显示了优势。
- **性能显著**：最高提升6.6pp，表明方法有效。

### 不足与局限

- **实验覆盖不足**：仅针对数学推理任务，未在一般文本生成、常识推理、长上下文等任务上评估，泛化性未知。
- **资源信息缺失**：未报告计算成本（GPU小时、参数量、推理时间等），无法判断效率优势的具体程度。
- **缺乏消融研究**：未详细分析缓存处理器大小、合并频率、top-k选择策略等超参数的影响。
- **与现有ALSC方法对比不全面**：只对比了停顿词元基线，未与其他缓存压缩/剪枝/合并方法（如H2O、Scissorhands等）比较，结论的竞争力有待进一步验证。
- **潜在偏差**：使用骨干模型及数据集选择可能有偏向性，未说明是否进行多次运行与显著性检验。
- **应用限制**：仅适用于解码器-仅推理场景，且需要换行分割的步骤边界，对自由文本推理不直接适用。

（完）
