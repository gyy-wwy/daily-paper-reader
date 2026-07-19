---
title: "MoMoREC: A Multi-agent Motivation Generation Framework for Residual Semantic ID-Aware Recommendation"
title_zh: MoMoREC：残差语义ID感知推荐的多智能体动机生成框架
authors: "Yige Wang, Mingming Li, Li Wang, Kaichen Zhao, Wangming Li, Weipeng Jiang, Xueying LI"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38623/42585"
tags: ["query:rec-sys"]
score: 8.0
evidence: 多智能体动机生成方法用于顺序推荐，融合大语言模型
tldr: 现有LLM推荐方法对用户购买动机理解不足，且高维嵌入与ID嵌入不兼容。本文提出MoMoREC，采用多智能体协同生成用户动机解释，并设计残差语义ID感知机制融合LLM与ID嵌入。实验表明MoMoREC在顺序推荐任务上优于现有LLM方法，同时降低推理开销。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 大语言模型推荐缺乏对用户购买原因的理解，且嵌入维度不兼容。
method: 设计多智能体生成用户动机序列，通过残差连接融合ID嵌入和LLM语义。
result: 在多个数据集上，MoMoREC在召回率和可解释性上全面优于基线。
conclusion: 动机生成和残差融合能有效增强LLM在顺序推荐中的效果。
---

## Abstract
Recent advances in the field of sequential recommendation have highlighted the potential of Large Language Models (LLMs) in enhancing item embeddings and improving user understanding. However, existing approaches face three major limitations: 1) insufficient understanding of the reasons behind users' purchase decisions, 2) the high-dimensional embeddings directly produced by LLMs are not well compatible with traditional low-dimensional ID embeddings and 3) reliance on additional fine-tuning and high inference overhead to adapt LLMs to the recommendation task. In this paper, we propose MoMoREC, a simple yet effective user-understanding-based recommendation strategy. This method leverages the intrinsic comprehension capabilities of LLMs combined with residual semantic IDs to better understand users. Specifically, starting from common user purchasing behaviors and incorporating item characteristics, we employ a multi-agent framework to utilize LLMs in analyzing user shopping motivations and extracting high-dimensional dense embeddings. These embeddings are then transformed into low-dimensional IDs using a residual semantic ID approach via clustering and residual dimensionality reduction, which can be fed into the recommendation model. MoMoREC effectively integrates the understanding power of LLMs with the strengths of recommendation systems, preserving rich semantic language embeddings while reducing or eliminating the need for auxiliary trainable modules. As a result, it seamlessly adapts to any sequential recommendation framework. Experiments on three benchmark datasets show that MoMoRec significantly improves traditional recommendation models, demonstrating its effectiveness and flexibility.

---

## 论文详细总结（自动生成）

# 论文《MoMoREC：残差语义ID感知推荐的多智能体动机生成框架》详细总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：现有基于大语言模型（LLM）的推荐方法存在三个主要缺陷：
  - 对用户购买决策背后的**深层动机**理解不足（仅基于交互序列）。
  - LLM生成的高维密集嵌入与传统的低维ID嵌入**不兼容**，导致性能不升反降。
  - 依赖额外的**微调**和巨大的推理开销，不适合真实推荐场景（如图2所示，参数/延迟差达数百倍）。
- **整体含义**：提出一种不依赖LLM生成推荐项、而是利用LLM的语义理解能力提取用户动机、并通过残差语义ID机制与轻量级推荐模型融合的框架，实现低延迟、高可解释性的推荐。

## 2. 方法论
### 核心思想
通过多智能体协同分析用户历史购买序列，生成用户购物动机（文本），并将动机通过聚类和残差量化转化为离散语义ID，作为传统推荐模型的附加特征输入。

### 关键技术细节（三阶段流程）
1. **动机生成（Motivation Generation）**
   - **分析Agent**：由生成模块和摘要模块组成。生成模块输出长文本动机和推理，摘要模块压缩为简短动机（≤20字）。
   - **判断Agent**：基于Chatbot Arena的ELO机制，对不同生成Agent（不同LLM/不同提示）的动机进行成对比较和评分，选出最优动机。

2. **动机传播（Motivation Spreading）**
   - **嵌入模型**：使用对比学习（InfoNCE + CoSENT损失）训练轻量级编码器，将动机文本和商品映射到同一语义空间。
   - **传播模块**：
     - **聚类模块**：采用LSH（局部敏感哈希）粗分桶 + 桶内密度聚类（图着色法）自动发现簇，无需预设K值。
     - **相似度模块**：对每个长尾商品，找到Top-K最近邻的已动机商品，通过多数投票分配动机。

3. **残差语义ID检索（Residual Semantic ID Retrieval）**
   - **码本计算**：使用分布式K-means对动机嵌入向量分层聚类，每层得到码本矩阵。
   - **残差ID计算**：
     - 第k层ID：`id_k = argmin_i ||h_{k-1} - c^i_k||_2`
     - 残差嵌入：`h_k = h_{k-1} - c^{id_k}_k`
     - 第一层输入为动机嵌入，得到多层（本文6层）整数ID向量。
   - 将多层语义ID嵌入后与原始物品ID嵌入拼接，经DNN输出推荐。

### 公式
- ELO评分：`ELO(R_i, R_j) = 1/(1+10^{(R_j-R_i)/β})`
- InfoNCE损失：`L_InfoNCE = -Σ_i log(exp(cos(m_i, i_i^+)/τ) / Σ_j exp(cos(m_i, i_j^-)/τ))`
- CoSENT损失：`L_Co = log[1 + Σ_{s(m_i,i_j)>s(m_m,i_n)} exp((cos(m_m,i_n)-cos(m_i,i_j))/τ)]`
- 残差ID计算：同上。

## 3. 实验设计
### 数据集与场景
- **三个Amazon真实数据集**：Beauty（12k用户/112k商品）、Video Game（2.77M用户/137k商品）、Baby Product（3.39M用户/217k商品）。
- **在线实验**：淘宝88VIP畅销榜（14天A/B测试）。

### Benchmark与对比方法
- **基座模型**：SASRec（经典顺序推荐Transformer）。
- **基线方法**：LLMESR、MoRec、whitenREC、LLMInit、RLMRec、UniSRec、AlphaFuse等（均为近期融合语义信息的推荐方法）。

### 评价指标
- 离线：Recall@10/20、MRR@10/20、Precision@10/20。
- 在线：用户点击率（UCTR）、交易转化率（TCR）、商品交易总额（GMV）。

## 4. 资源与算力
- **未明确说明**：论文未提供GPU型号、数量或总训练时长。仅提及：
  - 动机生成使用Qwen3-8B，判断Agent使用gpt-4o-mini-0718等。
  - 嵌入模型（Conan-embedding-v1）训练3个epoch，batch size 512，FP16混合精度，AdamW优化器。
  - 残差ID码本使用分布式K-means（多线程）。
- 整体算力消耗未量化，但强调推理时延仅增加7.9%（从550ms到593.54ms），参数从8.89M增加到41.63M。

## 5. 实验数量与充分性
- **实验组数丰富**：
  - 主实验：3个数据集 × 2种截断（@10/@20） × 9种方法，共54个结果条目。
  - 消融实验：去除动机传播和残差ID两个组件（Beauty数据集）。
  - 动机生成模块：对比4种生成LLM的ELO得分、判断Agent与人类一致性（12k条标注）。
  - 嵌入模型：对比5种嵌入模型的Pearson/Spearman相关系数。
  - 残差ID维度影响：从1到6维度测试。
  - 全参数移除可行性：对比微调与未微调嵌入模型（仅差4%）。
  - 在线A/B实验：14天。
- **充分性评价**：实验设计较为全面，覆盖离线/在线、消融、超参数、模型选择等多个维度，对比基线涵盖主流语义推荐方法，公平性较好（全部使用SASRec骨干）。但未包含更多基座模型（如BERT4Rec）的验证。

## 6. 主要结论与发现
- MoMoREC在三个数据集上**全面显著优于所有基线**：Recall@10提升4.07~11.67倍，MRR@10提升8.49~31.00倍，Precision@10提升6.69~23.51倍。
- 两个核心组件（动机传播、残差语义ID）均**不可或缺**：去除后性能大幅下降（Beauty上Recall@10从0.1725降至0.0791/0.0942）。
- **推理效率高**：相比基座SASRec，推理时延仅增7.9%（至593.54ms），远低于LLM生成推荐（如LLMESR +56.5%）。
- **在线收益明显**：GMV提升6.3%，TCR提升1%，UCTR小幅下降1.3%（可接受）。
- **可完全免微调潜力**：未微调的gte-Qwen2嵌入模型与微调模型仅差4%相关性，表明任务特定微调可能并非必需。

## 7. 优点
- **创新性强**：首次将多智能体LLM协同用于生成用户购买动机，而非直接生成推荐项，提升了可解释性。
- **设计精巧**：残差语义ID机制巧妙解决LLM高维嵌入与ID推荐不兼容问题，无需额外训练模块即可融合。
- **工程友好**：整体框架可无缝接入任意顺序推荐模型，在线延时增量极小（+7.9%），适合工业部署。
- **全面实验**：涵盖离线、在线、消融、超参数、模型选择等，且公开了详细实施细节（如提示词模板、聚类参数等），可复现性强。
- **对长尾问题有效**：通过动机传播覆盖低频商品，解决了LLM方法通常偏向高频项的缺陷。

## 8. 不足与局限
- **动机传播的噪声风险**：基于语义相似度传播可能将不相关的动机分配给长尾商品，引入噪声（论文未量化传播误差）。
- **LLM依赖**：动机生成质量受限于所选LLM的能力和成本（论文对比了多种LLM，但未讨论成本差异）。
- **泛化性局限**：实验仅在Amazon（电商）和淘宝（国内电商）进行，未验证其他领域（如新闻、视频、音乐）或多语言场景。
- **计算资源未透明**：未报告GPU型号、总训练时长和能耗，无法评估复现成本。残差ID码本K-means在多线程下开销未分析。
- **消融覆盖不足**：仅在一个数据集（Beauty）上做了完整的消融，其他两个数据集仅展示主实验结果。未对比不同基座模型（如Transformer vs. RNN）下的稳定性。
- **在线A/B测试细节有限**：仅给出相对变化（如+6.3% GMV），但未提供置信区间或p值，无法判断统计显著性。

（完）
