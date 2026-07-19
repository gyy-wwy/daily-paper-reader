---
title: Task-Aware Meta-Learning on Heterogeneous Knowledge Graph for POI Recommendation
title_zh: 基于异质知识图谱的任务感知元学习用于POI推荐
authors: "Jingyuan Wang, Zhichun Wang, Tong Lu, Yiming Guan"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38615/42577"
tags: ["query:rec-sys"]
score: 9.0
evidence: 通过POI推荐实现旅游推荐
tldr: POI推荐在兴趣建模和适应性方面面临挑战。本文提出TMHKG，构建双视图POI知识图谱融合地理邻近性与类别转移，并利用任务感知元学习捕捉用户演化兴趣。在真实数据集上，TMHKG显著提升了POI推荐准确率，为位置服务提供了更智能的解决方案。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有POI推荐方法难以有效建模用户偏好多样性和动态性。
method: 构建双视图POI知识图谱，结合任务感知元学习进行推荐。
result: 在真实POI数据集上，TMHKG优于多种基线模型。
conclusion: 融合知识图谱与元学习有效提升POI推荐性能。
---

## Abstract
Point-of-Interest (POI) recommendation plays a pivotal role in location-based services by guiding users to discover new and relevant places. While graph-based methods have shown promising results, effectively modeling the diversity and dynamics of user preferences remains a key challenge. Addressing this requires richer representations of both POIs and user interests, as well as more adaptive learning strategies.
In this work, we propose TMHKG, a Task-aware Meta-learning framework with a Heterogeneous Knowledge Graph for POI recommendation. To enhance representation learning, TMHKG constructs a dual-view POI knowledge graph that integrates geographical proximity and user-aware category transitions, and models users' evolving interests from sequential visit histories. On top of enriched features, TMHKG adopts a task-aware meta-learning paradigm, treating each user's recommendation task as a separate meta-task. A generalizable recommendation policy is first learned from diverse training tasks and then quickly adapted to each user's unique behavior, enabling highly personalized predictions.
Extensive experiments on two real-world datasets demonstrate that TMHKG consistently outperforms state-of-the-art baselines, highlighting its effectiveness in capturing complex user-POI interactions.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **核心问题**：POI（兴趣点）推荐中，用户偏好具有高度的多样性和动态性，现有基于图神经网络的方法通常采用统一的聚合规则（“一刀切”方式），难以灵活适应个体用户的独特行为模式。同时，传统训练范式学习单一静态参数集，无法快速个性化。
- **研究动机**：需要更丰富的POI和用户兴趣表示、更自适应的学习策略。元学习（meta-learning）虽然被用于冷启动问题，但未充分挖掘其为所有用户提供个性化适应的潜力。因此，本文探索将推荐系统重构为元学习问题——每个用户作为一个独立任务。
- **整体含义**：提出TMHKG框架，通过构建增强型异质知识图谱与任务感知元学习，旨在学习高度泛化的元模型，并能快速适应每个用户的具体行为，实现高度个性化POI推荐。

## 2. 方法论

### 2.1 核心思想
TMHKG由四个紧密耦合的组件构成：
- **交互图学习**：构建属性增强的异质交互图（用户-POI边携带时间、空间、频率属性），通过边感知注意力GNN学习节点表示；同时使用LSTM编码用户轨迹序列（兴趣演化路径），并通过残差融合得到最终用户表示。
- **知识图谱学习**：构建POI知识图谱，引入两种新型关系：（1）类别转移边（基于用户轨迹中的共现模式）；（2）空间邻近边（地理半径内距离衰减加权）。采用关系感知传播（基于TransR投影思想）学习POI语义表示，并与交互图表示通过门控融合。
- **元学习优化**：基于MAML框架，每个用户视为一个任务，含支持集和查询集。参数分组：内环更新部分参数（交互图、用户融合、预测模块），外环优化全部参数。
- **记忆感知任务采样**：计算每个任务的综合信息得分（结构多样性+行为熵），并对频繁选中的任务施加惩罚，从而从候选池中按软分布采样，提升训练多样性和泛化性。

### 2.2 关键技术细节
- 交互图边属性：时间槽嵌入、距离桶嵌入、归一化频率，拼接成边表示，用于边门控（sigmoid）和注意力系数计算。
- 兴趣演化路径编码器：单层LSTM+注意力池化。
- 用户表示残差融合：轨迹重要性由MLP预测的标量加权。
- 知识图谱关系感知传播：每种关系有独立投影矩阵Mr和变换矩阵Wr，加上自环关系保留自身特征。
- 评分函数：MLP([hu ∥ hp ∥ hu ⊙ hp])。
- 损失函数：BPR排名损失 + 用户表示对齐损失（余弦距离） + POI双视图对齐损失，通过超参数α、β平衡。
- 内环学习率rinner、外环学习率router，任务批处理更新。

## 3. 实验设计

### 3.1 数据集与场景
- **Yelp**：真实业务数据（含评论、签到、属性），经筛选后：用户30,887，POI 18,995，签到860,888。
- **NYCRestaurant**：Foursquare纽约餐厅数据（签到、小费、标签），筛选后：用户3,112，POI 3,298，签到27,149。
- 按3:1:1划分训练/验证/测试。知识图谱基于POI元数据和Freebase构建。

### 3.2 Benchmark与对比方法
- 知识图谱增强方法：KGIN、KGAT。
- 行为演化方法：GETNext、STGCN。
- 元学习方法：MetaKG、HyperMAN。
- 另外还进行了消融实验（5种变体）和冷启动场景分析。

### 3.3 评估指标
- Recall@K (K=5,10,20) 和 NDCG@K (K=5,10,20)。

## 4. 资源与算力

- **文中未明确说明使用的GPU型号、数量、训练时长**。仅在参数设置中提到使用PyTorch、Adam优化器、batch size 4096、学习率网格搜索、嵌入维度64、Xavier初始化。因此无法提供具体的算力资源信息。

## 5. 实验数量与充分性

- **主要实验**：全模型对比（表2）在2个数据集上报告6个指标，共12个对比数值。
- **消融实验**（表3）：5种变体，在2个数据集上报告Recall@10和NDCG@10，共20个数值。
- **参数分析**（图2）：对L（交互图层数）、L'（知识图层数）、α、β四个参数进行单变量调优实验。
- **冷启动实验**（表4）：在用户冷启动设置下对比2个基线，2个数据集，4个指标。
- **充分性**：实验设计较为充分，覆盖了主效果对比、消融、超参数分析和冷启动场景。对比方法涵盖三类代表性方法，且使用两次真实数据集，统计显著性检验（标记最强基线）。实验客观公平，但未报告模型训练效率或收敛时间。

## 6. 主要结论与发现

- TMHKG在所有评估指标上显著优于所有基线方法。在Yelp上NDCG@20提升5.77%，NYCRestaurant上提升4.77%。
- 消融实验表明：知识图谱的贡献最大（w/o KG导致Recall@10从0.0901降至0.0664）；动态兴趣建模和记忆感知采样的去除也带来明显下降；基础知识图谱（无本文增强）和一致性损失的去除影响较小但仍可察觉。
- 参数分析表明存在最优深度和平衡权重，极端值会导致性能下降。
- 冷启动实验显示TMHKG大幅优于GETNext和HyperMAN（Yelp上Recall@10为0.0820 vs 0.0411/0.0566），证明了任务感知元学习对稀疏用户的适应性。

## 7. 优点

1. **新颖的问题重构**：将个性化POI推荐转化为元学习问题，将每个用户视为独立任务，打破了传统静态参数范式。
2. **双图协同架构**：交互图与知识图谱独立学习后融合，知识图谱中引入行为驱动的类别转移边和空间邻近边，增强语义和空间归纳偏置。
3. **记忆感知任务采样**：有效提升元训练任务多样性和代表性，避免冗余，提升泛化。
4. **冷启动处理能力强**：实验证明在用户冷启动下显著优于现有元学习方法。
5. **消融实验全面**：细致验证了每个组件的贡献。
6. **代码开源**（GitHub链接），可复现性好。

## 8. 不足与局限

1. **计算资源信息缺失**：未报告GPU型号、数量、训练时间，难以评估实际部署成本。
2. **数据集规模与领域限制**：仅使用两个POI数据集（Yelp和NYCRestaurant），且经过严格筛选（用户>10次签到、POI>10次访问），可能丢失稀疏性特征。结果是否泛化到其他领域（如旅游POI、城市级数据）未验证。
3. **冷启动定义局限**：仅测试了用户冷启动（UC），未测试POI冷启动或跨域冷启动场景。
4. **超参数敏感性**：α、β、L、L'等需精心调整，在非最优设置下性能下降明显（如消融实验w/o Sampling下降较大），可能影响实际部署稳定性。
5. **复杂性与可解释性**：模型组件多（交互图、知识图谱、LSTM、元学习循环），整体复杂度高，可解释性较弱。
6. **未与大规模LLM或预训练模型对比**：论文未提及与近年来基于语言模型的方法对比（如CoSTIG等），可能限制实验时效性。
7. **元学习计算开销**：内环外环优化导致训练成本增加，文中未分析时间效率。

（完）
