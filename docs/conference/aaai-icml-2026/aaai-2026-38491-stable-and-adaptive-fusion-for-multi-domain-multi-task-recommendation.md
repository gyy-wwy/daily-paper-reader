---
title: Stable and Adaptive Fusion for Multi-domain Multi-task Recommendation
title_zh: 稳定自适应融合用于多域多任务推荐
authors: "Ke Fei, Da Luo, Kangyi Lin, Zibin Zhang, Jingjing Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38491/42453"
tags: ["query:rec-sys"]
score: 9.0
evidence: 多域多任务推荐的深度学习融合框架
tldr: 针对多域多任务推荐中特征与目标间的伪相关导致负迁移问题，提出稳定自适应融合框架SAF。该框架引入加权的Hilbert-Schmidt独立性准则损失，学习样本权重以解耦无关特征，并在底层和专家层促进稳定表征。采用随机傅里叶特征实现可扩展计算。实验证明SAF有效缓解负迁移，提升推荐准确性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有多域多任务推荐方法常因特征与目标间的伪相关导致负迁移。
method: 提出SAF框架，引入加权HSIC损失解耦无关特征，利用RFF实现可扩展计算。
result: 在多个多域多任务数据集上显著缓解负迁移，提升推荐性能。
conclusion: SAF通过稳定表征学习有效提升多域多任务推荐的鲁棒性。
---

## Abstract
Multi-Domain Multi-Task (MDMT) recommendation aims to provide personalized recommendations by leveraging information across multiple domains and tasks. However, existing methods often suffer from spurious correlations between irrelevant features and the target, leading to negative transfer. To address this, we propose a Stable and Adaptive Fusion (SAF) framework for MDMT recommendation. SAF introduces a weighted  Hilbert-Schmidt Independence Criterion (HSIC) loss to decorrelate irrelevant features from the target, learning sample weights that promote stable (i.e., robust to spurious correlations) representations in both bottom and expert layers. We employ Random Fourier Features (RFF) to enable scalable computation of the HSIC loss. We further employ adaptive feature and expert gating to select these stable features, enabling the model to capture intricate cross-domain and cross-task dependencies. The learned sample weights are also used to reweight the MDMT loss during training. Experiments on large-scale datasets show that SAF outperforms state-of-the-art baselines by up to 2% in AUC. To facilitate further research, we release a new industrial dataset with 30 million interactions across 3 domains and 2 tasks, with 300 features.

---

## 论文详细总结（自动生成）

# 论文《Stable and Adaptive Fusion for Multi-domain Multi-task Recommendation》详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：多域多任务推荐（MDMT）中，特征与目标之间常存在**伪相关**（spurious correlations），导致模型依赖无关特征，引发**负迁移**（negative transfer），降低推荐质量。
- **背景**：现有MDMT方法（如STAR、PLE、PEPNet等）主要通过参数共享和门控机制来捕获跨域/跨任务共性，但**无法区分因果相关特征与仅统计相关的虚假特征**，尤其在数据稀疏、分布不平衡时，负面效应更严重。
- **意义**：需要一种能主动解耦特征、抑制伪相关、促进稳定表征的方法，以提升MDMT推荐的鲁棒性和泛化能力。

## 2. 方法论：核心思想、关键技术细节
### 核心思想
- 学习**样本权重**，使重加权后的特征分布中无关特征与目标解耦，从而让模型关注稳定（因果）特征。
- 通过**自适应门控**选择这些稳定特征，动态融合专家知识。

### 关键技术细节
1. **加权HSIC损失**：
   - 采用**Hilbert-Schmidt独立性准则**（HSIC）量化特征间的统计依赖性。
   - 引入可训练样本权重 **w**，最小化加权后的HSIC：
     \[
     \text{HSIC}_{\text{RFF}} = \frac{1}{n^2} \| (\mathbf{H} \text{diag}(\mathbf{w}) \mathbf{Z}_A)^\top (\mathbf{H} \mathbf{Z}_B) \|_F^2
     \]
     其中 \(\mathbf{H}\) 为中心矩阵，\(\mathbf{Z}_A, \mathbf{Z}_B\) 为RFF投影后的特征矩阵。
2. **Random Fourier Features（RFF）**：
   - 近似核计算，使HSIC在大批量数据上可扩展。映射维度 \(n_x = 3\)。
3. **自适应特征门控与专家门控**：
   - 特征门控：基于用户-物品嵌入和域ID生成域缩放因子 \(\pi\)，对共享特征进行缩放。
   - 专家门控：对 \(K=5\) 个专家输出，使用域/任务专用门控动态选择融合。
4. **交替优化**：
   - 每轮先固定模型参数，优化样本权重 \(w\)（最小化HSIC损失）；再固定权重，优化MDMT损失（权重由 \(w_f\) 和 \(w_{ex}\) 组成）。
5. **损失函数**：
   - 总损失 = 加权MDMT损失 + HSIC损失（辅助）。样本权重同时用于重加权任务损失。

### 算法流程（文字说明）
- 输入：批数据（特征、域ID、标签）。
- 共享嵌入层 → 共享线性变换 → 特征门控（生成π） → K个专家MLP → 专家门控（域/任务专用） → 任务塔输出预测。
- 每批次：计算HSIC损失 → 更新样本权重；计算加权MDMT损失 → 更新模型参数；维持特征和权重的指数移动平均。
- 推理时仅使用训练好的主干网络，不涉及样本重加权。

## 3. 实验设计
### 数据集
- **Ali-CCP**（公共，2个域？实际文章表1显示d1,d2,d3，可能为3域？文章描述Ali-CCP是3域？根据表1：d1,d2,d3共3个域，CTR和CVR共2个任务）；**工业数据集**（30亿交互、3个域、2个任务、300个特征，新发布）。
- 涵盖CTR（点击率）和CVR（转化率）两类任务。

### 基准方法（共11种）
- 基础模型：MLP、SharedBottom、MMOE、PLE
- 多域方法：STAR、Adasparse、Hamur
- 多任务方法：AdaTask
- MDMT方法：PEPNet、M2M、M3OE

### 评估指标
- AUC（主指标）、Logloss（辅助）。统计显著性检验采用双尾t检验（p<0.05），5次不同随机种子平均结果。

## 4. 资源与算力
- **文中未明确说明使用的GPU型号、数量、训练时长**。
- 仅提及：SAF增加训练时间约4%，推理时间无增加。HSIC损失计算因RFF和批次大小（通常<10k）而高效。

## 5. 实验数量与充分性
- **实验组数**：
  - 整体性能对比：2个数据集 × 每个数据集含多个域/任务（Ali-CCP 5个任务，工业6个任务）→ 表1。
  - 深层分析（显著图，图3、表2）：定性+3个量化指标（熵、基尼系数、相关性）。
  - 消融实验（表3）：去除SL、FG、EG等组件，共6种配置。
  - 超参数敏感分析（图4）：RFF维度nx、移动平均α、专家数K。
  - 额外分析：自适应学习率与任务损失调优对比（文中提及）。
- **充分性评价**：
  - **充分**：覆盖多个数据集、多域/任务、多种基线、统计显著性检验、消融分析、超参调优。
  - **客观公平**：所有基线在同一设置下运行，结果取平均，使用t检验证明改进显著。
  - **局限**：仅针对CTR/CVR两类任务，未验证其他类型（如评分、转发等）；未提供工业数据集的具体分布细节。

## 6. 主要结论与发现
- SAF在所有任务和域上**一致优于**所有基线，平均AUC提升**1.5%**（Ali-CCP）和**2.5%**（工业）。
- 在数据稀疏的域/任务（如domain 3和CVR）上提升尤为显著，证明能有效缓解负迁移。
- 显著图分析表明SAF产生**稀疏、低相关性的特征归因**，聚焦于稳定特征集；而基线方法（PEPNet、M3OE）归因弥散、相关性高。
- 消融实验证实**稳定学习（SL）贡献最大**，其次是特征门控和专家门控；三者协同。

## 7. 优点
- **创新性**：首次将基于HSIC的稳定学习引入MDMT推荐，通过样本重加权显式解耦伪相关。
- **实用性**：使用RFF实现可扩展计算，训练开销仅增4%，适合工业部署。
- **设计精巧**：交替优化、维护全局特征和权重的指数移动平均，保证批次级与数据集级的一致性。
- **实验严谨**：大规模工业数据集验证、多种基线、统计检验、消融、超参分析完备。
- **开放贡献**：发布新工业数据集（30亿交互、300特征），促进后续研究。

## 8. 不足与局限
- **超参数敏感**：RFF维度 \(n_x=3\)、移动平均 \(\alpha=0.8/0.9\)、专家数 \(K=5\) 需针对数据集调优，泛化到新场景时可能需要重新搜索。
- **任务类型覆盖有限**：仅验证CTR和CVR两类二分类任务，未测试回归任务或多分类任务。
- **理论假设的理想化**：定理1要求特征在重加权后完全独立，实际中只能近似，HSIC零值难以严格达到。
- **跨域泛化分析不足**：未讨论模型在不同分布偏移（如新增域、时序偏移）场景下的鲁棒性。
- **计算资源细节缺失**：未公开GPU型号、训练时长、内存消耗等，不利于复现对比。

（完）
