---
title: "KeepKV: Achieving Periodic Lossless KV Cache Compression for Efficient LLM Inference"
title_zh: "KeepKV: 实现周期性无损KV缓存压缩以高效推理LLM"
authors: "Yuxuan Tian, Zihan Wang, Yebo Peng, Aomufei Yuan, Zhiming Wang, Bairen Yi, Xin Liu, Yong Cui, Tong Yang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40611/44572"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 通过自适应合并实现无损KV缓存压缩
tldr: LLM推理中KV缓存不断增长导致内存瓶颈。现有压缩方法如驱逐或合并会引入信息损失或注意力分布不一致。本文提出KeepKV，一种自适应KV缓存合并方法，在严格内存约束下通过智能合并保留更多信息，同时保持注意力分布一致性。实验表明在相同内存预算下，KeepKV比现有方法生成质量更高，且无额外推理开销。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存压缩方法导致信息丢失或注意力分布不一致，影响生成质量。
method: 提出自适应KV缓存合并策略，在合并时保持注意力分布一致性。
result: 在严格内存约束下保持生成质量，优于现有压缩方法。
conclusion: KeepKV实现了无损压缩，为LLM高效推理提供可行方案。
---

## Abstract
Efficient inference of large language models (LLMs) is hindered by an ever-growing key-value (KV) cache, making KV cache compression a critical research direction. Traditional methods selectively evict less important KV cache entries, which leads to information loss and hallucinations. Recently, merging-based strategies have been explored to retain more information by merging KV pairs that would be discarded; however, these existing approaches inevitably introduce inconsistencies in attention distributions before and after merging, causing degraded generation quality. To overcome this challenge, we propose KeepKV , a novel adaptive KV cache merging method designed to preserve performance under strict memory constraints, achieving single-step lossless compression and providing error bounds for multi-step compression. KeepKV introduces the Electoral Votes mechanism that records merging history and adaptively adjusts attention scores. Moreover, it further leverages a novel Zero Inference-Perturbation Merging method, compensating for attention loss resulting from cache merging. Extensive experiments on various benchmarks and LLM architectures demonstrate that KeepKV substantially reduces memory usage while successfully retaining essential context information, achieving over 2 times inference throughput improvement and maintaining superior generation quality even with only 10% KV cache budgets.

---

## 论文详细总结（自动生成）

# 论文总结：KeepKV: Achieving Periodic Lossless KV Cache Compression for Efficient LLM Inference

## 1. 核心问题与整体含义（研究动机与背景）
- **问题背景**：Transformer大语言模型（LLM）推理时，KV缓存随上下文长度增长而迅速膨胀，成为主要内存瓶颈（例如LLaMA-3-70B在批量128、8K上下文时需要320GB KV缓存）。
- **现有方法局限**：
  - **驱逐方法**（如StreamingLLM、H2O、PyramidInfer）：丢弃不重要的KV对，导致信息永久丢失和幻觉。
  - **合并方法**（如CaM、D2O、KVMerger）：通过加权合并保更多信息，但会引入“注意力凹陷”（Attention Sag）——合并后注意力分数低于合并前原分数之和，造成输出扰动。
- **研究目标**：提出一种自适应、理论上无损的KV缓存合并方法，在严格内存约束下保持生成质量，并首次实现**单步无损压缩**和**多步压缩的误差界**。

## 2. 方法论：核心思想、关键技术细节
- **整体设计**：KeepKV包含两大创新组件——**Electoral Votes（选举票数）** 和 **Zero Inference-Perturbation Merging（ZIP-Merging）**，并通过EMA（指数移动平均）预测注意力分数以扩展到多步生成。

### 关键技术细节
1. **Electoral Votes机制**：
   - 记录每个KV对被合并的次数（初始为1），类似于选举人团制度。
   - 合并后，新KV对的票数为原两票之和 (\(p_r = p_e + p_c\))。
   - 注意力计算时，每个KV对的得分乘以票数，从而近似还原合并前多个KV的影响。

2. **ZIP-Merging（零扰动合并）**：
   - 定义合并公式（式7）：
     \[
     k_r = \frac{(w_e k_e + w_c k_c) \ln\frac{w_e+w_c}{p_e+p_c}}{w_e \ln s_{te} + w_c \ln s_{tc}}, \quad v_r = \frac{w_e v_e + w_c v_c}{w_e + w_c}, \quad w_e = p_e s_{te}, w_c = p_c s_{tc}
     \]
   - 其中 \(s_{te}, s_{tc}\) 是注意力分数。该合并方式在**当前步骤**保证输出零扰动（Theorem 3）。

3. **多步扩展：EMA注意力分数预测**：
   - 基于注意力分数的强局部性（图2d），使用带偏置校正的指数移动平均预测未来步骤的分数（式8）。
   - 替换ZIP-Merging中的真实分数为EMA预测值，实现多步压缩。
   - 提供理论误差界（Theorem 5）：当预测误差 \(\epsilon\) 越小、合并的KV对越相似，输出扰动越小（Lemma 6）。这为高相似性合并提供了理论支撑。

4. **候选选择**：
   - 对于每个待驱逐的KV对，选择保留集中**余弦相似度最高**的KV对作为合并目标（设定阈值 \(T=0.8\)），避免动态计算开销。
   - 在预填充阶段，先按相似度合并，再执行驱逐策略（与常见顺序相反，可提高生成质量）。

## 3. 实验设计
### 数据集与场景
- **标准长度任务**：lm-eval-harness（MathQA, OpenBookQA等）、HELM（XSUM摘要, CNN/DailyMail摘要）。
- **长上下文任务**：LongBench（包含单文档QA、多文档QA、摘要、合成任务、代码共13个子任务）。

### 基准方法
- **驱逐方法**：StreamingLLM, H2O, PyramidInfer, Local Window。
- **合并方法**：CaM, D2O。
- **全缓存基线**（Full cache）。

### 模型架构
- Llama-2（7B, 13B）、Llama-3-8B、Mistral-7B。

## 4. 资源与算力
- **主要GPU**：NVIDIA A100 80GB（用于吞吐量测试），RTX 4090（用于延迟、FLOPs、内存测量）。
- **未明确说明**：使用的GPU数量、训练时长（论文为后训练优化方法，不需要训练）。
- **计算开销**：额外操作仅为EMA更新和余弦相似度计算，相比D2O（需计算注意力分布方差）更低。

## 5. 实验数量与充分性
- **主要实验**：
  - 图4：在多个数据集上以不同压缩比（10%–100%）比较KeepKV与所有基线，覆盖2种模型（LLaMA-2/3-7B,13B）。
  - 表3：在LongBench的13项任务上使用4种模型（LLaMA-2-7B/13B, LLaMA-3-8B, Mistral-7B），压缩比20%，对比5种方法。
  - 表1：吞吐量对比（Llama-2-7B, 4K上下文, 20%压缩比）。
  - 图5：消融实验——将KeepKV与H2O、PyramidInfer结合，验证通用性。
- **充分性**：实验覆盖了多种模型大小、任务类型和压缩比，对比了当前最先进驱逐与合并方法，并进行了消融。结果客观公平（遵循标准评估框架，参数一致）。

## 6. 主要结论与发现
- KeepKV在所有压缩比下**均优于**现有驱逐和合并方法，尤其在极低缓存预算（10%）时仍能保持接近全缓存的生成质量。
- 吞吐量提升超过2倍（在4K上下文、20%预算下从116.54 tokens/s提升至255.99 tokens/s）。
- 首次从理论上证明单步无损压缩，并提供多步误差界，为后续工作奠定基础。

## 7. 优点
- **理论贡献**：首次系统分析合并导致注意力凹陷的原因，并提出零扰动合并方法，给出误差界。
- **方法新颖**：Electoral Votes机制使合并后能复原合并前的影响力，ZIP-Merging确保单步零扰动。
- **通用性强**：可无缝集成现有驱逐策略（只需指定合并对）；不限制缓存分配方案。
- **低额外开销**：合并计算简单，延迟仅0.034 s/token，远低于同类合并方法D2O。

## 8. 不足与局限
- **模型规模有限**：实验最大为13B（LLaMA-2），未在70B等更大模型上验证（论文提及但未报告结果）。
- **合并阈值固定**：阈值 \(T=0.8\) 靠经验设定，可能非最优；不同任务可能需要自适应调整。
- **EMA预测依赖**：多步压缩质量依赖于注意力分数预测误差 \(\epsilon\)，若注意力在长期内突变（如长程依赖任务），预测误差可能增大，导致性能下降。
- **未讨论训练开销**：虽然KeepKV无需重新训练，但预填充阶段需要计算所有KV对的余弦相似度，对于极长上下文可能带来额外预填充延迟（论文未量化）。
- **消融实验范围**：仅测试了与H2O和PyramidInfer的结合，未与其他驱逐策略（如SnapKV）或合并方法（如KVMerger）组合。

（完）
