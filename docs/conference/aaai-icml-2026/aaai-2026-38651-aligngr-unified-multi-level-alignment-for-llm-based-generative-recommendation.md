---
title: "Align³GR: Unified Multi-Level Alignment for LLM-based Generative Recommendation"
title_zh: Align³GR：面向大语言模型生成式推荐的统一多层次对齐
authors: "Wencai Ye, Mingjie Sun, Shuhang Chen, Wenjin Wu, Peng Jiang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38651/42613"
tags: ["query:rec-sys"]
score: 8.0
evidence: 基于大语言模型的生成式推荐，多层次对齐
tldr: 大语言模型应用于推荐系统时存在语义和行为对齐问题。本文提出Align3GR框架，通过双重分词、双向语义对齐和渐进式DPO策略，统一了令牌级、行为建模级和偏好级对齐。实验表明该方法显著提升了生成式推荐的性能，推动了LLM在推荐中的实用化。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 将大语言模型用于推荐时存在语义和行为层面的不对齐问题。
method: 提出Align3GR，融合协同信号的双重分词、双向语义对齐和渐进式偏好优化。
result: 在多个推荐基准上，Align3GR超越了现有基于LLM的推荐方法。
conclusion: 统一多层次对齐能有效释放大语言模型在推荐中的潜力。
---

## Abstract
Large Language Models (LLMs) demonstrate significant advantages in leveraging structured world knowledge and multi-step reasoning capabilities. However, fundamental challenges arise when transforming LLMs into real-world recommendation systems due to semantic and behavioral misalignment. 
To bridge this gap, we propose Align³GR, a novel framework that unifies token-level, behavior modeling-level, and preference-level alignment. Our approach introduces: Dual tokenization fusing user-item semantic and collaborative signals. Enhanced behavior modeling with bidirectional semantic alignment. Progressive DPO strategy combining self-play (SP-DPO) and real-world feedback (RF-DPO) for dynamic preference adaptation. Experiments show Align³GR outperforms the SOTA baseline by +17.8% in Recall@10 and +20.2% in NDCG@10 on the public dataset, with significant gains in online A/B tests and full-scale deployment on an industrial large-scale recommendation platform.

---

## 论文详细总结（自动生成）

# 论文《Align³GR: Unified Multi-Level Alignment for LLM-based Generative Recommendation》中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：大语言模型（LLM）在结构化世界知识和多步推理方面具有显著优势，但将其直接应用于真实推荐系统时存在根本性的**语义和行为不对齐**问题。LLM原生擅长基于语义信息的“下一个词预测”，而推荐系统需要基于用户交互行为建模隐含偏好，二者之间存在鸿沟。
- **整体含义**：为了将LLM真正转变为推荐系统，需要在多个层面进行对齐。已有工作（如EAGER、DAS等）尝试在分词、多任务微调、偏好对齐等环节引入协同信号，但通常将用户和物品独立处理，忽略了二者之间的语义与协同依赖性，且偏好对齐缺乏渐进学习机制，难以适应动态真实的用户兴趣。
- **论文贡献**：提出**Align³GR**框架，统一了令牌级、行为建模级和偏好级的对齐，通过双SCID分词、双向语义对齐和渐进式DPO策略，系统性地弥合LLM与推荐系统之间的差距。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程

### 核心思想
- 构建一个**三层对齐流水线**：令牌级对齐（Dual SCID Tokenization）→ 行为建模级对齐（Multi-task SFT）→ 偏好级对齐（Progressive DPO: SP-DPO + RF-DPO）。每一层都以前一层为基础，逐步增强LLM对推荐任务的适应能力。

### 关键技术细节

#### (1) 令牌级对齐：Dual SCID Tokenization
- **双编码器**：对用户和物品分别提取语义特征（如文本描述）和协同特征（如行为模式）。语义编码器使用冻结的T5，协同编码器使用冻结的DIN。
- **融合与量化**：将两类特征拼接后通过混合语义-协同编码器（SC Encoder，如MLP）得到统一SC嵌入，再利用RQ-VAE量化得到层次化离散的**SCID**（Semantic-Collaborative ID）。
- **训练损失**（公式1和2）：
  - 用户-物品行为损失 \(L_{U2I}\)：使用采样softmax，拉近正交互对（用户SC嵌入与物品SC嵌入）的距离。
  - 联合损失 \(L = \alpha L_{U2I} + \gamma (L_{User\_RQ} + L_{Item\_RQ})\)，分阶段优化：先调\(\alpha=1,\gamma=0\)稳定行为对齐，再切换至\(\alpha=0.1,\gamma=1\)优化量化。
- **优势**：同时编码用户和物品，通过联合优化保持交互关系，压缩令牌空间，为下游任务提供更丰富表示。

#### (2) 行为建模级对齐：Multi-task SFT
- **基础任务**：基于LC-Rec的多个生成任务：顺序物品预测、非对称物品预测、基于用户意图的物品预测、个性化偏好推断。
- **增强设计**：
  - 在所有任务提示中**注入用户SCID令牌**（图3中的User SCID）。
  - 新增**双向对齐任务**（B.2）：从用户画像文本预测SCID（text→SCID），以及从SCID重构画像文本（SCID→text），显式对齐结构化ID与语义信息。
- **效果**：使LLM不仅能理解物品序列，还能把握用户-物品协同关系和用户语义。

#### (3) 偏好级对齐：Progressive DPO with Self-Play and Real-world Feedback
- **基于Softmax-DPO**，构建包含多个拒绝响应的训练样本，优化目标如公式3所示。
- **渐进式SP-DPO**（自对弈DPO）：
  - 将训练分为Easy、Medium、Hard三个阶段，根据**prefix-ngram匹配**衡量SCID前缀重叠程度，重叠越多难度越大。
  - 模型用自身生成的结果与真实标签对比构造偏好对，逐步提高区分难度。
- **渐进式RF-DPO**（真实反馈DPO）：
  - 利用真实用户反馈（喜欢、中性、不喜欢）作为偏好数据。公开数据集通过LLM情感模型评分映射为三级。
  - 训练也分阶段：Easy阶段用强烈不喜欢项为负样本，Hard阶段用中性项为更难负样本。
- **迭代训练**：每个阶段的微调模型作为下一阶段的参考模型（\(\pi^i_\theta \to \pi^{i+1}_{ref}\)），实现平滑适应。

## 3. 实验设计

### 数据集与场景
- **公开数据集**（离线实验）：
  - **Instruments**（Amazon子集，音乐设备）
  - **Beauty**（Amazon子集，美容产品）
  - **Yelp**（Yelp挑战数据，用户-商家交互）
  - 预处理：过滤交互<5的用户和物品，留一法划分训练/验证/测试集，用户历史最长20。
- **工业场景**（在线A/B测试）：
  - 某大型广告推荐平台，分配10%流量（约4000万+用户），多周测试，最后全量部署。

### 基准方法
- 传统推荐：MF、LightGCN
- 序列推荐：Caser、HGN、Bert4Rec、SASRec、BigRec
- 生成式/LLM推荐：P5-SemID、P5-CID、TIGER、LETTER-TIGER、LC-Rec、LETTER-LC-Rec、EAGER-LLM（最强基线）
- Align³GR基于LLaMA2-7B，LoRA微调。

### 评估指标
- 离线：Recall@5/10，NDCG@5/10
- 在线：Recall@100，Revenue提升百分比

## 4. 资源与算力

- **GPU型号与数量**：4块NVIDIA RTX A800 GPU。
- **训练步数**：20,000步。
- **批量大小**：1024。
- **其他细节**：学习率从{1e-3, 5e-4, 1e-4}中调优；使用AdamW优化器；LoRA参数高效微调。
- **备注**：论文未明确给出总训练时长或具体小时数，但根据通常经验，4*A800上20k步约需数小时至一天（取决于模型大小与数据量）。

## 5. 实验数量与充分性

- **实验数量**：
  - 离线实验：3个公开数据集，对比13种方法，主表（表1）涵盖每个方法在4个指标上的结果。
  - 在线A/B测试：1个工业场景，对比TIGER和工业基线（表2）。
  - 消融实验：3组（令牌级、行为级、偏好级），分别见表3、表4、表5；此外还有图4的增量对齐配置对比。
- **充分性与公平性**：
  - 覆盖了传统、序列、生成式多种主流基线，且使用相同预处理协议。
  - 消融实验控制变量，逐步累积模块验证各层贡献。
  - 在线A/B测试使用真实流量，统计显著，且全量部署验证实际收益。
  - 所有实验报告5次随机种子平均结果（表1脚注），保证稳定性。
- **客观性**：论文公开了实现细节，但未提供代码链接；贡献声称有严格对比，数据可信。

## 6. 主要结论与发现

- **离线性能**：Align³GR在所有数据集和指标上均超越所有基线。在Instruments上，Recall@10提升17.8%，NDCG@10提升20.2%（对比EAGER-LLM）。在Beauty和Yelp上也持续取得15-28%的提升。
- **在线A/B测试**：Recall@100从0.218（工业基线）提升至0.242（TIGER为0.229），Revenue提升**1.432%**，并全量部署。
- **消融结论**：
  - 双SCID分词比单侧分词显著提升，联合优化U-I对齐进一步增益。
  - 行为对齐中，用户SCID注入+双向语义对齐（B2）贡献最大。
  - 偏好对齐中，渐进式SP-DPO+RF-DPO携手最优，渐进策略优于静态。
- **核心发现**：统一多层次对齐是充分发挥LLM推荐潜力的关键，令牌级、行为级、偏好级三者缺一不可。

## 7. 优点

- **方法论创新性**：首次提出系统性的三层对齐框架，将分词、微调、偏好优化连贯起来，且每个级别都有针对性设计（双SCID、双向对齐、渐进DPO）。
- **协同与语义融合**：在令牌级同时编码用户和物品的协同+语义，并通过U2I损失联合优化，避免孤立建模。
- **渐进式DPO**：引入课程学习思想，从易到难训练偏好模型，结合自对弈和真实反馈，更贴合推荐场景的复杂偏好。
- **工业验证**：在千万级用户平台上进行在线A/B测试并全量部署，证明方法实际有效。
- **实验全面**：跨领域数据集、多种基线、充分消融，且消融实验控制变量合理。

## 8. 不足与局限

- **实验覆盖范围**：公开数据集仅3个（均为Amazon/Yelp），多样性有限；未在更多领域（如新闻、短视频）验证。
- **基线选择**：虽然对比了EAGER-LLM等最新方法，但可能遗漏其他同期未公开工作；且传统方法（如SASRec）性能偏低，可能未进行最优调参。
- **资源开销**：基于7B模型+LoRA，仍需要4*A800训练20k步，部署成本较高。未报告推理延迟，可能不适合极低延迟场景。
- **消融实验局限性**：令牌级消融仅固定在偏好级最优设置下，未交互考察各层级间相互影响；渐进DPO中Easy/Medium/Hard的划分依赖prefix-ngram匹配阈值，其对不同数据集的敏感性未探讨。
- **真实反馈获取**：RF-DPO依赖用户行为（点赞、点击、消极反馈）或LLM情感评分，可能引入标签噪声或标注偏差，论文未分析噪声鲁棒性。
- **可复现性**：未提供代码或详细超参数设置（如RQ-VAE层数、β值调优细节），仅给出部分实现。

（完）
