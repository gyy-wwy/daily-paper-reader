---
title: "PARS: Partial-Label-Learning-inspired Recommender Systems"
title_zh: "PARS: 受部分标签学习启发的推荐系统"
authors: "Shanshan Ye, Kezhi Lu, Guangquan Zhang, Jie Lu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40002/43963"
tags: ["query:rec-sys"]
score: 5.0
evidence: 利用部分标签学习的新型推荐系统设置
tldr: 本文提出一种新的推荐场景，仅利用用户浏览历史而无显式反馈，并借鉴部分标签学习提出PARS方法。该方法假设浏览序列包含待获取物品，通过弱监督学习训练推荐模型。实验表明，PARS在无显式反馈条件下仍能有效推荐，为推荐系统在数据稀疏场景下提供了新思路。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现实场景中常缺乏用户显式反馈，仅有点击浏览等隐式数据。
method: 将浏览序列类比为部分标签学习问题，训练推荐模型。
result: 在基准数据集上，PARS在无显式反馈时仍取得良好推荐效果。
conclusion: 部分标签学习方法能有效利用浏览历史进行推荐。
---

## Abstract
Recommender systems are widely required and deployed to address real-world problems. In this paper, we study a new yet challenging real-world setting for recommender systems, where only user browsing histories are available without any explicit feedback. No item acquisition information, e.g., purchasing or rating, is given. By assuming that user browsing sequences are likely to contain the items to acquire, we draw an analogy to the setting of partial label learning in weakly supervised learning. This enables us to train reliable recommender systems only using browsing histories. We term the proposed method as Partial Acquisition Recommender System (PARS). Empirical results on real-world benchmark datasets show the effectiveness of the proposed method. Surprisingly, we also show that the proposed method even surpasses some baselines using item acquisition information.

---

## 论文详细总结（自动生成）

以下是基于论文《PARS: Partial-Label-Learning-inspired Recommender Systems》的详细中文总结。

---

## 一、论文的核心问题与整体含义（研究动机和背景）

- **研究动机**：传统推荐系统严重依赖用户显式反馈（如评分、购买记录），但在许多真实场景中（例如隐私保护、跨设备购买、延迟转化），这些显式获取信息不可用，仅有用户的浏览历史。论文试图回答：**能否仅凭浏览历史训练出可靠的推荐系统？**
- **核心假设**：用户浏览序列中的大多数物品很可能就是用户最终想要获取（购买）的物品。这一假设是解决该问题的关键。
- **整体意义**：提出一种全新的推荐设置——无任何显式获取标签，仅利用浏览历史。这具有实际价值（隐私友好、缓解冷启动）和理论价值（将部分标签学习引入推荐领域）。

## 二、方法论：核心思想、关键技术细节

- **核心思想**：将用户浏览序列类比为部分标签学习中的候选标签集，序列中的每个物品都是潜在的“正标签”，但真实获取物品未知。通过自监督学习（掩码语言模型）和部分标签学习的交替优化，逐步“消歧”出每个用户的真实购买物品。
- **关键技术细节**：
    1. **用户嵌入**：使用Transformer编码器，结合物品嵌入和位置嵌入，得到序列的全局表示 \( \mathbf{z}_u \)。
    2. **掩码语言模型（MLM）**：随机遮蔽序列中的部分物品，训练模型预测被遮蔽的物品，以学习语义和时序模式，损失 \( L_{\text{MLM}} \)。
    3. **部分标签学习（PLL）**：初始化部分标签矩阵 \( \bar{P} \)（每个用户对浏览过的物品均匀分配概率）。通过模型预测 \( \hat{p}_u = \text{softmax}(\mathbf{Z}_{\text{item}}^\top \mathbf{z}_u) \) 和当前部分标签计算交叉熵损失 \( L_{\text{PLL}} \)。然后根据模型预测更新部分标签（式(10)动量更新），形成自循环。
    4. **联合训练**：总损失 \( L = L_{\text{PLL}} + \lambda L_{\text{MLM}} \)，同时优化Transformer参数和部分标签矩阵。
- **算法流程**（Algorithm 1）：初始化参数和部分标签 → 每轮对每个batch执行：① 创建遮蔽序列，计算表示，计算MLM损失；② 计算用户嵌入和预测分数，更新部分标签，计算PLL损失；③ 反向传播更新参数。

## 三、实验设计

- **数据集**：三个真实电商数据集——YooChoose（会话数多）、RetailRocket（稀疏、序列短）、Taobao（序列长、正标签比例高）。
- **对比基准**：由于PARS是首个只用浏览历史的方法，论文对比了十种**使用显式购买标签**的CVR（转化率预测）方法，包括：DeepFM、ESMM、MMoE、BERT4Rec、ESM²、ESCM²-DR/IPS、DCMT、CL4CVR、DDPO、NISE、EVI。它们均在训练时使用了真实购买标签。
- **评估指标**：AUC、HR@k、NDCG@k、Precision@k、Recall@k、F1@k（k=1,3,5）。

## 四、资源与算力

- 论文**未明确说明**所使用的GPU型号、数量、训练时长等计算资源信息。仅提供了算法复杂度的一般性描述，但没有具体硬件细节。

## 五、实验数量与充分性

- **主要实验**：在三个数据集上与11种基线方法进行全面对比，结果表格（Table 2）展示了7个指标×3个k值共21个指标。
- **额外分析**：
    - **标签可用性影响**（图3）：给基线提供不同比例（10%~100%）的购买标签，观察性能变化，并与PARS对比。
    - **训练稳定性**（图4）：在YooChoose上绘制300个epoch内6个指标的曲线，展示PARS的收敛速度和稳定性。
- **充分性评价**：实验规模较大，覆盖多数据集、多基线、多指标，且模拟了标签稀疏场景，比较客观。但缺少**消融实验**（如去掉MLM或PLL组件的影响），这算一个局限。

## 六、主要结论与发现

- PARS**完全不使用购买标签**，在所有数据集上的AUC、HR@1等核心指标大幅超过使用100%标签的基线（如YooChoose上AUC提升至0.9398 vs. 最佳基线0.6964）。
- 基线方法对标签比例极其敏感：当标签比例低于10%时性能急剧下降；而PARS稳定有效。
- 在Taobao数据集上，PARS虽略逊于部分强基线（但仍是第二/三名），这归因于Taobao原始序列中已有75.45%的序列包含真实购买标签，使得基线受益。
- PARS训练稳定性好，收敛快，无明显振荡。

## 七、优点

1. **问题新颖且实用**：首次形式化“仅浏览历史”的推荐问题，解决隐私和冷启动难题。
2. **方法论优雅**：巧妙地将推荐类比为部分标签学习，自监督+弱监督协同工作，无需任何获取标签。
3. **实验全面**：在三个不同特征的真实数据集上对比了11种CVR方法，并加入了标签比例敏感性分析。
4. **性能突出**：在大部分指标上显著超越使用完整标签的先进基线，表明方法有效提取隐式购买信息。
5. **隐私友好**：仅需浏览历史，减少对敏感购买记录的依赖。

## 八、不足与局限

1. **缺少消融分析**：未单独评估MLM和PLL组件的贡献，无法判断各自重要性。
2. **缺乏理论保证**：论文未提供关于假设条件的理论分析或泛化界，仅在实验上验证了有效性（未来方向中提到会补充）。
3. **假设依赖**：核心假设“大多数浏览序列包含真实购买物品”虽然在YooChoose和RetailRocket上通过伪标签得到了支撑（图2），但在某些极端场景（如纯粹随机浏览）可能失效。
4. **数据集偏差**：仅使用三个电商数据集，未涉及其他领域（新闻、视频、音乐），可能限制结论的通用性。
5. **计算资源未报告**：无法复现训练成本，例如GPU型号、训练时间等关键信息缺失。
6. **对比基线均为有监督方法**：尽管PARS无标签获胜，但缺乏与无监督/自监督推荐方法的比较（例如传统的基于相似度的协同过滤、基于序列的贝叶斯个性化排序等），这些方法也可能不使用购买标签。

（完）
