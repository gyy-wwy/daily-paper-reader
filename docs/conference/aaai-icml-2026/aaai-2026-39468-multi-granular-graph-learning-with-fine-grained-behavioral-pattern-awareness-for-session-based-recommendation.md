---
title: Multi-Granular Graph Learning with Fine-Grained Behavioral Pattern Awareness for Session-Based Recommendation
title_zh: 多粒度图学习与细粒度行为模式感知的会话推荐
authors: "Ming Li, Zihao Yan, Yuting Chen, Lixin Cui, Lu Bai, Feilong Cao, Ke Lv, Zhao Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39468/43429"
tags: ["query:rec-sys"]
score: 9.0
evidence: 基于会话的推荐，序列建模
tldr: 现有会话推荐方法往往忽略用户意图中瞬时和细粒度的行为模式。本文提出GraphFine，通过多粒度图学习感知细粒度行为模式，能够捕捉从即时偏好到长期目标的多样化用户意图。实验表明该方法在多个基准数据集上显著优于现有模型，为会话推荐提供了更精准的个性化建模方案。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有会话推荐方法依赖于全局聚合，忽略了会话中短暂的细粒度行为模式。
method: 提出GraphFine模型，采用多粒度图学习框架，从即时、片段共现和长期目标等多个粒度感知用户行为模式。
result: 在多个公开数据集上，GraphFine在召回率和MRR等指标上优于现有会话推荐方法。
conclusion: 多粒度图学习能有效捕捉用户意图的动态变化，提升会话推荐准确性。
---

## Abstract
Session-based recommendation aims to predict users’ next actions by modeling their ongoing interaction sequences, particularly in scenarios where long-term user profiles are unavailable. While existing methods have achieved promising results by leveraging sequential and graph-based structures, they often rely on global aggregation strategies that emphasize dominant user interests while overlooking the transient and fine-grained behavior patterns embedded in sessions. In practice, user intent evolves across sessions and is reflected through diverse behavioral patterns, ranging from immediate preferences to segmented co-occurrence interests and long-range goals. To address these limitations, we propose GraphFine, a novel multi-granular graph learning framework that achieves fine-grained behavioral pattern awareness for session-based recommendation. Our approach models user behavior at different temporal and semantic granularities through a combination of graph and hypergraph neural networks. Specifically, we employ a position-aware graph to capture short-term item transitions, and construct segmented co-occurrence hypergraphs to uncover high-order semantic relations among co-occurred items. To preserve diverse user intents, we further introduce a multi-view intent readout mechanism that extracts and adaptively integrates intent signals from short-term actions, segmented co-occurrence patterns, and entire sessions. Extensive experiments on benchmark datasets demonstrate that GraphFine consistently outperforms existing state-of-the-art methods, confirming its effectiveness in capturing fine-grained and dynamic user preferences for more accurate recommendation.

---

## 论文详细总结（自动生成）

# Multi-Granular Graph Learning with Fine-Grained Behavioral Pattern Awareness for Session-Based Recommendation —— 中文详细总结

## 1. 核心问题与整体含义（研究动机和背景）

- **核心问题**：会话推荐系统（Session-Based Recommendation, SBR）旨在仅根据用户当前匿名交互序列预测下一个交互物品。现有方法（如RNN、GNN、超图网络）多采用全局聚合策略，强调主导用户兴趣，却忽略了会话中嵌入的**瞬时、细粒度的行为模式**（例如：短期意图转移、分段共现兴趣、长期偏好）。
- **现实观察**：用户在同一会话内的行为并非单一主导意图，而是包含多个共存、动态演变的意图模式，如短暂探索（①）、分段子目标（②③）、全局偏好（④）。现有方法无法有效捕捉这种**多粒度、多层次**的用户意图。
- **研究动机**：提出一种能够显式建模不同时间与语义粒度下用户行为模式的新框架，从而提升会话推荐精度。

## 2. 方法论：核心思想、关键技术细节

### 2.1 核心思想
提出 **GraphFine**，一个**多粒度图学习框架**，通过图神经网络（GNN）和超图神经网络（HyperGNN）的组合，从短期、分段共现和全局三个视角建模用户行为，并通过多视图意图读取与自适应融合机制进行预测。

### 2.2 关键组件与技术细节

**（I）上下文聚合网络（Contextual Aggregation Network）**
- 构建全局**上下文物品图** \(G_c\)：基于滑动窗口（窗口大小 \(r\)）内物品共现频率建边。
- 使用注意力机制进行邻居聚合（公式1-2），得到上下文增强的物品嵌入 \(H \in \mathbb{R}^{N \times d}\)。

**（II）物品语义网络（Item Semantic Network）**
- 针对每个会话构建**位置感知语义图** \(G_s\)：基于物品嵌入相似度及可学习位置编码（式3-8），动态计算边权。
- 通过残差门控机制（式7-8）保留初始嵌入，更新会话内物品表示 \(\tilde{H}_S\)。

**（III）分段超图网络（Segmented Hypergraph Network）**
- 构建**分段共现超图** \(G_p\)：对会话进行滑动窗口（大小 \(\omega=2,\dots,\mu\)）生成超边，每个超边连接一组连续共现物品。
- 使用超图注意力网络：节点→超边聚合（式9-11），超边→节点聚合（式12-13），迭代更新节点表示，捕获高阶分段模式。

**（IV）多视图意图读取与融合（Multi-View Intent Readout and Integration）**
- **物品视图（短期意图）**：取最近 \(\pi\) 个物品的嵌入，通过线性变换和余弦相似度计算得分，再经 max/mean pooling 融合（式14-16）。
- **分段视图（分段共现兴趣）**：对每个超边（滑动窗口的最后一个）提取特征，类似计算得分并 pooling（式17-19）。
- **会话视图（长期偏好）**：对整个会话使用 entmax 注意力（式20-22）得到会话级表示，计算得分。
- **自适应融合**：三个视图的得分通过可学习参数 \(\eta_I, \eta_E, \eta_S\) 加权求和（式23），权重经 sigmoid 归一化。
- **损失函数**：二元交叉熵（式24）。

## 3. 实验设计

### 3.1 数据集与场景
- **三个基准数据集**：Tmall、Yoochoose、RetailRocket。
- 预处理遵循 Wu et al. (2019)、Wang et al. (2020) 等标准协议。统计信息见表1：训练/测试会话数、物品数、平均会话长度。

### 3.2 Benchmark 与对比方法
- 对比 **17 种基线方法**，涵盖三类：
  - **传统模型**：FPMC、SKNN、STAN
  - **序列模型**：NARM、CSRM、MTAW、GTPAN、DPDM
  - **图/超图模型**：SR-GNN、GCE-GNN、DHCN、COTREC、GSN-IAS、SPARE、RESTC、SDHID、RAIN
- 评估指标：**HR@20** 和 **MRR@20**。

## 4. 资源与算力

- **文中未明确说明使用的GPU型号、数量、训练时长**。仅提到训练参数设置：嵌入维度100，batch size 100，Adam优化器（学习率0.001，每3个epoch衰减0.1），L2正则化系数1e-5，训练最多20个epoch，早停。未提及具体硬件资源。

## 5. 实验数量与充分性

- **实验数量**：
  - **整体性能对比**（RQ1）：在3个数据集上与17个基线比较（表2）。
  - **细粒度意图建模有效性**（RQ2）：将多视图意图建模嵌入SR-GNN、GCE-GNN，产生6个变体（表3）。
  - **消融实验**（RQ3）：移除各关键模块（-CA、-IS、-SH、-IV、-EV、-SV），共6组（表4）。
  - **超参数分析**（RQ4）：调整 \(\pi\)（1~9）和 \(\mu\)（2~6）的影响（图3、图4）。
  - **会话长度影响**（RQ5）：分为短、中、长三组，与两个基线对比（图5）。
- **充分性与客观性**：
  - 数据集覆盖电商和公开基准，具有代表性。
  - 对比方法涵盖各个类别，且按标准协议复现（标注*表示重新实现），结果公平。
  - 消融实验全面验证了各组件贡献，超参数分析涵盖主要敏感参数。
  - 结论均基于多次实验的平均结果，未明显偏差。

## 6. 主要结论与发现

- **总体性能**：GraphFine 在所有数据集上 HR@20 和 MRR@20 均达到**最佳**，相比第二名提升显著（Tmall上HR提升14.15%）。
- **细粒度建模有效性**：将多视图意图模块集成到现有GNN模型（SR-GNN、GCE-GNN）后，各变体均获得提升，尤其多视图变体最优，证明该策略具有通用增强性。
- **关键组件贡献**：消融实验表明，**物品视图（短期意图）** 贡献最大（移除后性能下降最明显），其次是上下文聚合和超图分段建模；会话视图和分段视图虽单独贡献较小，但联合使用产生协同效应。
- **超参数影响**：\(\pi\)（最近物品数）和 \(\mu\)（滑动窗口个数）存在最佳范围，过大可能引入噪声。
- **会话长度泛化**：GraphFine 在短、中、长会话上均优于 SR-GNN 和 GCE-GNN，证明其鲁棒性。

## 7. 优点

- **方法创新**：首次将**多粒度图学习**（位置感知图 + 分段超图）与**多视图意图读取**结合，显式建模用户行为中的短期、分段、长期模式，而非简单全局聚合。
- **技术细节丰富**：使用位置编码、残差门控、entmax注意力、超图注意力等模块，设计合理且有理论支撑。
- **实验充分**：对比17个基线，消融实验覆盖每个组件，超参数分析全面，且验证了可集成到其他模型。
- **结果显著**：在三个数据集上均大幅超越SOTA，尤其是Tmall数据集HR提升14%，证明对稀疏、多意图场景的有效性。
- **可扩展性**：提出的多视图意图读取机制不依赖特定主干，可即插即用，具有实用价值。

## 8. 不足与局限

- **算力与效率报告缺失**：未提供训练耗时、GPU型号或内存消耗，难以评估实际部署成本。
- **超参数敏感性**：\(\pi\) 和 \(\mu\) 的最佳值依赖于数据集，调参可能复杂；文中未给出自动选择方法。
- **未探讨冷启动**：实验数据中物品频率较高，但未专门分析对长尾、冷启动物品的推荐效果。
- **未对比更近期方法**：虽然对比了17种，但发表时间集中在2023-2025，缺少2026年最新模型（如Mamba架构等）的比较。
- **理论分析有限**：未从理论上证明多粒度建模为何优于全局聚合，例如复杂度分析或泛化界。
- **应用限制**：仅考虑物品共现和顺序，未包含额外上下文（如时间戳、用户属性、跨会话信息），可能限制在更复杂场景的泛化。

（完）
