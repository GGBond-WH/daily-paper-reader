---
title: Efficient Multimodal Large Language Model via Dynamic KV Cache Quantization
title_zh: 通过动态KV缓存量化实现高效多模态大语言模型
authors: "Jiahao Fan, Chien-Ming Chen"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39241/43202"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 多模态大语言模型的动态KV缓存量化
tldr: 多模态大语言模型部署中KV缓存占用大量内存，本文提出动态KV缓存量化方法。针对键（K）和值（V）的不同分布特性，对K进行逐通道量化、对V进行逐令牌量化，并引入自适应令牌分配机制。实验在图文理解等任务上表明，该方法在几乎不损失性能的前提下将KV缓存内存减少约4倍，促进了资源受限设备上的实时部署。该工作将混合精度量化定制化地应用于多模态场景。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 多模态大模型推理时KV缓存内存占用大，且现有量化策略未针对键值分布差异进行优化。
method: 对K采用逐通道量化、V采用逐令牌量化，并设计自适应令牌分配以平衡精度与压缩率。
result: 在COCO Caption等基准上，量化后模型性能与全精度相当，缓存内存降低至1/4。
conclusion: 针对键值分布特性的动态量化可有效压缩多模态大模型KV缓存，利于边缘部署。
---

## Abstract
Multimodal large language models (LMMs) have demonstrated remarkable capabilities across diverse vision-language tasks, including image captioning, visual question answering, and text-image retrieval. However, their computational complexity and memory footprint, particularly in the key-value (KV) cache during inference, pose significant challenges for real-time deployment, especially on resource-constrained devices. In this paper, we propose Dynamic KV Cache Quantization, a novel quantization strategy tailored for multimodal LMMs. Our approach applies per-channel quantization to (K) and per-token quantization to (V), leveraging their respective statistical distributions to optimize precision allocation. Additionally, we introduce an adaptive token and channel recording mechanism that dynamically adjusts quantization parameters based on real-time distribution tracking, effectively mitigating the impact of outliers. To further enhance compression efficiency, we implement fine-grained grouping, which partitions KV tensors into localized subgroups, enabling more adaptive quantization. Experimental results on LLaVA-1.5 (7B/13B) and Qwen-VL across multiple multimodal benchmarks demonstrate that our method significantly outperforms existing KV-cache quantization approaches, achieving a superior trade-off between memory efficiency and model accuracy.

---

## 论文详细总结（自动生成）

# 论文中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：多模态大语言模型（LMMs）在推理时，KV 缓存（key-value cache）随序列长度线性增长，导致极大的内存占用和带宽需求，严重阻碍了在资源受限设备（如边缘设备、有限内存 GPU）上的实时部署。
- **现有局限**：现有的 KV 缓存量化技术主要针对纯文本 LLMs 设计，假设 KV 值分布相对平滑。但在多模态场景中，视觉特征引入大量异常值和高方差，导致传统量化方法（如均匀量化、KIVI、KVQuant 等）在多模态任务上出现显著性能下降。
- **本文目标**：提出一种专门针对多模态 LMMs 的动态 KV 缓存量化框架，在保持推理精度的前提下大幅降低内存占用，实现高效部署。

## 2. 论文提出的方法论

### 核心思想
基于对键（K）和值（V）统计分布差异的观察，分别采用不同的量化粒度：K 采用**逐通道量化**，V 采用**逐令牌量化**；并引入**动态令牌与通道记录机制**及**细粒度分组**，自适应处理异常值，优化压缩精度。

### 关键技术细节
- **动态 KV 缓存量化**：
  - 对 K 缓存：逐通道计算缩放因子 \( s_{K_c} \) 和零点 \( z_{K_c} \)，独立量化每个通道。
  - 对 V 缓存：逐令牌计算缩放因子 \( s_{V_t} \) 和零点 \( z_{V_t} \)，独立量化每个令牌。
  - 公式示例（以 K 为例）：
    \[
    s_{K_c} = \frac{\max(K_c) - \min(K_c)}{2^B - 1}, \quad z_{K_c} = \min(K_c)
    \]
    \[
    Q(K_c) = \left\lfloor \frac{K_c - z_{K_c}}{s_{K_c}} \right\rceil, \quad K'_c = Q(K_c) \cdot s_{K_c} + z_{K_c}
    \]
- **令牌与通道记录器及缩放**：
  - 使用指数移动平均（EMA）在线跟踪每个通道/令牌的均值和方差，更新公式如：
    \[
    \mu_K^{(t)} = \lambda \mu_K^{(t-1)} + (1-\lambda) \mathbb{E}[K^{(t)}]
    \]
    \[
    \sigma_K^{(t)} = \lambda \sigma_K^{(t-1)} + (1-\lambda) \sqrt{ \mathbb{E}[(K^{(t)} - \mu_K^{(t)})^2] }
    \]
  - 根据统计量计算缩放向量 \( S_K = 1/(\sigma_K + \epsilon) \)，对值进行归一化，再通过可学习的参数 \( \alpha, \beta \) 自动调整缩放平衡（auto-relaxation）。
- **细粒度分组**：
  - 将 KV 张量沿序列维度（逐通道分组）或模型维度（逐令牌分组）划分为局部子组。
  - 根据每个维度上的异常值比例动态分配组数 \( G_d \)，使高方差区域获得更细的量化粒度。
  - 每个子组独立拥有缩放因子和零点，避免全局异常值主导量化范围。
  - 量化后内存复杂度从 \( O(L \times d \times b_{fp}) \) 降至 \( O(L \times d \times B + G \times b_{fp}) \)。

（算法流程已在第3节详细描述，此处用文字概括。）

## 3. 实验设计

### 使用数据集/场景
- **基准任务**：VQAv2（视觉问答）、TextVQA（图像中文本理解）、GQA（组合推理）、ScienceQA（科学推理）、POPE（幻觉鲁棒性）。
- **场景**：多模态理解与推理，包括开放域问答、知识推理、文本读取、幻觉检测。

### 对比方法
- 基线：全精度（FP16）模型。
- 对比方法：Uniform（均匀量化）、SKVQ、KIVI、KVQuant、MiKV、ZIPVL。
- 所有方法均采用 4-bit 量化（部分消融实验中也包含其他位宽）。

### 模型
- LLaVA-1.5 (7B/13B)、Qwen-VL。

### 实验组数
- 主要结果：3 个表格（Table 1-3），分别针对 LLaVA-1.5-7B、LLaVA-1.5-13B、Qwen-VL 的 4-bit 量化性能。
- 消融研究：4 个表格（Table 4-6），分析组件贡献、分组超参数（g=16/32/64/128）、K/V 量化策略对比。
- 讨论部分还包含吞吐量、内存减少、延迟等额外效率评估。

## 4. 资源与算力

- **硬件环境**：Ubuntu 20.04.3，64 核 CPU，64GB RAM，单张 NVIDIA A100（80GB VRAM）。额外验证使用单张 NVIDIA V100（32GB VRAM）。
- **训练时长**：论文明确说明**无需额外微调**，所有量化方法均直接在预训练模型上进行推理时应用。因此无训练时长。文中也未报告量化过程的耗时。
- **内存与速度**：在 A100 上对 LLaVA-1.5-7B 实现 2.4× 速度提升和 3.8× 内存减少；在 V100 上 Qwen-VL 吞吐量达 128 tokens/s（未量化仅 76 tokens/s）；在 T4 上达 135 tokens/s（未量化 58 tokens/s）。存储开销仅增加 0.07%。

## 5. 实验数量与充分性

- **实验数量**：共 3 个主表、4 个消融表，覆盖 5 个基准、3 个模型、7 种对比方法。消融研究涵盖组件贡献、分组大小、K/V 量化策略三大维度。
- **充分性与公平性**：
  - 对比方法均为近期 SOTA；实验在同一硬件软件环境下运行，保持一致性。
  - 评估指标为常用准确率，结果清晰。
  - 但**缺少 8-bit 或 3-bit 量化对比**（仅在讨论中提及 3-bit 表现），且未报告方差或多次运行结果。另外，仅在 LLM 级别验证，未在更多样化的多模态架构（如 BLIP-2、InstructBLIP）上测试。
- **客观性**：作者承认所有方法均保持相同设置，差异直接归因于量化方法，结论可信。

## 6. 论文的主要结论与发现

- 所提出的动态 KV 缓存量化方法在 4-bit 下，在所有基准任务上**显著优于**现有方法，性能最接近全精度基线（差距 < 1%）。
- 对于 LLaVA-1.5-7B，在 TextVQA 上达到 58.01%（基线 58.19%），GQA 上 62.13%（基线 61.93%），甚至略有提升。
- 在 Qwen-VL 上同样保持高精度，表现与 KIVI 相当且略优。
- 消融实验表明：细粒度分组和记录器+缩放两个组件都至关重要；分组大小 g=32 为最优平衡点；逐令牌量化优于逐通道量化。
- 效率方面：实现 2.4× 速度提升、3.8× 内存节省，推理吞吐量显著提高，支持在 V100/T4 上部署大模型。

## 7. 优点

- **原创性**：首次系统性针对多模态 KV 缓存中 K/V 分布差异设计不同量化粒度，并提供动态自适应机制。
- **有效性**：在多个模型和基准上一致领先，无需微调，即插即用。
- **实现细节**：EMA 跟踪、异常值感知分组、可学习缩放参数等设计新颖且实用。
- **效率验证全面**：不仅报告准确率，还测量延迟、吞吐量、内存占用，并在不同 GPU（A100/V100/T4）上验证了部署可行性。
- **消融充分**：逐一剥离组件并分析超参数，证明各模块必要性。

## 8. 不足与局限

- **实验覆盖不足**：未在更大规模模型（如 LLaVA-NeXT-13B 以上）或更多多模态架构（如 BLIP-2、InstructBLIP）上进行测试，泛化性存疑。
- **量化位宽单一**：主要展示 4-bit 结果，3-bit/8-bit 仅作为辅助提及，缺乏系统对比。
- **偏差风险**：实验仅使用 LLaVA 和 Qwen-VL，可能偏向于这些模型的分布特性，对其他架构（如融合式多模态模型）的适用性未验证。
- **应用限制**：动态记录和分组引入额外计算开销，虽然文中声称增加很小（0.07% 存储），但端到端延迟增加未与纯量化方法直接对比；实时性要求极高的场景可能需要权衡。
- **超参数固定**：EMA 衰减因子 λ、异常值阈值 τ 等均需手动设定（文中给出经验值），未探讨自适应调整方法，可能存在数据集依赖性。
- **缺乏统计显著性和多次运行**：未报告多次实验的标准差或置信区间，无法评估结果稳定性。

（完）
