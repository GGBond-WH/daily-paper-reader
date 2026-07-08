---
title: Head-Aware KV Cache Compression for Efficient Visual Autoregressive Modeling
title_zh: 面向高效视觉自回归建模的注意力头感知KV缓存压缩
authors: "Ziran Qin, Youru Lv, Mingbao Lin, Hang Guo, Zeren Zhang, Danping Zou, Weiyao Lin"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39686/43647"
tags: ["query:llm-kv-cache"]
score: 7.0
evidence: 面向视觉自回归模型的注意力头感知KV缓存压缩
tldr: 视觉自回归模型在多尺度生成中面临KV缓存内存和计算压力。本文发现注意力头分为语义维持型（Contextual Heads）和空间结构型（Structural Heads），现有统一压缩策略不适用。据此提出头感知KV缓存压缩方法，对不同类型头采用差异化压缩策略。实验在保持生成质量前提下，大幅降低KV缓存内存占用和计算复杂度。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 视觉自回归模型中KV缓存随尺度积累，现有统一压缩方法忽略注意力头功能差异。
method: 根据注意力头功能分类（语义/结构），应用差异化压缩策略。
result: 在保持生成质量的同时显著减少KV缓存内存和计算开销。
conclusion: 头感知压缩为视觉自回归模型提供高效缓存管理，可启发LLM中类似设计。
---

## Abstract
Visual Autoregressive (VAR) models adopt a next-scale prediction paradigm, offering high-quality content generation
with substantially fewer decoding steps. However, existing
VAR models suffer from significant attention complexity and
severe memory overhead due to the accumulation of key-value (KV) caches across scales. In this paper, we tackle
this challenge by introducing KV cache compression into the
next-scale generation paradigm. We begin with a crucial observation: attention heads in VAR models can be divided into
two functionally distinct categories: Contextual Heads focus
on maintaining semantic consistency, while Structural Heads
are responsible for preserving spatial coherence. This structural divergence causes existing one-size-fits-all compression
methods to perform poorly on VAR models. To address this,
we propose HACK, a training-free Head-Aware KV cache
Compression frameworK. HACK utilizes an offline classification scheme to separate head types, enabling it to apply pattern-specific compression strategies with asymmetric
cache budgets for each category. By doing so, HACK effectively constrains the average KV cache length within a
fixed budget B, reducing the theoretical attention complexity
from O(n4) to O(Bn2). Extensive experiments on multiple
VAR models across text-to-image and class-conditional tasks
validate the effectiveness and generalizability of HACK. It
achieves up to 70% KV cache compression without degrading output quality, resulting in memory savings and faster in-
ference. For example, HACK provides a 1.75× memory reduction and a 1.57× speedup on Infinity-8B.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）
- **视觉自回归（VAR）模型**采用“下一尺度”预测范式，能以较少解码步骤生成高质量图像，但多尺度生成导致关键值（KV）缓存随尺度累积，带来 **O(n⁴)** 的注意力复杂度和严重的内存开销。
- 现有面向大语言模型（LLM）的KV缓存压缩方法（如H2O、SnapKV等）直接应用于VAR模型时效果不佳，因为VAR的注意力头存在功能分化：**Contextual Heads**（语义维持）和**Structural Heads**（空间结构保持）。统一压缩策略无法适配这种异质性。
- 本文提出**HACK**（Head-Aware KV Cache Compression），一种免训练的头感知KV缓存压缩框架，首次针对VAR模型设计差异化压缩，在保持生成质量的同时显著降低内存和计算开销。

### 2. 论文提出的方法论：核心思想、关键技术细节
- **核心观察**：
  - Contextual Heads产生低列方差的垂直注意力模式，关注关键语义token；Structural Heads产生高列方差的多对角模式，关注空间邻接token。两者在不同样本和尺度下稳定一致。
  - Contextual Heads对压缩鲁棒（90%压缩仍保持质量），Structural Heads敏感（超过50%压缩显著退化）。
  - 初始尺度和最近尺度的token比中间尺度的更重要。

- **关键技术步骤**：
  1. **离线头分类**：在最终生成步，计算每个注意力头的列方差之和，形成方差矩阵 \(\sigma \in \mathbb{R}^{L \times H}\)。全局排序后，按预设的**上下文头比例α**将头分为Contextual（低方差）和Structural（高方差）。分类后每层的头索引集 \(H_C\) 和 \(H_S\) 确定。
  2. **非对称预算分配**：给定平均每头预算 \(B\)，令 \(B = \alpha B_C + (1-\alpha)B_S\)，其中 \(B_C \ll B_S\)，为Structural Head保留更多容量。
  3. **模式特定压缩策略**：
     - **Contextual Head策略 \(f_C\)**：通过均匀采样子集查询计算近似注意力分数，直接选择累积分数最高的Top-\(B_C\)个KV对。最后一步还采用软合并（参考LOOK-M）以减少语义丢失。
     - **Structural Head策略 \(f_S\)**：保留前 \(N_{\text{init}}\) 个初始token和最近尺度的 \(N_k\) 个token，剩余预算 \(M = B_S - N_{\text{init}} - N_k\) 通过子集注意力选择中间尺度的重要token，保持空间连续性。
  4. **复杂度分析**：压缩后，注意力计算复杂度从 \(O(n^4)\) 降至 \(O(B n^2)\)，其中 \(n\) 为当前尺度token数。额外开销来自子集注意力（\(O(N_{\text{obs}} n^2)\)），但 \(N_{\text{obs}} \ll n\)。

### 3. 实验设计：数据集、benchmark与对比方法
- **评估模型**：
  - **文本到图像生成**：Infinity-2B、Infinity-8B、Hart。指标：GenEval（语义对齐）、ImageReward（IR）、HPSv2.1（HPS）、MJHQ30k（FID & CLIP）。
  - **类别条件生成**：VAR-d30。指标：ImageNet-1K上的FID、Inception Score (IS)、Precision、Recall。
- **对比方法**：
  - 驱逐型：StreamingLLM、H2O、SnapKV、CAKE。
  - 合并型：LOOK-M、Meda。
- **压缩比**：定义为 \(\rho = 1 - B/T_K\)，主要实验采用 \(\rho=70\%\)，并在不同比率（30%、50%、70%、90%）下评估。

### 4. 资源与算力
- 论文在**NVIDIA A100 GPU**上进行实验（批次大小为1），但**未明确说明使用的GPU数量、训练时长**。HACK为免训练方法，仅需少量离线分类样本，但推理时压缩过程会增加少量计算开销（约6–8%的总体延迟）。作者提供了内存和延迟的具体数值（见表3），但算力细节不完整。

### 5. 实验数量与充分性
- **实验组数**：覆盖3个文本到图像模型（Infinity-2B/8B、Hart）和1个类别条件模型（VAR-d30），共4个模型。
- **多压缩比**：除70%主结果外，还在30%、50%、90%下进行了定量和定性比较（图6、图8）。
- **消融实验**：
  - 组件消融：非对称预算 + 模式特化策略的贡献（表4）。
  - 策略变体消融：去除合并、去除初始/最近尺度保护、交换策略（表5）。
  - 查询子集选择方法比较：随机、初始 tokens、最近 tokens、均匀采样（表6）。
- **效率分析**：测量内存和延迟（表3），并对比不同分辨率下的延迟（图9）。
- **充分性**：实验设计较为完整，涵盖多种模型、任务、指标和压缩比率，消融实验验证了各组件必要性。对比方法均为代表性压缩方法，且在同一压缩比下比较，结果公平。但未在不同随机种子下进行多次重复测试（如FID标准差未给出），也未在更大规模模型（如>10B参数）上验证。

### 6. 论文的主要结论与发现
- HACK在**70%压缩比**下达到**无损甚至略优**的生成质量（部分指标超过全KV缓存），而所有baseline均显著退化。
- 在极端压缩（90%）下，HACK仍能保持视觉保真度，而baseline严重失真。
- 实际效率提升明显：Infinity-8B上内存减少1.75×，推理加速1.57×；在高分辨率（1024×1024）下加速可达5.8×。
- 分类方法稳定：基于注意力方差的离线分类能有效区分两类头，且函数角色通过掩码实验（图3）得到验证。

### 7. 优点：方法与实验设计的亮点
- **针对性创新**：首次识别VAR模型中注意力头的功能分化，并据此设计异质压缩，而非简单套用LLM方法。
- **免训练**：无须额外训练或微调，仅需少量离线数据计算方差，实用性强。
- **压缩策略简洁高效**：Contextual头采用TopK+合并，Structural头保留关键尺度 + 中间选择，均有理论依据和实验支撑。
- **实验覆盖全面**：涵盖多种模型、任务和指标，消融实验系统性强，效率分析包含压缩开销。
- **复杂度分析清晰**：给出了从 \(O(n^4)\) 到 \(O(Bn^2)\) 的推导，为后续研究提供理论基础。

### 8. 不足与局限
- **分类依赖方差阈值**：全局比例α需要人工设定，不同模型或任务可能需要调整，未探讨自适应α方法。
- **实验局限**：
  - 仅验证了4个VAR模型（Infinity-2B/8B、Hart、VAR-d30），未在更大规模（如Infinity-20B+）或其他架构（如Lumina、MARS）上测试。
  - 类别条件任务仅使用VAR-d30，缺少在其他VAR模型上的类别条件实验。
  - FID/IS等指标未提供置信区间或多次运行统计，可能受随机性影响。
- **压缩开销**：虽然子集注意力开销较小，但在极长序列或高维模型中仍可能成为瓶颈，且合并操作仅在最后一步执行，可能无法完全避免中间步的语义丢失。
- **与其他优化方法的兼容性**：未讨论与FlashAttention、量化等的联合使用效果，实际部署可能需进一步整合。
- **理论基础**：注意力方差作为分类依据虽实验有效，但缺乏对更复杂注意模式的解释（如是否存在混合头），可能在某些预训练模型中不成立。

（完）
