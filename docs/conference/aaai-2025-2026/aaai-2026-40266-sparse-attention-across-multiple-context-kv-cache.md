---
title: Sparse Attention Across Multiple-Context KV Cache
title_zh: 跨多上下文KV缓存的稀疏注意力
authors: "Ziyi Cao, Qingyi Si, Jingbin Zhang, Bingquan Liu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40266/44227"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 多上下文KV缓存稀疏注意力重用
tldr: 大型语言模型在长序列推理中面临高成本问题。本文提出跨多上下文KV缓存稀疏注意力方法，解决传统单上下文稀疏注意力无法直接应用于检索增强生成中多文档独立缓存的问题。通过稀疏选择最相关KV缓存，减少序列长度，提升推理吞吐量。实验表明该方法在RAG场景下有效降低延迟并保持生成质量。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有KV缓存稀疏注意力局限于单上下文，无法处理RAG中独立缓存的多文档场景。
method: 提出跨多上下文KV缓存稀疏注意力机制，根据相关性选择最相关的KV缓存子集。
result: 在RAG场景下显著降低推理延迟，保持生成质量。
conclusion: 该方法扩展了稀疏注意力到多上下文场景，有效提升RAG推理效率。
---

## Abstract
Large language models face significant cost challenges in long-sequence inference. To address this, reusing historical Key-Value (KV) Cache for improved inference efficiency has become a mainstream approach. Recent advances further enhance throughput by sparse attention mechanisms to select the most relevant KV Cache, thereby reducing sequence length. However, such techniques are limited to single-context scenarios, where historical KV Cache is computed sequentially with causal-attention dependencies. In retrieval-augmented generation (RAG) scenarios, where retrieved documents as context are unknown beforehand, each document’s KV Cache is computed and stored independently (termed multiple-context KV Cache), lacking cross-attention between contexts. This renders existing methods ineffective. Although prior work partially recomputes multiple-context KV Cache to mitigate accuracy loss from missing cross-attention, it requires retaining all KV Cache throughout, failing to reduce memory overhead. This paper presents SamKV, the first exploration of attention sparsification for multiple-context KV Cache. Specifically, SamKV takes into account the complementary information of other contexts when sparsifying one context, and then locally recomputes the sparsified information. Experiments demonstrate that our method compresses sequence length to 15% without accuracy degradation compared with full-recomputation baselines, significantly boosting throughput in multi-context RAG scenarios.

---

## 论文详细总结（自动生成）

### 论文详细中文总结

#### 1. 论文的核心问题与整体含义（研究动机和背景）
- **研究动机**：大型语言模型（LLM）在长序列推理中面临高昂的计算成本和显存占用。重用历史Key-Value（KV）缓存是提高推理效率的主流方法，而稀疏注意力机制进一步选择最相关的KV缓存以减少序列长度。
- **现有局限**：现有稀疏注意力技术仅适用于单上下文场景（如连续预填充的因果注意力），无法直接应用于检索增强生成（RAG）中的多上下文场景。在RAG中，每个文档的KV缓存是独立计算和存储的，缺乏跨上下文的注意力，因此单上下文方法失效。先前工作（如CacheBlend、EPIC）虽部分重计算多上下文KV缓存以弥补缺失的跨注意力，但要求保留全部KV缓存，无法降低显存开销。
- **论文目标**：提出SamKV，首次探索多上下文KV缓存的注意力稀疏化，在保持精度的同时大幅压缩序列长度，提升吞吐量。

#### 2. 论文提出的方法论：核心思想、关键技术细节
- **核心思想**：结合稀疏化与选择性重计算。先根据用户查询和其他上下文的信息过滤当前文档中的无关内容（稀疏化），再对稀疏后的关键Token进行局部重计算以恢复跨上下文注意力。
- **关键技术细节**：
  - **个性化查询嵌入模块**：生成每个文档专属的查询向量。
    - 先用初始和局部位置的KV缓存增量预填充用户查询，通过均值池化得到通用查询向量 \( Q_{que} \)。
    - 然后为每个文档 \( i \) 加入个性化偏置：整合其他文档局部Q缓存 \( Q_{doc-j}^{loc} \) 的加权信息，权重为余弦相似度的绝对值（除以文档数 \( D-1 \)），得到文档专属查询向量 \( \hat{Q}_{doc-i} \)。
  - **KV选择模块**：动态确定每个文档需要保留的KV缓存比例 \( P_{doc-i} \)。
    - 保留初始和局部位置的KV缓存不变，只对中间段进行稀疏化。
    - 基于锚点（初始/局部）K缓存 \( K_{anc} \)、最大/最小注意力得分块 \( K_{max}/K_{min} \) 计算 \( P_{doc-i}^{(n)} \) 公式（见原文式2），再跨层平均得到 \( P_{doc-i} \)。
    - 跨上下文比较后只保留全局最关键的块。
  - **重计算模块**：处理跨层块对齐问题，对稀疏后的KV缓存中需要恢复跨注意力的Token进行重计算。
    - 使用填充块对齐不同层的未对齐块。
    - 规则：若某Token在第 \( n \) 层需重计算，则需先计算前 \( n-1 \) 层的输出；同一层尽量复用已有缓存。
    - 提供两种更新策略：覆盖（Overwrite）和融合（Fusion，通过余弦相似度加权混合新旧KV缓存）。

#### 3. 实验设计
- **数据集**：使用LongBench中的三个多上下文QA数据集：2WikiMQA、MuSiQue、HotpotQA；另加DuReader进行消融。每个数据集200个测试样本。
- **基准方法**：
  - Recompute（完整重计算，作为基线）
  - Reuse（直接重用，不处理跨注意力）
  - Multi-InfLLM（将多上下文KV拼接后应用单上下文稀疏方法）
  - CacheBlend
  - EPIC
- **评估模型**：Mistral 7B Instruct、Llama 3.1 8B Instruct、Qwen2.5 3B Instruct。
- **评估指标**：F1分数。
- **实验数量与充分性**：
  - **主要对比**：在三个数据集上对每个模型测试了SamKV两种变体（overwrite/fusion）与五种比较方法，共形成约（2模型×7方法×3数据集 + 额外Qwen消融）组结果，数据量充分。
  - **消融实验**：在Qwen2.5和Llama 3.1上进行了三因素消融（是否选择中间段KV、是否加个性化偏置、是否重计算），共14行完整结果，验证了各模块贡献。
  - **客观公平**：所有方法采用相同超参数（贪心解码、温度0），对比方法复现自原论文设置。但未公开代码或随机种子，可能存在微小偏差。

#### 4. 资源与算力
- 论文明确说明实验环境：64GB NPU（华为昇腾？未具体型号）和256核鲲鹏920 CPU（超线程关闭），2TB DRAM，操作系统EulerOS 2.0。未提及GPU型号、数量或训练时长（仅推理实验），也未报告单个样本推理时间。因此无法评估算力总量，但显存占用通过KV缓存压缩至15%显著降低。

#### 5. 实验数量与充分性
- **实验数量**：较多，覆盖3个主流多上下文QA数据集 + 1个额外数据集消融；2种主要模型规模（7B/8B）+ 1个小模型；对比5种方法；每组实验F1报告方差？未提供标准差。
- **充分性**：消融实验设计合理，验证了各组件必要性；未见跨领域（如长文本摘要）或真实RAG流水线端到端测试。可能不够全面，但针对论文核心场景（多上下文QA）已较充分。
- **公平性**：所有方法使用相同基础模型和推理配置，对比方法复现合理。但SamKV引入额外稀疏化时间开销，文中仅定性声称比CacheBlend/EPIC总体时间低，未提供具体wall-clock时间对比，这一点可视为不足。

#### 6. 论文的主要结论与发现
- SamKV可将KV缓存压缩至原始长度的15%，且F1分数与完整重计算基线持平甚至略优（如在2WikiMQA和HotpotQA上超越Recompute）。
- 个性化偏置（融入其他上下文信标）比单纯使用用户查询能更有效识别跨文档共识。
- 重计算时采用融合策略（而非覆盖）有时可进一步提升性能。
- 稀疏化后只重计算关键Token，既降低了计算量，又缓解了跨注意力缺失。

#### 7. 优点
- **创新性**：首次将稀疏注意力应用于多上下文KV缓存，填补了该方向空白。
- **方法论完备**：包含个性化查询、动态比例选择、跨层对齐重计算、两种更新策略，设计精细。
- **效果显著**：在保持/提升精度的前提下显存压缩至15%，对RAG推理效率提升有实际价值。
- **消融验证充分**：清晰展示每个模块的贡献，证明设计必要。

#### 8. 不足与局限
- **实验覆盖有限**：仅测试QA任务，未涉及多跳推理、长文档摘要等；RAG场景也仅基于固定知识库，未评估动态检索场景。
- **时间开销对比缺失**：虽声称总体时间更低，但未提供具体推理延迟或吞吐量数值，削弱说服力。
- **泛化性存疑**：仅在Mistral/Llama/Qwen系列测试，未覆盖更大模型（如70B）或不同架构（如Mamba）。
- **稳定性风险**：动态TopP比例依赖锚点选择，可能对噪声敏感；余弦相似度加权可能引入误差。未报告多次运行的标准差或置信区间。
- **应用限制**：需要预先计算和存储所有文档KV缓存，不适合流式或动态更新场景。

（完）
