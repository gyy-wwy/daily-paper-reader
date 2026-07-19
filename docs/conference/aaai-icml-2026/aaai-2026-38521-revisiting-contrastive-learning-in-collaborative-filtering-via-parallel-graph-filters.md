---
title: Revisiting Contrastive Learning in Collaborative Filtering via Parallel Graph Filters
title_zh: 通过并行图滤波器重新审视协同过滤中的对比学习
authors: "Fang Kai, Yu Zhang, Kaibin Wang, Lei Sang, Yiwen Zhang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38521/42483"
tags: ["query:rec-sys"]
score: 8.0
evidence: 推荐系统中用于协同过滤的深度学习图对比学习
tldr: 本文重新审视了协同过滤中的图对比学习方法，提出并行图滤波器来解决多层图卷积导致的过平滑和负样本梯度饱和问题。该方法通过并行滤波增强表示区分度，在多个推荐数据集上显著提升了推荐精度，为图对比学习在推荐系统中的应用提供了新思路。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有图对比学习方法存在过平滑和梯度饱和问题，限制了推荐性能。
method: 提出并行图滤波器，分别处理高低频信号，避免过平滑并缓解梯度饱和。
result: 在多个基准数据集上，该方法在推荐准确率上优于现有图对比学习模型。
conclusion: 并行图滤波器有效提升了图对比学习在推荐系统中的表现。
---

## Abstract
Graph Contrastive Learning (GCL) has recently emerged as a powerful paradigm for modeling user–item interactions and learning high-quality representations in recommender systems. While existing GCL-based methods benefit from data augmentation and sampling strategies, they often overlook the inherent limitations of the contrastive objectives: 1) Stacking multiple Graph Convolutional Network layers to capture high-order information often causes the over-smoothing phenomenon, where node representations become overly similar. 2) Structurally similar negative sample pairs may exhibit high cosine similarity, causing gradient saturation during representation optimization. To address the above challenges, we revisit matrix factorization in recommendation models and uncover its implicit connection to a parallel graph filter bank. This perspective reveals how overly aggressive low-pass or high-pass filtering distorts feature distributions, contributing to gradient saturation. Building on this insight, we propose Light Cosine Similarity Collaborative Filtering (LightCSCF), a margin-constrained method that improves gradient optimization in contrastive learning by focusing on structurally hard examples, alleviating both gradient saturation and boundary over-smoothing. Extensive experiments on three real-world datasets demonstrate that LightCSCF consistently outperforms state-of-the-art baselines in recommendation accuracy and robustness to data sparsity.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究背景**：图对比学习（GCL）在推荐系统中广泛应用于建模用户-物品交互，通过数据增强和对比损失学习高质量表征。但现有方法存在两个固有缺陷：
  1. **过平滑（Over-smoothing）**：堆叠多层图卷积网络（GCN）以捕获高阶信息时，节点表示趋向一致，失去判别力。
  2. **梯度饱和（Gradient Saturation）**：结构相似的负样本对与正样本具有高余弦相似度，导致对比损失中的梯度消失或饱和，优化信号减弱。
- **核心问题**：如何在不增加额外图增强或复杂模型的前提下，同时缓解过平滑和梯度饱和，提升推荐精度和鲁棒性。
- **整体含义**：该论文从谱图理论出发，揭示矩阵分解（MF）与并行图滤波器组的隐含联系，并提出一种基于边界约束的余弦相似度方法，直接优化对比学习中的梯度信号。

## 2. 方法论：核心思想、关键技术细节

- **核心思想**：将对比学习中的梯度更新过程解释为**并行图滤波器组**（identity、低通滤波、高通滤波）的综合作用，从而利用低通滤波促进平滑、高通滤波增强局部差异。在此基础上，设计一种**边界约束的余弦相似度函数**，对高相似度的正负样本施加不同强度的梯度，缓解饱和。
- **关键技术细节**：
  - **并行图滤波器视角**：低秩投影矩阵 M 相当于低通滤波器，其补矩阵 N = I - M 为高通滤波器。对比学习中的吸引力梯度对应低通更新（μME），排斥力梯度对应高通更新（-νNE），整体更新规则为：E(t+1) = I + μM - νN。
  - **边界约束相似度函数**：
    \[
    \text{sim}_m = \exp\left(\frac{\cos(\mathbf{e}_u, \mathbf{e}_i)}{\tau}\right) + \exp\left(\frac{\max(\cos(\mathbf{e}_u, \mathbf{e}_i) - m, 0)}{\tau}\right)
    \]
    第一项为原始余弦相似度的指数形式（低通主导），第二项为高通过滤项：当余弦相似度超过阈值 m 时激活，对硬负样本施加更强梯度，避免饱和。
  - **损失函数**：基于上述相似度的对比损失：
    \[
    L_{\text{LightCSCF}} = -\log \frac{\text{sim}_m(\mathbf{e}_u, \mathbf{e}_i)}{\sum_{j \in B_u} \text{sim}_m(\mathbf{e}_u, \mathbf{e}_j)}
    \]
    其中 \(B_u\) 为批次内隐含负样本。
  - **多任务联合训练**：总损失 = BPR 主损失 + λ₁ L_LightCSCF + λ₂ 正则项。
- **公式与算法流程**（文字描述）：
  1. 用户-物品交互图 → 初始化嵌入矩阵 E(0)。
  2. 通过并行图滤波器（等价于 MF）生成更新梯度，但实际实现中不显式构建滤波器，而是通过损失函数中的相似度计算间接实现。
  3. 计算边界约束相似度 sim_m，用于对比损失。
  4. 联合优化 BPR 损失和 LightCSCF 损失，更新嵌入。
  5. 训练过程中不需额外图增强或复杂抽样，仅修改损失函数。

## 3. 实验设计

- **数据集**：三个真实世界稀疏数据集。
  | 数据集 | #用户 | #物品 | #交互 | 密度 |
  |--------|-------|-------|-------|------|
  | Amazon-book | 52,643 | 91,599 | 2,984,108 | 0.06% |
  | Douban-book | 13,024 | 22,347 | 792,062 | 0.27% |
  | Tmall | 47,939 | 41,390 | 2,619,389 | 0.13% |
- **场景**：隐式反馈推荐（用户-物品交互矩阵），80%训练，20%测试。
- **Benchmark 与对比方法**：共14种基线，分为5类：
  - 传统/BPR/对齐方法：BPR-MF, DirectAU, LightGCN
  - 生成式方法：MultVAE, CVGA, DiffRec
  - 对比学习方法：SGL, SimGCL, LightGCL, BIGCF, MixRec, LightCCF
  - 关系对比学习（RCL）方法：SCCF, RecDCL
- **评估指标**：Recall@K, NDCG@K（K=10,20）。

## 4. 资源与算力

- **说明**：论文全文**未明确提及**使用的 GPU 型号、数量、训练时长、显存等计算资源信息。仅提到统一框架实现、Xavier初始化、embedding大小64（除RecDCL为2048）、训练最多1000 epoch、早停20轮。其他算力细节缺失。

## 5. 实验数量与充分性

- **实验组数**：至少包含以下实验：
  1. **主实验**（表2）：在三个数据集上对比全部14种基线，报告6个指标，共54个数值。
  2. **嵌入维度分析**（图2）：在三个数据集上测试不同维度（64-1024）的表现。
  3. **鲁棒性分析**（图3）：在两个数据集上注入不同比例噪声，对比 LightCSCF 与 SCCF 的 NDCG@20 变化。
  4. **t-SNE 可视化**（图4）：在 Tmall 上随机采样5000个用户/物品嵌入并可视化。
  5. **稀疏性分析**（图5）：将用户分为四个活跃度等级（frequent / normal / occasional / sparse），比较 NDCG@20。
  6. **超参数分析**（图6）：网格搜索 m（0.0-1.1）和 τ（0.2-0.9等）对 Recall@20 的影响。
- **充分性与公平性**：
  - 覆盖主流基线类型，使用统一框架实现，fair comparison。
  - 嵌入大小统一为64（除 RecDCL 因架构要求2048），可能对 RecDCL 不公平（容量更大），但论文仍取得了明显优势。
  - **缺少消融实验**：未单独验证边界约束项（第二项）或并行滤波器假设的贡献，未分析各损失分量（BPR vs CL）的单独效果。
  - 缺少对计算效率、模型大小的比较。

## 6. 主要结论与发现

- LightCSCF 在所有三个数据集上均全面优于所有基线。在 Recall@20 上，相对第二好方法（LightCCF）提升 1.93%~4.39%；相对 RCL 方法（SCCF）提升 10.99%~13.95%。
- 边界约束方法能够有效识别并抑制结构相似的硬负样本，缓解梯度饱和和过平滑。
- 即使在极高噪声（40%）下，LightCSCF 的性能下降幅度小于 SCCF，显示更强的鲁棒性。
- t-SNE 可视化显示用户和物品嵌入分布更均匀，聚类现象减弱，说明模型避免了过度平滑。
- 在稀疏用户组上，LightCSCF 依然保持优势，尤其在 Amazon-book 上稀疏组表现甚至优于丰富组（可能由于 BPR 损失权重降低）。
- 超参数 m 和 τ 对性能有显著影响，最优值随数据集变化。

## 7. 优点

- **理论创新**：从谱图角度将矩阵分解与对比学习中的梯度更新统一为并行滤波器框架，揭示了过平滑和梯度饱和的根源，视角新颖。
- **方法简洁高效**：仅修改相似度函数，无需额外图增强、数据采样或模型结构改动，易于集成到现有框架中。
- **实验全面**：覆盖多数据集、多类别基线、多种分析（维度、鲁棒性、稀疏性、超参数、可视化），验证充分。
- **性能突出**：在极稀疏数据上仍有显著提升（如 Amazon-book 密度仅0.06%），证明方法对稀疏场景鲁棒。
- **可解释性强**：通过理论分析（定理1、2）和几何解释（定义1、2）清晰说明问题与解决方案。

## 8. 不足与局限

- **缺乏消融实验**：未将边界约束项单独移除做Ablation study，无法定量验证该组件的边际贡献。并行滤波器假设也需更直接的验证。
- **计算资源未报告**：缺少GPU型号、训练时间等，不利于可重复性和效率对比。
- **数据集多样性有限**：仅使用书籍类（Amazon、Douban）和电商类（Tmall）数据集，未在视频、音乐、新闻等其他领域测试，可能影响泛化性。
- **超参数敏感**：m 和 τ 需精细调参，且最优值在不同数据集上差异较大（图6），实际部署成本可能较高。
- **对比方法公平性**：RecDCL 要求 embedding size=2048，而其他方法统一为64，容量差异可能影响公平性，尽管 LightCSCF 仍大幅领先。
- **未讨论大规模或在线场景**：模型在超大规模图（如亿级节点）上的性能、内存开销、训练速度均未提及。

（完）
