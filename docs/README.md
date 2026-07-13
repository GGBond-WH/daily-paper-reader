<div class="dpr-home-notice-card">
  <h3 class="dpr-home-notice-title">🚀 Start Here</h3>
  <ul class="dpr-home-notice-list">
    <li><a href="#/tutorial/README">使用教程</a></li>
  </ul>
</div>

## 每次日报
- 最新运行日期：2026-07-13
- 运行时间：2026-07-13 20:30:17 UTC
- 运行状态：成功
- 本次总论文数：7
- 精读区：4
- 速读区：3

### 今日简报（AI）
今日聚焦两篇9.0高分论文，分别探索块稀疏注意力与KV缓存驱动的多智能体过程奖励建模。  
最值得精读的方向：COBS通过累积量阶稀疏突破注意力瓶颈，KV-PRM借助缓存传输实现高效过程奖励建模，两者在长序列计算和分布式推理中表现亮眼。  
建议普通读者优先精读这两篇，结合速读列表中TileLens的二维内存布局思路，可更全面理解高效Transformer与内存优化前沿。
- 详情：[/202607/13/README](/202607/13/README)

### 精读区论文标签
1. [COBS: Cumulant Order Block Sparse Attention](/202607/13/2607.09052v1-cobs-cumulant-order-block-sparse-attention)  
   标签：评分：9.0/10、query:llm-kv-cache
   evidence：缓解大语言模型KV缓存读取瓶颈
2. [KV-PRM: Efficient Process Reward Modeling via KV-Cache Transfer for Multi-Agent Test-Time Scaling](/202607/13/2607.09153v1-kv-prm-efficient-process-reward-modeling-via-kv-cache-transfer-for-multi-agent-test-time-scaling)  
   标签：评分：9.0/10、query:llm-kv-cache
   evidence：KV-PRM直接利用LLM生成阶段的KV缓存加速过程奖励模型
3. [Attention to Detail: Evaluating Energy, Performance, and Accuracy Trade-offs Across vLLM Configurations](/202607/13/2607.09172v1-attention-to-detail-evaluating-energy-performance-and-accuracy-trade-offs-across-vllm-configurations)  
   标签：评分：9.0/10、query:llm-kv-cache
   evidence：vLLM配置中的前缀缓存研究
4. [General Non-Clairvoyant KV-Cache Scheduling via Regime-Aware Routing](/202607/13/2607.09248v1-general-non-clairvoyant-kv-cache-scheduling-via-regime-aware-routing)  
   标签：评分：8.0/10、query:llm-kv-cache
   evidence：LLM推理中KV缓存调度及内存预算

### 速读区论文标签
1. [TileLens: Efficiently Using Large-Granularity Memory Systems with Transparent Two-Dimensional Memory Layout](/202607/13/2607.04031v1-tilelens-efficiently-using-large-granularity-memory-systems-with-transparent-two-dimensional-memory-layout)  
   标签：评分：7.0/10、query:llm-kv-cache
   evidence：通过二维布局优化LLM推理内存系统，减少读放大问题，间接提升KV缓存效率
2. [Sparse Delta Memory: Scaling the State of Linear RNNs through Sparsity](/202607/13/2607.07386v1-sparse-delta-memory-scaling-the-state-of-linear-rnns-through-sparsity)  
   标签：评分：7.0/10、query:llm-kv-cache
   evidence：线性RNN的稀疏键值记忆，可应用于KV缓存压缩
3. [BlockServe: Block-Grained Continuous Batching for High-Throughput Diffusion LLM Serving](/202607/13/2607.08930v1-blockserve-block-grained-continuous-batching-for-high-throughput-diffusion-llm-serving)  
   标签：评分：6.0/10、query:llm-kv-cache
   evidence：面向扩散LLM服务的块粒度调度与双缓存并行解码


<div class="dpr-home-promo-card">
  <h3 class="dpr-home-promo-title">💬 社区与支持</h3>
  <ul class="dpr-home-promo-list">
    <li>欢迎 Star / Fork / Issue / PR</li>
    <li>QQ群：583867967（欢迎交流，已有：1151人）</li>
  </ul>
</div>
