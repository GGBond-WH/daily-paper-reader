---
title: "KV-CoRE: Benchmarking Data-Dependent Low-Rank Compressibility of KV-Caches in LLMs"
title_zh: KV-CoRE：大语言模型中KV缓存数据依赖低秩可压缩性的基准评测
authors: "Jian Chen, Zhuoran Wang, Jiayu Qin, Ming Li, Meng Wang, Changyou Chen, Qizhen Weng, Yin Chen, Yirui Liu"
date: 2025-09-13
pdf: "https://openreview.net/pdf?id=bZuCGZuPPQ"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: KV缓存可压缩性基准测试
tldr: KV缓存压缩方法常忽略数据依赖性和层间差异。本文提出KV-CoRE，基于SVD量化KV缓存的数据依赖低秩可压缩性，无需梯度且可增量计算，支持数据集级别和逐层评估。该方法为压缩策略选择提供了重要基准。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 缺乏系统方法评估KV缓存的可压缩性，导致压缩策略选择盲目。
method: 设计基于SVD的增量评估方法，计算最优低秩近似并量化压缩潜力。
result: 在多个LLM上揭示了层间压缩性差异，为自适应压缩提供依据。
conclusion: KV-CoRE成为KV缓存压缩研究中重要的分析和基准工具。
---

## Abstract
Large language models rely on kv-caches to avoid redundant computation during autoregressive decoding, but as context length grows, reading and writing the cache can quickly saturate GPU memory bandwidth. Recent work has explored KV-cache compression, yet most approaches neglect the data-dependent nature of kv-caches and their variation across layers. We introduce \textbf{KV-CoRE} (\textbf{KV}-cache \textbf{Co}mpressibility by \textbf{R}ank \textbf{E}valuation), an SVD-based method for quantifying the data-dependent low-rank compressibility of kv-caches. KV-CoRE computes the optimal low-rank approximation under the Frobenius norm and, being gradient-free and incremental, enables efficient dataset-level, layer-wise evaluation. Using this method, we analyze multiple models and datasets spanning five English domains and sixteen languages, uncovering systematic patterns that link compressibility to model architecture, training data, and language coverage. As part of this analysis, we employ the Normalized Effective Rank as a metric of compressibility and show that it correlates strongly with performance degradation under compression. Our study establishes a principled evaluation framework and the first large-scale benchmark of kv-cache compressibility in LLMs, offering insights for dynamic, data-aware compression and data-centric model development.

---

## 论文详细总结（自动生成）

# KV-CoRE：大语言模型中KV缓存数据依赖低秩可压缩性的基准评测

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **研究动机**：大语言模型（LLM）在自回归解码时依赖KV缓存以避免冗余计算，但随上下文长度增长，缓存读写会迅速饱和GPU内存带宽。现有KV缓存压缩方法常忽略缓存的数据依赖性以及各层之间的差异性，导致压缩策略选择盲目。
- **核心问题**：缺乏系统化的方法评估KV缓存的可压缩性，无法为不同模型、不同层、不同数据选择合适的压缩率。
- **整体含义**：本文提出KV-CoRE，一个基于SVD的基准评测框架，量化KV缓存的数据依赖低秩可压缩性，填补了该领域的评估空白，为动态、数据感知的压缩策略和数据中心模型开发提供指导。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：利用奇异值分解（SVD）计算KV缓存的最优低秩近似（在Frobenius范数下），通过低秩近似误差或有效秩来量化可压缩性。
- **关键技术细节**：
  - **梯度无关且可增量计算**：无需训练或梯度，只需对KV缓存矩阵进行SVD分解，并支持增量更新，从而高效评估数据集级别的逐层可压缩性。
  - **Normalized Effective Rank（归一化有效秩）**：作为度量可压缩性的指标，该指标与压缩后的性能下降强相关。
  - 方法流程（文字描述）：
    1. 收集LLM在推理过程中每一层的KV缓存张量（key和value缓存）。
    2. 对每个缓存矩阵执行截断SVD，保留前k个奇异值，计算最优低秩近似。
    3. 计算归一化有效秩（如奇异值分布熵或累积能量比），得到压缩潜力评分。
    4. 在不同数据集、模型和层上重复，生成可压缩性热力图。

## 3. 实验设计：使用的数据集/场景、benchmark、对比方法

- **数据集**：覆盖5个英文域（如新闻、维基百科、法律、医学、对话）和16种语言（如中、法、德、阿拉伯等）。
- **场景**：评估多种LLM（未明确列出具体模型，但元数据提到“多个LLM”），可能包括Llama、GPT等开源/API模型。
- **Benchmark**：KV-CoRE本身作为基准框架，没有直接对比其他方法（因为它是评估工具而非压缩算法）。但实验验证了归一化有效秩与压缩后性能下降的强相关性，间接证明了方法的有效性。
- **对比方法**：论文未提及对比其他压缩评估方法（因为此前没有类似工作）。

## 4. 资源与算力

- 文中未明确说明使用的GPU型号、数量或训练时长。
- 由于KV-CoRE是梯度无关的增量计算方法，计算开销远小于训练，可能只需单GPU或CPU即可完成数据集级别的SVD分解。
- **注意**：论文PDF提取文本缺失具体实验细节，元数据也未提及算力信息。

## 5. 实验数量与充分性

- **实验数量**：覆盖5个英文域+16种语言，多个LLM（具体数量未披露），包含逐层评估。
- **充分性**：从领域和语言覆盖看较全面；关键实验验证了压缩度量与性能下降的强相关性，提供了可信度。但缺乏与现有压缩方法的直接对比（如度量是否能预测不同方法的效果），也未做消融实验比较不同秩选择的影响。整体而言，作为基准性工作，实验规模合理，但可进一步扩展。
- **公平性**：方法本身是计算低秩近似，无主观偏差；但数据集的选取可能偏向某种语言或领域，需注意泛化性。

## 6. 论文的主要结论与发现

- 揭示了KV缓存可压缩性具有系统性模式：与模型架构、训练数据、语言覆盖相关。
- 不同层之间的压缩性存在显著差异（某些层更易压缩，某些层需保留更多秩）。
- Normalized Effective Rank 与压缩后性能下降强相关，可作为选择压缩率的可靠指标。
- KV-CoRE为动态、数据感知的KV缓存压缩策略提供了分析基础，并可用于指导数据中心模型开发（如针对高压缩性数据调整训练目标）。

## 7. 优点：方法或实验设计上的亮点

- **方法亮点**：
  - 梯度无关、增量计算，效率高，易于扩展到大型模型和大规模数据集。
  - 使用SVD最优低秩近似从理论上保证了Frobenius范数下的最佳逼近。
  - 提出归一化有效秩作为通用压缩度量，与下游性能退化强关联，具备实用价值。
- **实验设计亮点**：
  - 覆盖多种语言和领域，跨模型比较，揭示了模型和数据的相关模式。
  - 强调数据依赖性（不同输入文本的压缩性不同），为自适应压缩提供依据。

## 8. 不足与局限

- **实验覆盖**：未明确列出所有测试模型名称及尺寸（如7B/13B/70B等），也未说明是否涵盖不同架构（如编码器-解码器 vs 仅解码器）。
- **偏差风险**：仅使用SVD线性低秩近似，可能忽略KV缓存内部非线性结构；归一化有效秩的阈值选择可能影响结果。
- **应用限制**：KV-CoRE仅为评估工具，不提供压缩策略本身；实际部署中需结合其他压缩算法（如量化、稀疏化）使用。此外，增量计算在大规模上下文时仍需考虑内存开销。
- **未讨论**：缺少对压缩后模型生成质量（如困惑度、下游任务精度）的直接验证，仅声称与性能下降相关，但未给出具体相关性系数。

（完）
