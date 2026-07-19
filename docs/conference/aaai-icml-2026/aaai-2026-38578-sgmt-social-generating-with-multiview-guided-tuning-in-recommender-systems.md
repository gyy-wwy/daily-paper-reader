---
title: "SGMT: Social Generating with Multiview-Guided Tuning In Recommender Systems"
title_zh: SGMT：推荐系统中基于多视角引导调优的社会生成方法
authors: "Jianghong Ma, Changran He, Dezhao Yang, Tianjun Wei, Haijun Zhang, Xiaofeng Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38578/42540"
tags: ["query:rec-sys"]
score: 8.0
evidence: 结合GNN和多视角对比学习的社交推荐系统
tldr: 协同过滤中用户-项目交互稀疏是主要难题。本文提出SGMT框架，通过生成社交链接和结合增强与非增强视角的对比学习，提升GNN推荐器的高阶表征能力。实验表明，SGMT在多个社交推荐数据集上显著优于现有方法。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 用户-项目交互稀疏制约协同过滤，现有社交推荐依赖显式关系。
method: 利用社交生成和多视角对比学习，增强GNN推荐器表征。
result: 在多个社交推荐数据集上显著优于现有方法。
conclusion: 多视角对比学习与社交生成有效缓解稀疏问题。
---

## Abstract
The sparsity of user–item interactions remains a fundamental obstacle in collaborative filtering, limiting the ability of Graph Neural Network (GNN)-based recommender systems to capture high-order user relationships without incurring over-smoothing and computational overhead. Existing social recommendation approaches mitigate this by incorporating social networks, yet most rely on explicit ties and fail to construct informative links in their absence. Meanwhile, contrastive learning (CL) has shown promise in improving representation quality, but current view generation strategies, augmentation-based for robustness and nonaugmentation-based for semantic fidelity, are seldom combined, leaving their complementary potential underexplored. We propose Social Generating with Multiview-guided Tuning (SGMT), a unified framework that addresses both challenges. First, an interest-aware social generation mechanism constructs synthetic user–user links from shared interaction patterns, theoretically shown to compress collaborative paths and uncover latent high-order relations. Second, we present two complementary CL modules, Noise-augmented View and Semantic-explored View, which we theoretically prove to preferentially enhance uniformity and alignment, respectively, two fundamental objectives in CL. Experiments on three real-world datasets show that SGMT outperforms state-of-the-art baselines, validating both the theoretical analysis and the practical efficacy of our model.

---

## 论文详细总结（自动生成）

# SGMT: Social Generating with Multiview-Guided Tuning In Recommender Systems — 详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：协同过滤中用户-项目交互稀疏严重制约了基于图神经网络（GNN）的推荐系统性能。深层GNN容易过平滑且计算成本高，而浅层GNN又无法捕获高阶用户关系。现有社交推荐方法虽然引入社交网络缓解稀疏性，但大多依赖显式社交链接，在缺乏显式关系时无法构建有效链接。同时，对比学习（CL）的视图生成策略分为增强型（提高鲁棒性）和非增强型（保持语义保真度），但两者很少被结合，其互补潜力未被充分挖掘。

- **研究动机**：通过生成兴趣感知的合成社交链接来压缩协同路径，使浅层GNN也能捕获高阶语义关系；并通过设计两种互补的对比学习视图（噪声增强视图和语义探索视图）分别提升表征的均匀性和对齐性，从而提升推荐质量。

## 2. 论文提出的方法论：核心思想、关键技术细节

### 核心思想
提出统一框架 **SGMT**，包含三个模块：
- **主模块**：社交协同过滤（Social Collaborative Filtering）——包括兴趣感知社交生成、社交集成传播、跨域门控聚合。
- **辅助模块1**：噪声增强视图（Noise-augmented View）——通过注入可控噪声提升表征均匀性。
- **辅助模块2**：语义探索视图（Semantic-explored View）——通过跨层对比保持语义一致性，提升对齐性。

### 关键技术细节

#### 2.1 兴趣感知社交生成（Interest-aware Generating）
- 基于用户-项目交互矩阵 \( R \) 计算共交互矩阵 \( S = R R^\top \)。
- 设置阈值 \( \theta_L \) 和 \( \theta_U \) 过滤稀疏或冗余边，保留中度共享交互的用户对。
- 计算结构感知嵌入 \( E_{ST}^U = S \cdot E_{CO}^U \)（\( E_{CO}^U \) 为协同域用户嵌入）。
- 计算兴趣置信度（IC）得分：\( IC_{u,v} = \big( \cos(e_{ST}^u, e_{ST}^v) + 1 \big)/2 \)。
- 仅保留 \( IC \ge \gamma \) 的边作为生成的社交图 \( \tilde{G}_s \)。
- 使用图注意力网络（GAT）编码得到社交域用户嵌入 \( E_{SO}^U \)。

#### 2.2 社交集成传播（Social Integrated Propagation）
- 融合嵌入：\( E_{FU}^U = E_{SO}^U \oplus E_{CO}^U \)（元素级相加）。
- 将融合的用户表示输入 LightGCN 在交互图 \( G_r \) 上传播，得到全局用户和项目嵌入 \( E_{GF}^U, E_{GI} \)。

#### 2.3 跨域门控聚合（Cross-Domain Gated Aggregation）
- 通过门控机制动态融合全局协同嵌入和社交嵌入：
  \[
  g_u = \sigma( W_1^\top e_{Gf}^u + W_2^\top e_{SO}^u ), \quad e_G^u = g_u \odot e_{Gf}^u + (1-g_u) \odot e_{SO}^u
  \]
- 最终用户嵌入 \( E_G^U \) 结合了协同和社交信息。

#### 2.4 噪声增强视图（Noise-augmented View）
- 向融合嵌入注入方向保持的噪声：\( \Delta = \bar{\Delta} \odot \text{sign}(E) \odot \epsilon \)。
- 生成两个扰动视图 \( E_G'^U, E_G''^U \) 和 \( E_G'^I, E_G''^I \)，使用 InfoNCE 损失 \( L_N \)。

#### 2.5 语义探索视图（Semantic-explored View）
- 利用 GNN 层语义：偶数层捕捉用户-用户相似性，奇数层捕捉用户-项目交互。
- 对观察到的交互 \( (u,i) \)，将第 \( k \) 层用户嵌入与第 \( k+1 \) 层项目嵌入配对为正例，随机未交互对为负例，使用 InfoNCE 损失 \( L_S \)。

#### 2.6 模型训练
- 主任务损失：Bayesian Personalized Ranking（BPR）损失。
- 总损失：\( L = L_{BPR} + \lambda_1 L_N + \lambda_2 L_S + \lambda_3 \|\Theta\|^2 \)。

### 理论分析
- **语义路径覆盖**：证明生成的社交图能将协同图中的高阶路径压缩为短路径，极大扩展语义传播范围（在Flickr上 \( \eta_{3\to3} = 1.38\times10^4 \)）。
- **双视图对比学习**：推导 InfoNCE 损失相对于正/负相似度的梯度，证明噪声增强视图优先提升均匀性（对硬负样本施加强排斥），语义探索视图优先提升对齐性（拉近低相似度正对），两者互补。

## 3. 实验设计

- **数据集**：三个真实社交推荐数据集：
  - Flickr（5,642用户, 21,176物品, 199,500交互）
  - Ciao（5,836用户, 10,708物品, 142,805交互）
  - Yelp（4,846用户, 5,695物品, 142,950交互）
  - 均采用 5-core 设置（保留至少有5个交互的用户和物品）。

- **基准方法**：涵盖四大类：
  - 传统方法：BPR
  - 协同过滤：LightGCN, DirectAU
  - 对比学习方法：SGL, SimGCL, CGCL
  - 社交推荐方法：SHaRe, RGCML

- **评估指标**：Top-5 推荐性能：Hit Ratio (H@5)、Precision (P@5)、Recall (R@5)、NDCG (N@5)。

## 4. 资源与算力

论文中未明确说明所使用的 GPU 型号、数量、训练时长等算力信息。仅提及在标准实验环境下进行训练（未量化）。

## 5. 实验数量与充分性

- **主要实验**：在三个数据集上进行 Top-5 推荐性能对比（表3），涵盖所有八类基线方法，对比充分。
- **消融实验**：进行了4组消融（w/o both、w/o SV、w/o NV、完整模型），在三个数据集上均报告了完整指标（表4），验证了每个辅助模块的贡献。
- **理论验证**：图4展示了训练过程中均匀性和对齐性的变化轨迹，直观验证了双视图的互补作用。
- **综合评价**：实验覆盖了不同稀疏程度的数据集（Flickr最稀疏，Ciao和Yelp中等），对比方法全面，消融充分，结果客观。但未进行超参数敏感性分析或更大规模数据集实验。

## 6. 论文的主要结论与发现

- SGMT 在所有三个数据集和所有指标上均取得最优结果，相对最强基线提升 4.4%–16.7%。
- 在极端稀疏的 Flickr 数据集上提升最大（H@5 +8.9%，R@5 +16.7%），验证了兴趣感知社交生成能有效压缩长协同路径。
- 消融实验表明，噪声增强视图和语义探索视图均贡献了可测量的性能提升，且同时使用时效果最佳。
- 理论分析与实证结果一致，证明双视图对比学习分别促进均匀性和对齐性，协同优化表征。

## 7. 优点

- **理论深度**：首次为兴趣感知社交生成提供语义路径覆盖的数学证明，并严格推导了双视图对比学习对均匀性与对齐性的不同影响。
- **方法创新**：结合增强与非增强对比学习视图，并给出互补性理论保证；兴趣感知社交生成通过共交互模式和结构嵌入自动构建高质量社交链接。
- **实验优秀**：在多个真实数据集上全面超越 SOTA，消融实验和理论可视化充分支撑了设计。
- **模型紧凑**：社交生成和双视图正则化引入额外参数少（仅门控矩阵和噪声幅度），在性能提升的同时保持较低复杂度。

## 8. 不足与局限

- **实验覆盖**：仅使用了三个中等规模数据集（最大约20万交互），未在更大规模数据集（如 Amazon、Yelp 完整版）上验证，可能存在泛化风险。
- **超参数敏感性**：未讨论阈值 \( \theta_L, \theta_U, \gamma \) 以及对比学习温度 \( \tau \)、噪声幅度 \( \epsilon \) 等超参数的影响，实际应用可能需要调参。
- **社交生成依赖**：兴趣感知生成依赖用户共交互模式，在极度稀疏场景下（两个用户无共交互）无法生成链接，仍存在冷启动问题。
- **计算资源未说明**：未提供训练时间、GPU 需求等，不利于复现和实际部署评估。
- **对比基线限制**：未与最新的大模型推荐方法或纯图增强方法（如 SGL 的变体）进行比较。

（完）
