---
title: Multi-Aspect Cross-modal Quantization for Generative Recommendation
title_zh: 面向生成式推荐的多视角跨模态量化方法
authors: "Fuwei Zhang, Xiaoyu Liu, Dongbo Xi, Jishen Yin, Huan Chen, Peng Yan, Fuzhen Zhuang, Zhao Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38664/42626"
tags: ["query:rec-sys"]
score: 8.0
evidence: 生成式推荐，跨模态量化
tldr: 生成式推荐依赖高质量的语义标识符，但现有方法难以充分利用多模态信息。本文提出多视角跨模态量化方法，通过层次化组织不同模态的交互，生成低冲突、结构化的物品标识符。实验证明该方法提升了生成式推荐的准确性和效率，为利用多模态数据提供了新思路。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 生成式推荐需要高质量的语义标识符，但现有方法缺乏对多模态信息的有效利用。
method: 提出多视角跨模态量化方法，通过层次化离散化多模态特征，生成结构化的物品标识符。
result: 在多个推荐数据集上，该方法在推荐准确性和效率方面均取得显著提升。
conclusion: 利用多视角跨模态量化可有效提升生成式推荐中物品标识符的质量。
---

## Abstract
Generative Recommendation (GR) has emerged as a new paradigm in recommender systems. This approach relies on quantized representations to discretize item features, modeling users’ historical interactions as sequences of discrete tokens. Based on these tokenized sequences, GR predicts the next item by employing next-token prediction methods. The challenges of GR lie in constructing high-quality semantic identifiers (IDs) that are hierarchically organized, minimally conflicting, and conducive to effective generative model training. However, current approaches remain limited in their ability to harness multimodal information and to capture the deep and intricate interactions among diverse modalities, both of which are essential for learning high-quality semantic IDs and for effectively training GR models.  To address this, we propose Multi-Aspect Cross-modal quantization for generative Recommendation (MACRec), which introduces multimodal information and incorporates it into both semantic ID learning and generative model training from different aspects. Specifically, we first introduce cross-modal quantization during the ID learning process, which effectively reduces conflict rates and thus improves codebook usability through the complementary integration of multimodal information. In addition, to further enhance the generative ability of our GR model, we incorporate multi-aspect cross-modal alignments, including the implicit and explicit alignments. Finally, we conduct extensive experiments on three well-known recommendation datasets to demonstrate the effectiveness of our proposed method.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究背景**：生成式推荐（Generative Recommendation, GR）是推荐系统的新范式，它将推荐任务转化为“下一个token预测”问题，依赖对物品特征的离散量化生成层次化、低冲突的语义标识符（semantic IDs），再利用大规模语言模型进行序列建模与预测。
- **现有局限**：当前GR方法主要使用单模态（如文本）嵌入进行量化，导致：
  - 单一模态语义区分性有限，例如同品牌不同产品因文本特征相似而难以区分。
  - RQ-VAE深层残差量化中易出现语义损失，使得token分配近乎随机，削弱层次语义。
  - 无法充分利用多模态信息（如文本+图像）的互补优势。
- **核心动机**：引入多模态信息（文本+图像）到GR的两个关键阶段——语义ID学习和生成模型训练，以提升ID质量和模型生成能力。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- **多视角跨模态量化**（MACRec）分为两阶段：
  1. **跨模态物品量化**：通过跨模态对比学习与重建对齐，生成层次化、低冲突的语义ID。
  2. **生成式推荐的多视角对齐**：在GR模型训练中引入隐式对齐（潜在空间对比）和显式对齐（任务级跨模态生成），促进多模态特征融合。

### 关键技术细节（公式/流程文字描述）

#### (1) 跨模态物品量化（Cross-modal Item Quantization）

- **双模态伪标签生成**：分别对文本嵌入（来自LLaMA）和视觉嵌入（来自ViT）进行K-means聚类（K=512），得到文本伪标签和视觉伪标签，用于构造对比学习的正样本对。
- **跨模态对比量化**：在RQ-VAE的每一层（共4层，码本大小256），对文本和视觉的残差表示独立量化，但引入跨模态对比损失：
  - 利用视觉伪标签增强文本残差，利用文本伪标签增强视觉残差。
  - InfoNCE损失公式：\( L_{con}^{l} = L_{t\to v}^{l} + L_{v\to t}^{l} \)，其中正样本为同一伪标签类别的表示。
- **跨模态重建对齐**：对量化后的表示 \( \hat{z}_t, \hat{z}_v \) 进行双向对比对齐损失 \( L_{align} \)，促进同一物品不同模态量化表示的一致性。
- **总损失**：\( L_{ID} = L_{RQ-VAE} + \lambda_{con} \sum_{l} L_{con}^{l} + \lambda_{align} L_{align} \)，其中 \( L_{RQ-VAE} \) 包含重建损失和码本学习损失。

#### (2) 生成式推荐的多视角对齐

- **隐式对齐**：将文本语义ID和视觉语义ID分别经GR模型编码器（T5）得到潜在表示，通过对比损失 \( L_{implicit} \) 对齐同一物品的不同模态表示。
- **显式对齐**：
  - 物品级：用文本ID生成视觉ID，反之亦然。
  - 序列级：用历史文本ID序列预测下一物品的视觉ID，反之亦然。
- **训练目标**：\( L_{rec} = -\sum \log P(y_t|y_{<t}, x) + \lambda_{implicit} L_{implicit} \)
- **推理阶段**：采用约束束搜索生成多模态候选，对文本和视觉两路结果进行分数平均集成。

## 3. 实验设计

- **数据集**：三个Amazon产品评论数据集：Musical Instruments、Arts Crafts and Sewing、Video Games。统计信息如下：
  - Instruments: 17,112用户, 6,250物品, 136,226交互, 稀疏度99.87%, 平均序列长7.96
  - Arts: 22,171用户, 9,416物品, 174,079交互, 稀疏度99.92%, 平均序列长7.85
  - Games: 42,259用户, 13,839物品, 373,514交互, 稀疏度99.94%, 平均序列长8.84
- **基准方法（Benchmark）**：对比了9种方法：
  - 传统序列推荐：BERT4Rec, SASRec, FDSA, S3-Rec
  - 多模态序列推荐：MISSRec, P5, VIP5
  - 生成式推荐：TIGER, MQL4GRec（多模态生成式SOTA）
- **评价指标**：HR@1, HR@5, HR@10, NDCG@5, NDCG@10；留一法（leave-one-out）全排序评估。

## 4. 资源与算力

- 论文中**未明确说明**使用的GPU型号、数量及训练时长。仅在实现细节中提及使用AdamW优化器、batch size=1024、学习率0.001，并在5个随机种子下取平均结果。未提供具体硬件配置或训练时间。

## 5. 实验数量与充分性

- **实验组数**：
  - 主实验（Table 2）：3个数据集 × 10种方法 × 5个指标 → 全面对比。
  - 消融实验（Table 3）：在3个数据集上分别去除4个模块（\( L_{con} \), \( L_{align} \), \( L_{implicit} \), 显式对齐），共4+1组×3数据集。
  - 碰撞率分析（Table 4）：MQL4GRec vs MACRec，文本和图像各3数据集。
  - 码本分配分布可视化（Figure 4）。
  - 超参数分析（Figure 3）：码本大小、ID长度、对比损失起始层、各损失权重等。
- **充分性评价**：实验设计较为充分，覆盖了主流基线、多模态消融、碰撞率、码本利用率和参数敏感性。但缺少对更大规模数据集或跨域迁移的实验验证，且未报告方差或显著性检验细节（仅注明p<0.05）。

## 6. 主要结论与发现

- MACRec在所有三个数据集上均**显著优于**所有基线，包括多模态生成式SOTA MQL4GRec。
- 跨模态对比量化（\( L_{con} \)）是贡献最大的模块，去除后性能下降最明显。
- 隐式对齐和显式对齐均有提升作用，两者结合效果更好。
- MACRec能够有效降低物品ID碰撞率（Table 4），提升码本利用率（Figure 4中分布更均匀）。
- 超参数分析表明：码本大小256、ID长度4、对比损失从第三层开始启用、各损失权重适中时效果最佳。

## 7. 优点

- **创新性**：首次在GR的量化阶段引入跨模态对比学习，并用伪标签约束残差表示，解决单模态语义损失问题。
- **完整性**：在两个关键阶段（ID学习和生成训练）都设计了跨模态交互，形成统一框架。
- **实验扎实**：在多个数据集、多种基线、多角度消融下验证了有效性，且包含码本利用率和碰撞率分析。
- **可复现性**：提供了开源代码（GitHub链接），增加了结果可信度。

## 8. 不足与局限

- **算力资源未报告**：缺乏GPU型号、数量、训练时长等信息，不利于复现成本评估。
- **数据集规模偏小**：三个Amazon子类数据集用户和物品数量有限（最大约42k用户/13k物品），在更大规模、更稀疏场景下的表现未知。
- **泛化性待验证**：仅使用Amazon数据，未在工业级数据集或不同领域（如视频、新闻）上测试。
- **对比基准的公平性**：为公平比较，未使用MQL4GRec的百万级跨域预训练，但MQL4GRec原文中预训练可能贡献部分收益。作者虽注明“did not utilize pre-training”，但未完全消除预训练差异带来的潜在偏差。
- **消融实验的显著性**：仅标注了主实验结果p<0.05，消融结果和碰撞率分析未提供统计检验。
- **多模态对齐计算开销**：显式对齐增加了多个生成任务，可能带来训练时间与内存开销，文中未讨论效率对比。

（完）
