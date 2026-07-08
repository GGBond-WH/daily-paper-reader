---
title: "DAVID: Dual-stage Adaptive Vision-text Integrated Decoupling for Multimodal KV Cache Eviction"
title_zh: DAVID：面向多模态KV缓存剔除的双阶段自适应视觉文本融合解耦
authors: "Yifeng Gu, Jianxiu Jin, Kailing Guo, Xiangmin Xu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39285/43246"
tags: ["query:llm-kv-cache"]
score: 8.0
evidence: 多模态KV缓存剔除，感知模态融合程度
tldr: 多模态大模型KV缓存管理面临挑战。DAVID通过分析视觉与文本令牌的特征融合程度，在浅层采用解耦剔除策略，深层采用超模态剔除策略，并设计轻量级指标支持动态切换，有效减少KV缓存内存同时保持多模态理解性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 多模态输入导致KV缓存巨大，现有策略未考虑层间模态融合差异。
method: 根据模态融合度在浅层解耦剔除、深层超模态剔除，通过轻量指标动态切换。
result: 在多模态任务中降低KV缓存使用，性能损失极小。
conclusion: 层间融合感知的剔除策略可优化多模态KV缓存管理。
---

## Abstract
With the rapid development of multimodal large language models (MLLMs), deploying them on low-resource devices remains challenging. Beyond the model size, long multimodal inputs cause substantial memory overhead in the KV cache, making efficient cache management critical. In this paper, we propose DAVID, a KV cache eviction strategy that adapts to the degree of modality fusion across layers. By analyzing the feature distributions of vision and text tokens, we observe low fusion in early layers and high fusion in deeper layers. Based on this observation, DAVID adopts a decoupled eviction strategy in shallow layers and a super-modal eviction strategy in deeper layers. To support this dynamic switching, we design a lightweight metric that quantifies cross-modal fusion and uses a threshold to determine which layers require decoupling. Experimental results show that DAVID achieves state-of-the-art performance on multiple benchmarks and offers a new perspective on KV cache eviction for MLLMs.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **问题**：多模态大语言模型（MLLMs）在低资源设备上部署面临挑战。除了模型规模本身，长多模态输入导致 KV 缓存内存开销巨大，高效缓存管理成为关键。
- **背景与动机**：现有 KV 缓存剔除策略（如 H2O、SnapKV）主要针对文本单模态设计，而多模态扩展方法（如 LOOK-M、MEDA）强行在所有层分离视觉与文本令牌并执行模态特定保留。然而，作者发现这种固定分离有时甚至不如单模态方法。根源在于 MLLMs 中模态融合是分层的：浅层视觉与文本令牌保持语义分离，深层则融合形成“超模态”表示。在融合阶段继续解耦会破坏上下文推理连贯性。因此需要根据层间融合程度自适应切换剔除策略。

## 2. 方法论：核心思想、关键技术细节、公式或算法流程

- **核心思想**：提出 Dual-stage Adaptive Vision-text Integrated Decoupling (DAVID)，根据层间模态融合度动态调整剔除模式。浅层（低融合）采用模态解耦剔除（分别对视觉和文本令牌计算重要性并保留）；深层（高融合）切换为统一剔除（类似单模态方法）。切换由轻量级指标——归一化跨模态注意力率（NCAR）控制。
- **关键技术细节**：
  - **解耦剔除**：对每个层，分别计算视觉和文本令牌的累积注意力分数（基于最近 r 个令牌的注意力之和），然后各自执行 Top-K 保留。保留比例 ρ = B_V / B_T 根据视觉和文本令牌数比例设置（实验中 OCRBench 和 MMVet 设为 2，MathVista 设为 1）。
  - **归一化跨模态注意力率 NCAR（θ）**：衡量某层中跨模态注意力强度。公式为 θ = (1/(nV * r)) * Σ_{i=n-r}^{n} Σ_{j∈V} A_{i,j}，其中 A_{i,j} 是注意力分数，V 为视觉令牌索引集。
  - **切换规则**：计算相邻层 NCAR 差异 Δ_i = θ_{i-1} - θ_i。当 Δ_i < 固定阈值 θ_T（实验中设为0.3）时，认为进入高融合阶段，停止解耦剔除，改为统一剔除（所有令牌一同评分，Top-K 保留）。
- **算法流程**（文字描述）：
  1. 输入总预算 B、最近缓存 Br、选择缓存 Bs、预算比例 ρ、融合阈值 θ_T。
  2. 初始化标志 Flag=True，θ_0=1。
  3. 遍历 LLM 的每一层 m：
     - 计算注意力矩阵 A^m。
     - 计算选择得分 S^m = 最近 r 个令牌的注意力之和。
     - 用 MaxPool 平滑得到 Ŝ^m。
     - 如果 Flag=True，计算该层 NCAR θ_m，计算 Δ_m = θ_{m-1} - θ_m。若 Δ_m < θ_T，设置 Flag=False。
     - 若 Flag=False（深层），对整个 Ŝ^m 执行 TopK 选择 B_s 个。
     - 若 Flag=True（浅层），分别对视觉和文本屏蔽后计算得分，根据预算比例 ρ 分配 B_V 和 B_T，各自 TopK 选择后拼接。
     - 保留最近 Br 个令牌，与选择的组合为最终 KV。
  4. 输出每层每头的保留索引。

## 3. 实验设计：数据集、基准、对比方法

- **数据集**：OCRBench（文本识别、场景VQA、文档VQA、KIE、HMER）、MMVet（REC、OCR、KNOW、GEN、SPAT、MATH）、MathVista（FQA、GPS、MWP、TQA、VQA等子任务）。均为开放生成式评测，OCRBench用准确率，MMVet和MathVista用GPT-4评分。
- **基准与对比方法**：
  - 文本类方法：H2O、SnapKV。
  - 多模态类方法：LOOK-M、CSP、MEDA。
  - 还包括全缓存（Full Cache）作为上界。
- **模型**：LLaVA-1.5-7B、LLaVA-1.5-13B、Qwen-VL（扩展实验）。
- **评估框架**：使用 VLMEvalKit。

## 4. 资源与算力

- **硬件**：单张 NVIDIA A800 80GB GPU 和 NVIDIA GeForce RTX 3090。
- **训练/推理**：论文未明确说明训练时长或预训练步骤（方法为训练后即插即用，无需重训练）。仅报告推理延迟和内存（见表5）：全缓存解码延迟 26.23ms/token，GPU内存1.26GB；DAVID（20%预算）延迟19.17ms/token，内存0.34GB。
- **算力成本**：本文方法只需单次推理，无需额外训练，算力需求较低。

## 5. 实验数量与充分性

- **主要实验**：在三个数据集（OCRBench、MMVet、MathVista）上进行主实验，每个数据集又包含多个子任务（如OCRBench含5个子任务，MMVet含6个，MathVista含12个）。模型大小包括7B和13B两个版本。
- **消融实验**：在OCRBench上对两个组件（解耦剔除DE和NCAR切换）进行消融，设置三种配置：无DE和NCAR、仅有DE、完整DAVID。
- **预算影响分析**：在MMVet上测试不同稀疏率（5%~60%）下的性能，画出曲线。
- **效率分析**：测量不同预算下的解码延迟和GPU内存。
- **兼容性实验**：将DAVID与H2O结合（基于H2O的DAVID），在OCRBench上验证。
- **跨模型扩展**：在Qwen-VL上以0.4稀疏率测试OCRBench和MMVet。
- **充分性评价**：实验覆盖了多模态常见场景、不同模型规模、不同预算、多种对比方法，消融和兼容性验证充分。对比方法选取了当前主流文本类和模态类方法，且重复5次取均值减少随机性。整体实验设计客观、公平。

## 6. 论文的主要结论与发现

- DAVID 在所有三个基准上均取得最优总分：OCRBench 7B 305分（全缓存308），13B 324分（全缓存325）；MMVet 7B 30.18（全缓存31.05），13B 31.28（全缓存32.20）；MathVista 7B 26.2（全缓存25.1），13B 27.5（全缓存26.6）。部分子任务甚至超过全缓存，说明剔除冗余视觉令牌可聚焦关键信息。
- 消融实验表明：单独使用解耦剔除（DE）提升3分，加入NCAR切换进一步提升7分，证明自适应切换的必要性。
- 在不同预算下，DAVID始终优于其他方法，尤其在60%稀疏率时超过全缓存。
- DAVID显著降低内存和加速推理（5%预算时内存降至0.18GB，延迟16.46ms/token）。
- DAVID可兼容文本类方法（如H2O），并成功推广到Qwen-VL模型。

## 7. 优点

- **创新性**：首次显式刻画层间模态融合过程并利用其指导剔除，将单模态与多模态剪枝范式桥接。
- **适应性**：动态切换机制无需手动指定切换层，由轻量指标NCAR自动判断。
- **即插即用**：无需重新训练或修改模型架构。
- **实验全面**：覆盖多种数据集、模型规模、预算比例，消融和兼容性验证充分，结果有统计稳定性。
- **效率高**：额外计算开销极小（仅需计算每层NCAR）。

## 8. 不足与局限

- **阈值固定**：NCAR切换阈值 θ_T=0.3 为固定值，可能不是最优，对于不同模型或任务可能需要调参。
- **预算比例 ρ 依赖先验**：浅层视觉与文本保留比例 ρ 需根据平均视觉/文本令牌数设定（OCRBench和MMVet用2，MathVista用1），缺乏自适应。
- **实验覆盖有限**：仅在LLaVA-1.5系列和Qwen-VL上验证，未在更大规模MLLM（如LLaVA-NeXT、Qwen2.5-VL）或混合专家模型上测试。未涉及视频或多图场景。
- **忽略其他压缩方向**：仅关注KV缓存剔除，未与视觉令牌压缩等技术正交结合（如FastV、Dynamic-LLaVA）。
- **潜在偏差**：NCAR基于最近r个令牌的注意力，假设近令牌代表性，可能不适用于长上下文某些场景。
- **应用限制**：方法依赖模型为“视觉令牌+文本令牌”拼接结构，对于完全并行处理的模型可能需调整。

（完）
