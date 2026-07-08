---
title: "SparK: Query-Aware Unstructured Sparsity with Recoverable KV Cache Channel Pruning"
title_zh: SparK：基于查询感知非结构化稀疏性与可恢复KV缓存通道剪枝
authors: "Huanxuan Liao, Yixing Xu, Shizhu He, Guanchen Li, Xuanwu Yin, Dong Li, Emad Barsoum, Jun Zhao, Kang Liu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40466/44427"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于查询感知稀疏性的KV缓存通道剪枝
tldr: 针对长上下文LLM推理中KV缓存内存瓶颈问题，现有压缩方法忽略通道维度重要性差异。SparK提出查询感知的通道剪枝方法，根据查询和位置动态识别重要通道，对不重要的进行剪枝并可恢复，在保持精度的同时大幅减少KV缓存内存和计算开销。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存压缩方法忽略通道维度重要性差异，难以平衡效率与准确性。
method: 提出查询感知的KV缓存通道剪枝，根据通道对查询的重要性进行非结构化稀疏化并支持剪枝后恢复。
result: 在长上下文推理中显著降低内存占用，同时保持模型精度。
conclusion: 通道维度的细粒度重要性分析可有效提升KV缓存压缩效果。
---

## Abstract
Long-context inference in large language models (LLMs) is increasingly constrained by the KV cache bottleneck: memory usage grows linearly with sequence length, while attention computation scales quadratically. Existing approaches address this issue by compressing the KV cache along the temporal axis through strategies such as token eviction or merging to reduce memory and computational overhead. However, these methods often neglect fine-grained importance variations across feature dimensions (i.e., the channel axis), thereby limiting their ability to effectively balance efficiency and model accuracy. In reality, we observe that channel saliency varies dramatically across both queries and positions: certain feature channels carry near-zero information for a given query, while others spike in relevance. To address this oversight, we propose SPARK, a training-free plug-and-play method that applies unstructured sparsity by pruning KV at the channel level, while dynamically restoring the pruned entries during attention score computation. Notably, our approach is orthogonal to existing KV compression and quantization techniques, making it compatible for integration with them to achieve further acceleration. By reducing channel-level redundancy, SPARK enables processing of longer sequences within the same memory budget. For sequences of equal length, SPARK not only preserves or improves model accuracy but also reduces KV cache storage by over 30% compared to eviction-based methods. Furthermore, even in an aggressive pruning ratio of 80%, SPARK maintains performance with less degradation than 5% compared to the based eviction method, demonstrating robustness and effectiveness. Our code will be available at \url{https://github.com/AMD-AIG-AIMA/AMD-Spark}.

---

## 论文详细总结（自动生成）

# 论文总结：SparK

## 1. 核心问题与整体含义（研究动机和背景）

长上下文推理中，大型语言模型（LLM）的关键-值（KV）缓存成为主要瓶颈：内存占用随序列长度线性增长，注意力计算复杂度呈二次增长。现有压缩方法主要沿时间轴（token剔除或合并）进行压缩，但忽略了特征通道维度的细粒度重要性差异——不同查询与位置下，通道的显著程度变化极大，某些通道信息几乎为零，而其他通道则信息密集。这种忽视导致效率与准确性难以平衡。SparK提出一种查询感知、训练无关的即插即用方法，通过在通道维度执行非结构化剪枝并动态恢复被剪枝条目，在维持或提升模型准确性的同时大幅减少KV缓存存储。

## 2. 方法论

### 核心思想
- 将通道剪枝重构为关键通道集合选择问题：为每个注意力头和每个token选择最具显著性贡献的T个通道（T << D），最大化所选通道的聚合显著分数。
- 引入可恢复性机制：在注意力分数计算中，通过轻量级恢复函数F近似被剪枝通道的贡献，而非直接丢弃，从而缓解信息损失。

### 关键技术细节
- **显著性度量**：采用代理分数 \( w_{j}^{i,t} = \|q_{j}^{i,t}\|_2 \|k_{j}^{i,t}\|_2 \) 来上界通道j对Frobenius范数的贡献。为降低计算开销，使用观察窗口内的平均查询向量替代逐token查询。
- **非结构化剪枝**：对每个token，保留Top-T个最高显著性得分的通道，构建二进制掩码S。
- **恢复机制**：在解码阶段，通过预填充阶段缓存的分布统计量（均值μ、标准差σ）或仅均值（退化分布）来采样近似分数，进而反推出键条目：\( \tilde{k}_{j}^{i,t} = \frac{\tilde{w}_{j}^{i,t}}{\|q_{j}^{i}\|_2} \)。支持高斯、指数、退化三种分布，退化分布表现最为鲁棒。
- **算法流程**：
  1. 预填充：计算每个head的通道显著性、构建mask、存储统计信息。
  2. 解码：使用恢复函数F重建被剪枝通道，拼接回完整键，执行标准全注意力。

### 公式说明
剪枝问题的优化目标最小化剪枝前后的注意力权重差异，近似等价于最大化保留通道显著分数的和，可通过贪心算法高效求解（选择Top-T通道）。

## 3. 实验设计

### 数据集与场景
- **LongBench**：包含多文档QA、单文档QA、摘要、少样本学习、合成代码等16个子任务，评估长上下文理解能力。
- **RULER**：包含多项检索和推理任务（如多键查找、数值推理等），评估长上下文下的精确检索能力。

### 基准对比方法
- 全KV缓存（Vanilla）
- 时间轴压缩：StreamingLLM、ExpectedAttention、TOVA、SnapKV、PyramidKV
- 通道轴结构化剪枝：ThinK（作为主要对比对象）
- 同时评估了与上述方法的集成效果（如SnapKV + SparK、PyramidKV + ThinK等）

### 模型
- LLaMA-3/3.1-8/70B-Instruct、Qwen3-8B/32B

### 消融实验
- 不同恢复分布（高斯、指数、退化）
- 自适应变体：SparK-p（动态阈值）、SparK-g（分组渐进剪枝）
- 不同剪枝率（λ=0.5/0.8）、不同KV缓存大小（128/512/1024/2048）
- 输入长度（8k~128k）下的吞吐量
- 联合键-值缓存通道剪枝

## 4. 资源与算力

论文未明确说明所使用的GPU型号、数量、训练时长等算力信息。由于SparK是**训练无关（training-free）**方法，实验仅涉及推理阶段，无需额外训练，因此不会产生训练算力开销。但具体推理硬件配置（如A100、H100等）未提及。

## 5. 实验数量与充分性

- **实验数量丰富**：在LongBench（16个子任务）和RULER（13个子任务）上进行了全面评估，覆盖不同压缩率、不同缓存大小、不同模型规模。
- **消融实验充分**：考察了恢复分布、自适应变体、联合KV剪枝、剪枝率影响、吞吐量等。
- **对比公平**：使用一致的超参数设置，与多种主流方法（包括结构化剪枝ThinK和token剔除方法）直接对比。
- **客观性**：结果以表格形式呈现，包含平均性能；无选择性报告。
- **潜在偏差**：主要使用LLaMA和Qwen系列模型，未在Mistral、Falcon等其他架构上验证；部分任务（如RULER中的VT、CWE）上性能下降稍大，未深入分析原因。

## 6. 主要结论与发现

- SparK在所有压缩率下均显著优于结构化剪枝方法ThinK，尤其在高剪枝率（80%）下保持性能下降＜5%，而ThinK下降超35%。
- 恢复机制是关键：即使使用简单的退化分布（仅均值）也能大幅缩小性能差距。
- 与现有token剔除方法（SnapKV、PyramidKV）兼容，进一步减少30%以上缓存存储。
- 非结构化、查询感知的剪枝比结构化剪枝更有效，能适应token间通道重要性变化。
- SparK在长上下文（128k）下保持稳定吞吐量，而全缓存方法易内存溢出。

## 7. 优点

- **训练无关、即插即用**：无需任何微调或重训练，可直接应用于任意LLM。
- **正交性**：与时间轴、空间轴压缩及量化技术兼容，可叠加使用实现更大加速。
- **恢复机制创新**：通过缓存统计量近似重建剪枝通道，避免信息永久丢失，提升高压缩比下的鲁棒性。
- **轻量化**：预填充阶段计算开销可接受（仅需一次额外统计），解码阶段恢复开销极小。
- **自适应变体**：提供无超参数版本（SparK-p、SparK-g），便于实际部署。

## 8. 不足与局限

- **额外计算开销**：预填充阶段需要计算每个token的通道显著性，增加少量计算；恢复机制在解码时增加轻微延迟（但实验显示吞吐量几乎不变）。
- **恢复误差**：依赖于分布假设（高斯/指数/退化），可能在某些任务上引入近似误差；虽然实验表明退化分布足够好，但理论上可进一步优化。
- **模型覆盖有限**：仅测试LLaMA-3/3.1和Qwen3系列，未验证在Mistral、Falcon、Gemma等模型上的泛化性。
- **未与低秩分解、量化等方法联合评估**：虽然作者声称正交性，但未提供与KIVI、FP8量化等方法的联合实验结果。
- **对某些任务敏感**：在RULER的“Variable Tracking”和“Common Word Extraction”等任务中，SparK的收益相对较小，可能因任务特性对精确通道信息要求高。

（完）
