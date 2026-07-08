---
title: "Sparse-dLLM: Accelerating Diffusion LLMs with Dynamic Cache Eviction"
title_zh: Sparse-dLLM：利用动态缓存剔除加速扩散大模型
authors: "Yuerong Song, Xiaoran Liu, Ruixiao Li, Zhigeng Liu, Zengfeng Huang, Qipeng Guo, Ziwei He, Xipeng Qiu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40586/44547"
tags: ["query:llm-kv-cache"]
score: 4.0
evidence: 针对扩散大模型的动态缓存剔除，内存效率
tldr: 扩散大模型推理中缓存占用大量内存。Sparse-dLLM提出训练无关的动态缓存剔除框架，利用跨层稀疏性识别并剔除低重要性的令牌，结合稀疏注意力，在加速扩散模型推理的同时显著降低KV缓存内存。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 扩散LLM推理内存开销大，现有缓存方法无法分辨令牌重要性。
method: 提出动态缓存剔除与稀疏注意力结合，基于跨层重要性保留关键令牌。
result: 在扩散LLM推理中降低内存并加速，保持生成质量。
conclusion: 动态剔除策略可有效减少扩散模型缓存占用。
---

## Abstract
Diffusion Large Language Models (dLLMs) enable breakthroughs in reasoning and parallel decoding but suffer from prohibitive quadratic computational complexity and memory overhead during inference. Current caching techniques accelerate decoding by storing full-layer states, yet impose substantial memory usage that limit long-context applications. Our analysis of attention patterns in dLLMs reveals persistent cross-layer sparsity, with pivotal tokens remaining salient across decoding steps and low-relevance tokens staying unimportant, motivating selective cache eviction. We propose Sparse-dLLM, the first training-free framework integrating dynamic cache eviction with sparse attention via delayed bidirectional sparse caching. By leveraging the stability of token saliency over steps, it retains critical tokens and dynamically evicts unimportant prefix/suffix entries using an attention-guided strategy. Extensive experiments on LLaDA and Dream series demonstrate Sparse-dLLM achieves up to 10 times higher throughput than vanilla dLLMs, with comparable performance and similar peak memory costs, outperforming previous methods in efficiency and effectiveness.

---

## 论文详细总结（自动生成）

# Sparse-dLLM：利用动态缓存剔除加速扩散大模型——中文详细总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：扩散大语言模型（dLLMs）在推理时存在二次计算复杂度和巨大的内存开销。虽然现有缓存技术（如KV缓存）可以加速解码，但存储全层状态导致内存占用过高，限制了长上下文应用。
- **研究背景**：dLLMs 通过多次迭代去掩码生成文本，每次推断需重新计算整个序列的QKV状态，产生 O(L²) 复杂度。传统自回归LLM已有成熟的KV缓存优化，但dLLMs的双向注意力机制使直接缓存不适用。近期工作（如 dLLM-Cache、dKV-Cache、Fast-dLLM）尝试引入缓存，但均未对缓存内部进行稀疏化，内存负担仍然很大。
- **整体含义**：本文通过分析dLLMs注意力模式，发现跨层稀疏性和令牌重要性在解码步间的高度一致性，从而提出首个无需训练的、结合动态缓存剔除与稀疏注意力的框架Sparse-dLLM，显著提升吞吐量并保持内存与性能。

## 2. 论文提出的方法论：核心思想、关键技术细节

### 核心思想
利用dLLMs注意力图中持续存在的稀疏性，以及重要令牌在不同解码步间的稳定性，动态剔除低重要性的KV缓存条目，仅保留关键子集，从而减少计算和内存开销。

### 关键技术细节
1. **延迟双向稀疏缓存（Delayed Bidirectional Sparse Caching）**：
   - 将序列划分为固定块（block），当前解码块之外的令牌分为前缀（prefix）和后缀（suffix）两部分。
   - 计算当前块查询状态与候选键状态的注意力得分，经最大池化（kernel size=3）后通过 top-k 选择保留比例为 r 的关键令牌索引。
   - 同时考虑前缀和后缀的KV条目，实现双向稀疏化。
2. **延迟缓存更新**：
   - 观察到步骤0到步骤1间KV状态变化较大，因此将缓存更新延迟一个解码步，以提高稳定性。
3. **缓存刷新机制**：
   - 在切换到新解码块时，完全清空并刷新缓存。

### 算法流程（文字描述）
- 输入：序列状态 x_t，当前块偏移 o，块长度 b，保留比例 r，池化核大小 s。
- 候选KV：K_f = Concat(K[:o], K[o+b:])，V_f 同理。
- 计算注意力得分 A = Q_b K_f^T / √d_k。
- 应用最大池化，选取 top-k 索引（k = (L-b) × r）。
- 构建稀疏缓存 K_c = K_f[Indices]，V_c 同理。
- 解码 step 1 后才更新缓存（延迟一步）。

## 3. 实验设计

### 数据集/场景
- **通用、科学、数学、代码**：MMLU (5-shot)、ARC-c (0-shot)、PIQA (0-shot)、GPQA (5-shot)、GSM8K (4-shot)、Math (4-shot)、HumanEval (0-shot)。
- **长上下文**：LongBench 评估，以及不同序列长度（至 4k）下的吞吐量与内存。

### Benchmark 基准
- **基线**：原始 dLLMs（vanilla）无缓存。
- **对比方法**：dLLM-Cache、dKV-Cache、Fast-dLLM。
- **评估指标**：准确率、吞吐量（TPS，Tokens Per Second）、峰值内存（GB）。

### 模型
- LLaDA-8B-Instruct、LLaDA-1.5、Dream-v0-7B-Base、Dream-v0-7B-Instruct。

### 默认设置
- 块长度=32，固定随机种子2025，保留比例 r=0.5，池化核大小 s=3。

## 4. 资源与算力

- **文中明确说明**：所有实验在 NVIDIA 4090（48GB）GPU 上运行。
- **未说明**：GPU 总数、训练时长（本文为推理加速方法，无需训练）。由于是训练无关框架，算力需求主要体现在推理测试阶段，未提供具体运行时间统计。

## 5. 实验数量与充分性

- **主要实验**：在两个模型系列（LLaDA 和 Dream）共4个模型上进行7个标准基准测试，每个结果取三次独立试验平均。吞吐量和内存取10个随机实例平均。
- **长上下文实验**：在多种序列长度（1k~4k）下测量吞吐量和峰值内存，对比所有方法。
- **消融实验**：
  - 延迟步数（0-5步）对吞吐量和准确率的影响。
  - 稀疏策略（双向 vs. 仅前缀）的比较。
  - 保留比例 r 和池化核大小 s 的调参。
- **客观性与公平性**：实验覆盖多个模型系列、多种任务类型、公平对比当前最先进方法；报告三次平均，减少偶然性；默认超参数经消融确定。实验设计较为充分、客观。

## 6. 论文的主要结论与发现

1. **性能保持**：Sparse-dLLM 在几乎所有下游任务上保持与原始模型相当甚至略有提升的准确率。
2. **吞吐量大幅提升**：相较于 vanilla dLLMs，最高可达 10 倍加速（长上下文场景），在标准基准上平均加速 3~6 倍。
3. **内存效率优化**：峰值内存与 vanilla 水平几乎相同（差异小于0.5GB），在 Dream 模型上甚至更低（因为块式解码采样仅需当前块 logits）。
4. **长上下文优势**：在 4k 序列长度下吞吐量提升 10 倍，内存增长曲线近乎平坦，而其他方法在高长序列时可能 OOM。
5. **最佳配置**：延迟步数=1、保留比例 r=0.5、池化核大小=3 在效率与性能间达到最佳平衡。

## 7. 优点

- **训练无关**：无需额外训练或微调，可直接作为插件应用于任何 dLLM。
- **创新性**：首次将动态缓存剔除与稀疏注意力结合用于 dLLMs，并引入双向稀疏化（兼顾前后缀）。
- **理论支撑**：基于对 dLLMs 注意力模式深入分析，找到跨层和跨步一致性，有效指导剔除策略。
- **实验结果扎实**：在多个模型和基准上验证，吞吐量提升显著且内存开销极低，对比方法中表现最优。
- **代码开源**：提供 GitHub 仓库，促进可复现性。

## 8. 不足与局限

- **HumanEval 性能下降**：在 Dream 模型上代码生成准确率下降，归因于通用剔除策略可能删除了语法关键令牌（如变量类型、数字字面量）。
- **超参数依赖**：保留比例 r 和池化核大小 s 需要通过实验调优，不同任务或模型可能需要重新调整。
- **延迟步数设定**：1 步延迟是最优，但不同模型/任务的最佳延迟可能不同，文中未深入探讨。
- **仅考虑单GPU场景**：实验仅在单张 NVIDIA 4090 上进行，未在多GPU或分布式环境下验证扩展性。
- **未与自回归LLM的缓存优化直接对比**：虽然方法针对 dLLMs，但与同类 dLLMs 缓存方法对比已够充分，但缺少与自回归LLM优化（如 SnapKV）在类似设定下的比较（因架构不同可能不直接可比）。
- **长上下文最大测试长度**：仅测试到 4k，实际应用中可能需处理更长序列（如 8k/16k），文中未覆盖。

（完）
