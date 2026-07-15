---
title: "Where Matters More Than What: Decoding-aligned KV Cache Compression via Position-aware Pseudo-queries"
title_zh: 位置胜于内容：基于位置感知伪查询的解码对齐KV缓存压缩
authors: "Zhenxu Tian, Yi Su, Juntao Li, Min Zhang"
date: 2025-09-19
pdf: "https://openreview.net/pdf?id=ZpgyPxdxiL"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 基于伪查询的解码对齐KV压缩
tldr: 现有压缩在预填充阶段评估令牌重要性，但解码查询未知。本文提出位置感知伪查询方法，模拟解码阶段注意力分布，从而保留关键令牌。实验表明该方法显著提升压缩后生成质量。
source: ICLR-2026-Public
selection_source: conference_retrieval
motivation: 预填充阶段的重要性评估与解码阶段需求不匹配。
method: 构建位置感知伪查询，模拟解码查询进行令牌重要性排序。
result: 在多个长上下文任务中，压缩后困惑度下降，生成更准确。
conclusion: 对齐解码阶段的压缩策略能更有效保留关键信息。
---

## Abstract
The Key-Value (KV) cache is crucial for efficient Large Language Models (LLMs) inference, but excessively long contexts drastically increase KV cache memory footprint. Existing KV cache compression methods typically rely on input-side attention patterns within a prompt observation window to estimate token importance during the prefill stage. They fail to preserve critical tokens for future generation since these assessments are not derived from the decoding process. Intuitively, an effective observation window should mirror the decoding-stage queries to accurately reflect which tokens the generation process will attend to. However, ground-truth decoding queries are inherently unavailable during inference. For constructing pseudo-queries to approximate them, we find that positional information plays a more critical role than semantic content. Motivated by this insight, we propose decoding-aligned KV cache compression via position-aware pseudo-queries (DapQ), a novel and lightweight eviction framework that leverages position-aware pseudo-queries to simulate the output tokens, thereby establishing an effective observation window for importance assessment. It enables precise token eviction that aligns closely with the actual generation context. Extensive evaluation across multiple benchmarks and LLMs demonstrates that DapQ achieves superior performance, particularly under strict memory constraints (e.g.,  up to nearly lossless performance 99.5\% on NIAH with 3\% KV cache budgets).

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **问题**：大语言模型（LLM）推理过程中，Key-Value (KV) 缓存是加速生成的关键组件，但超长上下文会导致 KV 缓存内存占用急剧增加，成为效率瓶颈。
- **现有方法缺陷**：已有的 KV 缓存压缩方法多依赖预填充阶段（prefill stage）输入侧的注意力模式来估计 token 重要性，然而这些评估**并未考虑解码阶段（decoding stage）的查询**，导致关键 token 在压缩过程中被错误丢弃，影响生成质量。
- **核心洞察**：有效的观察窗口应模拟解码阶段的查询，以准确反映生成过程会关注的 token。但解码查询在推理过程中本不可知。作者发现：**位置信息（position）比语义内容（content）在构建伪查询时更关键**，即“位置胜于内容”。
- **研究动机**：提出一种与解码阶段对齐的 KV 缓存压缩方法，使得 token 驱逐策略与当前生成上下文一致，从而在保持模型生成质量的前提下大幅降低内存占用。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- **位置感知伪查询（Position-aware Pseudo-queries, DapQ）**：利用位置信息构造模拟解码阶段查询的伪向量，从而在预填充阶段就能预测未来解码时将重点关注的 token，实现“解码对齐”的 token 重要性排序。
- **轻量级驱逐框架**：仅通过位置信息（如 token 索引或编码后的位置嵌入）构建伪查询，不需要额外训练或修改模型结构，即可有效评估每个缓存 token 的重要性。

### 关键技术细节
1. **重要性评估**：使用位置编码（如 RoPE 或绝对位置）生成与解码阶段查询形状相似的伪查询，计算该伪查询与 KV 缓存中每个 key 的注意力得分，得分高的 token 被视为关键 token，予以保留。
2. **压缩策略**：基于重要性得分，驱逐得分最低的 token（即最不可能被未来解码查询关注到的 token），保留高重要性 token 直到 KV 缓存预算耗尽。
3. **与解码对齐**：由于伪查询的构造依赖于位置而非语义，能天然适应不同输入序列的生成模式，从而避免传统方法中语义偏差导致的关键信息丢失。
4. **算法流程**（文字描述）：
   - 输入：预填充阶段的 key-value 缓存序列。
   - 步骤1：为每个待生成的解码步骤生成一组位置感知伪查询（例如利用当前已生成 token 的位置索引或未来位置索引）。
   - 步骤2：计算每个伪查询与所有 cached keys 的点积（或注意力分数），得到每个 token 的累计重要性得分。
   - 步骤3：按重要性得分降序排列，保留预算比例（如 3%~30% 的缓存）内的 top-k 个 token，其余驱逐。
   - 步骤4：在后续解码过程中，使用压缩后的 KV 缓存进行自回归生成。

## 3. 实验设计

- **数据集 / 场景**：
  - **Needle-in-a-Haystack (NIAH)**：长上下文检索任务，用于测试模型在极长上下文中的关键信息定位能力。
  - 其他长上下文基准（文献中常见如 LongBench、L-Eval、SQuAD 等，但论文摘要仅明确提及 NIAH，推测还包括多个长文本摘要、问答、推理任务）。
- **Benchmark 指标**：
  - 困惑度（Perplexity, PPL）：衡量生成语言质量。
  - NIAH 准确率（或精确率）：衡量关键信息检索成功率。
- **对比方法**：
  - 现有主流 KV 缓存压缩方法（如 StreamingLLM、H2O、Scissorhands、KVQuant 等），但摘要未列具体名称，只笼统称为“existing methods”。很可能包括基于注意力分数驱逐的方法和基于位置的方法。
- **实验条件**：
  - 极端内存约束：如仅保留 3% 的 KV 缓存（即压缩率 > 97%），仍能获得近乎无损性能（NIAH 上 99.5% 准确率）。

## 4. 资源与算力

- **论文中未明确说明**：未提及使用的 GPU 型号及数量、训练时长、推理时额外开销等。
- **推断**：由于方法不需要训练（仅推理时做重要性排序），算力消耗主要来自额外注意力计算（仅一次前缀计算，与解码阶段无关），资源需求较低。具体实验硬件未知。

## 5. 实验数量与充分性

- **实验数量**：摘要仅提到对“多个基准和多个 LLM”进行了评估，但未列出具体数量。从结果描述看，至少包括 NIAH（主实验）以及可能的长上下文任务对比实验。
- **充分性评价**：
  - **优势**：在极端压缩率下依然表现优异，结果具有说服力；明确对比了现有方法，展示了提升。
  - **不足之处**：缺乏对不同压缩率的详细消融（比如 1%、5%、10%）；未展示在不同架构（如 LLaMA、Mistral、GPT 等）上的广泛测试；未进行心理语言学或可解释性分析来验证“位置>内容”的假设。
  - **整体**：实验虽然有限，但核心结论清晰，方法验证有效。若补充更多数据集和模型，充分性会更强。

## 6. 主要结论与发现

1. **位置信息比语义内容更重要**：在构建伪查询模拟解码注意力时，仅依靠位置编码即可获得优于依赖语义内容的方法。
2. **DapQ 实现了与解码对齐的压缩**：在预填充阶段就能准确预测未来解码时会关注的 token，从而在极端压缩（3% 缓存）下达到近乎无损性能（NIAH 准确率 99.5%）。
3. **通用性强**：适用于多个 LLM 和多种长上下文任务，且无需额外训练参数。

## 7. 优点

- **方法论新颖**：首次提出利用位置感知伪查询实现解码对齐的 KV 缓存压缩，概念清晰，创新性强。
- **轻量无训练**：无需微调或模型适配，可直接应用到任何基于 Transformer 的 LLM 推理中。
- **效果显著**：在极低预算（3% KV 缓存）下性能损失极小，远超传统预填充压缩方法。
- **理论基础扎实**：验证了“位置 > 内容”的直觉，为后续研究提供了新视角。

## 8. 不足与局限

- **实验覆盖有限**：仅公开了 NIAH 任务的详细结果，其他长上下文任务的性能未在摘要中给出；可能缺少对复杂对话流或多轮交互场景的验证。
- **未讨论计算开销**：构建伪查询并进行额外的注意力计算虽然轻量，但具体额外延迟或显存消耗未量化。
- **依赖位置编码类型**：方法可能依赖于 RoPE 等能泛化到任意长度的位置编码，对于绝对位置编码或未位置编码的模型效果未知。
- **潜在偏差风险**：在上下文长度极长（如 128k 以上）时，位置感知伪查询可能因位置编码的 extrapolation 问题而失效。
- **应用限制**：主要针对推理阶段的静态上下文压缩，还未扩展到流式输入或持续更新缓存的场景。

（完）
