---
title: "MM4Rec: Multi-Source and Multi-Scenario Recommender for Unified User Preference"
title_zh: MM4Rec：面向统一用户偏好的多源多场景推荐系统
authors: "Chu-Chun Yu, Ming-Yi Hong, Miao-Chen Chiang, Min Chen Hsieh, Che Lin"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38655/42617"
tags: ["query:rec-sys"]
score: 9.0
evidence: 多源多场景推荐
tldr: 该论文提出MM4Rec，一个统一的多源多场景推荐框架。现有方法分别建模各源场景，无法利用跨上下文信号。MM4Rec通过源感知Transformer编码器联合建模异构输入，多场景行为抽取层捕捉场景动态，趋势感知学习器增强时间表示，在多个场景中提升了推荐效果。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有推荐系统分别建模不同源和场景，忽略了跨上下文的共享和互补信号。
method: 提出MM4Rec框架，包括源感知Transformer、多场景行为抽取层和趋势感知学习器。
result: 在跨场景推荐中，MM4Rec显著优于单源单场景方法。
conclusion: MM4Rec实现了多源多场景下的统一用户偏好建模，提升了推荐系统的泛化能力。
---

## Abstract
As online ecosystems grow increasingly complex, personalized recommendation systems must integrate user preferences across heterogeneous content sources and interaction scenarios. However, conventional methods typically model each source and scenario in isolation, hindering their ability to capture shared and complementary signals across contexts. In this work, we propose MM4Rec, a unified framework for multi-source and multi-scenario recommendation. MM4Rec introduces a Source-Aware Transformer Encoder to jointly model heterogeneous inputs, a Multi-Scenario Behavior Extraction Layer based on a multi-mixture-of-experts architecture to capture scenario-specific dynamics, and a Trend-Aware Learner to enhance temporal representation learning. Extensive experiments on three real-world datasets demonstrate that MM4Rec consistently outperforms strong baselines across standard recommendation metrics. To facilitate future research, we also release two large-scale datasets encompassing diverse sources and scenarios.

---

## 论文详细总结（自动生成）

# MM4Rec：面向统一用户偏好的多源多场景推荐系统——详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **问题**：现有推荐系统通常针对单一内容源（如新闻、广告）和单一交互场景（如浏览、推送通知）分别建模，忽视了跨源、跨场景之间的共享信号和互补信息，导致用户偏好建模不完整、推荐效果受限。
- **动机**：真实平台中用户在不同场景（如主动浏览 vs 推送通知）和不同内容源（如新闻、广告、视频、文章）上的行为相互影响。例如，新闻浏览行为可增强广告响应。而现有方法（如Multi-Task Learning主要优化单一场景内的多目标，Multi-Scenario Learning忽略跨源关系）无法有效整合这些信息。
- **贡献**：提出首个同时处理多源和多场景的统一推荐框架MM4Rec，并发布两个包含推送通知数据的大规模真实世界数据集（AviviD DatasetA和DatasetB），填补了该领域的公共资源空白。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：通过三个关键模块（SATE、MSBE、TAL）在统一架构中联合建模多源异构输入、多场景动态行为以及时间演化趋势，实现跨上下文信息共享，同时缓解“跷跷板效应”（一个场景/任务提升导致另一个下降）。

### 关键技术细节

#### 2.1 特征表示
- **时间信息提取器（TIE）**：提取点击时间特征（工作日、小时、时间间隔等），归一化并嵌入为向量 `e_T`。
- **上下文信息提取器（CIE）**：使用冻结的预训练语言模型（TAIDE，基于Llama 2）将项目标题转换为上下文嵌入 `e_A`。
- **嵌入层**：将源（`o`）和场景（`c`）映射为可学习的嵌入向量 `e_o`、`e_c`。

#### 2.2 源感知Transformer编码器（SATE）
- **结构**：在标准Transformer基础上，引入共享FFN和源特定FFN，并通过源自适应门控（SoAG）动态平衡两者的贡献。
- **流程**：
  1. 输入序列经过多头自注意力层后得到 `H^{(l)}`。
  2. 对 `H^{(l)}` 做均值池化，与源嵌入 `e_o` 拼接，经过两层MLP和softmax得到门控权重 `[g_{sh}, g_1, ..., g_q]`。
  3. 输出 `H_out = g_{sh} * PFFN_shared(H) + Σ g_i * PFFN_i(H)`。
  4. 残差连接和层归一化后输出下一层。
- **两个并行分支**：T-SATE（处理时间嵌入序列）和C-SATE（处理上下文嵌入序列），最后将均值池化结果拼接得到 `h_hat`。

#### 2.3 多场景行为抽取层（MSBE）
- **基于MMoE架构**：包含多个专家网络（全连接层）和场景专用门控网络。
- **门控计算**：对每个场景 `c`，门控输入为 `[h_hat; e_c]`，通过两层MLP和softmax得到专家权重 `G_c`。
- **输出**：每个场景 `c` 的加权专家输出 `V_c`，所有 `V_c` 拼接后经塔网络（两层全连接+ReLU）得到最终的个性化表示 `V_P`。

#### 2.4 趋势感知学习器（TAL）
- 从最近一小时内所有用户的总体点击中提取Top-N*流行项目，获取其上下文嵌入，通过自注意力机制和均值池化得到趋势表示 `V_Tr`。
- 使用门控网络动态融合 `V_P` 和 `V_Tr` 得到最终用户表示 `V`。

#### 2.5 损失函数
- 训练时对每个用户序列生成一个正样本和N个负样本，计算余弦相似度后经softmax归一化，使用二元交叉熵损失。

## 3. 实验设计

### 3.1 数据集
- **AviviD DatasetA / DatasetB**：来自两个媒体平台，包含两种场景（推送通知`Push`和网页浏览`Browse`）和两种内容源（新闻`News`和广告`Ads`），共4个组合。过滤掉交互少于3次的用户，取最近20次交互。
- **Tenrec公共数据集**：包含两种场景（QB和QK）和两种内容源（视频和文章）。

### 3.2 对比方法（Baselines）
- **单模型**：SASRec（自注意力序列推荐）、Push4Rec（将CTR方法适配到序列推荐），每个源-场景单独训练。
- **多场景模型**：Shared Bottom、MMoE、PLE、STAR（每源训练但跨场景）。
- **多源多场景模型**：STAR-MM（STAR的晚期融合变体）、Push4Rec-MM（Push4Rec + 双MMoE模块），以及与MM4Rec直接对比。

### 3.3 评估指标
- Hit Ratio@10 (HR@10) 和 Normalized Discounted Cumulative Gain@10 (NDCG@10)。每个实验运行5次取平均。

## 4. 资源与算力

- 论文未明确说明使用的GPU型号、数量或训练时长。仅在致谢中感谢台湾国家高速网络与计算中心（NCHC）提供计算和存储资源。未提供具体算力开销数据。

## 5. 实验数量与充分性

- **主要实验**：在两个数据集（AviviD DatasetA、Tenrec）上进行了完整的性能对比（表1、表2），覆盖4个源-场景组合，共8组指标。
- **消融实验**：在AviviD DatasetA上进行了3组消融：
  - 移除MSBE、移除SATE（分别从时域和上下文路径）、移除SATE完全。
  - 额外分析了SATE中SoAG和源特定FFN的作用（表4）。
- **附录**：提供了AviviD DatasetB的完整结果（表A.4-A.6）、参数分析、时间复杂度讨论。
- **充分性评估**：
  - **充分**：对比了单模型、多场景模型、多源多场景模型共8种baseline，覆盖了主流方法；消融实验验证了各模块有效性；统计显著性检验（配对t检验）表明结果可靠。
  - **客观**：统一数据划分、调参策略（基于验证集NDCG@10）、多次运行取平均。
  - **不够充分之处**：仅在一个公共数据集（Tenrec）和两个自建数据集上验证，缺乏更多公开跨场景跨源benchmark；未在冷启动场景下评估。

## 6. 论文的主要结论与发现

1. **MM4Rec显著优于所有baseline**：在两个数据集的大多数指标上取得最高NDCG@10和HR@10，且通过统计显著性检验。
2. **多源多场景联合建模有效**：所有多源多场景模型（STAR-MM、Push4Rec-MM、MM4Rec）均优于单独训练的单模型，说明跨上下文信息共享能提升推荐质量。
3. **有效缓解跷跷板效应**：其他多场景模型（如STAR）在提升一个场景时导致另一场景下降，而MM4Rec在所有场景上一致提升（例如Push Ads NDCG从0.337提升至0.473，同时Browse Ads从0.468提升至0.469）。
4. **各模块不可或缺**：消融实验表明移除MSBE、SATE或SoAG均导致性能下降，验证了设计的必要性。

## 7. 优点

- **方法创新**：首次将多源和多场景问题统一到一个端到端框架中，设计新颖的源感知Transformer（SATE）和基于MMoE的多场景行为提取层（MSBE），有效实现了跨上下文信息共享与场景特异性建模。
- **工程贡献**：发布两个包含推送通知数据的大规模真实数据集，填补了公共资源空白，有利于后续研究。
- **实验严谨**：对比了多个维度的baseline，进行了充分的消融和组件分析，使用统计检验验证显著性，结果可信。
- **鲁棒性验证**：在两个不同来源的数据集（媒体平台和Tenrec）上均取得一致优势，且克服了跷跷板效应。

## 8. 不足与局限

1. **稀疏行为下的性能**：论文承认在用户行为极度稀疏时可能性能下降，未专门针对冷启动场景进行评估。
2. **结构简单性假设**：仅考虑了两种源和两种场景的简单组合，未探索更复杂的多源多场景结构（如三个以上源/场景）。
3. **泛化性验证不足**：仅在三个数据集上测试（其中两个来自同一家媒体平台），需要更多平台验证框架的通用性。
4. **算力信息缺失**：未提供GPU型号、训练时长等，不利于成本评估和复现。
5. **计算复杂度分析有限**：附录中虽有时间复杂度讨论，但未对比各模块实际运行时间，也未分析内存开销。
6. **趋势感知模块依赖整体流行度**：TAL基于全局短时流行度，可能无法捕捉个性化细粒度趋势变化。

（完）
