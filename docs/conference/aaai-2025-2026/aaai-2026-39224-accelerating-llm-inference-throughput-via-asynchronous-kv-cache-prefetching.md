---
title: Accelerating LLM Inference Throughput via Asynchronous KV Cache Prefetching
title_zh: 通过异步KV缓存预取加速LLM推理吞吐量
authors: "Yanhao Dong, Yubo Miao, Weinan Li, Xiao Zheng, Chao Wang, Jiesheng Wu, Feng Lyu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39224/43185"
tags: ["query:llm-kv-cache"]
score: 9.0
evidence: 异步KV缓存预取以隐藏HBM延迟
tldr: LLM推理受限于高带宽内存带宽瓶颈。本文提出面向L2缓存的异步KV缓存预取方法，通过将KV缓存预先加载到GPU L2缓存中，实现计算与访存重叠，从而隐藏HBM访问延迟。在NVIDIA H20 GPU上实验表明，注意力核效率提升2.15倍，有效突破内存带宽限制。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: LLM推理内存带宽瓶颈导致HBM访问延迟成为性能关键。
method: 利用计算空闲时间片异步预取KV缓存到L2缓存，实现计算-访存重叠。
result: 注意力核效率提升2.15倍。
conclusion: 异步预取有效隐藏HBM延迟，提升LLM推理吞吐量。
---

## Abstract
Large Language Models (LLMs) exhibit pronounced memory-bound characteristics during inference due to High Bandwidth Memory (HBM) bandwidth constraints. In this paper, we propose an L2 Cache-oriented asynchronous KV Cache prefetching method to break through the memory bandwidth bottleneck in LLM inference through computation-load overlap. By strategically scheduling idle memory bandwidth during active computation windows, our method proactively prefetches required KV Cache into GPU L2 cache, enabling high-speed L2 cache hits for subsequent accesses and effectively hiding HBM access latency within computational cycles. Extensive experiments on NVIDIA H20 GPUs demonstrate that the proposed method achieves 2.15× improvement in attention kernel efficiency and up to 1.97× end-to-end throughput enhancement, surpassing state-of-the-art baseline FlashAttention-3. Notably, our solution maintains orthogonality to existing optimization techniques and can be integrated with current inference frameworks, providing a scalable latency-hiding solution for next-generation LLM inference engines.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：大型语言模型（LLM）在推理过程中，尤其是自回归解码阶段，呈现显著的“内存受限”特性。由于高带宽内存（HBM）带宽有限，频繁的KV Cache加载操作导致高延迟HBM访问，成为制约推理吞吐量的关键瓶颈。
- **现有方案不足**：现有优化（如FlashAttention、DeepSpeed-Inference）通过减少HBM访问次数来缓解瓶颈，但未能实现计算与访存的重叠（latency hiding），仍然受限于内存带宽。
- **研究动机**：通过硬件性能剖析发现，vLLM的XFormers后端注意力核存在极低的L1/L2缓存命中率（L2命中率仅0.06%），导致大量warp停顿（Stall Long Scoreboard占CPI的77%），同时内存带宽利用率仅47.10%，存在大量空闲带宽资源未被利用。本文旨在利用这些空闲带宽，通过异步预取将后续KV块提前加载到L2缓存，实现计算与访存重叠，从而隐藏HBM延迟。

## 2. 论文提出的方法论
- **核心思想**：在计算当前迭代的Q·K<sup>T</sup>（或logits·V）的同时，利用GPU的异步预取指令，将下一次迭代所需的KV块从HBM预取到L2缓存。这样，当后续迭代需要加载KV块时，可直接从高速L2缓存命中，避免等待HBM访问。
- **关键技术细节**：
  - 基于vLLM的分块KV Cache管理（每个KV Block包含16个token，每个注意力头独立存储）。
  - 利用NVIDIA Hopper架构的PTX指令 `cp.async.bulk.prefetch.L2`，该指令非阻塞，可在计算期间并行执行预取。
  - 具体流程（以Q·K<sup>T</sup>为例，见Algorithm 1）：
    1. 每个warp加载当前K Block到寄存器。
    2. 如果当前迭代还有后续K Block（索引为 `block_idx + w`，其中w是warp数），则异步预取该Block到L2缓存。
    3. 使用当前K Block进行Q·K<sup>T</sup>计算，同时预取操作在后台进行。
    4. 迭代下一次，此时预取的块已在L2中，实现命中。
  - 对V Block同样适用，在加载当前V Block时预取下一个。
- **性能边界分析**：单个KV Block大小为 `b * d_h * T_block`（字节），每次迭代加载的Block总量为 `M_block * (N_thread/32) * H * B`。H100的60MB L2缓存理论上可容纳120批次的KV Block（batch size=1）。但当batch size过大超出L2容量时，部分预取块仍可能被驱逐，但通过调整缓存驱逐优先级可缓解。

## 3. 实验设计
- **数据集/场景**：使用多种主流开源LLM模型进行推理，包括 Llama2-7B、Llama3-8B、Qwen2.5-7B、Qwen2.5-14B，均为FP16精度。输入token长度固定为512，输出token长度从512到8192变化，batch size从1到128变化。
- **Benchmark**：在vLLM v0.7.1框架下，对比三种后端实现：
  - 原生XFormers backend（baseline）
  - FlashAttention-3（FA3，state-of-the-art baseline）
  - 本文提出的方法（This Work）
- **评估指标**：
  - 注意力核性能：核持续时间、计算吞吐率、内存吞吐率、L2缓存命中率、每指令周期数（CPI）、Stall Long Scoreboard周期数、加速比。
  - 端到端推理吞吐量（tokens/s）：单GPU及多GPU（2/4/8 GPU张量并行）配置。

## 4. 资源与算力
- **硬件平台**：4个Intel Xeon Platinum 8469C处理器（192核） + 8个NVIDIA H20 GPU（每个含60MB L2缓存、96GB HBM，内存带宽4.0 TB/s）。全部实验在H20 GPU上完成，未提及训练过程，所有结果均为推理阶段性能。
- **所有实验均在此集群上进行，单GPU或多GPU配置**。

## 5. 实验数量与充分性
- **实验数量**：论文报告了以下实验：
  - 4种模型在固定输出2048 token下，4个batch size（16/32/64/128）的单GPU吞吐量对比（图3）。
  - 2种模型（Llama3-8B、Qwen2.5-7B）在输出token长度512-8192、batch size 1-128全组合下的加速比热力图（图4）。
  - 2种模型（Llama3-8B、Qwen2.5-14B）在2/4/8 GPU张量并行下的吞吐量对比（图5）。
  - 注意力核性能对比表（表3），涵盖4种模型在固定batch size=64、输出4K条件下的详细指标。
- **充分性**：实验覆盖了不同模型架构（MHA与GQA不同比例）、不同序列长度、不同batch size、不同GPU数量，并提供了详细的硬件性能剖析数据。对比方法包括当前最强基线FlashAttention-3。实验设计较为充分，结论有数据支撑。
- **客观公平性**：所有实验均在同一硬件平台、相同输入输出长度下进行，基准设定明确。但未明确说明是否对FlashAttention-3进行了同等优化或调整为最佳配置，可能存在一定偏差。

## 6. 论文的主要结论与发现
- 本文提出的异步KV Cache预取方法显著提升了注意力核性能：
  - L2缓存命中率从原生XFormers的0.06%（Llama2-7B）提升至43.70%，其他模型提升至73%~83%。
  - Stall Long Scoreboard周期从21.34 cycles降低至4.13 cycles（Llama2-7B），其他模型降幅类似（CPI降低65.8%-67.5%）。
  - 核持续时间缩短，加速比达1.84×-2.15×。
- 端到端推理吞吐量提升：
  - 单GPU：相比XFormers提升41%-57%，相比FlashAttention-3最高提升110%（Llama2-7B）。但在Qwen2.5-7B（极端GQA 7:1）上，性能比FA3低2-5%，原因是GQA减少了KV头数量，预取收益衰减。
  - 多GPU：相比XFormers提升4%-76%，且随着TP规模增大，加速比因每GPU处理头数减少而递减。
- 加速效果随输出序列长度和batch size增加而增强，但在极大batch size下受限于L2缓存容量和显存限制。
- 该方法与现有优化技术正交，可无缝集成到现有推理框架中。

## 7. 优点
- **方法创新**：首次提出利用计算空闲期异步预取KV Cache到L2缓存，实现计算与访存重叠，从根本上隐藏HBM延迟，而非仅减少访问次数。
- **硬件-软件协同设计**：充分利用NVIDIA Hopper架构的 `cp.async.bulk.prefetch.L2` 指令，展现了底层硬件特性与上层算子优化的结合。
- **正交性与兼容性**：方法可直接应用于现有推理框架（如vLLM），不依赖其他优化技术，且能与FlashAttention等协同工作。
- **实验全面**：覆盖多种模型、多种负载配置，提供了详尽的硬件性能剖析数据（表1、表3），深入解释了性能瓶颈的来源。

## 8. 不足与局限
- **硬件依赖**：方法依赖于Hopper架构（Compute Capability 9.0），无法在更早的GPU架构（如Ampere、Turing）上使用，限制了推广范围。
- **GQA架构下的收益有限**：对于极端GQA比例（如Qwen2.5-7B 7:1），预取收益被压缩，甚至可能略逊于FlashAttention-3。论文未深入分析如何针对GQA进一步优化。
- **L2缓存容量限制**：当batch size过大（>120批次，以H100为例）超过L2缓存容量时，预取收益可能下降。论文仅通过实验观察到此现象，但未提出解决溢出的具体方法（仅提及调整驱逐优先级，但未验证）。
- **未与其他预取方法对比**：论文未与Cross-Device预取（如ZeRO-Infinity、DeepUM）或multi-GPU方法PRESERVE进行直接对比，尽管这些方法场景不同，但缺乏对比削弱了新颖性论述。
- **实验平台单一**：所有实验仅在一台H20 GPU集群上进行，未在H100、A100等常见芯片上验证，通用性未知。
- **缺少显存限制下的行为分析**：当输出token长度很长时vLLM可能触发重计算，论文提到了此现象但未详细分析预取方法在重计算时的表现。

（完）
