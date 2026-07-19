---
title: Sign-Aware Multimodal Graph Recommendation
title_zh: 符号感知多模态图推荐
authors: "Yahong Lian, Haotian Tian, Chunyao Song, Tingjian Ge"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38547/42509"
tags: ["query:rec-sys"]
score: 9.0
evidence: 基于符号感知的多模态图推荐深度学习
tldr: 现有多模态推荐系统忽视正负反馈的语义差异。本文提出符号感知的多模态图推荐方法，通过符号图分别建模正负反馈信号，有效利用负反馈信息提升推荐性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 传统多模态推荐忽略高低评分交互的语义差异，未能充分利用负反馈信息。
method: 提出符号图机制，分别建模正负反馈信号，并融合多模态特征进行图推荐。
result: 实验证明区分正负反馈能显著提升推荐准确率。
conclusion: 符号感知建模是提升推荐系统性能的有效途径。
---

## Abstract
A multimodal recommendation system (MRS), which leverages rich multimodal information to model user preferences, has recently attracted significant research interest. Most existing MRSs focus primarily on developing sophisticated encoders for feature extraction, typically relying on simple aggregation of interaction-based features for final predictions. However, this conventional paradigm fails to account for the critical semantic difference between high- and low-rating interactions: while high ratings indicate user preference, low ratings explicitly convey dissatisfaction. Such oversight of negative feedback semantics may significantly limit the system’s recommendation performance. Recently, sign graphs—which model positive and negative feedback signals separately—have gained considerable attention. Inspired by this approach, we propose Sign-Aware Multimodal Graph Recommendation (SiMGR), a novel framework incorporating signed graphs into multimodal recommendation systems. SiMGR fuses multimodal features with signed interactions in a unified graph framework by integrating modality-specific representations and applying user-level thresholds to separate positive and negative subgraphs. A balanced pseudo-edge augmentation strategy is introduced to alleviate sparsity and enhance generalization. Experiments on three public multimodal recommendation datasets show that SiMGR outperforms state-of-the-art baselines, achieving an average 4.28% improvement in NDCG@20.

---

## 论文详细总结（自动生成）

# Sign-Aware Multimodal Graph Recommendation 论文总结

## 1. 核心问题与整体含义（研究动机和背景）

现有多模态推荐系统（MRS）通常将所有用户-商品交互（如评分1-5）视为正向信号，忽略了高低评分之间的语义差异：高评分表示偏好，低评分明确表达不满。这种对负反馈的忽视限制了推荐性能。作者通过初步实验证明，仅使用高评分（4-5分）训练比使用全部评分效果更好，说明低评分数据实际上会降低精度。受符号图（signed graph）建模正负反馈的启发，提出**SiMGR**，将符号图引入多模态推荐，以充分利用显式评分中的正负语义。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
在统一图框架中融合多模态特征与符号化交互，分别对正反馈和负反馈进行建模，并通过个性化阈值和伪边增强解决稀疏问题。

### 关键技术细节
- **Signed Graph Augmentation (SGA)**：  
  将原始用户-物品图根据阈值δ（默认δ=3）拆分为正子图G⁺和负子图G⁻。对正子图使用多层GCN（式5-8）得到正嵌入Ẽ⁺；对负子图使用一层GNN + MLP（式9）得到负嵌入Ẽ⁻；然后通过线性注意力融合正负信息得到新的交互图嵌入E_id^(new)（式10-11），替代原始模型的交互图嵌入。

- **Personalized Sign Threshold (PST)**：  
  针对不同用户的评分习惯，为每个用户u计算个性化阈值δ_u = ν*μ_u - (1-ν)*ε_u（式14），其中μ_u是用户平均评分，ε_u是标准差，ν是超参数。用δ_u替代全局δ划分正负子图。

- **Adaptive Pseudo Feedback Injection (APFI)**：  
  解决符号图稀疏问题。将物品按度分为尾项（低频）和流行项（高频）。对尾项：选择与用户历史语义最相似的未交互项作为伪正边加入A⁺；对流行项：选择最不相似的未交互项作为伪负边加入A⁻。注入阈值由ζ⁺和ζ⁻控制。

- **优化目标**：  
  BPR损失 + 符号余弦损失L_sign（式4） + 对比学习损失（InfoNCE，对齐用户/物品的交互嵌入与最终嵌入），总损失如式23。

## 3. 实验设计

### 数据集
- 三个Amazon review子集：**Baby**、**Sports**、**Electronics**。经过5-core滤波，8:1:1划分训练/验证/测试。密度分别为0.117%、0.045%、0.148%。

### Benchmark对比方法
- **经典方法**：MF-BPR、LightGCN
- **符号图方法**：SiReN、SiGRec、LSGRec
- **多模态方法**：MGCN、BM3、Freedom、LGMRec、DiffMM、TMLP

### 评估指标
- Recall@10、Recall@20、NDCG@10、NDCG@20

## 4. 资源与算力

论文在“Implementation Details”中提到：
- 硬件：NVIDIA RTX 3090（未说明数量）
- 框架：MMRec
- 超参数：学习率0.001，嵌入维度64，批大小2048，GCN层数2
- 未明确报告训练总时长或GPU数量。因此算力资源信息不完整。

## 5. 实验数量与充分性

共进行了以下实验：
- **整体性能对比**（Table 2）：在3个数据集上对比12个基线，报告Recall和NDCG。
- **消融实验**（Figure 3）：在3个数据集上验证Base、Base+SGA、w/o APFI、w/o PST、完整SiMGR，共5种配置。
- **超参数敏感性分析**（Figure 4-6）：分析ν（个性化阈值权重）、ξ（尾项比例）、ζ⁺/ζ⁻（伪边相似度阈值）的影响，在2-3个数据集上展示趋势。

实验设计较为全面：覆盖不同稀疏度数据集、多种类型基线、消融验证各模块贡献、超参数调优。结论客观，有统计显著性表述（平均提升4.28% NDCG@20）。但未做统计显著性检验（如t检验），也未讨论冷启动用户/物品的单独评估。

## 6. 主要结论与发现

- SiMGR在所有三个数据集上均优于所有基线，平均Recall@20提升4.36%，NDCG@20提升4.28%。
- 符号图增强（SGA）显著提升性能，说明区分正负反馈至关重要。
- 个性化阈值（PST）和自适应伪边注入（APFI）进一步带来增益，完整模型最优。
- 多模态方法整体优于传统协同过滤方法。

## 7. 优点

1. **创新性**：首次将符号图系统性地引入多模态推荐，填补了负反馈建模的空缺。
2. **模块设计合理**：SGA、PST、APFI三个模块分别解决信号分离、个性化、稀疏三大关键问题，逻辑清晰。
3. **实验充分**：多个数据集、多种基线、完整的消融和超参数分析，验证了每个组件的有效性。
4. **性能提升显著**：在多个指标上取得一致提升，效果稳健。

## 8. 不足与局限

1. **计算资源未明确**：未报告训练时间、GPU数量，难以复现资源需求。
2. **实验覆盖有限**：仅三个Amazon子集，未在更大规模或不同领域（如视频、新闻）上验证；未评估冷启动场景。
3. **超参数依赖**：ν、ξ、ζ⁺/ζ⁻需要调优，不同数据集取值不同，可能影响泛化。
4. **未讨论负反馈的噪声问题**：低评分可能包含误点或异常行为，简单按阈值划分可能引入噪声。
5. **复杂度分析缺失**：未比较模型参数量或推理时间，实际部署成本不明。
6. **对比方法选择**：未包含最新的大模型或多模态预训练方法，基准版本略旧（2025年以前）。

（完）
