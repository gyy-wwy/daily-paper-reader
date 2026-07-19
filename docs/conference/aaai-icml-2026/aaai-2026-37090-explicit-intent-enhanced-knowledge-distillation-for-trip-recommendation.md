---
title: Explicit Intent-Enhanced Knowledge Distillation for Trip Recommendation
title_zh: 显式意图增强的知识蒸馏用于旅行推荐
authors: "Shuliang Wang, Xiaoting Leng, Sijie Ruan, Dingqi Yang, Yicheng Tang, Qianyu Yang, Qianxiong Xu, Jiabao Zhu, Hanning Yuan"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/37090/41052"
tags: ["query:rec-sys"]
score: 9.0
evidence: 显式意图增强知识蒸馏用于旅行推荐
tldr: 旅行推荐旨在根据用户查询生成兴趣点序列，现有监督方法难以捕获POI转换模式，自监督方法无法全面建模查询意图。本文提出EKD-Trip框架，利用特权知识蒸馏显式对齐用户查询意图与轨迹，先训练轨迹编码器，再蒸馏到轻量推荐模型。实验证明该方法在真实旅行数据集上显著优于基线模型。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有旅行推荐方法难以同时捕获POI转换和用户查询意图。
method: 提出EKD-Trip，通过显式意图对齐的知识蒸馏提升推荐。
result: 在真实旅行数据集上显著优于基线模型。
conclusion: 显式意图增强的知识蒸馏有效提升旅行推荐质量。
---

## Abstract
Trip recommendation aims to generate a sequence of points of interest (POIs) under a user's query input. Existing data-driven methods mainly fall into two categories: supervised approaches and self-supervised approaches. The former cannot fully capture the transition patterns among POIs, while the latter fail to comprehensively model user's query intents.
Fortunately, privileged knowledge distillation (PKD) provides us an unique opportunity to align user's query intents with its corresponding trip in historical data. However, such knowledge alignment is implicit, which may not directly reflect the query intents. To this end, in this paper, we propose EKD-Trip, an explicit intent-enhanced knowledge distillation framework. EKD-Trip first trains a trajectory encoder (teacher model) and a trip generator jointly in a self-supervised manner. Then, a query encoder (student model) is trained via multi-task learning to extract implicit knowledge by PKD from teacher and explicit knowledge from an auxiliary task, respectively. At inference time, we use the query encoder and the trip generator to recommend trips. Extensive experiments on four real-world datasets demonstrate that EKD-Trip outperforms all baselines over three metrics, with a particularly notable improvement of 13.70% in pairs-F1.

---

## 论文详细总结（自动生成）

# 论文中文总结：显式意图增强的知识蒸馏用于旅行推荐（EKD-Trip）

## 1. 核心问题与整体含义（研究动机和背景）
- **研究背景**：旅行推荐任务旨在根据用户的查询（包含起点、终点、开始时间、结束时间、期望POI数量）生成一个连续的POI序列。现有方法分为监督学习（如DeepTrip、AR-Trip）和自监督学习（如SelfTrip、Bert-Trip）两类。
- **核心问题**：
  - 监督方法依赖有限样本，难以充分捕获POI间的转移模式；
  - 自监督方法能利用大量历史轨迹学习转移知识，但无法全面建模查询中的意图（如时间、数量约束），导致推荐结果不符合用户实际需求。
- **整体含义**：需要一种方法同时兼顾轨迹转移模式的学习和查询意图的捕获。论文引入特权知识蒸馏（PKD），让学生模型（仅使用查询输入）通过蒸馏教师模型（使用完整轨迹）隐式学习轨迹知识，并显式地通过辅助任务（出行模式预测）增强对查询意图的理解。

## 2. 方法论：核心思想与关键技术细节
- **核心思想**：EKD-Trip框架分为两个训练阶段：
  1. **自监督训练阶段**：联合训练轨迹编码器（教师模型）和轨迹生成器，从历史轨迹中学习POI转移模式。
  2. **多任务蒸馏训练阶段**：冻结教师和生成器，训练查询编码器（学生模型）。学生通过两个损失进行学习：
     - **隐式知识**：通过MSE损失使查询表示逼近轨迹表示（PKD）。
     - **显式知识**：通过辅助任务——出行模式预测（分类为接近、远离、掉头、不规则四类）——增强查询表示。
- **关键技术细节**：
  - **轨迹编码器**：采用双向Mamba模块（替代传统LSTM/Transformer），对时空嵌入（POI ID、时间bin、相对于起终点的距离上下文）建模。
  - **轨迹生成器**：使用单向Mamba解码器，引入上下文感知机制（剩余POI数量、到终点的距离），避免重复推荐。
  - **查询编码器**：将起点、终点、时间、长度等信息嵌入后，通过多层融合得到查询表示。
  - **联合损失函数**：$J'_{Dis} = \lambda J_{Dis} + (1-\lambda) J_{Mode}$，其中 $J_{Dis}$ 包含蒸馏损失和生成损失，$J_{Mode}$ 为出行模式分类的交叉熵损失。
- **推理阶段**：仅使用查询编码器和预训练的生成器，输入查询即可输出推荐轨迹。

## 3. 实验设计：数据集、基准与对比方法
- **数据集**：四个真实世界数据集——Glasgow（2227条轨迹，27个POI）、Osaka（1115条）、Toronto（6057条）、Tokyo（6414条）。数据来自Flickr和Foursquare。
- **基准方法**：
  - 启发式方法：POIPopularity、POIRank、Markov、Rank+Markov。
  - 监督方法：DeepTrip、AR-Trip。
  - 自监督方法：SelfTrip、CTLTR、GraphTrip、Bert-Trip。
- **评估指标**：F1（POI级别的精确率/召回率调和平均）、pairs-F1（POI顺序对的正确性）、REP（推荐重复POI的比例，越低越好）。
- **交叉验证**：五折交叉验证。

## 4. 资源与算力
- 论文明确说明：使用单张GeForce RTX 4090（24GB）GPU，训练环境为56核CPU、128GB内存。但未提供具体的训练时长或总计算量。

## 5. 实验数量与充分性
- **实验组数**：
  - 整体对比：在4个数据集上，对比11种基线方法，报告F1、pairs-F1、REP三个指标。
  - 消融实验：在Toronto和Tokyo上进行了6组变体实验（去除KD、去除出行模式预测头、去除上下文感知机制、Mamba替换为Transformer、去除查询编码器、在Bert-Trip上加出行模式预测）。
  - 参数实验：在Toronto和Tokyo上调整超参数λ（从0.3到0.7），观察性能变化。
  - 案例研究：在Toronto数据集上展示了AR-Trip、Bert-Trip与EKD-Trip的直观对比。
- **充分性评价**：实验设计较为充分，覆盖了主流基线、多个数据集、消融和参数敏感性分析，结果统计上一致优于所有基线。但缺少对更大规模数据或不同分布数据的泛化性验证，且未报告统计显著性检验。

## 6. 主要结论与发现
- EKD-Trip在所有四个数据集上均优于所有基线方法，尤其在pairs-F1指标上平均提升13.70%。
- 消融实验表明：每个组件（知识蒸馏、出行模式预测、上下文感知、Mamba模块、查询编码器）都对性能有贡献，去除后性能下降。
- 多任务学习平衡参数λ=0.6时效果最佳，说明隐式与显式知识需要适当平衡。
- 案例显示：EKD-Trip能生成与真实轨迹高度一致且符合时间窗口的路线，而监督方法在起点附近偏差大，自监督方法忽略查询约束。

## 7. 优点
- **方法论创新**：首次将特权知识蒸馏引入旅行推荐，并结合显式意图增强，有效解决了监督与自监督方法的缺陷。
- **技术亮点**：使用Mamba模块（比Transformer更高效地处理序列）；生成器引入上下文感知机制（剩余POI数、距终点距离）减少了重复推荐。
- **实验充分**：多数据集、多基线、多视角的消融和参数实验，结果稳健。
- **可复现性**：开源代码（GitHub链接）。

## 8. 不足与局限
- **实验覆盖**：仅使用了4个公开数据集，且POI数量较少（最多30个），未验证在更大规模、更多样化场景（如城市级、语义丰富POI）下的表现。
- **未报告统计显著性**：未给出置信区间或p值，无法判断提升的统计显著性。
- **忽略POI语义**：论文声明未考虑POI类型等语义信息，虽可扩展但当前实验未验证该扩展的有效性。
- **算力细节模糊**：未提供训练时间等成本信息，不利于实际部署评估。
- **应用限制**：假设用户历史轨迹可用（作为教师），但在冷启动场景下可能无法获取足够历史数据；出行模式预测基于距离趋势，可能不适用于复杂交通网络。

（完）
