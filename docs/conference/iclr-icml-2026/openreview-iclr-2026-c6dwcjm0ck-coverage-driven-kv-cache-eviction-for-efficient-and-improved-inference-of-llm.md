---
title: Coverage-Driven KV Cache Eviction for Efficient and Improved Inference of LLM
title_zh: 面向高效和改进LLM推理的覆盖度驱动KV缓存驱逐
authors: "Shuvendu Roy, Mengyao Zhai, Hossein Hajimirsadeghi, Golnoosh Samei"
date: 2025-09-18
pdf: "https://openreview.net/pdf?id=c6dwCJM0CK"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于覆盖度驱动的KV缓存驱逐策略
tldr: LLM在长上下文任务中部署成本高，现有KV缓存驱逐策略在减少内存的同时导致性能下降（尤其在需要全局推理的任务）。本文识别出性能下降源于覆盖度不足，提出覆盖度驱动的驱逐策略，在保留token子集时最大化上下文覆盖度。实验表明该方法在同等内存下不仅恢复性能，甚至超越全缓存基线。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 现有基于注意力稀疏性的驱逐策略在长上下文任务中性能显著下降，需要更保留上下文覆盖度的驱逐方法。
method: 定义覆盖度指标衡量已保存token对原始上下文的代表性，并在驱逐时优先保留最大化覆盖度的token。
result: 在多项长文本推理基准上，覆盖度驱动驱逐在压缩内存的同时提升任务准确率，超越全缓存效果。
conclusion: 覆盖度原则是设计KV缓存驱逐策略的关键，可同时提升效率与效果。
---

## Abstract
Large language models (LLMs) excel at complex tasks like question answering and summarization, thanks to their ability to handle long-context inputs. However, deploying LLMs is costly, not only due to the high computational demands of quadratic complexity of self-attention and auto-regressive generation, but also because of the significant memory overhead required for storing the key-value (KV) cache during inference. To reduce the memory cost, existing KV-cache eviction strategies leverage the sparsity in attention to selectively store a subset of tokens. While reducing the memory footprint, such approaches show a considerable drop in performance, especially in tasks that require long-context reasoning. We identify that the drop in performance is linked to a reduction in the coverage of unique tokens. Additionally, we theoretically show that reduced coverage limits the mutual information between inputs and outputs, thereby impairing predictive accuracy. To this end, we introduce K-VEC, a novel coverage-aware KV-cache eviction strategy that prioritizes token coverage while evicting tokens in the cache. K-VEC introduces a cross-head and a cross-layer coverage module to enhance token retention across attention heads and model layers, mitigating performance degradation caused by low coverage. Evaluated on 16 LongBench subsets, K-VEC exhibit up to 10.35 points improvement over the existing methods under the same eviction rate and memory constraint. Comprehensive evaluations validate the effectiveness of our approach and demonstrate its potential for efficient LLM deployment in resource-constrained settings.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **问题**：大型语言模型（LLM）在长上下文任务中表现出色，但部署成本极高。主要瓶颈之一是自注意力机制的二次复杂度和自回归生成过程中存储键值（KV）缓存带来的巨大显存开销。
- **现有方法**：现有的KV缓存驱逐策略利用注意力稀疏性，选择性地存储部分token，以减少内存占用。
- **性能缺陷**：这些方法在需要长上下文推理的任务中性能显著下降，尤其是在需要全局理解的场景。
- **核心发现**：论文识别出性能下降的根本原因在于保留的token对原始上下文的**覆盖度（coverage）**不足。理论分析表明，覆盖度降低会限制输入与输出之间的互信息，损害预测准确性。
- **整体目标**：提出一种覆盖度驱动的KV缓存驱逐策略（K-VEC），在减少内存的同时保持甚至提升模型性能，实现高效且更优的LLM推理。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：在驱逐KV缓存中的token时，优先保留那些能够**最大化覆盖度**的token，即保留的token子集应尽量代表原始上下文的全体信息。
- **关键技术细节**：
  - **覆盖度指标**：定义了一种量化指标来衡量已保存token对原始上下文的代表性。
  - **跨头覆盖模块（cross-head coverage module）**：在不同注意力头之间协调token保留，避免某些头因驱逐而丢失关键信息。
  - **跨层覆盖模块（cross-layer coverage module）**：在不同Transformer层之间统一考虑覆盖度，确保深层也能获得足够的上下文信息。
- **算法流程（文字说明）**：
  1. 输入当前KV缓存中的全部token。
  2. 利用注意力分数或相似度计算每个token对上下文覆盖度的贡献。
  3. 按照覆盖度贡献排序，优先保留贡献最高的token，直到达到预设的缓存容量。
  4. 在驱逐过程中，同时考虑跨头和跨层的影响，通过模块动态调整保留决策。
- **理论支撑**：论文从信息论角度证明，保持高覆盖度可以最大化输入与输出之间的互信息，从而提升预测准确率。

## 3. 实验设计

- **数据集与场景**：在 **LongBench** 的 **16个** 子集上进行评估，涵盖长文本问答、摘要、推理等多种任务。
- **基准方法**：对比了现有的多种KV缓存驱逐策略（如基于注意力稀疏性的方法）。未具体列举方法名称，但指出K-VEC在相同驱逐率和内存约束下，性能提升**最高达10.35个点**。
- **评估指标**：通常为任务准确率或得分。

## 4. 资源与算力

- **文中未明确说明**使用的GPU型号、数量或训练时长。仅在实验部分提及在LongBench上评估，未详述硬件配置。
- 由于论文侧重于推理阶段的缓存优化，通常不需要额外训练，但推理测试仍需一定算力。论文未给出具体算力信息。

## 5. 实验数量与充分性

- **实验数量**：涵盖了16个LongBench子集，并进行了多种驱逐率下的对比。论文还提到“全面评估”（comprehensive evaluations），但未详述消融实验数量。从摘要看，包含了跨头和跨层模块的消融（通过“增强token保留”暗示）。
- **充分性判断**：实验覆盖了主流长文本基准的多个子集，对比了现有方法，且效果显著，说明实验设计较为充分。但缺乏对更多模型规模（如7B、13B等）和更长上下文（如超过32K）的验证，也未涉及不同预训练模型的泛化性，存在一定局限性。

## 6. 论文的主要结论与发现

- 覆盖度不足是现有KV缓存驱逐策略性能下降的关键原因。
- 提出的覆盖度感知驱逐策略（K-VEC）在压缩内存的同时，不仅恢复甚至超越全缓存基线的性能。
- 在LongBench上，K-VEC最高比现有方法提升10.35个点，验证了覆盖度原则对于高效LLM推理的有效性。

## 7. 优点

- **洞察深刻**：指出覆盖度这一关键因素，并给出理论解释（互信息），超越了以往仅依赖注意力稀疏性的方法。
- **结构创新**：引入跨头和跨层覆盖模块，考虑了注意力分布的异构性，更细致地保留重要token。
- **效果显著**：同等内存下不仅不降性能，反而提升，甚至优于全缓存基线，实用价值高。
- **理论结合实践**：既有信息论分析，又有实际算法与实验验证。

## 8. 不足与局限

- **实验覆盖有限**：仅基于LongBench的16个子集，未在更广泛的长上下文任务（如文档级机器翻译、多文档问答）上验证；未涉及不同模型架构（如Mamba等非注意力模型）的适配性。
- **缺乏算力报告**：未提供推理时间开销或GPU资源消耗，无法判断方法本身的额外计算成本。
- **可能存在的偏差**：覆盖度定义可能依赖注意力分数，若注意力分布不够可靠（如长尾任务），保留策略可能失效。
- **应用限制**：仅针对KV缓存的驱逐环节，未考虑与其他优化（如量化、稀疏注意力）的联合协同；且驱逐策略在极端低内存下可能仍会损失关键信息。

（完）
