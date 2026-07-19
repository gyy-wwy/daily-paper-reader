---
title: "Tokenize Once, Recommend Anywhere: Unified Item Tokenization for Multi-domain LLM-based Recommendation"
title_zh: 一次分词，处处推荐：面向多领域LLM推荐系统的统一物品分词
authors: "Yu Hou, Won-Yong Shin"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38505/42467"
tags: ["query:rec-sys"]
score: 9.0
evidence: 面向多领域LLM推荐系统的统一物品分词
tldr: 现有物品分词方法需为每个领域训练独立模型，泛化性差。本文提出UniTok，集成混合专家架构和多码本，将物品转化为离散标记，实现多领域统一且可扩展的LLM推荐。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有物品分词方法跨领域泛化能力有限，无法统一处理多领域物品。
method: 提出UniTok框架，采用混合专家架构和系列码本将物品转换为离散标记，支持多领域统一分词。
result: 在多个领域数据集上取得优异性能，支持冷启动推荐。
conclusion: 统一物品分词是实现可扩展多领域LLM推荐的关键技术。
---

## Abstract
Large language model (LLM)-based recommender systems have achieved high-quality performance by bridging the discrepancy between the item space and the language space through item tokenization. However, existing item tokenization methods typically require training separate models for each item domain, limiting generalization. Moreover, the diverse distributions and semantics across item domains make it difficult to construct a unified tokenization that preserves domain-specific information. To address these challenges, we propose UniTok, a Unified item Tokenization framework that integrates our own mixture-of-experts (MoE) architecture with a series of codebooks to convert items into discrete tokens, enabling scalable tokenization while preserving semantic information across multiple item domains. Specifically, items from different domains are first projected into a unified latent space through a shared encoder. They are then routed to domain-specific experts to capture the unique semantics, while a shared expert, which is always active, encodes common knowledge transferable across domains. Additionally, to mitigate semantic imbalance across domains, we present a mutual information calibration mechanism, which guides the model towards retaining similar levels of semantic information for each domain. Comprehensive experiments on wide-ranging real-world datasets demonstrate that the proposed UniTok framework is (a) highly effective: achieving up to 51.89% improvements over strong benchmarks, (b) theoretically sound: showing the analytical validity of our architectural design and optimization; and (c) highly generalizable: demonstrating robust performance across diverse domains without requiring per-domain retraining, a capability not supported by existing baselines.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：现有的基于大型语言模型（LLM）的推荐系统通过物品分词（item tokenization）将物品空间与语言空间对齐，但在多领域场景中，现有方法通常需要为每个领域训练独立的词元器，导致训练开销大、泛化能力差；同时，不同领域的分布和语义差异使得构建统一且保留领域特定信息的词元化方案非常困难。
- **研究背景**：随着推荐任务跨领域（如电商、美妆、手机等）的需求增加，统一多领域模型成为趋势（如NLP和CV中的多域学习）。但现有分词方法（如TIGER、LC-Rec、LETTER）均针对单领域设计，无法直接迁移，且训练多套模型效率低下。UniTok旨在解决两大挑战：**C1**（训练冗余）和**C2**（语义对齐），首次提出面向多领域的统一物品分词框架。

## 2. 论文提出的方法论：核心思想、关键技术细节

### 2.1 核心思想
- **UniTok**：一个统一的物品分词框架，集成**混合专家（Mixture-of-Experts, MoE）**架构与系列码本（codebook），将物品转换为离散标记，在多领域间保持语义信息的同时实现可扩展的分词。

### 2.2 关键技术细节
1. **共享自编码器（Shared Autoencoder）**：所有领域的物品先通过一个共享编码器 \(f_\theta\) 映射到统一潜在空间，解码器 \(g_\phi\) 负责重建原始语义嵌入，损失函数采用L2重建损失 \(L_{\text{Rec}}\)。
2. **TokenMoE（MoE分词模块）**：
   - 路由函数 \(G(\cdot)\) 根据输入潜在嵌入 \(z_k^i\) 为每个物品分配权重到 **K个领域特定专家**，同时**始终激活一个共享专家**。
   - 领域特定专家学习各领域独特语义，共享专家编码跨领域通用知识。专家选择采用 top-N 稀疏激活（通常N=1或2）。
   - 公式：\(\hat{z}_k^i = \sum_k G_k E_k(z_k^i) + E_{\text{share}}(z_k^i)\)。
3. **码本标识符（Codebook-based Identifiers）**：每个专家内部采用残差量化（RQ），包含L级码本（每级T个码向量），将潜在嵌入量化为离散标记序列。
4. **互信息校准（MI Calibration）**：
   - 计算每个领域输入嵌入 \(X_k\) 与潜在嵌入 \(Z_k\) 之间的HSIC（Hilbert-Schmidt独立性准则）作为互信息代理。
   - 损失 \(L_{\text{MI}} = \text{Var}[\hat{I}(k)] - \beta \mathbb{E}[\hat{I}(k)]\)，惩罚跨领域MI方差，鼓励每个领域保留足够信息。
5. **总损失**：\(L_{\text{total}} = L_{\text{Rec}} + \lambda_{\text{RQ}} L_{\text{RQ}} + \lambda_{\text{MI}} L_{\text{MI}}\)。

### 2.3 理论分析
- **定理1**：UniTok产生的标记空间熵严格高于标准码本方法，增加了标记空间容量。
- **定理2**：UniTok的期望量化误差低于标准方法，专家专业化降低了领域特定误差。
- **定理3**：跨领域性能方差受MI方差上界控制，MI校准有助于稳定一致性。

## 3. 实验设计

- **数据集**：10个真实世界数据集，涵盖10个领域：Beauty, Cellphones, Grocery, Instruments, Office, Pet Supplies, Tools, Toys, Games, Yelp。每个物品包含标题、类别、特征等元数据。
- **基准方法**：
  - 传统协同过滤：MF, LightGCN, SASRec, Bert4Rec。
  - 基于物品分词+LLM的推荐：P5-TID, P5-SemID, TIGER, LC-Rec, LETTER。
- **评价指标**：Recall@M 和 NDCG@M（M=5,10），采用全排名协议。
- **对比设置**：所有基准方法为每个数据集训练独立分词器；UniTok在所有10个数据集上联合训练单一模型。另外进行了单统一训练（competitors共享参数）的对比，以及零样本跨域测试（Clothing, Health, Sports）。

## 4. 资源与算力

- **明确说明**：所有实验在 **两张 NVIDIA RTX 3090 GPU** 上完成。
- **未明确信息**：未报告具体训练时长（如epoch数、总时间），但强调其参数效率（9.63倍减少）。

## 5. 实验数量与充分性

### 实验数量
- **主实验（表1）**：在10个数据集上与9个基准方法对比NDCG@10，报告了每个结果。
- **效率分析（表2）**：比较参数量（代码本+自编码器+路由器）。
- **单统一训练对比（表3）**：在3个代表性数据集上（Beauty, Cellphones, Grocery）与TIGER/LC-Rec/LETTER对比。
- **零样本泛化（表4）**：在3个未见领域（Clothing, Health, Sports）上与同样对比。
- **消融实验（表5）**：在3个数据集上逐步移除TokenMoE、共享专家、MI校准，共4个变体。
- **其他**：论文还提及更全的结果由于页数限制未展示，但声称在所有数据集上均有统计显著性（p=0.0219）。

### 充分性与公平性
- **充分**：覆盖多域、零样本、消融、效率等多角度，实验设计较为完整。
- **公平**：基准方法均按原始论文复现或使用公开实现，UniTok与它们在同一设置下对比；单统一训练中控制了参数量；零样本设置避免了数据泄漏。但注意：表1中基准方法各自独立训练，而UniTok联合训练，这对方法本身是有利的，但论文在表3中额外做了公平比较（同样联合训练），证明了UniTok的优势不仅来自联合训练。

## 6. 论文的主要结论与发现

- **有效性**：UniTok在所有10个数据集上均优于最强基线，最高提升51.89%（NDCG@10, Tools数据集）。
- **效率**：与码本基线相比，可训练参数减少约9.63倍（从87.78M到9.11M）。
- **泛化性**：在零样本设置下，对未见领域（Clothing, Health, Sports）依然优于需要重训练的基线，提升12%~17%。
- **组件贡献**：TokenMoE和MI校准均对性能有显著正向作用，消融实验验证了各模块必要性。
- **理论验证**：理论分析与实验现象一致（高熵、低量化误差、低MI方差）。

## 7. 优点

- **创新性**：首次将MoE应用于多领域物品分词，提出领域特定专家+共享专家的结构，解决了多域专业化与知识共享的矛盾。
- **简洁性**：端到端训练，无需领域标签或额外用户数据，仅依靠物品元数据。
- **理论支撑**：提供三个定理证明设计合理性，结合实验验证。
- **全面实验**：涵盖10个领域（丰富多样）、零样本、消融、效率分析，结果统计显著。
- **实用性**：支持冷启动（无需重训练即可处理新域），对工业部署友好。

## 8. 不足与局限

- **用户协同信号缺失**：方法只依赖物品内容，未利用用户-物品交互数据，可能丢失协同过滤信号，在严重冷启动或个人化场景中受限（作者在任务定义中明确说明）。
- **计算资源未充分报告**：未提供训练时间（epoch数、总GPU小时），难以评估训练开销。
- **代码未开源验证**：虽提供GitHub链接，但论文投稿时可能未公开，复现性待确认。
- **零样本测试领域少**：仅3个领域，不足以全面评估泛化力，且它们是否与训练集领域有重叠未知（如Clothing可能语义相似性高）。
- **超参数敏感性**：MI损失中的β、λ_RQ、λ_MI的调参过程未详细讨论，可能存在对特定领域的依赖。
- **基线方法在零样本下的不公平性**：基线在零样本时需要重新训练，而UniTok直接使用预训练模型，对比上有先天优势，虽然论文在表3中做了公平比较，但零样本实验的基线条件描述不够清晰（是否确保基线没有使用目标域数据？）。

（完）
