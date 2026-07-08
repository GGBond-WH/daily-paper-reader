---
title: "QJL: 1-Bit Quantized JL Transform for KV Cache Quantization with Zero Overhead"
title_zh: QJL：零开销的1比特量化JL变换用于KV缓存量化
authors: "Amir Zandieh, Majid Daliri, Insu Han"
date: 2025-04-11
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/34773/36928"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 零开销的1比特KV缓存量化
tldr: KV缓存量化中，传统方法因存储量化常数产生额外内存开销。QJL结合约翰逊-林登斯特劳斯变换与符号量化，将KV嵌入压缩至1比特，无需存储零点或缩放因子，在极低比特率下保持模型性能，显著减少KV缓存内存占用。
source: AAAI-2025-Accepted
selection_source: conference_retrieval
motivation: 传统KV缓存量化需要存储量化常数，带来额外内存开销。
method: 采用JL变换后接符号位量化，实现1比特KV缓存表示且无需存储量化常数。
result: 消除了量化常数存储开销，在极低内存下保持模型精度。
conclusion: JL变换与符号量化结合可实现高效无开销KV缓存压缩。
---

## Abstract
Serving LLMs requires substantial memory due to the storage requirements of Key-Value (KV) embeddings in the KV cache, which grows with sequence length. An effective approach to compress KV cache is quantization. However, traditional quantization methods face significant memory overhead due to the need to store quantization constants (at least a zero point and a scale) in full precision per data block. Depending on the block size, this overhead can add 1 or 2 bits per quantized number. We introduce QJL, a new quantization approach that consists of a Johnson-Lindenstrauss (JL) transform followed by sign-bit quantization. In contrast to existing methods, QJL eliminates memory overheads by removing the need for storing quantization constants. We propose an asymmetric estimator for the inner product of two vectors and demonstrate that applying QJL to one vector and a standard JL transform without quantization to the other provides an unbiased estimator with minimal distortion. We have developed an efficient implementation of the QJL sketch and its corresponding inner product estimator, incorporating a lightweight CUDA kernel for optimized computation. When applied across various LLMs and NLP tasks to quantize the KV cache to only 3 bits, QJL demonstrates a more than fivefold reduction in KV cache memory usage without compromising accuracy, all while achieving faster runtime.

---

## 论文详细总结（自动生成）

# QJL: 1-Bit Quantized JL Transform for KV Cache Quantization with Zero Overhead – 中文详细总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **问题**：大型语言模型（LLM）在自回归解码时需要存储所有先前生成的Key-Value（KV）嵌入到KV缓存中。随着序列长度增长，KV缓存的内存消耗成为主要瓶颈。
- **传统量化方法的不足**：现有KV缓存量化方法（如KIVI、KVQuant）需要将数据分组，并为每组存储量化常数（至少一个零点和缩放因子），通常使用全精度（16位）存储。根据分组大小，每个量化数会额外增加1~2比特的开销，导致“内存开销”问题。
- **本文目标**：提出一种**零额外内存开销**的KV缓存量化方法——QJL（Quantized Johnson-Lindenstrauss transform），通过随机投影+符号位量化，将每个Key嵌入压缩为1比特，无需存储任何量化常数，同时保持内积估计的低失真。

## 2. 论文提出的方法论

### 2.1 核心思想
- 使用Johnson-Lindenstrauss（JL）变换（随机高斯投影）将高维向量投影到低维空间，其内积是原始内积的无偏估计且失真小。
- 对Key嵌入进行JL变换后，**仅保留结果的符号位（sign bit）**，即1比特量化；而Query嵌入只做JL变换不做量化。这种**非对称**方式可以构造出内积的无偏估计器（Lemma 2）。
- 内积估计器定义为：  
  `Prod_QJL(q, k) = (√(π/2) / m) * ||k||₂ * ⟨Sq, sign(Sk)⟩`  
  其中S为m×d的随机高斯矩阵。

### 2.2 关键技术细节
- **无偏性**（Lemma 2）：证明上述估计器的期望等于⟨q, k⟩。
- **低失真界**（Lemma 3）：当投影维度 m ≥ O(ε⁻² log(1/δ)) 时，估计误差以高概率被 ε·||q||₂·||k||₂ 界住。
- **关键缓存量化算法**（Algorithm 1）：
  1. 在线生成随机矩阵S。
  2. 每来一个key k，计算量化向量 ˜k = sign(Sk)，并存储 ||k||₂ 范数（标量）。
  3. 查询时，对query q计算Sq，然后利用上述公式估计与每个key的内积，最后做softmax得到注意力分数。
- **理论保证**（Theorem 4）：当key和query范数有界时，仅需 m ≈ ε⁻² log n 比特（即每token比特数），注意力分数的相对失真 ≤ 3ε 以高概率成立。
- **处理异常值**：深层的key嵌入某些通道（坐标）存在大幅值异常。通过检测这些通道，将它们与正常通道分开，各使用独立的QJL量化器（异常通道分配更多bit）。
- **正交化JL矩阵**：对随机矩阵S的行做QR正交化，可实际提升性能。

### 2.3 价值缓存量化
- 价值（Value）缓存使用简单的逐token量化（归一化+舍入到低位整数），沿用KIVI和KVQuant的做法。

## 3. 实验设计

### 3.1 基准数据集与任务
- **长序列任务**：LongBench（Bai et al. 2023），包括NarrativeQA、Qasper、MultiQA-en、MultiQA-zh、HotpotQA、2WikiMultiQA等问答数据集。最大序列长度31500。
- **标准短序列任务**：LM-eval框架下的Lambada-OpenAI、HellaSwag、PIQA、MathQA、MMLU。
- **模型**：Longchat-7b-v1.5-32k（基于Llama-2 7B微调，32k上下文）、Llama-3.1-8B-Instruct。

### 3.2 对比方法
- **基线**：FP16 / BF16（16bit全精度）
- **竞争方法**：KIVI（3bit和5bit版本）、KVQuant（默认4.3bit）
- **QJL**：提供3bit、4bit、5bit版本（总比特数包括存储范数等开销）

### 3.3 评估指标
- 长序列：F1分数（主要指标，见Table 1）
- 标准任务：准确率（Table 2）
- 运行时：壁钟时间（prompt编码、KV缓存量化、解码）
- 峰值内存

## 4. 资源与算力

- **硬件**：单张NVIDIA A100 GPU，80GB显存。
- **训练/运行**：论文未进行模型训练，仅对预训练模型进行量化推理。未明确报告GPU运行总时长，但给出了不同序列长度下的具体耗时图（图4）。
- **实现**：使用PyTorch封装，关键操作（JL变换、sign量化、内积估计）通过定制的轻量CUDA kernel实现。

## 5. 实验数量与充分性

- **实验组数**：
  - 主表（Table 1）：在6个LongBench数据集上比较3种量化方法（4个设置）和基线，共约30个结果。
  - Table 2：在5个标准数据集上比较Llama-2-7B和Llama-3-8B模型，约20个结果。
  - 消融实验（Figure 3）：不同层、不同m/d下的失真MSE。
  - 异常值分析（Figure 2）：32层中3个代表性层的关键通道分布。
  - 运行时/内存比较（Figure 4, 5）：不同序列长度（1k~128k）下的时间、峰值内存。
- **充分性**：覆盖了长序列和短序列任务，对比了主流方法，同时提供了消融和效率分析。实验较为全面。
- **公平性**：对比方法使用其默认或推荐超参（KIVI 3bit/5bit, KVQuant 4.3bit）。QJL选择匹配的比特数。数据集、评估方法均遵循原框架（LongBench official, LM-eval）。但未与所有最新方法（如KIVI的升级版、其他零开销方法）对比；KVQuant未在峰值内存图中出现（因太慢被排除）。

## 6. 论文的主要结论与发现

1. **零内存开销有效**：QJL消除了传统量化存储常数开销，实现真正的“无开销”1比特量化。
2. **3bit即可达到16bit基线性能**：在LongBench多项任务上，3bit QJL的F1与16bit基线相当或更好；在LM-eval标准任务中准确率几乎无下降。
3. **内存节省显著**：KV缓存内存减少5倍以上（3bit vs 16bit），峰值内存（含模型参数）降低超过2倍。
4. **解码速度提升**：QJL的3bit版本在解码阶段比基线更快（尤其长序列），且比KVQuant快得多；对Llama-3支持良好（因支持BF16和分组查询注意力）。
5. **异常值处理重要**：识别并单独量化异常通道可进一步提升高层注意力层性能。

## 7. 优点

- **理论扎实**：提供了无偏估计和失真的严格证明（Lemma 2,3, Theorem 4），常数比原始JL变换更小。
- **零开销创新**：从根本上避免量化常数存储，设计新颖。
- **高效实现**：轻量CUDA kernel，支持多种精度（FP16, BF16, FP32），尤其支持Llama-3的BF16。
- **可扩展性**：每token比特数仅与log(序列长度)有关，与嵌入维度无关，适合极长上下文。
- **数据无关**：无需模型微调或数据校准，即插即用。

## 8. 不足与局限

- **第一层量化误差较大**：消融实验（Figure 3）显示第1层的失真远高于其他层，可能成为瓶颈。
- **部分数据集性能波动**：例如在MultiQA-zh上QJL的F1低于基线（41.18 vs 42.83 at 4.3bit），但未深入解释原因。
- **对比的完整性**：未与最新其他零开销方法（如RaBitQ）、基于采样的方法或更高压缩比的方案对比。KVQuant因速度太慢被排除出部分实验，可能影响公平性。
- **实现未完全用CUDA**：目前仅核心kernel用CUDA，其余用PyTorch，未来纯CUDA版本可能更快。
- **价值缓存量化并非创新**：沿用现有简单逐token量化，未提供理论保证或新设计。
- **异常值检测基于离线观察**：需要预设阈值或固定位置，可能不适用于动态变化的数据。
- **实验规模**：仅测试了7B和8B模型，未在更大模型（如13B, 70B）或更多长序列任务上验证。

（完）
