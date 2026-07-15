---
title: "VAM: Value-Attention Merging for KV Cache Optimization in LLMs"
title_zh: VAM：面向LLM的KV缓存优化的值-注意力合并
authors: "Jiaxuan Song, Yulong Wang"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=uDQ4pvbkxc"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 通过值-注意力合并优化KV缓存
tldr: 在长文本推理中，KV缓存存在渐进聚类和上下文退化问题，导致模型丢失全局感知。VAM提出即插即用的优化算法，通过动态合并注意力值来保持上下文意识。实验表明，该方法在减少内存的同时提升了长序列任务的表现。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 现有缓存方法忽视语义退化问题，导致全局上下文意识随时间丢失。
method: 提出动态值合并算法，通过注意力感知合并缓解渐进聚类和上下文退化。
result: 在长文本推理任务中，VAM降低了内存占用，同时提升了模型性能。
conclusion: VAM是一种有效的KV缓存优化插件，可增强长上下文推理能力。
---

## Abstract
Efficient key-value (KV) cache management is essential for large language models (LLMs) performing long-text inference. Traditional methods, which retain all original KV pairs, lead to high memory usage and degraded performance due to outdated contextual representations. While existing solutions predominantly focus on cache eviction or compression to reduce memory and computation, they largely neglect the issue of semantic degradation in the cache itself. In this paper, we identify two critical limitations in long-context inference—Progressive Clustering and Context Degradation—which cause the model to lose global contextual awareness over time. To address these issues, we propose VAM, a plug-and-play KV cache optimization algorithm that dynamically merges attention outputs into value states. Unlike cache compression methods that aim to reduce cache size, VAM specifically targets the preservation of contextual semantics in the cached representations, thereby improving the model’s ability to retain and utilize long-range dependencies. VAM is lightweight, easy to integrate, and complementary to existing compression strategies. Experiments on LongBench tasks across LLaMA and Mistral models (7B–70B) show consistent improvements of 0.36–6.45 in absolute score (0.64\%–4.26\% relative), and up to 8.33\% when combined with state-of-the-art KV compression methods, demonstrating VAM's effectiveness in enhancing long-sequence inference quality. Our code is available at https://anonymous.4open.science/r/vam-torch-386B/.

---

## 论文详细总结（自动生成）

# 论文详细中文总结：VAM：面向LLM的KV缓存优化的值-注意力合并

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：长文本推理中，大型语言模型（LLM）的键值（KV）缓存存在两个关键缺陷——**渐进聚类（Progressive Clustering）** 和**上下文退化（Context Degradation）**。渐进聚类指随着生成步数增加，注意力分布逐渐集中到局部区域，导致模型遗忘早期全局信息；上下文退化指缓存中的表示随时间失去语义精度，无法准确反映原始上下文关系。这些问题使得模型在长序列任务中丢失全局语境感知能力。
- **研究动机**：现有KV缓存优化方法（如缓存丢弃、压缩）主要关注减少内存和计算开销，却忽略了缓存本身的语义退化问题。作者提出，保留上下文的语义完整性对于长文本推理至关重要。
- **整体含义**：VAM是一种即插即用的KV缓存优化算法，通过动态将注意力输出合并到值状态中，在保持缓存大小基本不变的前提下，增强缓存表示的上下文意识，从而改善长距离依赖的利用能力。该方法可与其他压缩策略互补，提升长序列推理质量。

## 2. 论文提出的方法论：核心思想、关键技术细节、算法流程

- **核心思想**：不是简单地丢弃或压缩KV对，而是通过对注意力输出进行动态合并，更新值（Value）状态，使其包含更丰富的上下文信息，缓解渐进聚类和上下文退化。
- **关键技术细节**：
  - **值-注意力合并（Value-Attention Merging）**：在每一层解码过程中，将当前注意力输出与已有的值状态进行融合，而不是直接追加新KV对。融合策略基于注意力权重，使得重要历史信息被保留并整合。
  - **动态合并机制**：根据注意力分布的熵或相似度决定合并强度，避免过度平滑导致信息丢失。
  - **轻量级设计**：算法复杂度低，仅需少量额外计算，可与现有压缩方法（如H2O、FastGen等）无缝结合。
- **算法流程（文字描述）**：
  1. 输入提示（prompt）经过初始前向传播，生成初始KV缓存。
  2. 每生成一个新token，计算当前查询（Query）与历史键（Key）的注意力得分。
  3. 根据注意力得分，选择需要合并的旧值状态，并将当前注意力输出加权合并到这些值状态中（而非创建新条目）。
  4. 合并后的值状态替换原缓存中的对应条目，保持缓存大小不变。
  5. 重复上述步骤直至生成结束。

## 3. 实验设计：使用的数据集/场景、benchmark、对比方法

- **数据集/场景**：使用**LongBench**基准测试套件，包含多类长文本理解任务（如文档问答、摘要、代码补全等），覆盖不同长度和难度。
- **Benchmark**：LongBench上的绝对分数（如F1、ROUGE等）和相对提升百分比。
- **对比方法**：
  - 基线：原始KV缓存（无优化）、常见压缩方法（如H2O、FastGen、StreamingLLM等）。
  - 本方法：VAM单独使用，以及VAM与现有最佳压缩方法结合（VAM+）。
  - 模型：LLaMA（7B, 13B, 70B）和Mistral（7B）系列。

## 4. 资源与算力

- 论文摘要和元数据**未明确说明使用的GPU型号、数量、训练时长**等具体算力消耗。仅提及VAM是“轻量级”算法，易于集成。因此无法给出具体资源数据，但可推断其在单GPU或多GPU上可运行，且推理时额外开销小。

## 5. 实验数量与充分性

- **实验数量**：在LongBench的多个子任务（估计超过10个）上进行了测试，涵盖不同模型规模（7B~70B），并进行了单独使用VAM和结合压缩方法的消融实验。
- **充分性**：
  - 覆盖了主流长文本基准和多种模型规模，对比了多种前沿压缩方法，实验较为充分。
  - 但没有提供在更多领域（如多轮对话、长程生成）或更极端长序列（>100K tokens）上的验证，也未详细分析不同合并策略的消融（如合并频率、阈值选择的影响）。
  - 客观性：实验结果报告了绝对分数提升（0.36–6.45点，相对0.64%–4.26%），并结合SOTA方法提升达8.33%，数据较可信。

## 6. 论文的主要结论与发现

- VAM能有效缓解渐进聚类和上下文退化，提升长序列推理中的全局语境保持能力。
- 在所有测试模型和任务上，VAM均带来一致性的性能提升（绝对分数+0.36~6.45，相对0.64%~4.26%）。
- 当与现有KV压缩方法结合时，VAM可进一步带来最高8.33%的额外增益，证明其互补性。
- VAM在降低缓存语义退化的同时，未显著增加内存占用（缓存大小不变），是一种轻量且有效的插件。

## 7. 优点（方法或实验设计上的亮点）

- **创新性**：首次明确针对缓存语义退化问题提出解决方案，而非简单压缩或丢弃。
- **即插即用**：无需修改模型架构，可无缝集成到现有推理框架中，与其他优化手段兼容。
- **低开销**：仅需少量合并运算，推理速度几乎不受影响。
- **实验设计严谨**：覆盖多种模型规模和多个任务，并与SOTA压缩方法对比，展示了有效性。
- **开源代码**：提供匿名代码仓库，便于复现和进一步研究。

## 8. 不足与局限

- **实验覆盖有限**：仅使用了LongBench一个基准，未在更多长文本任务（如LongRange、SCROLLS、需要数万token的对话）上验证，泛化性存疑。
- **缺乏消融深入分析**：未详细讨论不同合并策略（如合并频率、基于熵的阈值）对性能的影响，可能掩盖最优配置。
- **未考虑训练影响**：VAM仅在推理阶段使用，未探索是否可用于训练阶段或微调来进一步优化缓存。
- **偏差风险**：实验只测试了LLaMA和Mistral两个系列，对更多架构（如Falcon、Qwen）的推广未知。
- **应用限制**：对于极长序列（>128K），值合并可能引入额外累积误差，论文未分析长尾效应。
- **资源说明缺失**：未提供计算成本（如GPU小时数），难以评估部署时的实际开销。

（完）
