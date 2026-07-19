---
title: Bidirectional Counterfactual Distillation for Review-Based Recommendation
title_zh: 双向反事实蒸馏用于基于评论的推荐
authors: "Sheng Sang, Shujie Li, Shuaiyang Li, Kang Liu, Teng Li, Wei Jia, Dan Guo, Feng Xue"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38598/42560"
tags: ["query:rec-sys"]
score: 8.0
evidence: 双向反事实蒸馏方法用于基于评论的推荐
tldr: 基于评论的推荐方法在跨行为知识迁移中面临评分分布偏差和特征同质化问题。本文提出双向反事实蒸馏框架，通过反事实干预消除评分偏差，并设计双向蒸馏避免特征同质化。实验证明，该方法在多个推荐数据集上有效提升了推荐准确性和鲁棒性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有跨行为知识迁移受评分分布偏差和特征同质化影响，损害推荐效果。
method: 采用反事实推理消除评分偏差，并引入双向蒸馏机制保持特征多样性。
result: 在多个真实推荐数据集上，该方法在点击率和评分预测上均优于基线。
conclusion: 反事实蒸馏能有效消除偏差，提升推荐系统的鲁棒性和准确性。
---

## Abstract
Review-based recommendation methods typically integrate multiple behaviors, including interactions, reviews, and ratings, to model user preferences. To effectively extract preference signals from diverse behaviors, some studies train multiple student models to capture distinct behavioral patterns, and leverage online distillation to facilitate collaborative learning among them. However, we argue that these techniques suffer from bias contamination from rating distributions and feature homogenization during cross-behavior knowledge transfer: (1) Rating distribution bias, arising from non-uniform historical ratings, propagates across behaviors through distillation, contaminating the true preference representations of other behaviors. (2) Static distillation strategies often lead to homogenized behavioral features, hindering the learning of behavior-specific preferences. 
To address these issues, we propose a novel Bidirectional Counterfactual Distillation (BiCoD) framework for review-based recommendation. In BiCoD, we first design an adversarial counterfactual distillation module to suppress the impact of non-uniform rating distributions on distillation, thereby preventing it from contaminating the user's true preference representations across behaviors. Subsequently, we introduce a stage-aware bidirectional distillation strategy to enhance the distinctiveness of behavioral features, facilitating the effective learning of behavior-specific preferences. Extensive experiments on five real-world datasets validate the effectiveness and superiority of the proposed framework.

---

## 论文详细总结（自动生成）

# 论文详细中文总结：Bidirectional Counterfactual Distillation for Review-Based Recommendation

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **研究领域**：基于评论的推荐系统（Review-based Recommendation），利用用户的多类型行为（交互、评论、评分）来建模用户偏好。
- **现有方法局限**：现有方法通过多个学生模型并行学习不同行为模式，并采用在线蒸馏促进跨行为知识迁移。但存在两个关键问题：
  - **评分分布偏差污染（Bias contamination）**：历史评分呈非均匀分布（如高评分居多），模型在蒸馏时会将这种偏差传播到其他行为表征中，污染用户真实偏好。
  - **特征同质化（Feature homogenization）**：静态蒸馏策略（固定方向和强度）使不同行为的特征表征趋同，限制了行为特有偏好的学习。
- **核心目标**：提出一种新框架，在跨行为蒸馏中有效抑制评分偏差传播，同时保持行为特征差异性，提升推荐准确性和鲁棒性。

## 2. 论文提出的方法论：核心思想、关键技术细节

### 整体框架（BiCoD：Bidirectional Counterfactual Distillation）

- **并行多行为建模（MB）**：构建三个视图（交互视图、评论视图、方面-情感视图），每个视图使用图卷积网络（GCN）聚合信息，节点特征包括ID嵌入、评论语义、方面情感、评分嵌入。
- **核心模块一：对抗反事实蒸馏（Adversarial Counterfactual Distillation）**
  - **反事实推理**：构造反事实世界，在该世界中用户/物品特征仅由评分决定（去除其他信息）。然后从事实特征中减去反事实特征，以消除评分分布偏差对蒸馏的影响。
  - **对抗学习**：引入可学习参数γ控制偏差抑制强度。主模型（θ_MB）希望最小化蒸馏损失（倾向于保留偏差），而γ通过对抗最大化损失，从而迫使蒸馏过程抑制偏差。通过梯度反转层（GRL）实现交替优化。
  - **损失函数**：蒸馏损失改为$f_{sce}(e_{x,k} - \gamma e^*_{x,k}, e_{y,k} - \gamma e^*_{y,k})$，其中$e^*$为反事实特征。
- **核心模块二：阶段感知双向蒸馏（Stage-aware Bidirectional Distillation）**
  - **前向蒸馏（Forward Distillation, FD）**：训练初期，促进跨行为知识迁移，帮助各视图表征对齐。
  - **反向蒸馏（Reverse Distillation, RD）**：训练后期，增大行为间特征差异，迫使模型学习行为特有表征。
  - **动态调度**：蒸馏系数$\phi_n$按余弦函数从1衰减至0.1，奇数epoch执行FD，偶数epoch执行RD（符号相反），自动调整蒸馏方向和强度。
- **总损失**：评分预测损失（MSE）+ 动态双向蒸馏损失，通过交替优化使主模型最小化、对抗参数最大化。

## 3. 实验设计

- **数据集**：5个真实世界数据集：
  - Amazon评论数据集的 Art Crafts, Digital Music, Luxury Beauty, Software
  - Yelp评论数据集
- **数据划分**：80%训练，10%验证，10%测试。
- **评价指标**：MSE（均方误差）和MAE（平均绝对误差），用于回归任务。
- **对比方法**：分为三组：
  - **双塔架构**：DeepCoNN, NARRE, DAML, APRE
  - **GNN方法**：RGCL, SGDN, MAGCL, ReHCL
  - **知识蒸馏方法**：KDCRec, LUME, EMKD, FreqD
- **实现细节**：嵌入维度d=64，GCN层数L=1，学习率0.01，最大训练轮数200，早停耐心50，γ初始化为0.5。

## 4. 资源与算力

- 论文**未明确说明使用的GPU型号、数量及训练时长**。仅提及“The computation is completed on the HPC Platform of Hefei University of Technology.”，即使用合肥工业大学的高性能计算平台，但未提供详细硬件配置。

## 5. 实验数量与充分性

- **总体实验**：在5个数据集上对比了12种基线方法，报告了MSE/MAE均值及统计显著性（t检验，p<0.01）。
- **消融实验**：
  - 对双向蒸馏进行消融：去除FD、RD、动态调度、完整OD，共4个变体，在4个数据集上展示MSE结果（图3）。
  - 对对抗反事实蒸馏进行消融：去除AL/FD、CD/FD、AL/RD、CD/RD、整体AL、整体CD，以及固定γ=0.5/1，共8个变体，在5个数据集上报告MSE（表2）。
- **额外分析**：
  - 损失趋势分析：对比LCD与LOD训练曲线（图4）。
  - 灵敏度分析：固定调度系数ϕ_n不同取值下的性能（图5）。
- **充分性评价**：实验覆盖了多个领域数据集、多种类型基线、多个消融组件，且报告了统计显著性。但在数据集规模、计算效率、超参数敏感性等方面未做全面探索。

## 6. 论文的主要结论与发现

- BiCoD在所有5个数据集上均取得最优MSE/MAE，相对最佳基线提升3.19%~12.71%，表明框架有效。
- 消融实验证实：
  - 双向蒸馏优于单一方向蒸馏，且动态调度优于固定强度。
  - 对抗反事实蒸馏显著优于无对抗或无反事实的变体，固定γ效果差于自适应γ。
- 损失分析显示：对抗机制使LCD保持较高值，表明成功抑制了蒸馏中的偏差利用，且训练稳定。
- 双向蒸馏能够自动适应不同训练阶段，前向促进知识迁移，后向保持特征多样性。

## 7. 优点

- **理论分析深入**：从梯度角度严格推导了评分偏差在蒸馏中的传播机制，为方法设计提供了理论依据。
- **方法创新性强**：首次将反事实推理和对抗学习结合用于在线蒸馏中的偏差抑制，并引入阶段感知双向蒸馏，概念清晰且实用。
- **实验设计规范**：对比基线涵盖主流方法，消融实验系统，验证了每个模块的必要性。
- **代码开源**：提供GitHub仓库（https://github.com/hfutmars/BiCoD），便于复现和后续研究。

## 8. 不足与局限

- **资源信息缺失**：未报告GPU型号、数量、训练时间等，不利于其他研究者评估计算成本。
- **数据集规模有限**：仅使用中等规模数据集（Amazon子集和Yelp），未在大规模推荐数据集（如MovieLens-20M、Amazon完整数据集）上验证。
- **超参数敏感性未充分分析**：仅对调度系数ϕ_n做了灵敏度实验，对γ初始值、学习率、GCN层数等未做详细调参研究。
- **应用限制**：方法依赖评论、评分、交互等多行为数据，在缺乏丰富文本评论的场景下可能不适用。
- **理论分析可进一步深化**：非线性GCN中的偏差传播仅从简单SCE损失梯度推导，未考虑更复杂的图结构影响。
- **未见部署或效率讨论**：未分析模型推理速度、参数量、实际部署可行性。

（完）
