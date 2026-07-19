---
title: "DRSoRec: Dual-Rectification of Social Networks for Recommendation"
title_zh: DRSoRec：面向推荐的社会网络双重修正
authors: "Liangxun Yang, Tianzi Zang, Jiayi Sun, Juan Li, Yicong Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38645/42607"
tags: ["query:rec-sys"]
score: 8.0
evidence: 面向推荐的社会网络修正
tldr: 现有社交推荐中原始社交网络包含噪音。本文提出DRSoRec，通过不变社会理由发现和自适应连接修正，去除噪音边并保留有用信息，提升社交推荐鲁棒性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 社交推荐中原始社会网络存在噪音和缺失边，影响推荐质量。
method: 提出双重修正模型，包含不变社会理由发现模块和自适应连接修正模块。
result: 在多个社交推荐数据集上显著提升推荐准确率。
conclusion: 社交网络修正是提升社交推荐性能的有效方法。
---

## Abstract
Leveraging social homophily to enhance user preference modeling, social recommendation has become a cornerstone of modern recommender systems. However, the raw social network contains inherent unreliability as it teems with noise---misclicks, bot-generated and transient ties---while many meaningful links remain unobserved. In this study, we propose DRSoRec, a dual-rectification model to rectify the raw social networks by simultaneously removing noisy signals and preserving useful information. Specifically, the invariant social rationale discovery module distills each user's influential core social circle of the current recommendation, whereas the adaptive social connection refinement module employs a mixture-of-experts structure learner to prune spurious edges and uncover latent links. A contrastive optimization objective is designed to align and mutually enhance these two modules, and the refined user representations are fused with collaborative representations generated from interactions for the final recommendation. Experiments on three public datasets confirm that DRSoRec consistently gains over state-of-the-art baselines.

---

## 论文详细总结（自动生成）

# DRSoRec 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心动机**：社会推荐系统利用用户之间的社会同质性（homophily）来增强偏好建模。然而，原始社会网络存在**固有不准确性**：包含误点击、机器人生成、临时关系等噪声边，同时许多有意义的潜在链接（如兴趣相似但未直接相连的用户）未被观察到。现有方法对噪声处理不彻底，且存在三个关键局限：
  1. 忽视潜在但有意义的社交链接；
  2. 未能针对不同推荐目标识别多样化的核心社交圈（同一个用户在不同商品上受不同朋友影响）；
  3. 缺乏有效的多源知识整合机制。
- **本文目标**：提出 DRSoRec（Dual-Rectification of Social Networks for Recommendation），通过对社会网络进行**双重修正**，同时去除噪声信号并保留有用信息，提升社交推荐的鲁棒性和准确性。

## 2. 论文提出的方法论

### 核心思想
DRSoRec 包含**交互视图**（Interaction View）和**社会视图**（Social View）。社会视图由两个并行分支组成，分别负责不同的修正任务，再通过对比学习对齐，最后自适应融合三个视图的表示进行推荐。

### 关键技术细节

#### 2.1 不变社会理由发现模块（Invariant Social Rationale Discovery）
- **理由分数计算**：使用多头自注意力计算每条社会边（u, v）作为 informative rationale 的概率：
  - \( f^k(u,v) = (e_u^s W_k^Q) \cdot (e_v^s W_k^K)^\top / \sqrt{d/K} \)  
  - 对所有头的结果通过 softmax 聚合得到概率分布 \( \alpha(u,v) \)。
- **社交理由图构建**：对原始社会图，保留具有高理由分数的边，去除低分数边（视为任务无关）。引入 Gumbel 扰动和节点度数缩放，按比率 \( \rho_r \) 选择低分数边集合 \( C_S \) 并移除，得到理由图 \( \mathcal{G}_r^S \)。
- **理由感知编码**：在理由图上按权重 \( \alpha(u,v) \) 进行 L 层聚合，得到用户表示 \( \mathbf{h}_u^r \)。
- **掩码重建自监督**：构造掩码图 \( \mathcal{G}_m^S \)（移除高分数边），通过变分自编码器（重参数化）重建被掩码的边，使用 ELBO 损失 \( \mathcal{L}_m \) 增强模块提取关键信息的能力。

#### 2.2 自适应社交连接修正模块（Adaptive Social Connection Refinement）
- **多专家相似度学习**：使用 P 个专家（每个专家有一个投影向量 \( \mathbf{w}^p \)），计算用户对 (u,v) 的加权余弦相似度 \( g^p(u,v) \)，然后平均得到 \( \tilde{S}(u,v) \)。
- **图稀疏化**：设置阈值 \( \Delta \)，仅保留相似度高于阈值的边，得到稀疏邻接矩阵 \( \hat{S} \)。
- **残差连接**：引入原始社会结构 S 与学习结构 \( \hat{S} \) 的残差，稳定训练：\( \hat{S} \leftarrow \mu S + (1-\mu)\hat{S} \)。
- **邻居加权聚合**：类似公式 (5) 在修正后的图上得到用户表示 \( \mathbf{h}_u^s \)。

#### 2.3 社交知识对比增强模块（Social Knowledge Contrastive Enhancement）
- 将两个分支的表示通过不同的 MLP 映射到同一语义空间：\( \mathbf{z}_u^r, \mathbf{z}_u^s \)。
- 使用 InfoNCE 损失 \( \mathcal{L}_{cl} \) 对齐同一用户在不同分支的正样本对，推开不同用户的负样本对，增强两个模块的知识一致性与互补性。

#### 2.4 自适应多视图表示融合
- 交互视图使用 LightGCN 得到用户协同表示 \( \mathbf{h}_u^c \) 和物品表示 \( \mathbf{h}_i^c \)。
- 三个视图的用户表示（\( \mathbf{H}_U^r, \mathbf{H}_U^s, \mathbf{H}_U^c \)）通过两层注意力网络自适应融合，得到最终用户表示 \( \mathbf{H}_U \)。
- 推荐分数通过点积 \( \hat{y}_{ui} = \mathbf{h}_u^\top \mathbf{h}_i^c \) 计算。

#### 2.5 模型优化
- 总损失：\( \mathcal{L} = \mathcal{L}_{rec} + \lambda_1 \mathcal{L}_m + \lambda_2 \mathcal{L}_{cl} + \lambda_3 \|\Theta\|_2^2 \)，其中 \( \mathcal{L}_{rec} \) 是 BPR 损失。

## 3. 实验设计

### 数据集
- **三个真实世界数据集**：Delicious（1868用户，69224物品，0.0802%交互密度，0.4200%社交密度）、Ciao（7375用户，106797物品，0.0255%交互密度，0.2040%社交密度）、Yelp（17236用户，38341物品，0.0293%交互密度，0.0484%社交密度）。
- 数据划分：训练/验证/测试 = 8:1:1，仅保留至少5次交互的用户。

### 基准方法
- 对比了10种方法：SoRec、GraphRec、DiffNet、LightGCN、DiffNet++、Design、MADM、SSD-ICGA、GBSR、RGCML。涵盖矩阵分解、GNN、自监督学习、图去噪等类别。

### 评估指标
- Recall@K 和 NDCG@K，K=10 和 20。
- 对每个测试用户，从未交互物品中随机采样100个与正样本混合排序。

### 超参数设置
- 嵌入维度 64，学习率 1e-3，层数 l=2，对比损失权重 λ2=1e-2，重建损失权重 λ1=1e-1，KL权重 β 依数据集调整，掩码比例 ρm=0.1，理由选择比例 ρr=0.4，温度 τ 依数据集不同，相似度阈值 Δ=0，专家数 P=4。

## 4. 资源与算力

- **GPU**：论文明确说明所有实验运行在 **NVIDIA RTX 4090** 上。未提及 GPU 数量（通常为单卡）及训练时长。
- **其他**：未报告参数总量、训练轮次或推理时间。

## 5. 实验数量与充分性

- **全量对比实验**：在三个数据集上对比10种基线，报告 Recall@10/20、NDCG@10/20（表2），共3×3×3=27个性能数值。
- **消融实验**：在Ciao和Yelp上移除四个关键组件（w/o-cl、w/o-isrd、w/o-ascf、w/o-fus），对比 Recall@10、NDCG@10、Recall@20（表3），共2×3×5=30个数值。
- **超参数敏感性分析**：在Yelp数据集上分析 λ1、λ2、τ、l 四个超参数对 Recall@10 和 NDCG@10 的影响（图4），展示变化趋势。
- **可视化分析**：在Delicious上随机选取400用户，用t-SNE展示四个视图的分布（图3）。
- **充分性评价**：实验覆盖多个数据集、多个基线、关键消融和超参数，设计较为全面。但不足包括：①仅在三个中等规模数据集（Delicious规模较小）上验证，缺乏更大规模或工业场景的测试；②未在更多社交密度场景下对比；③未提供统计显著性检验的详细结果（仅提了paired t-test，无具体p值表格）；④没有对模型的收敛性、训练时间、参数量进行对比。整体实验充分但可进一步扩展。

## 6. 论文的主要结论与发现

1. **DRSoRec在所有数据集和所有指标上均显著优于所有基线**，证明了双重修正策略的有效性。例如在Ciao上 Recall@10 比最好基线 GBSR 提升 4.75%，NDCG@10 提升 4.98%。
2. **在低社交密度场景下优势更明显**：Yelp社交密度仅0.0484%，DRSoRec通过推断潜在链接显著提升了推荐性能。
3. **两个修正分支具有互补性**：消融实验表明移除任何一个分支都会导致较大性能下降（Ciao上Recall@10分别下降5.37%和4.95%），可视化也显示它们的表示分布互补。
4. **对比学习有助于对齐和增强两个模块**：移除对比损失（w/o-cl）导致性能小幅下降，验证了其贡献。
5. **自适应融合优于简单平均**：替换为平均池化后性能下降，表明注意力融合有效。

## 7. 优点

- **问题剖析深入**：明确指出现有方法的三点局限，并针对性地设计双重修正模块。
- **创新性**：同时进行去噪（移除噪声边）和补边（发现潜在边）的双重修正，并使用对比学习对齐两个分支，整体架构新颖。
- **技术细节扎实**：理由分数计算结合多头注意力与Gumbel噪声；多专家相似度学习+稀疏化+残差连接设计合理；变分重建自监督增强理由图学习。
- **实验较充分**：覆盖多个数据集、多种基线、关键消融和超参数分析，结果清晰有力。
- **可视化辅助分析**：t-SNE展示不同视图表示互补性，直观支撑结论。
- **代码可复现性**：论文给出了详细超参数设置（虽然未公开代码，但描述了足够细节）。

## 8. 不足与局限

- **计算资源信息不充分**：仅提到使用RTX 4090，未报告训练时间、参数量、收敛轮次，难以评估计算开销。
- **超参数数量较多**：涉及λ1、λ2、λ3、β、ρr、ρm、τ、Δ、P等多个超参数，调参可能依赖经验，泛化性存疑。
- **数据集规模有限**：Delicious、Ciao、Yelp均为中等规模（最大约1.7万用户），未在百万级用户或更大社交网络（如Epinions、Last.fm）上验证，也未考虑工业级稀疏场景。
- **未讨论模型复杂性**：没有分析推理速度、内存消耗，或与基线模型的效率对比。
- **未涉及动态社交网络**：假设社会图静态，忽略了社交关系随时间变化的特性。
- **未开源代码**：论文未提供代码链接，影响可复现性。
- **统计显著性报告不完整**：仅说“paired t-test p<0.01”，但未在表中列出具体p值或置信区间。
- **潜在偏差风险**：负采样方式为随机采样100个未交互物品，未采用主流的高效负采样策略，可能影响公平比较。

（完）
