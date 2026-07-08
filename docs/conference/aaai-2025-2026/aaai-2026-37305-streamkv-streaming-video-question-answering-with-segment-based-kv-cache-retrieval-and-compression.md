---
title: "StreamKV: Streaming Video Question-Answering with Segment-based KV Cache Retrieval and Compression"
title_zh: "StreamKV: 基于分段的KV缓存检索与压缩的流式视频问答"
authors: "Yilong Chen, Xiang Bai, Zhibin Wang, Chengyu Bai, Yuhan Dai, Ming Lu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/37305/41267"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 针对视频大模型的KV缓存检索与压缩
tldr: StreamKV针对视频大模型处理长视频时KV缓存检索和压缩未充分探索的问题，提出无训练框架。它动态划分视频流，实现查询相关的KV缓存检索与压缩。实验证明在视频问答任务上提升效率而不损失准确性。为视频LLM的高效推理提供了新方案。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 视频LLM处理长视频时KV缓存检索和压缩方法不足。
method: 动态划分视频流，进行基于段的KV缓存检索和压缩。
result: 在视频问答任务上提高效率并保持准确性。
conclusion: 动态分段检索压缩可有效释放视频LLM的KV缓存瓶颈。
---

## Abstract
Video Large Language Models (Video-LLMs) have demonstrated significant potential in the areas of video captioning, search, and summarization. However, current Video-LLMs still face challenges with long real-world videos. Recent methods have introduced a retrieval mechanism that retrieves query-relevant KV caches for question answering, enhancing the efficiency and accuracy of long real-world videos. However, the compression and retrieval of KV caches are still not fully explored. In this paper, we propose StreamKV, a training-free framework that seamlessly equips Video-LLMs with advanced KV cache retrieval and compression. Compared to previous methods that used uniform partitioning, StreamKV dynamically partitions video streams into semantic segments, which better preserves semantic information. For KV cache retrieval, StreamKV calculates a summary vector for each segment to retain segment-level information essential for retrieval. For KV cache compression, StreamKV introduces a guidance prompt designed to capture the key semantic elements within each segment, ensuring only the most informative KV caches are retained for answering questions. Moreover, StreamKV unifies KV cache retrieval and compression within a single module, performing both in a layer-adaptive manner, thereby further improving the effectiveness of streaming video question answering. Extensive experiments on StreamingVQA benchmarks demonstrate that StreamKV significantly outperforms existing Online Video-LLMs, achieving superior accuracy while substantially improving both memory efficiency and computational latency.

---

## 论文详细总结（自动生成）

# 论文详细中文总结：StreamKV

## 1. 核心问题与整体含义（研究动机和背景）
- **研究动机**：现有视频大语言模型（Video-LLMs）在处理长视频时面临三大挑战：① 如何高效处理流式视频；② 如何平衡视觉上下文保留与内存消耗；③ 如何快速准确检索历史信息以回答用户问题。
- **整体含义**：针对目前方法（如ReKV）采用均匀分段、保留全部KV缓存导致高内存和检索不灵活的问题，本文提出StreamKV——一个无需训练（training-free）的框架，利用语义分割和压缩检索一体化机制，显著提升流式视频问答的准确性和效率。

## 2. 方法论：核心思想、关键技术细节
### 核心思想
- 动态将视频流划分为**语义连续的分段**，每个分段生成**摘要向量**保留段级信息；编码后对KV缓存进行压缩（仅保留信息量最大的块）并存入KV Bank；收到用户问题时，利用**统一层自适应KV选择模块**检索相关KV缓存并生成答案。

### 关键技术细节
1. **语义分段与摘要向量**  
   - 以固定间隔采样帧，通过视觉编码器提取每帧的ViT patch特征。  
   - 计算相邻帧特征的余弦相似度，**低相似度帧作为段边界**。  
   - 为避免短片段，设置排除窗口m；为限制片段长度，若超过阈值M，则合并最相似的相邻帧。  
   - 每个片段Si内所有帧的特征在空间位置取平均得到摘要向量 \( f_s^i \)。

2. **基于分段的滑动窗口编码**  
   - 逐段编码，当前段Si、摘要向量及局部窗口L（过去KV对）共同计算注意力：\( O = \text{Attn}(W_Q X_i, [L_k, W_K X_i], [L_v, W_V X_i]) \)，其中 \( X_i = [S_i \parallel f_s^i] \)。  
   - 每个帧产生一个KV块，其代表键向量为该帧所有patch键的均值。

3. **统一层自适应KV选择模块**  
   - 将压缩和检索均视为**从候选集中选择最相关KV条目**的问题。  
   - 输入：各层候选代表键向量集合 \(\{R^l\}\)、选择准则向量（压缩用引导提示词，检索用问题向量）、总预算N。  
   - **步骤**：
     1. 计算每个候选与准则的余弦相似度。
     2. 对每层归一化并排序，得到优先级序列。
     3. 通过二分搜索找到全局累积相似度阈值p，使各层选中的候选总数等于N；每层分配预算 \(K_l\)，最终选出索引子集。  
   - 该模块统一应用于压缩和检索，实现自适应预算分配。

4. **KV压缩**  
   - 压缩在每段编码后立即执行，而非解码阶段。  
   - 使用**引导提示词**（如“包含人、物体、关键事件、时间关系等”）作为选择准则，确保保留语义关键元素，不依赖特定用户问题。  
   - 摘要向量对应的KV块永久保留，不参与压缩。

5. **KV检索与问答**  
   - 收到用户问题后，用问题向量作为准则，从KV Bank中检索最相关的KV块（总预算 \(N_r \times L\)）。  
   - 检索到的KV块作为上下文，与问题、已生成token拼接，执行注意力计算。

6. **位置编码**  
   - 分段编码时，RoPE仅应用于局部窗口（LM-Infinite方式）；问答时，将检索到的token视为连续，按相对位置编码。

## 3. 实验设计
- **数据集/场景**：使用**StreamingBench**（包含18个子任务，分为三类：实时视觉理解、全源理解、上下文理解）。  
- **基准线**：对比方法包括：
  - 专有MLLM：Gemini1.5 pro、GPT-4o、Claude3.5 Sonnet。
  - 开源MLLM：Qwen2-VL-7B、MiniCPM-V-2.6-8B、InternVL-V2-8B等。
  - 流式MLLM：Flash-VStream-7B、VideoLLM-online-8B、Dispider-7B、ReKV-7B。
- **基线模型**：LLaVA-OneVision-Qwen2-7B-OV（基础模型）。
- **配置**：0.5 FPS处理，局部窗口15K，动态分段最小/最大长度4/64帧，划分阈值0.99，检索帧数8（除消融外）。

## 4. 资源与算力
- **硬件**：NVIDIA H20 GPU（96GB显存），FP16精度。  
- **训练时长**：未明确说明（StreamKV为training-free框架，仅需推理，无需训练）。  
- **算力数量**：未提及具体GPU数量。

## 5. 实验数量与充分性
- **主实验**：表1展示在StreamingBench全部18个子任务上的准确率，覆盖三类场景，对比了8种流式/非流式方法及3种专有模型。  
- **消融实验**：
  - 表2：语义分段 vs 均匀分段（在10%到90%压缩率下测试10个点）。
  - 表3：摘要向量有无（同样10个压缩率）。
  - 表4：层自适应模块（压缩/检索分别使用均匀或自适应，共4种组合，5个压缩率）。
  - 图4：检索帧数变化（4~64帧）对比ReKV。
- **充分性评价**：实验设计较系统，控制变量充分，消融实验覆盖核心模块。但**仅在一个基准（StreamingBench）上评估**，未在多个长视频数据集上验证泛化性；未测试不同基础模型规模或架构的影响。

## 6. 主要结论与发现
- **显著性能提升**：在60%压缩率下，StreamKV在Clip Summarization任务上达到87.7%准确率，比ReKV高9.1%，甚至超越所有专有模型。  
- **全源理解**：在四个全源任务上超过GPT-4o和Claude3.5，表明能准确捕捉细粒度视觉信息。  
- **异常检测能力**：Anomaly Context Understanding任务表现最佳，证明能有效识别微妙语义变化。  
- **检索精炼**：减少检索帧数反而提升性能（4帧最优），而ReKV需要更多帧补偿低精度。  
- **90%压缩仍保持高准确率**（56.7% overall），远超其他流式方法。

## 7. 优点（亮点）
- **训练-free**：无需额外微调，可直接部署于现有Video-LLM。
- **语义分段**：动态保留语义边界，比均匀分段更符合视频内容结构。
- **摘要向量**：保留段级全局信息，对压缩和检索都至关重要。
- **引导提示词压缩**：无需用户问题即可在编码阶段压缩，适合多轮对话场景。
- **层自适应分配**：基于各层信息分布差异合理分配预算，优于统一分配。
- **统一模块**：同一机制处理压缩与检索，简化设计并提升效率。
- **精准检索**：以少量帧实现高精度，降低计算开销。

## 8. 不足与局限
- **单一基准评估**：仅在StreamingBench上验证，缺乏在其他长视频数据集（如Video-MME、MLVU等）的泛化性测试。  
- **引导提示词设计**：文中仅给出参考示例，未分析不同提示词对压缩效果的影响，可能存在依赖经验选择的偏差。  
- **摘要向量计算开销**：每段需要对所有帧特征进行平均，在极高速率场景下可能带来额外延迟。  
- **范围限制**：未与其他类型的KV压缩方法（如FastV、SparseVLM）直接对比；也未分析更大模型（如14B、70B）上的表现。  
- **多模态局限**：尽管任务包含音频，模型本身仅处理视觉，依赖文本转述，未真正融合音频模态。

（完）
