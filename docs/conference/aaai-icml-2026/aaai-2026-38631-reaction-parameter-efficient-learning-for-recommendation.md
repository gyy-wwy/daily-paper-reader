---
title: "REACTION: Parameter-Efficient Learning for Recommendation"
title_zh: REACTION：面向推荐的参数高效学习
authors: "Song-Li Wu, Zhaocheng Du, Qinglin Jia, Zhenhua Dong"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38631/42593"
tags: ["query:rec-sys"]
score: 8.0
evidence: 面向深度推荐模型的参数高效学习
tldr: 本文从信息论角度证明现有深度学习推荐模型存在特征冗余和结构冗余。提出REACTION方法，通过信息论裁剪冗余特征和参数，实现参数高效学习，降低计算复杂度而不损失性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 深度学习推荐模型存在显著的特征和结构冗余，导致计算复杂度高。
method: 从信息论角度识别冗余，并提出REACTION方法进行参数高效学习。
result: 在保持推荐性能的同时大幅减少参数和计算量。
conclusion: 信息论指导的参数压缩是提升推荐模型效率的有效手段。
---

## Abstract
While deep learning (DL) has demonstrated significant success in recommender systems, it suffers from high computational complexity and poor scalability. In this work, we demonstrate, from an information-theoretic perspective, the redundancy of existing DL-based recommender models in two aspects: (1) Feature Redundancy. We show that many features are highly mutually correlated, noisy, or weakly predictive of user-item interaction labels. (2) Structural Redundancy. We further show that a large proportion of parameters in the dense layers contribute minimally to overall performance, indicating significant redundancy within the model architecture. To address these challenges, we propose REACTION (paRameter-Efficient LeArning for recommendaTION), an information-theoretic framework designed to reduce model complexity without sacrificing performance. REACTION consists of two core components: Adaptive Feature Extraction (AFE) leverages mutual information to  project high-dimensional sparse features into a compact, informative subspace. This adaptively filters noisy or weak features, reduces embedding parameters, and preserves implicit feature interactions without explicit high-order computation. 
Dynamic Tower Fusion (DTF) bridges the representational gap between dual-tower expressiveness and single-tower efficiency. It facilitates rich cross-tower interactions during training, then merges the towers into a unified, low-latency single tower for inference. 
Extensive experiments on four large-scale benchmarks demonstrate that REACTION not only outperforms existing methods in accuracy but also achieves a drastic reduction in both model parameters and inference costs, thus establishing a new paradigm for efficient and scalable recommendation systems.

---

## 论文详细总结（自动生成）

# 中文详细总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：现有深度学习推荐模型存在两方面的冗余，导致高计算复杂度和低可扩展性：
  - **特征冗余**：大量特征相互高度相关、包含噪声或对预测标签贡献微弱，约46%的特征可被移除而不损失性能。
  - **结构冗余**：稠密层中大部分参数对整体性能贡献极小，保留仅10%的稠密层参数可维持98.5%的性能。
- **整体含义**：论文从信息论角度证明了这些冗余的存在，并提出REACTION框架，旨在通过**参数高效学习**减少模型复杂度而不牺牲推荐精度，从而建立高效可扩展的推荐系统新范式。

## 2. 论文提出的方法论

### 核心思想
- 设计两个协同模块：**自适应特征提取（AFE）** 和 **动态塔融合（DTF）**，分别解决特征冗余和结构冗余。
- AFE通过互信息最大化实现线性复杂度的特征子集选择；DTF通过训练中的多层级融合损失将双塔表示统一，使推理时仅需单塔运行。

### 关键技术细节

#### Adaptive Feature Extraction (AFE)
- **目标**：从高维稀疏特征中选择一个高度信息性的小子集 \( S' \)，最大化 \( I(S'; L) \)（特征与标签的互信息）。
- **挑战**：直接计算条件互信息 \( I(X_m; L | S) \) 需要指数级复杂度。
- **解决方案**：
  - 引入**推荐标签独立分布假设（RLID）**，将联合分布近似为标签与特征间成对相互作用的乘积，复杂度从指数级降为线性 \( O(|L||S|) \)。
  - **低阶因子分解（定理1）**：将条件互信息分解为三个可解释的低阶项：
    - **偏好信息（PI）**：特征与单个标签的直接相关性。
    - **行为信息（BI）**：特征在不同标签之间（如点击、购买）的耦合信息。
    - **协作信息（CI）**：特征与已选特征之间的协同增益。
  - 特征选择评分 \( J(X_m) = PI + BI + CI \)，通过贪心前向选择逐步添加特征。

#### Dynamic Tower Fusion (DTF)
- **目标**：在训练阶段实现双塔（S1和S2）之间的深层融合，使推理时可仅使用单塔，保留表达能力的同时大幅降低延迟。
- **三步融合策略**：
  - **空间特征融合（Spatial Fusion）**：对两塔特征图逐位置施加余弦相似度损失 \( L_{srf} \)，并设置边界 \( m_1 \) 减少对已对齐特征的惩罚。
  - **关系特征融合（Relational Fusion）**：对齐两塔内部特征间的成对相似性矩阵，损失 \( L_{rrf} \) 保持几何结构一致性。
  - **跨塔实例对比融合（Instance-Contrastive Fusion）**：基于InfoNCE损失（\( L_{contra} \)）拉近同一样本在两塔的表示，推远不同样本。
- **联合优化目标**：每个塔的损失为 \( L_{S_k} = L_{CE} + \alpha L_{srf} + \beta L_{rrf} + \gamma L_{contra} \)，其中 \( L_{CE} \) 为交叉熵损失。

### 算法流程（文字描述）
1. **训练前**：通过AFE从全部d个特征中选出k个信息性最强的特征（k ≪ d），并构建紧凑的嵌入表。
2. **训练阶段**：输入特征经过嵌入后分别送入两个轻量MLP塔。每个小批量计算两塔的预测交叉熵损失，同时计算空间融合、关系融合和对比融合损失，反向传播更新两塔参数。
3. **推理阶段**：舍弃其中一个塔，仅用另一个塔进行前向推理，实现单塔低延迟。

## 3. 实验设计

### 数据集
- **Criteo**：45M样本，39个字段，2.1M特征
- **Avazu**：40M样本，23个字段，1.5M特征
- **MovieLens**：2M样本，3个字段，90K特征
- **UGC**：102M样本，92个字段，10.6M特征

### Benchmark 与方法对比
- 对比方法涵盖多种SOTA模型：
  - 浅层模型：FmFM、FwFM
  - 深度交互模型：xDeepFM、DCNV2、FiBiNet、AutoInt+、FiGNN
  - 知识蒸馏方法：ECKD、BKD、KD-DAGFM
  - 其他高效方法：APG、AutoDis、FinalMLP、LLM-ESR
- 评估指标：**AUC**、**Logloss**、**参数量（Params）**、**FLOPs**、**推理延迟（Latency）**

## 4. 资源与算力

- **GPU型号**：单张NVIDIA GeForce RTX 3090
- **运行设置**：每个方法随机种子运行10次取平均，未明确报告每次的训练时长（如epoch数或总时间），仅提到使用Adam优化器、学习率搜索范围1e-6~1e-2，L2正则化搜索范围1e-6~1e-3。
- **已知缺失信息**：未说明具体批次大小、训练轮数或每个实验的平均耗时，因此无法精确评估总计算成本。

## 5. 实验数量与充分性

- **整体性能对比**：在4个数据集上，REACTION与14种基线方法比较，报告了AUC、Logloss、参数量、FLOPs、延迟，并进行t检验（p < 1e-2或1e-4），实验充分。
- **消融实验**：
  - **DTF消融**：对比Baseline、Baseline+Entropy Regularization(ER)、Baseline+Knowledge Distillation(KD)、Baseline+DTF，在4个数据集上DTF显著优于其他变体。
  - **AFE消融**：在UGC数据集上与PCA、LASSO、IDARTS、AutoField、TayFCS比较，AFE取得最高AUC和最低Logloss。
- **缩放实验**：在Criteo和UGC上考察增加塔数量（1~8个）的影响，结果表明确实有性能提升但边际收益递减。
- **充分性评价**：实验覆盖主流数据集和基线，消融设计合理，统计检验增加了可信度。但缺少超参数敏感性分析、不同embedding维度下的对比、以及与其他参数高效方法（如剪枝、量化）的直接比较。

## 6. 论文的主要结论与发现

1. 深度学习推荐模型存在显著的特征冗余和结构冗余，大量参数可被移除而不明显影响性能。
2. AFE模块通过低阶互信息分解实现了线性复杂度的特征选择，有效过滤噪声并保留关键交互信号。
3. DTF模块使双塔在训练中实现深度表示对齐，推理时仅需单塔即可达到甚至超越双塔的精度，大幅降低计算开销。
4. 完整的REACTION框架在四个大规模基准上均取得最优AUC和Logloss，同时参数量最少、FLOPs最低、延迟最短（例如Criteo上4.83M参数，0.148ms延迟）。
5. 随着塔数量的增加，模型性能进一步提升，但效率增益需在多样性和成本之间权衡。

## 7. 优点

- **创新性**：从信息论角度系统性地识别并解决两类冗余，方法理论扎实。
- **效率显著**：AFE将选择复杂度从指数级降至线性，DTF实现推理时单塔，参数和计算量减少一个数量级以上。
- **实验全面**：在四个不同规模数据集上与14种SOTA方法对比，涵盖不同架构类型（浅层、深层、蒸馏、大模型），结果具有说服力。
- **可扩展性**：塔数量可灵活调节，验证了模型在不同规模下的扩展能力。
- **实用性**：推理延迟低至0.148ms，非常适合工业实时部署。

## 8. 不足与局限

- **实验覆盖局限性**：仅在一个数据集（UGC）上比较了AFE与其他特征选择方法，其他三个数据集上未进行类似消融，可能削弱AFE泛化性的证据。
- **超参数未深入分析**：DTF中三个融合损失的权重（α, β, γ）如何影响性能未作系统研究；AFE中选择特征数k的最优值也未讨论。
- **缺乏与其他效率技术的对比**：未与模型剪枝、量化、低秩分解等标准压缩方法比较，无法凸显REACTION在参数高效学习中的相对优势。
- **冷启动与长尾处理**：论文未分析AFE对低频或长尾特征的处理效果，这些特征在推荐中至关重要。
- **资源报告不完整**：未提供训练时间、收敛步数等关键效率指标，影响对整体计算成本的评估。
- **未进行线上A/B测试**：所有结果基于离线实验，实际工业环境中的性能与稳定性有待验证。

（完）
