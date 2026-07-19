---
title: "GUIDER: Uncertainty Guided Dynamic Re-ranking for Large Language Models Based Recommender Systems"
title_zh: "GUIDER: 面向大语言模型推荐系统的不确定性引导动态重排序"
authors: "Cai Xu, Xujing Wang, Ziyu Guan, Wei Zhao, Meng Yan"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38639/42601"
tags: ["query:rec-sys"]
score: 7.0
evidence: 基于深度学习的大语言模型推荐系统的不确定性引导重排序
tldr: 本文针对大语言模型推荐系统的不确定性和幻觉问题，提出GUIDER不确定性引导动态重排序框架。该方法通过量化LLM生成推荐的置信度，动态调整排序以过滤不可靠输出。实验表明，GUIDER在减少幻觉、提升推荐可信度方面效果显著，为LLM在推荐中的可靠应用提供了新方案。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: LLM推荐系统存在数据稀疏、幻觉和缺乏透明性等问题。
method: 提出不确定性量化方法，动态重排序以降低不可靠推荐的影响。
result: 在推荐数据集上，GUIDER提升了推荐准确性和用户信任度。
conclusion: 不确定性引导重排序能有效增强LLM推荐系统的可靠性。
---

## Abstract
Large Language Models (LLMs) are increasingly integral to recommendation systems, offering sophisticated language understanding and generation capabilities. However, their practical application is often hindered by challenges such as data sparsity, the generation of unreliable or hallucinated recommendations, and a general lack of transparency in their decision-making processes. Existing mitigation strategies frequently introduce significant complexity or computational overhead. To address these limitations, particularly the critical gap in quantifying the confidence of LLM-generated recommendations, we propose GUIDER: Uncertainty Guided Dynamic Re-ranking for Large Language Models based Recommender Systems. This new framework innovatively leverages the logits produced by LLMs as evidence for recommended items. By employing a Dirichlet distribution, GUIDER decomposes the total predictive uncertainty into distinct Data Uncertainty (DU), reflecting inherent data ambiguity, and Model Uncertainty (MU), indicating the model's own conviction. This principled decomposition, achieved with a single inference pass, enhances transparency and trustworthiness. Based on the quantified DU and MU levels, our system dynamically adapts its recommendation strategy---adjusting output diversity---through a four-quadrant analysis that tailors responses to specific uncertainty profiles. Extensive experiments conducted in zero-shot recommendation settings validate the effectiveness of our approach. GUIDER consistently outperforms existing methods in reliability-aware scenarios, demonstrably improving recommendation quality. This framework not only advances the practical deployment of LLM-based recommenders by making them more dependable but also provides a robust foundation for future research into uncertainty-aware generative systems.

---

## 论文详细总结（自动生成）

# 论文详细总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：大语言模型（LLM）在推荐系统中展现出强大的语义理解和生成能力，但实际部署时面临数据稀疏、幻觉（hallucination）和决策过程不透明等问题。现有缓解方法（如Chain-of-Thought、RAG）往往带来额外复杂度和计算开销。
- **核心问题**：缺乏对LLM生成推荐结果置信度的量化手段，无法区分推荐不可靠的来源——是用户数据本身的歧义性（数据不确定性，DU），还是模型自身知识不足（模型不确定性，MU）。
- **整体含义**：本文提出GUIDER框架，利用单次推理得到的logits作为证据，通过狄利克雷分布将总预测不确定性分解为DU和MU，并基于此动态调整重排序策略，以提升LLM推荐系统的可靠性与可解释性。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- 将LLM输出的原始logits解释为每个候选物品的证据（evidence），构建狄利克雷分布，从而分解不确定性。
- 根据DU和MU的高低将推荐场景分为四个象限，每个象限采用不同的动态排序策略。

### 关键技术细节
1. **从logits到证据**：  
   对每个候选物品k，将其logit \( z_k \) 通过激活函数（softplus）加常数1得到浓度参数 \( \alpha_k = \text{softplus}(z_k) + 1 \)，构成狄利克雷分布 \( \text{Dir}(p|\alpha_1,\dots,\alpha_K) \)。
2. **不确定性分解**：
   - **数据不确定性 (DU)**：  
     \[
     DU = -\sum_{k=1}^K \frac{\alpha_k}{\alpha_0} \left( \psi(\alpha_k+1) - \psi(\alpha_0+1) \right)
     \]  
     其中 \( \alpha_0 = \sum \alpha_k \)，\( \psi \) 为digamma函数。DU衡量数据内在歧义性。
   - **模型不确定性 (MU)**：  
     \[
     MU = \frac{K}{\sum_{k=1}^K (\alpha_k + 1)}
     \]  
     当总证据 \( \sum \alpha_k \) 较小时MU较高，反映模型缺乏信心。
3. **四象限动态重排序**：
   - **象限I（高DU，高MU）**：采用流行度混合策略，融合LLM得分与全局流行度。
   - **象限II（低DU，高MU）**：对后几名候选进行随机洗牌式的探索。
   - **象限III（低DU，低MU）**：直接使用原始logits排序（信任模型）。
   - **象限IV（高DU，低MU）**：使用最大边际相关性（MMR）增强多样性，避免过拟合单一兴趣。
- 所有计算仅需一次LLM前向推理（单token预测），高效且低开销。

## 3. 实验设计

- **数据集**：三个公开序列推荐数据集：
  - MovieLens 10M（ML-10M，71,567用户，10,681物品）
  - Amazon Grocery and Gourmet Food（Amazon-Grocery）
  - Steam
- **基准模型**：对比方法包括：
  - 传统序列推荐：SASRec、BERT4Rec
  - LLM基推荐：PepRec、HSTU、RankGPT、LLM4Rerank（SOTA）
  - 内部基线：Standard LLM（不采用不确定性框架）
- **骨干LLM**：Llama3-8B、Qwen2.5-7B、Mistral-7B（零样本设置，不微调）
- **评估指标**：NDCG@K和Recall@K（K=10,20）
- **实施细节**：每个用户候选集大小 \( N_c = 100 \)，历史长度 \( L_{\max} = 20 \)，阈值为验证集平均值。

## 4. 资源与算力

- **文中明确说明**：所有实验在 **单张 NVIDIA A100-80G GPU** 上使用PyTorch执行。
- **训练时长**：未明确报告训练时间。由于为零样本设置，无需微调LLM，仅需离线一次验证集阈值计算，推理时间极短。
- **推理效率**：GUIDER平均每样本约147.3ms，远快于Standard LLM（530.9ms）和LLM4Rerank（12339ms）。

## 5. 实验数量与充分性

- **实验组数**：
  - 主实验：3个数据集 × 3个骨干LLM × 2个指标（共多个对比表格）。
  - 不确定性有效性验证（RQ1）：使用Gini impurity和物品流行度将用户分组，绘制分裂小提琴图。
  - 影响因素分析（RQ3）：调节历史长度（Lmax=10,20）和候选集大小（Nc=50,100,150）观察DU/MU及NDCG变化。
  - 消融实验：移除动态策略、移除DU、移除MU，分别对比性能下降。
  - 复杂度分析：对比推理时间。
- **充分性判断**：实验覆盖多种数据集、多骨干模型、多因素分析，且进行了消融和超参数影响分析，设计较为全面。对比方法包含传统序列模型和多种LLM方法（含SOTA），公平性较好（均零样本）。唯一可改进的是未在微调设置下验证，但论文聚焦于零样本场景。

## 6. 主要结论与发现

1. **不确定性分解的有效性**：DU与用户历史偏好多样性正相关，MU与用户偏好物品的流行度负相关，验证了指标合理性。
2. **动态重排序显著提升性能**：GUIDER在所有数据集上超过所有基线，最佳变体Ours(Qwen2.5-7B)达到SOTA，NDCG@20在ML-10M上达0.6693，Recall@20达0.8149。
3. **各组件均不可或缺**：消融实验显示移除DU或MU均导致性能大幅下降（NDCG@20从0.5899降至0.4729/0.4932），移除整个动态策略下降最严重（0.3608）。
4. **提示因素影响不确定性**：更长的历史降低DU和MU并提升性能；更大的候选集增加不确定性并降低性能。
5. **效率优势**：单次推理 + 轻量计算，推理时间远低于需要多次采样的方法。

## 7. 优点

- **方法创新**：首次将LLM的logits视为证据并用狄利克雷分布分解DU和MU，为LLM推荐系统的可信度提供可解释量化。
- **高效实用**：仅需单次推理，无额外LLM调用，复杂度低，适合部署。
- **动态自适应策略**：四象限设计合理区分了不同风险场景，分别采用流行度、探索、信任、多样化策略，提升鲁棒性。
- **实验严谨**：多数据集、多骨干、多维度分析，消融实验验证了每个模块的必要性。
- **开放代码**：提供GitHub仓库，可复现。

## 8. 不足与局限

- **零点设置限制**：仅验证零样本推荐，未探讨微调场景下的效果；实际工业系统往往需要微调以适应领域数据。
- **阈值设定依赖验证集**：高低DU/MU的阈值基于验证集平均值，可能无法适应动态分布变化。
- **仅适用于list-wise ranking**：框架建立在候选集重排序场景，不适用于更广泛的生成式推荐（如自由文本输出）。
- **未讨论场景泛化**：四个象限的固定策略（如象限I的流行度混合）可能不适用于流行度分布极端的数据集。
- **缺少用户研究**：未通过用户调研验证推荐结果的真实可信度提升。
- **潜在偏差**：负采样采用随机均匀采样，可能引入偏差；真实场景中负样本分布不均匀。

（完）
