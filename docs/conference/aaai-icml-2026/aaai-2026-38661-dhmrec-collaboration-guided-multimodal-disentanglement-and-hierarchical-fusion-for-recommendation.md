---
title: "DHMRec: Collaboration-Guided Multimodal Disentanglement and Hierarchical Fusion for Recommendation"
title_zh: "DHMRec: 协作引导的多模态解耦与层次化融合推荐"
authors: "Xiaohan Zhan, Yuliang Shi, Jihu Wang, Shijun Liu, Fanyu Kong, Zhiyong Chen"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38661/42623"
tags: ["query:rec-sys"]
score: 7.0
evidence: 基于深度学习的多模态推荐解耦与融合
tldr: 本文提出DHMRec框架，用于多模态推荐系统中的模态解耦与层次化融合。该方法通过协作引导解耦冗余信号和噪声，并采用层次融合策略整合不同模态表示，以反映用户偏好。实验证明，DHMRec在多个多模态推荐任务上优于现有方法，有效提升了推荐质量。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 多模态推荐中存在模态信息冗余和融合困难的问题。
method: 提出协作引导的多模态解耦和层次化融合方法，分离冗余信号并自适应整合。
result: 在真实数据集上，DHMRec在推荐准确率上超越了多种多模态推荐模型。
conclusion: 解耦与层次化融合能有效提升多模态推荐性能。
---

## Abstract
Multimodal recommender systems have emerged as a pivotal paradigm for harnessing diverse data modalities to deliver personalized services. Contemporary research predominantly focuses on integrating heterogeneous modality information through graph learning. However, these approaches face two key challenges: (1) the inherent complexity of modalities, characterized by entangled redundant signals and noise; and (2) the challenge of effectively integrating multimodal representations, each of which may exert varying degrees of influence on users' preferences. To address these challenges, we propose a novel Collaboration-Guided Multimodal Disentanglement and Hierarchical Fusion for Recommendation (DHMRec), which simultaneously achieves intra-modal denoising disentanglement and inter-modal hierarchical fusion. Specifically, we introduce a collaboration-related modality disentanglement module to distinguish between modality-common and modality-specific features. Then, through multi-view graph learning to capture both item-item dependencies and user-item interaction patterns. Additionally, we implement hierarchical fusion between the disentangled multimodal features and ID embeddings using a positive-negative attention-aware fusion module and an interaction distribution-based alignment module. Extensive experiments  on three benchmarks demonstrate that our DHMRec surpasses various state-of-the-art baselines, highlighting its effectiveness in intra-modal disentanglement and multimodal features fusion.

---

## 论文详细总结（自动生成）

# 论文结构化总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **问题**：多模态推荐系统面临两大挑战：  
  - **C1（模态内部）**：多模态特征中存在冗余信号和与用户偏好无关的噪声，导致特征纠缠，难以提取纯净的语义表示。  
  - **C2（模态间）**：不同模态对用户偏好的影响程度不同，现有方法往往平等对待所有模态，无法自适应融合，可能放大无关信号。  
- **动机**：现有方法（如MMGCN、LATTICE等）虽利用图学习整合多模态信息，但未能有效解耦和去噪，且缺乏动态融合策略。  
- **整体含义**：提出DHMRec框架，同时实现**模态内去噪解耦**和**模态间层次化融合**，以提升推荐准确性和鲁棒性。

## 2. 方法论：核心思想、关键技术细节
- **核心思想**：通过协作信号引导的模态解耦分离出公共与私有特征，再通过多视图图学习捕获结构和语义关系，最后设计层次融合策略动态整合解耦特征与ID嵌入。  
- **关键技术细节**：  
  1. **协作相关模态解耦模块**：  
     - 将原始多模态特征（文本、视觉）投影到统一空间。  
     - 使用公共编码器和私有编码器分别提取公共表示 \(C_{i,m}\) 和私有表示 \(P_{i,m}\)。  
     - **协作信号门控去噪**：将私有表示输入门控网络，由ID嵌入（含协同信息）引导生成去噪后的模态特定表示 \(S_{i,m}\)（公式3）。  
     - 通过**Jensen-Shannon散度（JSD）**对齐公共表示，并利用**正交损失**使公共与私有特征分离。  
     - 公共特征通过Logit Pooling融合得到 \(C_i\)。  
  2. **多视图图学习模块**：  
     - 构建四个用户-物品图（ID、公共、文本、图像视图），采用度敏感边剪枝缓解过平滑。  
     - 利用KNN算法构建模态物品-物品图，选取top-k邻居，并将文本和图像相似度矩阵加权得到公共物品图。  
     - 进行多层图卷积，得到各视图的用户/物品嵌入。  
  3. **层次融合模块**：  
     - **第一层（解耦特征融合）**：设计**正负注意力（Pos-Neg Attention）**，同时捕获正相关（高相似度）和负相关（低相似度但有信息）信号，将三视图特征动态融合。  
     - **第二层（多模态与ID特征融合）**：通过计算交互分布，使用**KL散度**对齐多模态融合嵌入与ID嵌入的分布，最终加权求和得到最终嵌入。  
  4. **优化目标**：BPR损失 + 解耦损失（JSD+正交）+ KL对齐损失 + L2正则化。

## 3. 实验设计
- **数据集**：Amazon子集 **Baby**、**Sports**、**Clothing**（经典多模态推荐基准）。  
- **基准方法**：  
  - **通用CF**：BPR、LightGCN、LayerGCN。  
  - **多模态模型**：VBPR、MMGCN、GRCN、DualGNN、LATTICE、MICRO、BM3、FREEDOM、LGMRec、DA-MRS、DiffMM、TMLP、CMDL（共14种）。  
- **评估协议**：8:1:1随机划分训练/验证/测试，评价指标为 **Recall@K** 和 **NDCG@K**（K=10,20）。  
- **实现**：嵌入维度64，Xavier初始化，Adam优化器，网格搜索超参数。

## 4. 资源与算力
- **文中说明**：使用 **NVIDIA GeForce RTX 4080** GPU，batch size = 2048。  
- **复杂度分析**：报告了平均训练时间每epoch和GPU内存成本（图6），但未给出具体数值（仅以柱状图展示相对对比）。  
- **结论**：DHMRec在取得最高精度的同时，训练时间和内存成本处于中等水平，优于多数多模态方法（如CMDL需减小batch size才能运行）。

## 5. 实验数量与充分性
- **性能对比实验**：表1涵盖3个数据集 × 4个指标 × 17种基线，统计显著性检验（p<0.05）。  
- **消融实验**：表2，5个核心变体（去除解耦、去平均融合、去正负注意、去KL对齐、去物品-物品图）。  
- **超参数分析**：图4（融合权重 \(\alpha_{mm}\)）、图5（\(\gamma_{dis}\)与\(\gamma_{kl}\)组合）。  
- **可视化分析**：图7（t-SNE验证解耦效果）、图8（验证多视图图学习对嵌入分布的影响）。  
- **充分性评价**：实验覆盖全面，从准确性、消融、超参数、效率、可视化多个维度验证，公平性良好（所有基线按官方设置调优）。

## 6. 主要结论与发现
- DHMRec在所有数据集上一致超越所有基线，在Recall@20上相对最佳基线提升约1~3%。  
- 多模态信息显著提升推荐性能（对比纯CF模型）。  
- 解耦模块有效去除冗余噪声，正负注意机制优于标准自注意。  
- 层次融合策略自适应平衡各模态贡献，对稀疏数据（Clothing）需要更大融合权重。  
- 可视化证明解耦后特征分离清晰，多视图图学习使嵌入分布更紧凑。

## 7. 优点
- **创新性强**：首次将协作信号引导的解耦与正负注意力结合用于多模态推荐。  
- **设计合理**：解耦-图学习-层次融合的流程逻辑清晰，逐步去除噪声、捕获关系、动态融合。  
- **实验充分**：多数据集、多基线、多消融、可视化，验证了各组件有效性。  
- **代码开源**（GitHub），可复现性强。  
- **效率可控**：在中等GPU资源下取得SOTA性能。

## 8. 不足与局限
- **模态覆盖有限**：仅使用文本和视觉模态，未涉及音频、视频等，未来可扩展。  
- **数据集局限**：仅使用Amazon子集，领域偏向商品推荐，泛化性需在更多场景（如新闻、社交）验证。  
- **超参数敏感**：不同数据集的最优\(\alpha_{mm}\)差异较大（0.8~1.2），需手工调参。  
- **理论基础**：解耦损失（JSD+正交）的合理性主要靠实验验证，缺乏理论收敛性分析。  
- **未讨论冷启动**：论文未专门分析模型在冷启动物品/用户上的表现。

（完）
