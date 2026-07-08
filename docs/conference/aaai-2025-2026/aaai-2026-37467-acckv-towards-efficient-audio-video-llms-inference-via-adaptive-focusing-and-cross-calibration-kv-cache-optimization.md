---
title: "AccKV: Towards Efficient Audio-Video LLMs Inference via Adaptive-Focusing and Cross-Calibration KV Cache Optimization"
title_zh: AccKV：通过自适应聚焦与跨校准实现高效音视频大模型推理
authors: "Zhonghua Jiang, Kui Chen, Kunxi Li, Keting Yin, Yiyun Zhou, Zhaode Wang, Chengfei Lv, Shengyu Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/37467/41429"
tags: ["query:llm-kv-cache"]
score: 8.0
evidence: 面向音视频大模型的KV缓存优化
tldr: 音视频大模型中，KV缓存因长时序列而显著增大。AccKV提出自适应聚焦与跨校准优化策略，根据层间注意力分布动态调整不同模态的KV缓存保留，并通过跨模态校准减少冗余，有效降低内存消耗同时保持多模态对话性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 音视频模态导致KV缓存巨大，现有方法难以自适应分配缓存资源。
method: 提出自适应聚焦与跨校准KV缓存优化，根据层间注意力模态偏好动态保留关键KV。
result: 在音视频问答任务中减少KV缓存使用，推理效率提升。
conclusion: 跨模态注意力分析可指导KV缓存高效管理。
---

## Abstract
Recent advancements in Audio-Video Large Language Models (AV-LLMs) have enhanced their capabilities in tasks like audio-visual question answering and multimodal dialog systems. Video and audio introduce an extended temporal dimension, resulting in a larger key-value (KV) cache compared to static image embedding. A naive optimization strategy is to selectively focus on and retain KV caches of audio or video based on task. However, in the experiment, we observed that the attention of AV-LLMs to various modalities in the high layers is not strictly dependent on the task. In higher layers, the attention of AV-LLMs shifts more towards the video modality. In addition, we also found that directly integrating temporal KV of audio and spatial-temporal KV of video may lead to information confusion and significant performance degradation of AV-LLMs. If audio and video are processed indiscriminately, it may also lead to excessive compression or reservation of a certain modality, thereby disrupting the alignment between modalities. To address these challenges, we propose AccKV, an Adaptive-Focusing and Cross-Calibration KV cache optimization framework designed specifically for efficient AV-LLMs inference. Our method is based on layer adaptive focusing technology, selectively focusing on key modalities according to the characteristics of different layers, and enhances the recognition of heavy hitter tokens through attention redistribution. In addition, we propose a Cross-Calibration technique that first integrates inefficient KV caches within the audio and video modalities, and then aligns low-priority modality with high-priority modality to selectively evict KV cache of low-priority modality. The experimental results show that AccKV can significantly improve the computational efficiency of AV-LLMs while maintaining accuracy.

---

## 论文详细总结（自动生成）

### 论文总结：AccKV: Towards Efficient Audio-Video LLMs Inference via Adaptive-Focusing and Cross-Calibration KV Cache Optimization

#### 1. 核心问题与整体含义（研究动机和背景）
- **问题**：音视频大语言模型（AV-LLMs）推理时，由于视频和音频数据包含长时空序列，其 Key-Value (KV) 缓存量远大于静态图像，导致显著的内存和计算开销。现有 KV 缓存优化方法（如 H2O、SnapKV）主要针对纯文本或纯视觉场景，未能适应音视频模态的独特注意力模式（如高层注意力向视频收敛）以及跨模态信息融合冲突。
- **动机**：分析 AV-LLMs 的注意力分布，发现三个关键现象：
  - **注意力收敛**：在高层中，无论任务类型（音频QA或视频描述），模型注意力均偏向视频模态。
  - **异质模态合并冲突**：直接合并音频（时间特征）和视频（时空特征）的 KV 缓存会导致信息混淆。
  - **过度压缩/保留陷阱**：无差别处理音视频会破坏模态间的对齐与同步（如语音与唇动）。
- **目标**：设计一种兼顾模态注意力动态与跨模态对齐的 KV 缓存优化框架，在保持模型性能的同时大幅降低内存占用。

#### 2. 论文提出的方法论
- **核心思想**：通过**层自适应聚焦（Adaptive-Focusing）** 动态确定每层关键模态，结合**注意力重分配（Attention Redistribution）** 缓解早期 token 偏差，再引入**跨校准（Cross-Calibration）** 技术先进行模态内冗余合并，再以高优先模态为锚点对齐并丢弃低优先模态的不相关 KV。
- **关键技术细节**：
  - **注意力重分配**：针对注意力矩阵的低三角特性，对每个位置 \((i,j)\) 的注意力分数乘以权重 \(\frac{i+1}{l-j}\)，降低早期 token 的累积优势，重新分配注意力权重。
  - **自适应聚焦**：计算每层中文本 token 对所有视频 token 和音频 token 的平均累积注意力分数 \(\bar{S}_v\) 和 \(\bar{S}_a\)，归一化得到模态优先级权重 \(W_v, W_a\)，并据此加权调整该层中不同模态的注意力分数。
  - **模态内合并（Intra-modal Merging）**：基于累积注意力分数分别从视频和音频 token 中选出 top-k 重要 token，对各自模态内不重要的 token 取平均合并，生成压缩后的 KV 表示。
  - **模态间跨校准**：以高优先模态的 K 向量为锚点，计算低优先模态各 token 的 K 向量与所有高优先模态 K 向量的平均余弦相似度，设置阈值 \(\tau\)，丢弃低于阈值的低优先模态 KV cache，保留与其对齐的部分。
- **算法流程**（文字说明）：
  1. 对注意力矩阵进行重分配。
  2. 根据当前层计算模态优先级权重 \(W_v, W_a\)，加权调整注意力分数。
  3. 对每个模态分别选出 top-k 重要 token，合并剩余 token。
  4. 确定高优先模态（如视频），计算低优先模态（如音频）的 K 向量与高优先模态 K 向量的余弦相似度均值。
  5. 保留相似度高于阈值 \(\tau\) 的低优先模态 KV，与高优先模态 KV 及文本 KV 拼接形成最终缓存。

#### 3. 实验设计
- **数据集**：
  - **MVBench**：包含 20 个复杂视频理解任务，论文选取其中 10 个同时涉及音频和视频的子任务（Action Sequence, Action Prediction, Unexpected Action, Object Interaction, Object Shuffle, Action Localization, Scene Transition, Action Count, State Change, Character Order）。
  - **AVSD**（Audio-Visual Scene Aware Dialogue）：对话理解任务，使用 ROUGE 作为评估指标。
- **Benchmark与对比方法**：
  - 对比 **H2O** 和 **SnapKV**（纯文本 LLM 的 KV 优化方法），**LOOK-M** 和 **FastV**（纯视觉多模态优化方法）。
- **模型架构**：
  - **VideoLLaMA2**（语言解码器基于 Qwen2-7B-Instruct）
  - **AVicuna**（语言解码器基于 Vicuna-7B-v1.5）
- **评估指标**：各子任务准确率（MVBench）及 ROUGE（AVSD）。

#### 4. 资源与算力
- 文中明确说明：实验在 **NVIDIA A100 (40GB 内存)** 上进行。
- 未提及 GPU 数量、训练时长、具体运行时间等细节。

#### 5. 实验数量与充分性
- **实验组数**：超过 40 组主要对比（表 4、表 5 各包含 11 个任务×5 方法×2 种 cache budget = 至少 110 组数据点；另有消融实验表 6、影响分析表 1-3）。
- **消融实验**：针对三个关键组件（注意力重分配 A-R、自适应聚焦 A-F、跨校准 C-C）进行逐一移除实验，验证各自贡献。
- **不同 cache budget**：VideoLLaMA2 上测试 20% 和 10% 缓存预算；AVicuna 上测试 160 tokens 和 120 tokens 两种预算。
- **充分性评估**：实验覆盖了不同模型、不同压缩率、不同模态任务、多数据集，且与多种基线公平对比，结果稳定。消融实验证明了每个组件的有效性。实验设计较为全面、客观。

#### 6. 论文的主要结论与发现
- AccKV 在多数任务上优于所有对比方法，能在仅损失少量准确率的情况下实现 **80%-90% 的 KV 缓存压缩**。
- 验证了新发现的注意力收敛现象：高层中视频模态始终获得更高注意力，无论任务类型。
- 直接合并不同模态 KV 缓存会严重损害性能（表 2 显示性能从 0.70 降至 0.17）。
- 跨模态对齐至关重要：即使仅丢弃一种模态的完整缓存都会导致性能下降（表 3）。
- 注意力重分配可有效修正早期 token 的累积偏差，提升重要 token 识别准确度。

#### 7. 优点
- **现象驱动的方法设计**：基于对 AV-LLMs 注意力模式的深入观察（收敛、合并冲突、过度压缩陷阱），提出针对性解决方案，而非简单迁移现有方法。
- **多层次优化**：分别在 token 级（注意力重分配）、模态内（合并）、模态间（对齐）三个层面协同优化，结构清晰。
- **跨模态对齐机制**：利用余弦相似度以高优先模态为锚点进行软对齐，避免了硬性丢弃导致的模态失衡。
- **鲁棒性**：在多种缓存预算下均保持稳定性能，特别是在极低预算（如 10% 或 120 tokens）时仍优于其他方法。
- **通用性**：在两个不同架构（Qwen2、Vicuna）的 AV-LLMs 上均验证有效。

#### 8. 不足与局限
- **实验覆盖范围有限**：仅测试了 7B 参数规模的模型，未在更大模型（如 13B、72B）或更小模型上验证；数据集仅包含 MVBench 和 AVSD，缺乏更多样化的音视频任务（如长视频问答、实时对话等）。
- **阈值依赖**：跨校准阶段的阈值 \(\tau\) 需要手动设定（文中实验了 0.9 和 0.6），不同任务可能最优值不同，缺乏自动调节机制。
- **计算开销未详细分析**：虽然 AccKV 减少了 KV 缓存空间，但额外的注意力重分配、模态内合并、跨模态相似度计算本身也引入了一定计算量，文中未对比实际推理延迟或 FLOPs 变化。
- **偏置风险**：注意力重分配假设最近 token 更重要，但在某些任务（如需要回顾早期视觉信息的场景）中可能错误压制早期 token。
- **应用限制**：方法依赖于模型的注意力模式，若未来 AV-LLMs 架构改变（例如采用非自注意力机制），AccKV 可能需要重新适配。

（完）
