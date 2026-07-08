---
title: "MHA2MLA-VLM: Enabling DeepSeek’s Economical Multi-Head Latent Attention Across Vision-Language Models"
title_zh: "MHA2MLA-VLM: 在视觉语言模型中启用DeepSeek经济高效的多头隐式注意力"
authors: "Xiaoran Fan, Zhichao Sun, Tao Ji, Lixing Shen, Tao Gui"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40319/44280"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 将VLM转换为MLA结构以压缩KV缓存
tldr: MHA2MLA-VLM针对VLM中KV缓存占用问题，提出参数高效的MLA转换框架。通过模态自适应部分RoPE策略和参数高效迁移，在不需昂贵预训练下将现成VLM转为MLA架构，极大压缩KV缓存并加速推理。实验验证了有效性和多模态兼容性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: VLM推理时KV缓存快速增长造成内存和计算瓶颈。
method: 通过模态自适应部分RoPE和参数高效迁移将VLM转换为多头隐式注意力架构。
result: 在不需预训练下实现KV缓存压缩和推理加速。
conclusion: 转换架构是优化VLM KV缓存的有效途径。
---

## Abstract
As vision-language models (VLMs) tackle increasingly complex and multimodal tasks, the rapid growth of Key-Value (KV) cache imposes significant memory and computational bottlenecks during inference. While Multi-Head Latent Attention (MLA) offers an effective means to compress the KV cache and accelerate inference, adapting existing VLMs to the MLA architecture without costly pretraining remains largely unexplored. In this work, we present \textbf{MHA2MLA-VLM}, a parameter-efficient and multimodal-aware framework for converting off-the-shelf VLMs to MLA. 
Our approach features two core techniques: (1) a modality-adaptive partial-RoPE strategy that supports both traditional and multimodal settings by selectively masking nonessential dimensions, and (2) a modality-decoupled low-rank approximation method that independently compresses the visual and textual KV spaces. Furthermore, we introduce parameter-efficient fine-tuning to minimize adaptation cost and demonstrate that minimizing output activation error, rather than parameter distance, substantially reduces performance loss. Extensive experiments on three representative VLMs show that MHA2MLA-VLM restores original model performance with minimal supervised data, significantly reduces KV cache footprint, and integrates seamlessly with KV quantization.

---

## 论文详细总结（自动生成）

### 论文详细中文总结

#### 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：视觉语言模型（VLM）在处理多模态任务时，Key-Value（KV）缓存随上下文长度快速增长，导致显存占用高和计算瓶颈，制约推理效率。
- **研究动机**：多头隐式注意力（MLA）架构能有效压缩KV缓存并加速推理，但现有VLM大多基于MHA/GQA架构，直接训练MLA需要高昂成本。如何在不进行大规模预训练的前提下，将现成的VLM高效转换为MLA架构，是一个亟待解决的关键问题。
- **整体含义**：论文提出 **MHA2MLA-VLM** 框架，通过参数高效、多模态感知的转换技术，使VLM能够低代价地采用MLA，显著降低KV缓存占用，同时保持或恢复原有性能，为VLM的规模化部署提供了实用解决方案。

#### 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程
- **核心思想**：将VLM从MHA/GQA架构转换为MLA架构，包含两个关键步骤：1) 将全RoPE改为部分RoPE（仅保留重要旋转维度）；2) 对KV向量进行低秩联合压缩，实现MLA所需的隐式表示。同时采用模态感知和数据驱动的方式实现最优转换。
- **关键技术细节**：
  - **模态自适应部分RoPE（Modality-Adaptive Partial-RoPE）**：提出了基于KL散度的贡献感知策略（MKL），通过计算每个频率子空间对注意力分布的敏感性，自适应地保留对位置理解最重要的旋转维度。该方法支持标准1D-RoPE和多模态M-RoPE，并针对视觉与文本模态的差异进行了优化。
  - **模态解耦的低秩近似（Modality-Decoupled SVD, MD-SVD）**：观察到视觉和文本激活具有不同特征，且二者低秩空间正交。因此分别为视觉和文本计算独立的低秩投影矩阵，最小化输出激活误差（而非参数距离），从而减少截断损失。理论证明（Theorem 2.1）解耦优化的损失上界低于联合优化。
  - **参数高效微调（PEFT）**：采用两阶段训练：阶段1仅微调查询和键的投影矩阵（约10%参数）；阶段2微调MLA内部参数。整个训练仅需少量数据（如0.002%原始数据）即可收敛。
- **算法流程**（文字说明）：
  1. 输入：原始VLM权重（MHA/GQA）和少量多模态数据。
  2. Step 1 部分RoPE转换：对每个注意力头，根据MKL策略选择保留的旋转子空间，丢弃其余维度，形成NoPE维度。
  3. Step 2 MD-SVD初始化：分别收集视觉和文本激活，计算各自协方差矩阵，进行SVD分解得到低秩压缩矩阵（W_up_m, W_down_m）。
  4. 两阶段微调：先微调query/key投影（阶段1），再微调MLA参数（阶段2），最终得到MLA架构的VLM。

#### 3. 实验设计：数据集、基准和对比方法
- **数据集**：使用8个视觉语言基准测试：AI2D（2016）、GQA（2019）、POPE（2023）、SEED-Bench（2023）、RealWorldQA（RWQ 2025）、MMBench（2024）、ChartQA（2022）、DocVQA（2021）。涵盖图表理解、视觉问答、物体幻觉、文档理解等多样任务。
- **模型**：三个代表性VLM：LLaVA-1.5 7B（MHA）、LLaVA-NeXT 8B（GQA）、Qwen2.5-VL 7B（GQA + M-RoPE）。覆盖不同架构和压缩设置。
- **对比方法**：
  - 原始模型（MHA/GQA）及微调baseline（dkv=256）。
  - 缓存剪枝方法：H2O、TOVA。
  - 缓存量化方法：Int4 Quanto、Int4 HQQ、Int2 Quanto、Int2 HQQ。
  - 消融实验：有无MD-SVD、有无两阶段训练、部分RoPE策略对比（S2-norm vs SMKL）。

#### 4. 资源与算力
- **文中明确提及**：
  - 训练数据量：仅使用0.002%~0.025%的原始数据（如Qwen2.5-VL用0.5B tokens中的0.002%即约10K tokens）即可达到效果。
  - 参数更新比例：仅微调约10%的模型参数。
  - 训练时间：MHA2MLA-VLM将Qwen2.5-VL的训练时间从22小时缩短至9小时（使用两阶段PEFT）。
- **未明确说明**：使用的GPU型号、数量、具体节点信息。论文仅提到“使用CFFF计算平台”，未提供硬件细节，因此无法评估算力成本的具体量级。

#### 5. 实验数量与充分性
- **实验数量**：涵盖了3个不同架构的VLM、多种压缩率（dkv = 128/64/32/16）、与3种缓存剪枝方法、2种量化方法（4bit和2bit）的对比、2类消融实验（模态解耦、两阶段训练、部分RoPE策略）、以及理论验证（损失比、频率分析）。总计实验组数十组。
- **充分性**：实验设计较为全面，覆盖了不同模型规模、压缩程度、方法对比和消融分析。对比基线包括当前流行方法（H2O、TOVA、HQQ等），且在同一尺度下比较。消融实验验证了每个组件的贡献。但缺少与完全从头训练MLA模型的直接比较（虽然这不是论文目标），以及在更长上下文（如视频多帧）或更大模型（>13B）上的验证。

#### 6. 论文的主要结论与发现
- MHA2MLA-VLM能够在极小数据量（0.002%原始数据）和少量参数微调（10%）下，将MHA/GQA架构的VLM成功转换为MLA架构，显著压缩KV缓存（最高96.43%），同时保持与原模型相当的性能（例如Qwen2.5-VL在dkv=64时平均得分79.47，原模型79.47）。
- 模态解耦的SVD（MD-SVD）和贡献感知的部分RoPE（MKL）均优于基线，MD-SVD的理论优势得到实验验证（损失比始终低于1）。
- 与缓存剪枝方法相比，MHA2MLA-VLM在相同压缩率下性能显著更好；与量化方法兼容，可复合压缩。
- 最小化输出激活误差（而非参数距离）明显降低性能损失。

#### 7. 优点：方法或实验设计上的亮点
- **方法创新**：首次将MHA2MLA思想从纯文本LLM扩展到多模态VLM，并专门设计了模态自适应部分RoPE和模态解耦SVD，解决了视觉和文本信息共存的挑战。
- **数据与参数高效**：极大降低转换成本，使资源受限的研究者也能受益。
- **理论支撑**：给出了MD-SVD的损失不等式证明，并有实验验证；同时通过KL敏感性分析（MKL）提供了部分RoPE选择的依据。
- **兼容性**：证明了与KV量化的无缝结合，拓展了实际应用范围。
- **实验严谨**：多条证据链（主结果、消融、分析）支撑结论，对比基准覆盖主流压缩方法。

#### 8. 不足与局限
- **模型覆盖有限**：仅验证了7B/8B规模，未涉及更大模型（如13B、72B）或更多VLM族（如InternVL、LLaVA-NeXT变体）。也未测试视频/多帧输入场景，其中KV缓存压力更大。
- **资源信息缺失**：未提供详细计算资源（GPU型号、数量），难以评估方法的实际部署门槛。
- **训练数据量极小**：虽然体现了数据高效，但恢复性能依赖于微调数据的质量；在长尾或领域专有任务上效果未知。
- **理论深度**：Theorem 2.1仅给出不等式，未分析最优解的具体形式或收敛性。
- **与原生MLA对比缺失**：未与从零训练的MLA模型进行性能对比（但论文目标并非胜过原生MLA，而是低成本转换）。
- **部分RoPE策略依赖超参数**：保留子空间数量r需要手动设定（受dkv决定），未提供自适应确定方法。

（完）
