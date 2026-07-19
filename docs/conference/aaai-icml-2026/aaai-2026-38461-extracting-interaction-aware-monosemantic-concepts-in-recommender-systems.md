---
title: Extracting Interaction-Aware Monosemantic Concepts in Recommender Systems
title_zh: 提取推荐系统中交互感知的单语义概念
authors: "Dor Arviv, Yehonatan Elisha, Oren Barkan, Noam Koenigstein"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38461/42423"
tags: ["query:rec-sys"]
score: 9.0
evidence: 可解释推荐系统，稀疏自编码器
tldr: 该论文提出一种从推荐系统用户和物品嵌入中提取单语义神经元的方法。通过稀疏自编码器和预测感知训练目标，学习到的神经元能捕捉类别、流行度、时间趋势等概念，同时保持用户-物品交互结构，显著提升了推荐系统的可解释性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 推荐系统嵌入缺乏可解释性，现有概念提取方法未考虑用户-物品交互。
method: 采用稀疏自编码器，并引入预测感知目标，通过冻结推荐模型反向传播来对齐潜在结构与预测。
result: 提取的神经元能捕捉类别、流行度等概念，并支持解释性应用。
conclusion: 该方法使推荐系统嵌入更可解释，有助于模型调试和用户信任。
---

## Abstract
We present a method for extracting monosemantic neurons, defined as latent dimensions that align with coherent and interpretable concepts, from user and item embeddings in recommender systems. Our approach employs a Sparse Autoencoder (SAE) to reveal semantic structure within pretrained representations. In contrast to work on language models, monosemanticity in recommendation must preserve the interactions between separate user and item embeddings. To achieve this, we introduce a prediction aware training objective that backpropagates through a frozen recommender and aligns the learned latent structure with the model’s user-item affinity predictions. The resulting neurons capture properties such as genre, popularity, and temporal trends, and support post hoc control operations including targeted filtering and content promotion without modifying the base model. Our method generalizes across different recommendation models and datasets, providing a practical tool for interpretable and controllable personalization.

---

## 论文详细总结（自动生成）

# 论文总结：Extracting Interaction-Aware Monosemantic Concepts in Recommender Systems

## 1. 核心问题与整体含义（研究动机和背景）

- **问题**：推荐系统中的用户和物品嵌入（embeddings）缺乏可解释性，限制了用户信任、调试和公平性审计。
- **背景**：近年来，基于稀疏自编码器（Sparse Autoencoder, SAE）的方法已成功从大语言模型（LLMs）中提取“单语义神经元”（monosemantic neurons），即与人类可理解概念对齐的潜在维度。然而，这些方法直接应用于推荐系统时失效，因为推荐系统的核心是用户-物品嵌入之间的交互（如点积或神经网络），而非单一单向传播。
- **目标**：设计一种专门适用于推荐系统的SAE框架，保持用户-物品交互语义的同时，提取可解释的概念神经元。

## 2. 方法论：核心思想、关键技术细节

- **核心思想**：在传统SAE的几何重建损失之外，引入“预测感知重建损失”（prediction-aware reconstruction loss），通过冻结的推荐模型反向传播，使重建后的嵌入能保持与原始模型一致的预测亲和度（affinity scores）。
- **模型架构**（基于两塔推荐模型）：用户和物品嵌入分别通过一个SAE（线性编码器+ReLU+线性解码器，字典大小m）输出稀疏潜在向量z，并重建为˜e。SAE采用Matryoshka结构（多层嵌套字典，层次化特殊化）。
- **损失函数**：
  - 重建损失 L_rec = α L_emb + β L_pred
    - L_emb = ‖e - ˜e‖₂²（嵌入级几何损失）
    - L_pred = (1/|S|) Σ_{u,i} (ŷ_{u,i} - ỹ_{u,i})²（预测级损失，通过冻结的推荐模型计算原始与重建亲和度的MSE）——这是关键创新。
  - 稀疏损失 L_sparse = λ₁·L₁ + λ₂·KL散度（鼓励每个神经元低激活率，避免死神经元问题，不使用LLM中常用的Top-K稀疏）。
- **训练流程**：冻结推荐模型参数，仅训练SAE。每次迭代随机采样用户-物品对，计算总损失，梯度反向传播通过冻结的推荐模型（L_pred路径），更新SAE参数。

## 3. 实验设计

- **数据集**：
  - MovieLens 1M（ML1M）：电影领域，二值化隐式反馈。
  - Last.FM：音乐领域，隐式反馈（聚合为艺术家级别）。
- **推荐模型**（基准模型）：
  - Matrix Factorization (MF)：嵌入维度d=20（ML1M）/ 100（Last.FM），内积计算亲和度。
  - Neural Collaborative Filtering (NCF)：两层MLP（64-32-16）+ sigmoid。
- **评估方法**：
  - 定性分析：可视化神经元激活模式（如类别对齐、时间趋势）。
  - 定量分析：语义纯度（semantic purity）= 神经元top-K激活物品中匹配给定标签的比例（K=10/20/50）。标签通过GPT-4.5自动标注。
  - 消融实验：调节预测损失权重β，观察推荐保真度（Rank-Biased Overlap, Kendall Tau）和单语义性得分的变化。
- **对比方法**：主要与自身消融（β=0，即无预测损失）对比，未与现有可解释方法直接对比（因无专门针对推荐系统的SAE方法）。

## 4. 资源与算力

- **未明确说明**：文中未提及使用的GPU型号、数量、训练时长等算力信息。仅给出了超参数（学习率、批次大小、训练轮数），无法评估计算开销。

## 5. 实验数量与充分性

- **实验组数**：总共包含：
  - 2个数据集 × 2个推荐模型 = 4个主要条件。
  - 每个条件下定性分析（图2-3）、定量纯度表（Table 1）、消融实验（图4）、Matryoshka层次分析、应用案例（图5）。
- **充分性评价**：
  - **优点**：覆盖了不同领域（电影、音乐）和不同复杂度模型（MF简单、NCF更复杂）；消融实验清晰展示了预测损失的重要性；提供了定性+定量双重证据。
  - **不足**：仅两个数据集，规模较小（尤其是Last.FM经聚合处理）；未在更大规模（如百万用户）或工业级数据集上验证；未与其他现有可解释性方法（如注意力权重、原型方法）进行定量对比；应用案例仅示例性展示，缺乏系统评估。

## 6. 主要结论与发现

- 提出的SAE框架成功从推荐嵌入中提取出对齐类别（如Children、Horror、Sci-Fi）、流行度、时间趋势（如90年代喜剧）的单语义神经元。
- 语义纯度较高：例如MF模型上Comedy和Horror在top-10中达到100%纯度；Last.FM上Metal、Electronic等也接近完美。
- 预测感知损失（L_pred）对保持推荐行为至关重要：无该损失时推荐保真度显著下降；适当权重（β）可在保真度与可解释性之间取得平衡。
- Matryoshka结构在Last.FM上展现层次化：早期神经元对应广泛受众（如主流电子），后期对应狭小众领域（如北欧死亡金属）。
- 神经元级干预（如调整Bob Dylan的特定神经元激活）可无参数地促进特定物品到不相关受众，展示了可控性潜力。

## 7. 优点

- **创新性强**：首次将单语义性提取从LLM适配到推荐系统，解决交互语义保持的关键问题。
- **无监督且通用**：无需外部元数据或标签，仅从行为数据中自发发现概念；方法适用于MF和NCF，且可扩展至其他两塔架构。
- **可解释+可控**：提取的神经元不仅用于理解，还可用于后处理干预（推广、过滤、多样化），无需重新训练模型。
- **严谨消融**：通过β调控证明预测损失的必要性，并揭示了保真度与可解释性的权衡，使结论可靠。

## 8. 不足与局限

- **实验范围有限**：仅两个数据集（其中Last.FM被聚合为艺术家级别，可能失去细粒度）；仅两种推荐模型，未测试图神经网络（如LightGCN）或深度双塔模型。
- **评估依赖自动标注**：使用GPT-4.5生成神经元标签，可能引入噪声或概念模糊（如“Crime”标签纯度下降可能因标注不完整）。
- **缺乏与现有可解释方法的对比**：未与基于特征归因、原型或注意力的可解释方法进行定量比较，无法证明语义纯度上的优势。
- **计算资源未报告**：难以评估方法在大规模部署中的可行性（SAE训练需冻结推荐模型并反向传播，可能较慢）。
- **干预效果未量化**：应用案例仅展示单个实例，未评估对推荐多样性、准确性、用户满意度等指标的影响。
- **Matryoshka效果不稳定**：在MovieLens上效果有限，可能因语义空间较平坦，方法适用性可能依赖领域特性。

（完）
