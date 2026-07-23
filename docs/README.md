<div class="dpr-home-notice-card">
  <h3 class="dpr-home-notice-title">🚀 Start Here</h3>
  <ul class="dpr-home-notice-list">
    <li><a href="#/tutorial/README">使用教程</a></li>
  </ul>
</div>

## 每次日报
- 最新运行日期：2026-07-23
- 运行时间：2026-07-23 21:22:00 UTC
- 运行状态：成功
- 本次总论文数：4
- 精读区：2
- 速读区：2

### 今日简报（AI）
1) 今日精选两篇高影响力论文，揭示KV缓存安全漏洞与高效解码新架构。  
2) 最值得关注方向：KV缓存重用存在劫持攻击风险（HijackKV，10分）；MoA结构化解码结合GQA/MQA可优化累积与加速（9分）。  
3) 建议普通读者优先评估自身KV缓存实现是否易受攻击，并关注MoA对长序列推理的潜力。
- 详情：[/202607/23/README](/202607/23/README)

### 精读区论文标签
1. [HijackKV: New Threat in Position-Independent KV Cache Reuse](/202607/23/2607.19957v1-hijackkv-new-threat-in-position-independent-kv-cache-reuse)  
   标签：评分：10.0/10、query:llm-kv-cache
   evidence：讨论KV缓存重用安全性，提出位置无关KV缓存重用中的新威胁（KV缓存劫持）
2. [MoA-Structured Decode Attention DNF Derivation, KV-Cache Accumulation, GQA/MQA, and OpenACC Kernel](/202607/23/2607.19456v1-moa-structured-decode-attention-dnf-derivation-kv-cache-accumulation-gqamqa-and-openacc-kernel)  
   标签：评分：9.0/10、query:llm-kv-cache
   evidence：推导了基于KV缓存累积的Transformer注意力内存最优推理方法

### 速读区论文标签
1. [LaCache: Exact Caching and Precision-Adaptive Inference for Diffusion Large Language Models](/202607/23/2607.16339v2-lacache-exact-caching-and-precision-adaptive-inference-for-diffusion-large-language-models)  
   标签：评分：7.0/10、query:llm-kv-cache
   evidence：为扩散大语言模型缓存中间状态以减少重计算
2. [ChronoStitch: Training-Free Composition of Visual KV Memories for Long-Horizon Temporal Reasoning](/202607/23/2607.19547v1-chronostitch-training-free-composition-of-visual-kv-memories-for-long-horizon-temporal-reasoning)  
   标签：评分：7.0/10、query:llm-kv-cache
   evidence：视觉KV缓存组合用于长视频问答


<div class="dpr-home-promo-card">
  <h3 class="dpr-home-promo-title">💬 社区与支持</h3>
  <ul class="dpr-home-promo-list">
    <li>欢迎 Star / Fork / Issue / PR</li>
    <li>QQ群：583867967（欢迎交流，已有：1151人）</li>
  </ul>
</div>
