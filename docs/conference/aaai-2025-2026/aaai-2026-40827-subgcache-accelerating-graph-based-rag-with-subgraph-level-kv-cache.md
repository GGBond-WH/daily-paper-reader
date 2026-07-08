---
title: "SubGCache: Accelerating Graph-based RAG with Subgraph-level KV Cache"
title_zh: SubGCache：基于子图级KV缓存的图增强生成加速
authors: "Qiuyu Zhu, Liang Zhang, Qianxiong Xu, Cheng Long, Jie Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40827/44788"
tags: ["query:llm-kv-cache"]
score: 8.0
evidence: 图检索增强生成中基于子图级别的KV缓存重用
tldr: 针对图增强生成（RAG）中不同查询可能检索到相似子图导致重复计算的问题，本文提出SubGCache。该方法首先对查询子图嵌入进行聚类，为每个簇构建代表性子图并预计算其KV缓存，簇内查询直接复用该缓存。实验表明，SubGCache有效降低了图RAG的推理延迟，同时保持回答质量。该工作将前缀缓存思想扩展至图结构提示，实现了跨查询的KV缓存复用。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 图RAG中不同查询常检索相似子图，导致重复计算KV缓存，浪费资源。
method: 对查询子图嵌入聚类，构建代表性子图并预计算KV缓存，簇内查询复用该缓存。
result: "在多个知识图谱问答基准上，SubGCache相较于基线减少了30%-50%的推理时间，精度损失很小。"
conclusion: 子图级KV缓存复用是一种有效的图RAG加速策略，适用于结构化知识检索场景。
---

## Abstract
Graph-based retrieval-augmented generation (RAG) enables large language models (LLMs) to incorporate structured knowledge via graph retrieval as contextual input, enhancing more accurate and context-aware reasoning. We observe that for different queries, it could retrieve similar subgraphs as prompts, and thus we propose SubGCache, which aims to reduce inference latency by reusing computation across queries with similar structural prompts (i.e., subgraphs). Specifically, SubGCache clusters queries based on subgraph embeddings, constructs a representative subgraph for each cluster, and pre-computes the key-value (KV) cache of the representative subgraph. For each query with its retrieved subgraph within a cluster, it reuses the pre-computed KV cache of the representative subgraph of the cluster without computing the KV tensors again for saving computation. Extensive experiments on three datasets across multiple LLM backbones and graph-based RAG frameworks demonstrate that SubGCache consistently reduces inference latency with comparable and even improved generation quality, achieving up to 6.68x reduction in time-to-first-token (TTFT).

---

## 论文详细总结（自动生成）

# 论文详细中文总结：SubGCache: Accelerating Graph-based RAG with Subgraph-level KV Cache

## 1. 核心问题与整体含义（研究动机与背景）

- **研究背景**：基于图的检索增强生成（Graph-based RAG）通过从外部文本图中检索相关子图作为提示，增强大语言模型（LLM）的结构化知识理解与上下文推理能力。然而，现有系统针对单查询设计，每个查询独立处理。
- **问题发现**：在实际场景（如学术问答、批量查询）中，多个查询可能检索到高度相似甚至重叠的子图，现有方法重复编码和推理这些重叠内容，造成大量冗余计算。
- **核心目标**：提出一种利用批查询中子图结构冗余来加速推理、降低延迟的方法，同时保持或提升生成质量。

## 2. 方法论：核心思想、关键技术细节、算法流程

### 2.1 核心思想
通过构建**子图级KV缓存**，实现跨查询的注意力状态复用。将批查询中检索到的相似子图聚类，为每个簇构建一个**代表性子图**，预计算其KV缓存，簇内所有查询直接复用该缓存，仅需计算查询专属问题令牌的KV。

### 2.2 关键技术细节与流程

1. **查询聚类（Query Clustering）**
   - 使用预训练的图神经网络（GNN）将每个检索到的子图编码为图嵌入，该GNN与下游图RAG模型共享，嵌入同时捕获语义和结构信息。
   - 基于嵌入进行层次聚类（Hierarchical Clustering），将子图高度重叠的查询自动分组到同一簇中。

2. **代表性子图构建（Representative Subgraph Construction）**
   - 对每个簇，取所有子图的节点和边的并集，合并成一个代表性子图。该子图保留了簇内所有查询所需的关系上下文。

3. **子图级KV缓存复用（Subgraph-level Cache Reuse）**
   - 对每个簇，先根据代表性子图构建提示文本，送入LLM预计算所有Transformer层的KV张量，存储在GPU内存中。
   - 簇内每个查询处理时，将查询问题令牌附加到缓存的提示末尾，LLM复用代表性子图的KV缓存，仅计算新令牌的KV。
   - 处理完一个簇后释放该KV缓存，再处理下一簇（簇间顺序处理）。

### 2.3 算法流程（文字说明）
输入：批查询集合 \( \{q_1,\dots,q_m\} \)，文本图 \(G\)，检索得到的子图集合 \( \{s_1,\dots,s_m\} \)
1. 对每个子图 \(s_i\)，用GNN计算嵌入 \(e_i\)。
2. 对嵌入集合 \( \{e_1,\dots,e_m\} \) 进行层次聚类，得到 \(c\) 个簇。
3. 对于每个簇 \(C_k\)：
   - 合并簇内所有子图的节点和边，得到代表性子图 \( \tilde{s}_k \)。
   - 根据 \( \tilde{s}_k \) 构建提示，输入LLM计算并存储KV缓存。
   - 对簇内每个查询 \(q_i\)，追加问题令牌到提示，复用缓存的KV，生成回答。
   - 处理完该簇后释放缓存。
4. 输出所有查询的回答。

- **可调粒度**：调整簇数量 \(c\) 可控制缓存复用程度。\(c=1\) 为最粗粒度（所有查询共享同一个代表性子图），\(c=m\) 退化为标准图RAG。

## 3. 实验设计

### 3.1 数据集
- **Scene Graph**（来源G-Retriever论文）
- **OAG**（Open Academic Graph）
- **DBLP**（学术文献图）

每个数据集均为文本图，包含学术实体关系等，可构造子图检索问答任务。

### 3.2 基准方法（Baseline）
- **G-Retriever**：检索节点和边后重构子图。
- **GRAG**：检索k跳ego网络并剪枝。
将SubGCache作为即插即用模块集成到上述两个方法中，得到：
  - G-Retriever + SubGCache
  - GRAG + SubGCache

### 3.3 对比方法
- 主要对比基线本身（无缓存）与集成SubGCache后的性能差异。
- 还探索了不同簇数量、不同批大小、不同LLM骨干的影响。

### 3.4 评估指标
- **ACC**：准确率（%）
- **RT**：总响应时间（ms）
- **TTFT**：首个令牌生成时间（ms）
- **PFTT**：预填充+首个令牌时间（ms）

## 4. 资源与算力

- 论文未明确说明使用的GPU型号、数量及训练时长。
- 实验设置中提到所有LLM均为冻结（inference-only），因此无需训练，仅推理阶段评测。
- 模型推理在4种LLM骨干上进行：Llama-3.2-3B、Llama-2-7B、Mistral-7B、Falcon-7B，均为开源模型。
- 聚类和代表性子图构建的计算开销较小，文中指出其占整体延迟的比例极低（Scene Graph上<2.1%，OAG上<6%）。

## 5. 实验数量与充分性

### 进行的实验组
- **主实验结果**：在Scene Graph和OAG两个数据集上，分别用4种LLM骨干，对比G-Retriever和GRAG两个基线，共 2 × 4 × 2 = 16 组主实验（表1，另含DBLP结果在附录）。
- **簇数量影响实验**：在Scene Graph和OAG上，以Llama-3.2-3B为骨干，测试了7个不同簇数（1,2,3,4,5,10,20），并绘图展示ACC和TTFT（图3）。
- **批大小影响实验**：在Scene Graph和OAG上，测试了50、100、150、200的批大小，以Llama-3.2-3B为骨干（表2），附录还补充了另外三个LLM骨干的类似实验。
- **簇处理时间分析**：在不同簇数下分析聚类阶段耗时与LLM响应时间（图4）。
- **案例研究**（附录D）：具体展示代表性子图构造效果。
- **不同连接策略对比**（附录F）：比较了不同的合并策略（如union vs intersection等）的影响。

### 充分性评价
- **实验较为充分**：覆盖多个数据集、多种LLM骨干、两种主流图RAG方法，并系统探究了簇数、批大小等关键超参数的影响。
- **公平性**：对比时保持LLM冻结、推理环境一致，SubGCache仅作为额外插件，不改变基准的检索与生成逻辑，对比公平。
- 但**仅测试了学术图数据集**，缺少对其他类型图（如知识图谱、社交网络）的评估，泛化性证据有限。

## 6. 主要结论与发现

- SubGCache在不同数据集、LLM骨干和图RAG框架下**一致降低推理延迟**，TTFT最高降低6.68倍，PFTT最高降低19.77倍。
- 延迟降低的同时**生成质量保持甚至提升**：大部分实验ACC持平或提高（最高+9%），仅在少数情况（如OAG上Falcon-7B）出现极小下降（-1%）。
- 簇数量选择存在**延迟-准确率权衡**：更少的簇（粗粒度）带来更多复用、更低的TTFT，但可能因引入冗余信息导致轻微质量下降；更多簇（细粒度）保留更多查询特异性，但复用减少。存在非单调变化。
- 聚类阶段引入的额外时间开销极小（<6%），不影响整体加速效果。
- SubGCache**可扩展至不同批大小**，随着批大小增加，加速效果更明显。

## 7. 优点（方法或实验设计亮点）

- **概念创新**：首次提出批查询设置下图RAG的加速问题，并将KV缓存思想扩展到子图级结构提示。
- **方法简洁且即插即用**：SubGCache作为轻量模块，可与现有图RAG方法无缝集成，无需修改模型或训练。
- **兼顾效率与效果**：通过合并子图保留有用上下文，避免简单丢弃信息，因此不仅不损失质量，有时还能通过更丰富的上下文提升准确率。
- **实验设计系统**：全面考虑了簇数、批大小、不同LLM骨干、不同基线框架的影响，并分析了各环节开销。
- **结果显著**：TTFT加速高达6.68倍，PFTT加速高达19.77倍，且质量持平或提升，实用价值高。

## 8. 不足与局限

- **数据集局限**：仅使用了三个学术图数据集（Scene Graph, OAG, DBLP），缺少对更广泛类型图（如通用知识图谱、社交网络、生物医疗图）的验证，泛化性有待进一步评估。
- **实验覆盖偏差风险**：所有测试的LLM均在7B参数量级以下，未测试更大模型（如13B、70B或GPT-4级别），子图级缓存对更大模型的加速效果及内存占用问题未讨论。
- **延迟指标仅包含推理阶段**：未考虑图检索本身的耗时，而批查询的图检索也可能存在冗余，文中未提及检索阶段的优化。
- **簇数选择依赖人为设定**：虽然提供了可调参数，但未给出自动选择最优簇数的方法，实际部署需要调优。
- **代表性子图合并策略**：当前为简单的节点和边并集，未考虑更精细的合并策略（如基于语义对齐或拓扑剪枝）可能带来的额外质量提升。
- **应用限制**：适用于批查询场景，对于实时单查询或流式查询，缓存复用机会较少，效果有限。
- **论文版本问题**：标注为“AAAI-2026 Accepted”，但发表时间为2026年，可能为未来会议，需注意其实际审阅和发表状态。

（完）
