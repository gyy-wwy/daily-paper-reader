---
title: "FusedRec: Fused Embedding Communication for Distributed Recommendation Training on GPUs"
title_zh: FusedRec：面向分布式推荐训练的融合嵌入通信方法
authors: "Xuanteng Huang, Fan Li, Riyang Hu, Jianchang Zhang, Yuan Peng, Yang Zhou, Fangying Chen, Xianwei Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38512/42474"
tags: ["query:rec-sys"]
score: 6.0
evidence: 深度学习推荐模型训练优化
tldr: 分布式训练DLRM存在通信碎片化问题。本文提出FusedRec，通过融合嵌入通信和查找机制减少碎片化，提升训练效率。实验表明FusedRec在多个GPU集群上显著加速训练过程，适用于大规模推荐服务。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有去重策略导致分布式DLRM训练中通信碎片化，影响性能。
method: 提出FusedRec，通过融合嵌入通信和查找减少碎片。
result: 在GPU集群上训练速度提升显著。
conclusion: 融合通信有效解决分布式推荐训练瓶颈。
---

## Abstract
Recent years have witnessed the wide adoption of deep learning recommendation models (DLRMs) for many online services. Unlike traditional DNN training, DLRMs leverage massive embeddings to represent sparse features, which are stored in distributed GPUs following the model parallel paradigm. Existing approaches adopt deduplication to eliminate replicated embeddings involved in AltoAll transfers to avoid unnecessary communication. In our practices, we have observed that such a deduplication design exacerbates interconnect inefficiency due to the fragmented embedding transfers with reduced message sizes, hindering the performance of distributed DLRM training.

This paper introduces FusedRec, a fused embedding communication and lookup mechanism to tackle the inefficiency due to deduplication. By seeking the opportunities to fuse embeddings from multiple categories into a group, FusedRec conducts the communication in a combined shot to alleviate bandwidth under-utilization. Meanwhile, a categorical-aware hashing algorithm is integrated into FusedRec to retain the category information during lookup without extra communication. Combining with efficient unique and recovery operations, comprehensive results show FusedRec achieves a 37.8% throughput speedup in average compared to the SOTA industry implementation, without hurting the recommendation qualities of our in-house models used in online production environments.

---

## 论文详细总结（自动生成）

好的，我将基于您提供的论文内容，按照要求生成一段详细的中文总结。

---

### 1. 核心问题与整体含义（研究动机和背景）

- **背景**：深度学习推荐模型（DLRM）广泛用于在线服务，训练时依赖大量嵌入表（embedding tables）表示稀疏特征。由于嵌入表规模巨大，通常采用模型并行（Model Parallel）方式将表格分布到多个 GPU 上，并通过 AlltoAll 通信进行分布式嵌入查找。
- **核心问题**：现有方法（如 RecD、PICASSO）为了减少通信量，会在批次内对每个类别内的重复嵌入 key 进行去重。然而，这种去重操作使每次 AlltoAll 传输的消息尺寸变得很小（多数小于 64KB），导致宝贵的互连带宽（NVLink/InfiniBand）利用率严重不足（通常低于峰值的一半）。此外，特征按类别独立通信的模式进一步加剧了这种碎片化，使得分布式训练性能受到限制。
- **研究动机**：如何在保留去重优势的同时，通过合并多个稀疏特征的嵌入通信来提升带宽利用率，并保持嵌入查找的语义完整性（不丢失类别信息）。

### 2. 方法论：核心思想、关键技术细节与算法流程

- **核心思想**：FusedRec 提出**融合嵌入通信与查找**机制。将具有相同规格（维度、优化器、初始化器）的不同稀疏特征融合成一个组，以组为单位进行合并的 AlltoAll 通信，从而增大单次传输的消息尺寸，提升带宽利用率。
- **关键技术细节**：
  - **融合与分段去重**：先根据规格将多个类别的嵌入 key 拼接成融合组。然后使用自定义的 **Segment Unique** GPU 算子，在保持类别边界的前提下，一次性地去除组内每个类别的重复 key，同时保留不同类别中冲突的 key（如 `a0` 和 `b0`）。该算子利用 warp-level 原语（ballot、popc）实现高效去重，并返回逆映射索引。
  - **路由与排序**：对去重后的唯一 key 按目标 worker 进行重排序（Rankwise Sort），使得发往同一目的地的 key 连续存放，满足 AlltoAll 的输入要求。
  - **无损查找**：在接收端，通过**延迟哈希**保留类别信息：先将接收到的 token 与类别名拼接（如 `a/2`），再哈希以消除跨 worker 的全局重复。这样既能过滤冗余，又能在查找时获知完整的类别-实例信息，保证无损。
  - **嵌入恢复**：利用之前保存的 Segment Unique 逆映射和 Rankwise Sort 逆映射，通过两次 `tf.gather` 操作将接收到的去重、融合后的嵌入恢复到原始批次顺序和类别分离的布局，供后续 MLP 计算。
- **算法流程**（Algorithm 1）：
  1. 对输入批次的 M 个类别按规格分组得到融合组 G。
  2. 对每个融合组 g：
     - 执行 Segment Unique，得到唯一组 gu 和逆映射 Iu。
     - 按目标 worker 排序得到有序组 gs 和逆映射 Is，并执行 AlltoAll 发送 key。
     - 接收端执行全局哈希去重，本地嵌入查找，然后利用逆映射恢复出有序的嵌入结果 Gl。
     - 将 Gl 通过 AlltoAll 传回请求方，利用 Is 和 Iu 恢复为密集布局（步骤 c.1）。
  3. 后向过程镜像执行，以合并方式计算梯度并通信更新。

### 3. 实验设计：数据集、基准与对比方法

- **数据集**：使用腾讯内部的 **4 个生产级推荐模型**（W0、W1、W2、W3），其嵌入表大小分别为 4.4 TB、5.6 TB、8 TB、12.8 TB。训练数据来自实时数据流，覆盖数亿日活用户。
- **基准（Baseline）**：
  - **TFRA**（TensorFlow Recommenders Addons）：业界领先的分布式 DLRM 训练框架。
  - **TFRA-fused**：基于相同融合准则的改进版本，但采用源端哈希去重并需额外通信传递类别信息。
- **对比方法**：FusedRec 与 TFRA 和 TFRA-fused 进行端到端吞吐量、通信效率、延迟和 GPU 利用率对比。

### 4. 资源与算力

- **GPU 型号**：每节点 8 块 **NVIDIA H20 GPU**（96 GB VRAM）。
- **互连**：节点内 NVLink 带宽 900 GB/s；节点间 Mellanox ConnectX-7 NIC，400 Gb/s 双向 RDMA。
- **GPU 数量**：实验从 8 个 GPU（1 节点）扩展到 128 个 GPU（16 节点）。
- 论文未明确给出总训练时长或训练迭代次数等详细信息。

### 5. 实验数量与充分性

- **主要实验组**：
  - 端到端吞吐量比较（图 6）：在 4 个模型、5 种 GPU 规模（8/16/32/64/128）下对比 FusedRec、TFRA、TFRA-fused，共约 60 个数据点。
  - 通信效率分析（图 7）：在 W0 模型、32 GPU 下统计通信轮次和感知带宽比（按不同嵌入维度细分）。
  - 延迟分析与 GPU 利用率（图 8）：在 4 个模型下对比单次迭代延迟分解和 GPU 利用率。
- **消融实验**：无显式消融实验（如单独验证 Segment Unique 或融合策略效果），但通过 TFRA-fused 间接说明融合+高效去重优于简单融合。
- **充分性**：实验覆盖了多模型、多规模、多性能指标，结论具有较强支持。但缺少公开数据集（如 Criteo）的结果，只在内部生产环境验证，公平性可能受限。未提供准确性（AUC/NDCG）的详细数值，仅声称 no hurt。

### 6. 主要结论与发现

- **吞吐量提升**：FusedRec 相比 TFRA 平均加速 **37.8%**；相比 TFRA-fused 额外提升 **16.7%**。
- **可扩展性**：在 8→128 GPU 时，可扩展因子达 **0.93**，表明在大规模集群中仍能保持高效。
- **通信效率**：AlltoAll 轮次平均减少 **11.4×**，单次传输嵌入数量增加 **8.16×**；感知带宽提升 **5.66×**。
- **延迟与利用率**：单次训练迭代延迟最多缩短 **20.3%**；GPU 利用率提升 **11.4%**。
- **无损**：通过延迟哈希保持类别信息，无需额外通信，不影响推荐质量。

### 7. 优点

- **创新性**：首次系统性指出去重导致的带宽碎片化问题，并采用融合组通信从根本上缓解。
- **工程高效**：设计的 Segment Unique 算子一次 GPU 内核完成多类别去重，避免多次内核启动和动态分配；结合逆映射的恢复流程实现零额外开销。
- **兼容性**：可与现有嵌入调度（如 Bagpipe 的预取方案）正交结合，进一步提升性能。
- **评估全面**：在多模型、多 GPU 规模下验证，并展示了延迟、带宽、利用率等多维度指标。

### 8. 不足与局限

- **实验偏差风险**：仅在腾讯内部模型上测试，模型结构和数据分布可能不具备通用性；未在公开数据集（如 Criteo、Avazu）上进行对比，难以独立复现。
- **准确性评估不足**：论文仅称“不影响推荐质量”，但未给出 AUC、NDCG 等量化指标，缺乏严格验证。
- **消融实验缺失**：未单独分析融合组大小、Segment Unique 性能增益、哈希冲突对效果的影响等。
- **应用限制**：需要多稀疏特征具有相同规格（维度、优化器、初始化器）才能融合；若特征规格多样，融合收益可能降低。此外，对于特征数很少的 DLRM，碎片化问题可能不显著。
- **硬件依赖**：实验基于 H20 和特定网络配置，在低端硬件或不同拓扑下结论可能变化。

---

（完）
