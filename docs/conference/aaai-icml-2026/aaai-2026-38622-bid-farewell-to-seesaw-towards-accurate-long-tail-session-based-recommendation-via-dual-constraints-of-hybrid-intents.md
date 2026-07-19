---
title: "Bid Farewell to Seesaw: Towards Accurate Long-Tail Session-Based Recommendation via Dual Constraints of Hybrid Intents"
title_zh: 告别跷跷板：通过混合意图的双重约束实现准确的长尾会话推荐
authors: "Xiao Wang, Ke Qin, Dongyang Zhang, Xiurui Xie, Shuang Liang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38622/42584"
tags: ["query:rec-sys"]
score: 9.0
evidence: 会话推荐，长尾分布
tldr: 该论文针对会话推荐中的长尾问题，提出HID方法。现有方法在提升长尾推荐时往往牺牲准确率，形成跷跷板效应。HID通过混合意图的双重约束，在提升长尾物品推荐的同时保持了准确率，实验证明有效缓解了该矛盾。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 会话推荐中长尾物品推荐与准确率之间存在跷跷板效应。
method: 提出混合意图的双重约束机制，识别并约束会话无关噪声，同时保留有用长尾信号。
result: 在多个会话推荐数据集上，HID在长尾指标和准确率上均取得提升。
conclusion: HID为会话推荐中的长尾-准确率权衡提供了有效解决方案。
---

## Abstract
Session-based recommendation (SBR) aims to predict anonymous users' next interaction based on their interaction sessions. In practical recommendation scenario, low-exposure items constitute the majority of interactions, creating a long-tail distribution that severely compromises recommendation diversity. Existing approaches attempt to address this issue by promoting tail items but incur accuracy degradation, exhibiting a "see-saw" effect between long-tail and accuracy performance. We attribute such conflict to session-irrelevant noise within the tail item set, which existing long-tail approaches fail to identify and constrain effectively. To resolve our fundamental conflict, we propose HID (Hybrid Intent-based Dual Constraint Framework), a plug-and-play framework that transforms the conventional "see-saw" into a "win-win" relationship through introducing the hybrid intent-based dual constraints. Two key innovations are incorporated in this framework: (i) Hybrid Intent Learning, where we reformulate the intent extraction strategies by employing attribute-aware spectral clustering to reconstruct the item-to-intent mapping. Furthermore, discrimination of session-irrelevant noise is achieved through the assignment of both target and noise intents to each sessions. (ii) Intent Constraint Loss, where we propose two novel constraint paradigms regarding the diversity and accuracy to regulate the representation learning process, and unify the two optimization objectives into a unique loss. Extensive experiments across multiple SBR models and datasets demonstrate that HID can enhance both long-tail performance and recommendation accuracy, establishing new state-of-the-art performance in long-tail recommender systems.

---

## 论文详细总结（自动生成）

# 论文详细总结：Bid Farewell to Seesaw

## 1. 核心问题与整体含义

- **研究动机**：会话推荐（Session-based Recommendation, SBR）中，物品曝光量呈现长尾分布——少数头部物品占据大部分交互，大量尾部物品被忽视，导致推荐多样性下降。现有长尾处理方法（如增强或重排序）在提升尾部物品推荐时往往牺牲准确率，形成“跷跷板效应”。
- **问题归因**：作者将这一冲突归因于尾部物品中包含与当前会话无关的噪声（session-irrelevant noise），现有方法无法有效识别和约束这些噪声，从而干扰了准确推荐。
- **整体目标**：提出一个即插即用的框架HID，通过混合意图的双重约束，同时提升长尾性能和推荐准确率，将“跷跷板”转化为“双赢”。

## 2. 方法论

### 核心思想
- **混合意图学习**：利用物品的属性和属性共现模式，通过谱聚类构建全局的“混合意图”（hybrid intent），并为每个会话分配目标意图（target intent，与真实下一物品相关的意图）和噪声意图（noise intent，与当前会话无关的其他会话目标意图）。
- **意图约束损失**：设计两个约束——长尾约束（减小会话与目标意图内物品的相似度方差，等价于最大化会话与目标意图向量的相似度）和准确率约束（增大会话与噪声意图的相似度并限制其方差），最终统一为基于余弦相似度的对比损失，并加入柔性系数和方差惩罚项。

### 关键技术细节
1. **初步意图**：以物品属性（如商品类别）作为初步意图。
2. **初步意图图**：基于所有会话中属性共现频率构建图。
3. **谱聚类**：对图归一化拉普拉斯矩阵进行特征分解，用k-means将属性划分为n个混合意图簇。
4. **目标意图与噪声意图定义**：
   - 目标意图：包含会话真实下一物品的混合意图。
   - 噪声意图：同一批次中其他会话的目标意图，但不属于当前会话的目标意图集。
5. **约束损失**：
   - 长尾约束近似为最小化会话与目标意图的距离（定理1）。
   - 准确率约束为最大化会话与噪声意图的距离，并约束方差小于阈值。
   - 最终损失形式：\( \mathcal{L}_c = -\sum_{S_u\in B} \log \frac{\exp(\cos(S_u, c_u)/\sigma)}{\sum_{c_v \in \hat{C}_u} \exp(\cos(S_u, c_v)/\sigma)} + \lambda p_u \)，其中 \( p_u = \max(0, \text{Var}(\cos(S_u, c_v)) - \eta) \)。
6. **多任务学习**：总损失 = 交叉熵损失 + \( \epsilon \times \) 意图约束损失。

## 3. 实验设计

- **数据集**：三个真实世界数据集——Tmall（电商）、Diginetica（CIKM Cup 2016）、RetailRocket（浏览日志）。
- **基准模型**：四种SBR模型作为基座——STAMP、GRU4Rec（序列模型）；SRGNN、GCEGNN（图模型）。
- **对比方法**：四个即插即用的长尾方法——TailNet（重排序）、CSBR（校准）、LOAM（增强）、LAP-SR（重排序）。
- **评估指标**：
  - 准确率：HR@20, MRR@20
  - 长尾指标：tHR@20, tMRR@20, tCov@20, Tail@20（尾部物品覆盖率等）

## 4. 资源与算力

- 论文正文及附录中**未明确说明**使用的GPU型号、数量、训练时长等算力信息。仅提及“附录C”有时间复杂度分析，但未提供具体硬件配置。因此无法总结算力细节。

## 5. 实验数量与充分性

- **实验组数**：共进行了多组实验——
  - **主实验**：在3个数据集上，对4种基座模型（STAMP、GRU4Rec、SRGNN、GCEGNN）分别集成5种长尾方法（TailNet、CSBR、LOAM、LAP-SR、HID），共约 4×6×3 = 72 个实验配置（报告中仅展示部分代表性结果）。
  - **消融实验**：在两种基座模型（STAMP、SRGNN）上，对HID的两个变体（去掉混合意图HI、去掉柔性系数FC）在三个数据集上进行了比较。
  - **超参数分析**：调节ICLoss权重 \( \epsilon \) 和混合意图簇数 \( n \)，在两个数据集上展示了趋势。
  - **显著性检验**：主实验中给出了p值（t检验）。
- **充分性与公平性**：实验覆盖了不同基座（序列+图）、不同长尾方法类型（增强+重排序）、多个指标，且进行了统计显著性检验。消融实验验证了各组件贡献。超参数探索合理。整体实验设计较为充分和客观。

## 6. 主要结论与发现

- **现有方法**：几乎所有现有长尾方法在提升长尾性能时都导致准确率下降，存在跷跷板效应，原因是忽略了会话无关的尾部噪声。
- **HID方法**：在几乎所有配置下，HID同时提升了准确率（HR、MRR）和长尾性能（tHR、tMRR、tCov），实现了“双赢”。例如，在Tmall上STAMP+HID使HR@20从26.10提升至28.26，tCov@20从69.46提升至73.65。
- **消融实验**：去掉混合意图（HI）主要影响长尾性能，去掉柔性系数（FC）主要影响准确率，支持了设计动机。
- **超参数影响**：ICLoss权重存在最优区间；混合意图簇数n存在峰值，过多或过少都会影响性能。

## 7. 优点

- **创新性**：首次提出通过意图建模区分会话相关尾部物品与无关噪声，并设计双重约束同时优化长尾和准确率，突破传统跷跷板困境。
- **理论严谨性**：对长尾约束和整体损失函数给出了理论等价证明（定理1、定理2），保证设计的合理性。
- **即插即用性**：HID是模型无关的插件，可轻松集成现有SBR模型，无需大幅修改原架构。
- **实验全面性**：覆盖多种基座模型、多个长尾方法、多个数据集，并进行消融和超参数分析，结果可靠。
- **性能提升显著**：在多个指标上均达到新SOTA，且统计显著。

## 8. 不足与局限

- **实验覆盖**：仅使用了三个公开数据集（均为电商/浏览场景），未在更多领域（如音乐、视频）验证；对比的长尾方法数量有限（仅4个），未与更多最新方法（如基于LLM的增强方法）对比。
- **部署计算成本**：混合意图的谱聚类虽可预计算，但训练时仍需维护意图嵌入，且多任务损失增加了计算复杂度，未给出实际推理时间对比。
- **参数敏感性**：柔性系数σ、方差阈值η、平衡系数ϵ等需要调参，在应用时对非专业用户可能不够友好。
- **意图聚类假设**：依赖物品的属性标签，若属性缺失或噪声大，聚类质量可能下降。
- **未提及公平性或偏斜风险**：长尾推荐可能引入新的偏差（如过度推荐某些小众物品），论文未讨论社会影响或伦理问题。

（完）
