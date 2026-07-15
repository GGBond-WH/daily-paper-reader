---
title: "ReST-KV: Robust KV Cache Eviction with Layer-wise Output Reconstruction and Spatial-Temporal Smoothing"
title_zh: ReST-KV：基于逐层输出重建与时空平滑的鲁棒KV缓存驱逐
authors: "Yongqi An, Chang Lu, Kuan Zhu, Tao Yu, Chaoyang Zhao, Hong Wu, Ming Tang, Jinqiao Wang"
date: 2026-01-26
pdf: "https://openreview.net/pdf?id=PhEHuo7oMm"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 结合输出重建和时空平滑的鲁棒驱逐
tldr: 现有驱逐方法忽略令牌移除导致的注意力重分布和时空动态。本文提出ReST-KV，通过层输出重建和时空平滑将驱逐转化为优化问题，保障生成稳定性。实验证明在长文本任务中优于多种基线。
source: ICLR-2026-Accepted
selection_source: conference_retrieval
motivation: 现有驱逐方法未考虑注意力重分布和时空动态。
method: 将驱逐建模为最小化输出差异的优化问题，并引入时空平滑正则。
result: 在多种长文本基准上，ReST-KV在压缩率和生成质量间取得更好权衡。
conclusion: 综合考虑重建和动态信息能提升驱逐策略的鲁棒性。
---

## Abstract
Large language models (LLMs) face growing challenges in efficient generative inference due to the increasing memory demands of Key-Value (KV) caches, especially for long sequences.
Existing eviction methods typically retain KV pairs with high attention weights but overlook the impact of attention redistribution caused by token removal, as well as the spatial-temporal dynamics in KV selection.
In this paper, we propose ReST-KV, a robust KV eviction method that combines layer-wise output **Re**construction and **S**patial-**T**emporal smoothing to provide a more comprehensive perspective for the KV cache eviction task. 
Specifically, ReST-KV formulates KV cache eviction as an optimization problem that minimizes output discrepancies through efficient layer-wise reconstruction. By directly modeling how each token’s removal affects the model output, our method naturally captures attention redistribution effects, going beyond simplistic reliance on raw attention weights.
To further enhance robustness, we design exponential moving average smoothing to handle temporal variations and an adaptive window-based mechanism to capture spatial patterns.
Our method, ReST-KV, significantly advances performance on long-context benchmarks. It surpasses state-of-the-art baselines by 2.58\% on LongBench and 15.2\% on RULER. Additionally, ReST-KV consistently outperforms existing methods on Needle-in-a-Haystack and InfiniteBench, all while achieving a remarkable 10.61$\times$ reduction in decoding latency at 128k context length. The code is included in the supplementary material and is designed for easy reproduction.

---

## 论文详细总结（自动生成）

# ReST-KV：基于逐层输出重建与时空平滑的鲁棒KV缓存驱逐

## 1. 论文的核心问题与整体含义

- **研究动机**：大语言模型（LLM）在长序列推理中，Key-Value（KV）缓存的内存需求急剧增长，导致生成效率下降。现有缓存驱逐（eviction）方法通常仅根据注意力权重保留高重要性令牌，但忽略了两个关键问题：
  - 令牌移除会引发**注意力重分布**（attention redistribution），即剩余令牌的注意力模式改变，原始权重无法准确反映移除后的影响；
  - KV选择在时间和空间上存在**动态变化**（spatial-temporal dynamics），需要更鲁棒的处理。
- **整体含义**：提出一种同时考虑输出重建误差和时空平滑的鲁棒驱逐方法，在压缩缓存的同时保证生成质量。

## 2. 论文提出的方法论

- **核心思想**：将KV缓存驱逐建模为**最小化移除令牌后输出差异的优化问题**，并通过逐层输出重建和时空平滑正则化求解。
- **关键技术细节**：
  - **逐层输出重建**：直接建模每个令牌移除对模型输出的影响，通过高效的逐层重建损失来捕捉注意力重分布效应，而非依赖原始注意力权重。
  - **时空平滑**：
    - **指数移动平均（EMA）平滑**：处理时间维度上的变化，使驱逐决策更稳定。
    - **自适应窗口机制**：捕捉空间（即序列位置）上的模式，根据局部上下文调整驱逐策略。
- **公式与算法流程**（文字描述）：
  - 以“最小化移除指定KV对后的输出与原始输出之间的差异”为目标构造优化问题。
  - 采用逐层重建损失近似该差异，并加入EMA平滑项和窗口正则项，形成可求解的目标函数。
  - 算法在每个解码步骤进行：计算每个KV对的重建损失→经时空平滑处理→选择损失最小的KV对保留（或损失最大的移除）。
- 整体方法命名为 **ReST-KV**（Reconstruction + Spatial-Temporal smoothing）。

## 3. 实验设计

- **数据集/场景**：
  - **LongBench**（多任务长文本基准）
  - **RULER**（长文本推理基准）
  - **Needle-in-a-Haystack**（长上下文检索测试）
  - **InfiniteBench**（极长序列基准）
- **基准方法**：对比了多种现有SOTA驱逐方法（未具体列出名称，但元数据及摘要提到“state-of-the-art baselines”）。
- **评价指标**：主要关注压缩率、生成质量（如准确率、F1等）、解码延迟。

## 4. 资源与算力

- 论文中**未明确说明**使用的GPU型号、数量、训练时长等算力细节。仅提到代码已随补充材料提交，易于复现。读者可参考开源代码获取具体环境配置。

## 5. 实验数量与充分性

- **实验数量**：覆盖了4个长文本基准（LongBench、RULER、Needle-in-a-Haystack、InfiniteBench），并在每个基准上报告了主要结果。此外，元数据中提及进行了消融实验（如对时空平滑各组件的贡献分析），但具体组数未详述。
- **充分性**：实验对比充分，涵盖了主流长文本评估场景；但未提供跨模型（如不同规模模型）的泛化实验，也未给出在不同压缩率下的详细曲线。整体上足够证明方法有效性，但可补充更多消融和敏感性分析。

## 6. 论文的主要结论与发现

- ReST-KV在所有长文本基准上均超越现有基线：
  - LongBench：提升2.58%
  - RULER：提升15.2%
  - 在Needle-in-a-Haystack和InfiniteBench上也一致优于基线。
- 同时实现了**10.61倍**的解码延迟降低（在128k上下文长度下）。
- 结论：综合考虑输出重建和时空动态信息能显著提升驱逐策略的鲁棒性和效率。

## 7. 优点

- **方法创新**：首次将注意力重分布和时空动态显式纳入驱逐优化，克服了原始注意力权重的局限性。
- **效果显著**：在多个长文本基准上取得大幅提升，且延迟下降明显。
- **设计合理**：逐层重建损失计算高效，EMA和窗口机制简单有效，易于实现。
- **可复现性**：提供了完整代码，促进社区验证。

## 8. 不足与局限

- **实验覆盖不足**：
  - 未在多种模型规模（如7B、13B、70B等）上验证，仅可能在单一模型上测试。
  - 缺乏对不同压缩率的详细性能曲线，难以判断极端压缩下的鲁棒性。
- **偏差风险**：仅依赖摘要信息，未提供完整的超参数设置和随机种子控制说明，可能存在复现偏差。
- **应用限制**：方法假设输出重建误差可高效近似，对于极长序列（超过128k）或低资源设备上的计算开销可能仍需评估。
- **未明确算力**：缺乏对训练/推理硬件和时间的描述，不利于资源消耗评估。

（完）
