---
title: LLM-Aligned Geographic Item Tokenization for Local-Life Recommendation
title_zh: LLM对齐的地理物品标记化框架用于本地生活推荐
authors: "Hao Jiang, Guoquan Wang, Donglin Zhou, Sheng Yu, Yang Zeng, Wencong Zeng, Kun Gai, Guorui Zhou"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38514/42476"
tags: ["query:rec-sys"]
score: 7.0
evidence: 结合地理标记的上下文感知推荐
tldr: 现有文本推荐方法在本地生活服务中无法捕捉细粒度空间特征。本文提出LGSID框架，利用强化学习对齐地理LLM，并通过层次化地理标记编码空间信息。实验表明LGSID在本地生活推荐任务中显著优于传统方法，提升了位置感知能力。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 文本推荐方法在本地服务中难以捕捉物品间的精细空间特征。
method: 提出LGSID，包含基于强化学习的地理LLM对齐和层次化地理标记。
result: 在本地生活推荐数据集上，LGSID显著提升推荐准确率。
conclusion: 结合地理信息的标记化有效增强了推荐系统的上下文感知能力。
---

## Abstract
Recent advances in Large Language Models (LLMs) have enhanced text-based recommendation by enriching traditional ID-based methods with semantic generalization capabilities. Text-based methods typically encode item textual information via prompt design and generate discrete semantic IDs through item tokenization. However, in domain-specific tasks such as local-life services, simply injecting location information into prompts fails to capture fine-grained spatial characteristics and real-world distance awareness among items. To address this, we propose LGSID, an LLM-Aligned Geographic Item Tokenization Framework for Local-life Recommendation. This framework consists of two key components: (1) RL-based Geographic LLM Alignment, and (2) Hierarchical Geographic Item Tokenization. In the RL-based alignment module, we initially train a list-wise reward model to capture real-world spatial relationships among items. We then introduce a novel G-DPO algorithm that uses pre-trained reward model to inject generalized spatial knowledge and collaborative signals into LLMs while preserving their semantic understanding. Furthermore, we propose a hierarchical geographic item tokenization strategy, where primary tokens are derived from discrete spatial and content attributes, and residual tokens are refined using the aligned LLM’s geographic representation vectors. Extensive experiments on real-world Kuaishou industry datasets show that LGSID consistently outperforms state-of-the-art discriminative and generative recommendation models. Ablation studies, visualizations, and case studies further validate its effectiveness.

---

## 论文详细总结（自动生成）

# LLM-Aligned Geographic Item Tokenization for Local-Life Recommendation 论文分析总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：在本地生活推荐（如餐厅、外卖等）中，用户只与有限地理半径内的物品交互，而现有基于文本的推荐方法（如用LLM生成语义ID）虽然提升了语义泛化能力，但简单将位置信息拼入prompt无法捕捉物品间**细粒度的空间特征**和**真实世界距离意识**。
- **核心问题**：预训练LLM倾向于优先考虑内容相关性而非地理邻近性，导致推荐出语义匹配但距离过远（如上海用户被推荐北京餐馆）的物品，影响系统效率与用户体验。
- **整体含义**：本文提出LGSID框架，通过强化学习对齐LLM的地理知识，并设计层次化地理物品编码，从而在语义理解基础上注入空间感知，使推荐结果更符合本地生活场景的真实约束。

## 2. 方法论

### 核心思想
LGSID采用**两阶段框架**：
1. **基于强化学习的地理LLM对齐**（RL-based Geographic LLM Alignment）——通过奖励模型和G-DPO算法将空间知识注入LLM的表示中。
2. **层次化地理物品标记化**（Hierarchical Geographic Item Tokenization, HGIT）——结合离散空间/内容属性与LLM地理表示，生成层次式语义ID。

### 关键技术细节

#### (1) RL-based Geographic LLM Alignment
- **奖励模型训练**：
  - 使用**距离感知的列表式采样**：对每个物品i，用其真实地理位置与K个负样本（按地理距离密度采样）构造prompt序列（内容固定、位置替换）。
  - 设计**列表式奖励模型**：LLM编码prompt得到表示，通过MLP预测奖励分数 \( r_{i,j} \)，软标签 \( p_{i,j} \) 基于距离排序（近者分高）。
  - 训练损失：加权二分类交叉熵 \( L_{RM} = -\frac{1}{N} \sum_i \sum_j p_{i,j} \log \sigma(r_{i,j}) \)。

- **G-DPO算法**：
  - 结合**两种偏好数据**：领域协同对（基于用户共现）和地理约束对（随机采样远处物品）。
  - 对齐损失 \( L_{align} \)（基于DPO的修订版，使用奖励模型评分）：
    \[ L_{align} = -\mathbb{E}_{(i^+, i^-)} \log \sigma\big(\beta (R(E_{\pi_\theta}(i^+)) - R(E_{\pi_\theta}(i^-)) - R(E_{\pi_{\text{ref}}}(i^+)) + R(E_{\pi_{\text{ref}}}(i^-)))\big) \]
  - 引入**相似性正则化** \( L_{sim} \)：在batch内拉近策略模型与参考模型的语义表示，避免语义漂移。
  - 总体损失：\( L_{G-DPO} = L_{align} + \lambda L_{sim} \)。

#### (2) Hierarchical Geographic Item Tokenization (HGIT)
- **第一层（主token）**：基于融合特征向量（地理位置编码、行政区划码、品类码、品牌码）进行MiniBatch K-Means聚类，聚类中心用LLM表示的均值初始化，得到粗粒度地理token。
- **后续残差层（l ≥ 2）**：使用可学习聚类中心，基于欧氏距离分配，并更新残差：
  - \( z^{(l)} = \arg\min_k \|R^{(l-1)} - \mu^{(l)}\|^2 \)
  - \( Q^{(l)} = \mu^{(l)}[z^{(l)}] \)，\( R^{(l)} = R^{(l-1)} - Q^{(l)} \)
- **训练目标**：重建损失 \( L_{recon} = \|X - \sum_l Q^{(l)}\|^2 \) + 熵正则化（防止簇塌陷）。

## 3. 实验设计

- **数据集**：快手本地生活真实工业数据集（未公开详细规模，但提及百万级候选池）。
- **Benchmark**：对比两类推荐范式：
  - **判别式模型**：DIN, DIEN, SIM, TWIN, ETA
  - **生成式模型**：TIGER, OneRec
- **对比方法**：
  - 判别式：原始ID + 各种量化方法（Res-KMeans, RQ-VAE, Lin et al., RQ-VAE-ngram, 以及LGSID）
  - 生成式：RO-VAE, Lin et al., RQ-VAE-ngram, LGSID
- **评估指标**：
  - 判别式：AUC
  - 生成式：Hit@5/10, NDCG@5/10

## 4. 资源与算力

- 论文中**未明确说明**使用的GPU型号、数量、训练时长等具体资源信息。仅提及在快手工业数据集上开展实验，但未公开算力细节。

## 5. 实验数量与充分性

- **实验数量**：主实验中包含5个判别式基线×5种方法 = 25组（表1），生成式2个模型×4种方法 = 8组（表2）；消融实验（表3）包含6种G-DPO变体，覆盖奖励模型设计、采样策略、正则化等维度；此外还有TSNE可视化（图3）、雷达图覆盖度（图4）、案例概率分布（图5）。
- **充分性评价**：实验设计**较为充分**，覆盖多种主流推荐架构、多种量化方案，且进行了细致的消融——验证了奖励模型类型（点式 vs 列表式）、密度采样、领域混合对、相似性正则化等各组件的有效性。可视化分析也直观展示了地理感知能力的提升。整体客观公正，对比基准均为SOTA方法。

## 6. 主要结论与发现

1. **LGSID在所有设置中一致优于现有方法**：在判别式模型上AUC提升0.0229~0.0417（绝对）；在生成式模型上Hit@5提升18.63%~27.01%，NDCG提升18.09%~27.05%。
2. **RL对齐有效提升地理感知**：消融显示，列表式奖励模型+密度采样+领域混合对+相似性正则化的完整G-DPO方案使Town覆盖率T@5提升83.64%（从0.1601到0.2940），同时保持了语义相似性（仅下降2.47%）。
3. **层次化标记与对齐互补**：第一层基于地理特征的聚类为粗粒度token提供稳定基础，后续残差层利用对齐后的LLM表示，使HIT在高分位数覆盖更优。
4. **案例展示**：对齐后同一类目（如BBQ）被凝聚到同一第一层token，而未对齐时分散在不同token，突出了LLM对齐对下游量化的关键作用。

## 7. 优点

- **创新性**：首次将强化学习（DPO变体）与地理知识注入结合，提出G-DPO算法，将空间距离作为偏好信号精细对齐LLM表示。
- **实用性**：层次化标记方案兼顾计算效率与表示精度，第一层用预计算地理聚类避免训练负担，后续层可学习适应下游任务。
- **实验严谨**：消融实验系统涵盖了奖励模型设计、采样策略、混合数据、正则化等技术细节，清晰证明了每个组件的贡献。
- **可解释性**：可视化分析（TSNE、雷达图、频率分布）直观展示了模型的地理感知能力，验证了框架的有效性。

## 8. 不足与局限

- **算力未公开**：未报告训练LLM所需GPU型号、数量、时长，难以评估实际部署成本。
- **实验泛化性**：仅使用快手单一工业数据集，未在公开基准（如Yelp、Foursquare）上验证，可能存在平台偏差。
- **未讨论跨域迁移**：方法针对本地生活场景设计，但未探讨是否可推广至其他空间敏感推荐（如POI、外卖）。
- **计算开销**：RL对齐需要训练奖励模型和G-DPO更新LLM，层次量化也需多次聚类，整体训练流程较复杂，但未分析实际推断效率。
- **假设条件**：依赖经纬度精确信息，在隐私或数据不完整的场景可能受限。

（完）
