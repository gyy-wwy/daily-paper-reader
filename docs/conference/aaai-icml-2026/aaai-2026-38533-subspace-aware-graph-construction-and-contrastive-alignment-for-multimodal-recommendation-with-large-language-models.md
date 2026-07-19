---
title: Subspace-Aware Graph Construction and Contrastive Alignment for Multimodal Recommendation with Large Language Models
title_zh: 子空间感知图构建与对比对齐：结合大语言模型的多模态推荐
authors: "Haodong Li, Lianyong Qi, Weiming Liu, Fan Wang, Chong Li, Shengye Pang, Wenwen Gong, Yanwei Xu, Xiaoxiao Chi, Yang Zhang, Xiaokang Zhou"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38533/42495"
tags: ["query:rec-sys"]
score: 8.0
evidence: 结合子空间感知图和对比对齐的多模态推荐方法
tldr: 多模态推荐中，现有方法仅构建浅层次图结构，且协同信号压制语义知识。本文提出SCALE框架，通过子空间感知图构建捕获用户-物品间复杂语义关系，并利用对比对齐平衡协同与语义。实验证明SCALE在多个多模态推荐数据集上超越现有方法，有效融合多模态信息。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有方法难以建模跨实体语义关系，且协同信号主导导致语义知识被压制。
method: 构建子空间感知的用户-物品图，并通过对比学习对齐协同和语义表征。
result: 在多个数据集上，SCALE在推荐准确率和多样性指标上均达到领先水平。
conclusion: 子空间感知图能够有效整合多模态信息和协同信号，提升推荐质量。
---

## Abstract
Multimedia content offers additional context for recommender systems to better understand user interests. Existing studies on multimodal recommendation primarily focus on constructing item-item semantic graphs. However, most of these methods capture only shallow semantic structures based on feature similarity and struggle to model more complex or cross-entity semantic relationships (e.g., user-item). Moreover, in these methods, collaborative signals often dominate and suppress semantic knowledge, which limits its role in representation learning. To address these issues, we propose SCALE, a novel framework that combines subspace-aware graph construction and contrastive alignment for multimodal recommendation with large language models. Specifically, we first use large language models and encoders to extract user and item features. Following the subspace clustering assumption, we apply the Orthogonal Matching Pursuit algorithm to mine complex semantic structures within the item-item, user-user, and user-item spaces, and integrate them into a unified semantic graph. We then perform graph convolution on both the semantic and interaction graphs, and aggregate the results for recommendation. Furthermore, contrastive losses are employed to enhance semantic fusion and alignment. Extensive experiments on five real-world datasets demonstrate that SCALE significantly outperforms state-of-the-art multimodal recommendation models, highlighting its effectiveness in modeling complex relationships and integrating semantic knowledge with collaborative signals.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：多模态推荐系统（MRS）通过整合文本、图像等异质信息，能够更准确地捕获用户偏好和物品特性。然而，现有方法存在两个关键挑战：
  - **挑战1**：如何从多模态内容中有效提取全面的语义知识？
  - **挑战2**：如何恰当融合这些语义知识与协同信号？
- **现有方法的不足**：
  - 大多数方法仅基于特征相似性构建物品-物品语义图，捕获的是浅层成对相似性，忽视了**跨实体语义关系**（如用户-物品）和**复杂多实体组合结构**（例如图1a中的用户兴趣组合）。
  - 协同信号往往主导并压制语义知识，导致表示空间中语义无关节点重叠（如图1b中不同类别物品混杂），限制了语义知识在表示学习中的作用。
- **整体含义**：本文提出SCALE框架，旨在通过**子空间感知图构建**和**对比对齐**，同时解决上述两个挑战，实现多模态语义与协同信号的深度融合。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- 利用LLM生成用户和物品的细化描述（Profile），再通过子空间聚类假设（Subspace Clustering Assumption）和**正交匹配追踪（OMP）** 算法，挖掘用户-用户、物品-物品、用户-物品空间中的复杂语义结构，构建统一的语义图。
- 对**交互图**和**语义图**分别进行图卷积，并设计两种对比对齐损失（语义对齐损失 + 关系对齐损失），在保持各自信号纯净的同时实现有效融合。

### 关键技术细节
1. **Profile Generator and Multimodal Encoder**：
   - 使用LLM（Llama-3.2-3B）生成物品描述 Profile $F_i$ 和用户兴趣 Profile $F_u$。
   - 文本编码器（Izacard et al. 2022）将Profile转为文本特征 $x^t_u, x^t_i \in \mathbb{R}^{d_t}$；视觉编码器（He & McAuley 2016a）提取物品图像特征 $x^v_i$。

2. **Semantic Graph Construction**：
   - **子空间感知图（SAG）**：基于子空间自表达性，假设同类实体位于低维子空间中，每个实体可稀疏重构。使用OMP算法（Algorithm 1）迭代选择与当前残差最相关的原子，得到稀疏系数向量，从而构建用户-用户图 $C^U$、物品-物品图 $C^I$、用户-物品图 $C^{UI}$。
   - **特征相似图（FSG）**：基于余弦相似度 + kNN稀疏化，构建模态特定的相似图 $G^U, G^v_I, G^t_I, G^{UI}$。
   - **融合**：对SAG和FSG逐元素逻辑交（AND）得到最终语义图 $\hat{G}$（公式6）。

3. **Interaction and Semantic Graph Convolution**：
   - 交互图 $\tilde{G}$ 经边剪枝后，进行LightGCN式图卷积（公式7），得到交互表示 $\tilde{H}$。
   - 语义图 $\hat{G}$ 同理得到语义表示 $\hat{H}$。
   - 最终表示 $H = \tilde{H} + \hat{H}$。

4. **Interaction and Semantic Alignment Module**：
   - **语义对齐损失** $L_{\text{align}}^s$（公式8-9）：对比交互表示和语义表示的用户/物品维度，促使两者一致。
   - **关系对齐损失** $L_{\text{align}}^r$（公式10-11）：在子空间感知图定义的语义关系对上，拉近正样本、推开负样本，显式增强语义信息在表示空间中的影响。

5. **Optimization**：
   - 总损失：$L = L_{\text{bpr}} + \lambda_s L_{\text{align}}^s + \lambda_r L_{\text{align}}^r$，其中 $L_{\text{bpr}}$ 为BPR损失。

## 3. 实验设计

### 数据集与场景
- **多模态推荐场景**：Amazon子集（Baby、Sports、Clothing），提供文本描述和图像。
- **LLM增强推荐场景**：Book（仅文本）、Yelp（仅文本）。
- **数据预处理**：文本特征使用预训练编码器（Izacard et al. 2022）提取768维，视觉特征使用4096维预提取特征（He & McAuley 2016a）。

### Benchmark与对比方法
- **传统推荐**：BPR、LightGCN、LayerGCN。
- **多模态推荐**：VBPR、LATTICE、SLMRec、BM3、FREEDOM、MGCN、LGMRec、DiffMM、MENTOR。
- **LLM增强推荐**：KAR、RLMRec（基于LightGCN和SGL两种骨干网络）。
- **评估指标**：Recall@10/20、NDCG@10/20。随机划分8:1:1。

## 4. 资源与算力

- **文中明确提及**：使用Llama-3.2-3B生成Profile，文本特征维度768，视觉特征维度4096，批量大小2048，学习率1e-3，Adam优化器，ID嵌入维度64。
- **未明确说明**：训练使用的GPU型号、数量、训练时长、显存消耗等具体算力信息。仅提到基于MMRec框架实现，采用早停策略。

## 5. 实验数量与充分性

- **实验组数**：
  - 主实验：5个数据集 × 2个指标（R@10/20, N@10/20）× 多层对比，表1、表2呈现结果。
  - 消融实验（图4a）：在Baby数据集上逐一移除LLM、SAG、FSG、语义对齐损失、关系对齐损失、以及各类内部关系（CU&GU、CUI&GUI、CI&GI）。
  - 对比损失可视化（图4b/c）：t-SNE展示关系对齐损失的效果。
  - 超参数敏感性（图5）：对齐损失权重（λs, λr）、负样本数量|V|、稀疏度ko、图卷积层数（LI, LS）。
- **充分性评价**：
  - 对比基线全面，覆盖传统CF、多模态SOTA、LLM增强方法。
  - 消融实验覆盖各核心组件（LLM、SAG、FSG、两种损失、不同关系类型），验证了每部分的贡献。
  - 超参数分析覆盖主要可调参数，并给出合理解释。
  - 实验结果一致表明SCALE显著优于所有基线，最大提升达27.96%（Recall@20 on Baby）。
- **客观性**：所有实验在相同数据集和评估协议下进行，实现细节和超参数通过网格搜索确定，没有明显偏向。

## 6. 主要结论与发现

1. **SCALE在多模态推荐任务上显著优于所有现有方法**，在Baby、Sports、Clothing数据集上Recall@20提升7.68%~27.96%，NDCG@20提升7.54%~47.05%。
2. **LLM生成的高质量Profile是性能提升的关键因素**（去掉LLM后性能下降最大）。
3. **子空间感知图（SAG）和特征相似图（FSG）互补**，二者联合构建的语义图比单独使用效果更好。
4. **语义对齐损失和关系对齐损失均有效**：前者促进交互与语义表示的一致，后者显式拉近语义相关实体，增强区分度（图4b-c可视化证实）。
5. **跨实体关系（用户-物品）的建模不可或缺**，移除后性能明显下降。
6. **超参数影响**：稀疏度ko=10最优；对比对齐损失权重需平衡（λs=1e-2, λr=5e-2较好）；语义图卷积层数不宜过多（LS=1或2最佳），交互图卷积层数LI=2或5较好。

## 7. 优点

- **方法论创新**：
  - 首次将子空间聚类思想（OMP算法）引入多模态推荐，构建包含用户-用户、物品-物品、用户-物品的复杂语义图，突破了传统成对相似图的局限。
  - 独立建模交互图和语义图，再通过对比对齐融合，既避免了语义噪声污染协同信号，又保持了语义知识的清晰作用。
  - 关系对齐损失直接利用子空间图中的语义关系，显式优化表示空间的区分度。
- **实验设计规范**：
  - 覆盖多种数据集类型（带图像/纯文本），对比基线丰富，消融实验系统。
  - 提供了t-SNE可视化直观验证关系对齐损失的效果。
  - 超参数敏感性分析全面。
- **代码实现**：基于MMRec开源框架，可复现性强。

## 8. 不足与局限

- **依赖LLM生成质量**：LLM生成的Profile质量直接影响后续特征提取和子空间图构建。实验中Clothing数据集文本描述缺失率93.83%、冗余率94.10%，导致SCALE在该数据集上提升幅度较小（Recall@20提升仅12.87%），说明对文本质量敏感。
- **计算开销**：OMP算法需要为每个用户和物品构建字典并迭代求解，加上LLM推理和多次图卷积，整体训练代价可能较高。文中未提供训练时间或内存消耗数据。
- **超参数依赖**：ko、ks、kf、λs、λr、τs、τr等超参数较多，需要通过网格搜索确定，实际部署调参成本较高。
- **实验覆盖有限**：
  - 仅验证了Recall/NDCG两个指标，缺少Precision、Hit Rate、多样性、覆盖率等维度。
  - 未讨论冷启动场景（例如新物品/新用户推荐），这是多模态推荐的重要应用场景。
  - 未在更大规模数据集（如Amazon全量、Movielens-1M/10M）上验证可扩展性。
- **偏差风险**：仅在Amazon和Yelp两个领域进行测试，可能存在领域偏差，在其他类型平台（如视频、新闻）上的泛化性未知。
- **应用限制**：需要LLM在线推理生成Profile，可能引入延迟；若使用API则还会增加成本。文中未讨论如何权衡效果与效率。

（完）
