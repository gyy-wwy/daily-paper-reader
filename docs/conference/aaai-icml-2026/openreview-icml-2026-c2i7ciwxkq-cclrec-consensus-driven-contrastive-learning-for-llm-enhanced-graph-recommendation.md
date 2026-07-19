---
title: "CCLRec: Consensus-driven Contrastive Learning for LLM-enhanced Graph Recommendation"
title_zh: CCLRec：基于共识的对比学习用于LLM增强的图推荐
authors: "Ting Guo, Dongyu Pei, Litiao Qiu, Xiaoying Liao, KE LIANG, Peng Song, Pinle Qin"
date: 2026-04-30
pdf: "https://openreview.net/pdf/10c5791d639bc10f91df7571fa8e09e298dc2932.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 图推荐，LLM增强
tldr: 该论文提出CCLRec，一个共识驱动的对比学习框架用于LLM增强的图推荐。现有方法分离处理图结构和LLM语义，导致结构近邻与语义相关性的监督差距。CCLRec通过对比学习深度融合两者，实验证明在多个推荐数据集上取得了最佳性能。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: LLM增强的图推荐中图结构信息和LLM语义知识分离，导致监督差距。
method: 提出共识驱动的对比学习，联合优化结构表示和语义表示。
result: 在多个基准数据集上，CCLRec超越了现有图推荐和LLM增强方法。
conclusion: CCLRec为图推荐与LLM的深度融合提供了有效范式。
---

## Abstract
Recommendation systems seek to accurately model user preferences from a large set of candidate items. Graph neural networks (GNNs) have emerged as a dominant approach in this domain due to their ability to capture high-order user–item interactions. Recent efforts have aimed to enhance GNN-based representation learning by incorporating the semantic reasoning capabilities of large language models (LLMs). However, existing methods often process graph structural information and LLM-derived semantic knowledge separately, creating a supervisory gap between structural proximity and semantic relevance. To bridge this gap, we propose CCLRec, a consensus-driven contrastive learning framework for recommendation. CCLRec deeply integrates structural and semantic information by identifying consistent signals. Specifically, we first use an LLM to extract semantic representations of items and to sample candidate positive/negative sets in the semantic space. We then introduce a structural–semantic consensus mining strategy that computes the intersection between a node’s structural neighbors in the graph and its semantically similar items. This allows us to identify high-confidence positive pairs endorsed by both collaborative filtering patterns and LLM-based reasoning. By centering contrastive learning on these consensus pairs and applying a weight-aware reinforcement mechanism during training, CCLRec significantly amplifies the contribution of high-quality consensus features during training. Experiments across multiple public benchmarks show that CCLRec consistently outperforms state-of-the-art methods on key metrics, demonstrating the effectiveness of our consensus-aware design.

---

## 论文详细总结（自动生成）

# CCLRec：基于共识的对比学习用于LLM增强的图推荐

## 1. 论文的核心问题与整体含义（研究动机和背景）
推荐系统旨在从大规模候选物品中准确建模用户偏好。图神经网络（GNN）因其能捕获高阶用户-物品交互而成为该领域的主流方法。近年来，研究者尝试利用大语言模型（LLM）的语义推理能力来增强基于GNN的表示学习。然而，现有方法往往将图结构信息和LLM语义知识分开处理，导致结构近邻与语义相关性之间存在监督差距（supervisory gap）。这限制了模型同时利用协作过滤信号和语义知识的能力。本文提出CCLRec框架，旨在通过共识驱动的对比学习深度融合结构和语义信息，从而弥合这一差距。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程（文字说明）
**核心思想**：通过识别结构近邻与语义相似集的一致性信号，构造高置信度的正样本对，并基于这些共识对执行对比学习，同时引入权重感知增强机制强化高质量共识特征的贡献。

**关键技术细节**：
- **LLM语义表示提取**：使用LLM提取物品的语义表示，并在语义空间中对候选正/负样本进行采样。
- **结构-语义共识挖掘**：计算图中节点的结构邻居与同一节点的语义相似物品集合的交集，得到同时被协作过滤模式和LLM推理认可的高置信正样本对。
- **对比学习以共识对为中心**：将对比学习的正样本构建限制在这些共识对上，而不是传统的随机或单一来源的样本。
- **权重感知增强机制**：在训练过程中，根据共识质量（例如交集的大小或置信度）动态调整样本权重，放大高共识特征的贡献。
- 整体框架将GNN编码的结构表示与LLM编码的语义表示通过对比学习联合优化，实现深度融合。

**算法流程（简略）**：
1. 输入：用户-物品交互图、物品原始文本特征。
2. 使用LLM生成物品语义嵌入，并基于语义相似度构建候选正/负集。
3. 在图结构上通过GNN传播得到结构嵌入，识别每个节点的结构邻居。
4. 对每个节点，计算结构邻居与语义相似物品集合的交集，得到共识正样本集。
5. 构建对比损失，以共识对为正样本，其他节点为负样本（可能采用温度系数及权重）。
6. 通过权重感知机制（如基于交集大小的权重）调整每个共识对的损失权重。
7. 联合优化GNN参数与LLM适配层，最小化对比损失。

（论文未提供具体公式，以上为基于摘要的合理推断）

## 3. 实验设计：数据集、基准、对比方法
**数据集**：在多个公开基准数据集上进行实验（摘要中未列举具体名称，但提及“multiple public benchmarks”）。
**对比方法**：包括最先进的图推荐方法（state-of-the-art GNN-based methods）以及LLM增强的推荐方法。
**评估指标**：主要指标包括推荐常用的点击率（CTR）、召回率（Recall）、NDCG等（具体指标未在摘要中详列，但统一报告了“key metrics”的超越）。
**实验设置**：未详细说明数据划分比例等，但通常遵循标准的时间或随机划分。

## 4. 资源与算力
**论文摘要及元数据中未提及任何具体的算力信息**（如GPU型号、数量、训练时长等）。因此，无法总结此部分。需要指出这一事实。

## 5. 实验数量与充分性
**实验数量**：至少包含多个公开数据集上的主实验，以及消融实验（ablation study）来验证共识挖掘和权重增强机制的有效性（从“consistently outperforms”及“demonstrating the effectiveness of our consensus-aware design”可以推断）。但具体消融实验组数、超参数敏感性分析、模型复杂度对比等未在摘要中说明。
**充分性与公平性**：结论声称在多个数据集上均超越当前最优方法，表明实验具有一定覆盖度。但缺乏对基线方法是否公平调优、随机种子重复次数等细节描述，无法完全判定。总体而言，基于摘要信息，实验设计基本合理，但细节不足。

## 6. 论文的主要结论与发现
- CCLRec通过共识驱动的对比学习，成功弥合了图结构近邻与LLM语义相关性之间的监督差距。
- 在多个公共基准上，CCLRec在关键指标上持续优于现有最先进的图推荐方法和LLM增强方法。
- 共识对识别和权重感知增强机制是性能提升的关键，证明了共识感知设计的有效性。

## 7. 优点：方法或实验设计上的亮点
- **创新性**：提出结构-语义共识挖掘，首次将两种不同模态的监督信号通过交集形式对齐，构建更可靠的正样本。
- **深度融合**：不是简单拼接或串联，而是通过对比学习直接拉近共识对，实现结构和语义的协同学习。
- **训练动态加权**：权重感知增强机制可自适应强调更可信的共识特征，避免噪声影响。
- **实验验证**：在多个数据集上验证了通用性，方法简洁而有效。

## 8. 不足与局限
- **数据集粒度缺失**：未具体列出数据集名称和统计信息，无法复现或评估域差异。
- **资源开销不明**：未报告LLM调用成本、训练时间或显存占用，对于实际部署需关注。
- **对LLM的依赖性**：共识挖掘依赖于LLM的语义表示质量，若LLM对物品语义理解偏差，则可能引入错误监督。
- **未讨论负样本策略**：对比学习中负样本的选择对性能至关重要，但摘要未提及细节。
- **缺乏更细粒度的消融**：例如，分别去除共识挖掘或权重增强的单独影响，仅笼统提及；也未分析共识阈值对性能敏感度。
- **可扩展性**：当图规模极大时，计算所有节点的结构邻居与语义相似集交集的复杂度可能较高，未说明优化措施。

（完）
