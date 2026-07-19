---
title: Capturing Dynamic User Interests Under Modality Imbalance for Multimodal Sequential Recommendation
title_zh: 模态不平衡下捕获动态用户兴趣的多模态序列推荐
authors: "Zilong Li, Jia Zhu, Chenglei Huang, Zhangze Chen, Hanghui Guo, Guoqing Ma, Jianxia Ling"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38544/42506"
tags: ["query:rec-sys"]
score: 10.0
evidence: 多模态序列推荐中动态用户兴趣和模态不平衡
tldr: 多模态序列推荐面临模态不平衡和用户兴趣动态变化挑战。本文提出DuAF-MAT框架，通过双感知自适应融合模块动态校准模态贡献，并使用时序建模捕获兴趣演化，有效提升多模态序列推荐准确性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 多模态序列推荐中现有融合策略难以处理模态不平衡和用户兴趣动态变化。
method: 提出DuAF-MAT框架，包含双感知自适应融合模块和模态感知Transformer。
result: 在多个多模态序列推荐数据集上取得最佳效果。
conclusion: 动态模态融合与时序建模相结合能有效提升多模态序列推荐性能。
---

## Abstract
Multimodal sequential recommender systems leverage diverse modal inputs to enhance the accuracy and relevance of personalized recommendations. However, existing fusion strategies often struggle to capture intricate cross-modal interactions, especially under the evolving dynamics of user intent. Moreover, they frequently neglect modality imbalance issues, leading to suboptimal utilization of multimodal information. To address these challenges, we propose DuAF-MAT, a novel framework for robust multimodal sequential recommendation. Our approach consists of three key components: (1) a Dual-Aware Adaptive Fusion (DuAF) module dynamically calibrates modality contributions by jointly modeling user preferences and temporal information, enabling the extraction of multimodal features aligned with evolving user interests; (2) by integrating Modality Adversarial Training with the Mixture-of-Experts paradigm, MAT-MoE employs an ensemble of expert generators to dynamically reconstruct missing modality representations, effectively mitigating modality imbalance challenges; (3) to address the inherent sparsity of sequential behavior data, we propose a Multi-Supervised Contrastive Learning strategy that integrates cross-modal alignment and virtual sequence augmentation. This approach enhances user interest modeling by leveraging diverse learning signals, resulting in improved model robustness and generalization capability. Extensive experiments on four public datasets demonstrate that DuAF-MAT significantly outperforms state-of-the-art baselines.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

多模态序列推荐系统通过融合文本、图像等多种模态信息来提升推荐的准确性和个性化。然而，现有融合策略面临三个关键挑战：

- **模态贡献动态性**：用户的兴趣随时间动态变化，不同场景下对不同模态的侧重点不同（例如，健身爱好者饮食控制期更关注文本营养信息，放纵期更关注视觉外观），现有静态或仅依赖用户偏好的融合方法无法捕获这种演变。
- **模态不平衡**：实际数据中，许多物品缺失某类模态信息（如图片或描述缺失），导致多模态信息利用不充分。
- **序列数据稀疏**：用户交互序列通常较短（尤其是冷启动用户），限制了模型对用户兴趣的有效建模。

针对上述问题，论文提出 **DuAF-MAT** 框架，旨在鲁棒地处理模态不平衡、动态用户兴趣和数据稀疏问题，提升多模态序列推荐的性能。

## 2. 方法论：核心思想、关键技术细节

### 2.1 整体架构
DuAF-MAT 包含三个核心模块：

1. **双感知自适应融合模块（DuAF）**  
2. **基于混合专家的模态对抗训练（MAT-MoE）**  
3. **多监督对比学习（MSCL）**，包括跨模态序列对比学习（CSCL）和虚拟序列对比学习（VSCL）。

### 2.2 详细技术
- **多模态物品表示**：每个物品包含 ID、文本（BERT提取）、视觉（ViT提取）和结构特征，通过线性投影到共同嵌入空间，并加入位置编码。
- **DuAF 模块**：  
  - 计算时间间隔嵌入和可学习的时间戳嵌入，两者拼接得到最终时间嵌入 \(e^{time}\)。  
  - 基于用户动态偏好 \(s_u\) 和物品各模态嵌入 \(e^m_i\) 及时间嵌入，通过内积和 softmax 计算各模态自适应权重 \(\omega^m_i(u,t)\)。  
  - 加权融合得到物品表示 \(h_i\)。  
  - 公式核心：\(\omega^m_i(u,t) = \frac{\exp(\langle s_u, \tanh(e^m_i + e^{time}_i) \rangle)}{\sum_{n \in M} \exp(\langle s_u, \tanh(e^n_i + e^{time}_i) \rangle)}\)。
- **MAT-MoE 模块**：  
  - 借鉴 WGAN 思想，生成器 \(G\) 由多个专家（两层 FFN）组成，基于物品类别嵌入计算路由权重，生成缺失模态（如视觉）的嵌入 \(e^{syn}\)。  
  - 判别器 \(D\) 判断文本-视觉嵌入对是否真实（正样本为真实对，负样本为生成对），采用二元交叉熵损失和梯度惩罚。  
  - 生成器和判别器交替优化，使生成器学习真实分布。
- **多监督对比学习**：  
  - **CSCL**：对同一用户的文本序列和图像序列（含生成嵌入）分别编码，将两个序列表示作为正样本，批次内其他序列作为负样本，进行对比学习。  
  - **VSCL**：在用户序列中通过相似物品插入（基于双模态语义相似度）构造虚拟序列，对原始序列和虚拟序列进行对比学习。  
  - 总损失 = 主任务交叉熵损失 + \(\lambda_1 L_{CSCL} + \lambda_2 L_{VSCL}\)。

## 3. 实验设计

- **数据集**：Amazon Review 标准基准，包含四个领域：Toys & Games、Video Games、Beauty、Home & Kitchen。每条用户交互按时间排序构建行为序列。
- **基准方法**：  
  - 纯ID方法：GRU4Rec、SASRec、LRURec  
  - 文本方法：FDSA、UniSRec、TedRec  
  - 多模态方法：NOVA、DIF-SR、MISSRec、M3SRec、IISAN  
- **评估指标**：NDCG@5/10、MRR@5/10，使用留一法评估（全物品集无采样）。
- **超参数调优**：学习率 {0.0003,0.001,0.003,0.01}，嵌入维度 {64,128,300}，专家数 {2,4,6,8,10}，温度 \(\tau\) 在 0.1~1.0，\(\lambda_{gp}\) 在 1e-2~1e-5，对比损失权重 \(\lambda_1,\lambda_2\) 在 0.2~1.0。

## 4. 资源与算力

论文未明确说明训练所使用的具体 GPU 型号、数量及训练时长。仅提及实验在 Linux 服务器上运行，使用 NVIDIA GeForce RTX 3090 显卡。未提供训练轮数或总时间。

## 5. 实验数量与充分性

- **总体比较**：在4个数据集上比较了11种基线方法，报告了8个指标（NDCG@5/10、MRR@5/10），共4×11×8 = 352个结果（表格中仅展示了部分，但趋势明确）。
- **消融实验**：在全部四个数据集上进行了四个变体的消融（去掉DuAF、去掉MAT-MoE、去掉CSCL、去掉VSCL），并给出可视化结果（图4）。
- **超参数研究**：针对专家数、对比损失权重、模态丢弃率等关键参数在两个数据集上进行了系统实验（图3）。
- **充分性评估**：实验覆盖了多种模态缺失场景（通过模态丢弃模拟），并分析了性能增益来源。比较方法涵盖主流 ID、文本、多模态三类，基准具有代表性。实验设计公平（统一框架 RecBole 实现，统一优化器和调参）。结论可信。

## 6. 主要结论与发现

- DuAF-MAT 在所有数据集和指标上均显著优于所有基线，最高提升达 16.11%（NDCG@5 在 Toys 上）。  
- 与最强的多模态方法相比，性能提升在 1.07% 到 16.11% 之间；与最强文本模型相比，提升达 12.91%~52.83%。  
- 各组件均有效：DuAF 最关键（动态融合>平均融合），MAT-MoE 提升鲁棒性，CSCL 贡献最大（跨模态对齐），VSCL 进一步改善稀疏序列。  
- 模态丢弃实验中，DuAF-MAT 在丢弃率 0.6 时表现最优，且始终优于随机初始化和零初始化，展示了对模态缺失的鲁棒性。

## 7. 优点

- **创新性**：首次将时间感知与用户偏好联合用于模态权重动态校准（DuAF），并将对抗生成与 MoE 结合用于缺失模态重建（MAT-MoE），思路新颖。  
- **系统性**：同时解决了模态不平衡、兴趣动态变化和数据稀疏三大挑战，且各模块设计合理、互补。  
- **实验全面**：多数据集、多种基线、详细消融和超参数分析，验证了方法有效性和鲁棒性。  
- **可复现性**：基于开源框架 RecBole 实现，超参数和代码可用。

## 8. 不足与局限

- **算力资源未明确**：缺少训练时间、GPU 数量等细节，不利于实际部署评估。  
- **模态种类有限**：仅考虑文本和视觉两种模态（加 ID 和结构），未探索音频、视频等更多模态。  
- **假设条件**：需要预提取的文本/视觉特征（BERT、ViT），且假设所有物品至少有一种模态可用，极端缺失场景（如无文本）未讨论。  
- **评价指标单一**：仅使用 NDCG 和 MRR，未使用 Recall 等其他常用指标，对比不够全面。  
- **冷启动用户分析**：虽然提到冷启动问题，但未专门对冷启动用户进行性能分析（如交互次数少的用户子集）。  
- **对比方法时效性**：基线中包含一些较老方法（如 GRU4Rec, SASRec），但最新 SOTA（如 2024-2025 方法）未全部覆盖。  
- **消融实验的可视化**：图4 仅展示了 Beauty 和 Toys 两个数据集的结果，未在另外两个数据上验证（但文中声称全部数据集均做了消融，可能只是未展示图）。

（完）
