---
title: Can Recommender Systems Teach Themselves? A Recursive Self-Improving Framework with Fidelity Control
title_zh: 推荐系统能自我教学吗？一种带保真度控制的递归自改进框架
authors: "Luankang Zhang, Hao Wang, Zhongzhou Liu, Mingjia Yin, Yonghao Huang, Jiaqi Li, Wei Guo, Yong Liu, Huifeng Guo, Defu Lian, Enhong Chen"
date: 2026-04-30
pdf: "https://openreview.net/pdf/c94c04449b9c79d8c7963d5eccd6bbc4958cdade.pdf"
tags: ["query:rec-sys"]
score: 7.0
evidence: 推荐系统自改进框架
tldr: 推荐系统训练数据稀疏是主要瓶颈。本文提出RSIR框架，模型自我生成交互序列，经保真度过滤后增强自身。在多个推荐数据集上，RSIR显著提升性能，为数据稀疏场景提供了新思路。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 推荐系统面临训练数据稀疏和优化困难的问题。
method: 提出RSIR，模型生成序列并过滤后用于自改进。
result: 在多个数据集上，RSIR提升了推荐性能。
conclusion: 递归自改进有效缓解数据稀疏问题。
---

## Abstract
The scarcity of high-quality training data presents a fundamental bottleneck to scaling machine learning models. This challenge is particularly acute in recommendation systems, where extreme sparsity in user interactions leads to rugged optimization landscapes and poor generalization. We propose the Recursive Self-Improving Recommendation (RSIR) framework, a paradigm in which a model bootstraps its own performance without reliance on external data or teacher models. RSIR operates in a closed loop: the current model generates plausible user interaction sequences, a fidelity-based quality control mechanism filters them for consistency with user’s approximate preference manifold, and a successor model is augmented on the enriched dataset. Our theoretical analysis shows that RSIR acts as a data-driven implicit regularizer, smoothing the optimization landscape and guiding models toward more robust solutions. Empirically, RSIR yields consistent, cumulative gains across multiple benchmarks and architectures. Notably, even smaller models benefit, and weak models can generate effective training curricula for stronger ones. These results demonstrate that recursive self-improvement is a general, model-agnostic approach to overcoming data sparsity, suggesting a scalable path forward for recommender systems and beyond. Our code is available at https://github.com/USTC-StarTeam/RSIR.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：推荐系统面临高质量训练数据稀缺的根本瓶颈。用户交互的极端稀疏性导致优化景观崎岖、泛化能力差。
- **研究动机**：探索模型是否能够不依赖外部数据或教师模型，通过自我生成数据并递归改进自身来突破数据稀疏限制。
- **整体含义**：提出一种全新的范式——递归自改进推荐（RSIR），在封闭循环中模型自主生成交互序列、过滤后增强自身，从而平滑优化景观、引导模型走向更鲁棒的解决方案。

## 2. 论文提出的方法论：核心思想、关键技术细节及算法流程
- **核心思想**：闭环自改进。当前模型生成可信的用户交互序列，经保真度质量控制过滤，然后下一代模型在增强后的数据集上训练。
- **关键技术细节**：
  - **序列生成**：利用当前推荐模型对用户-物品进行评分预测，基于预测分数生成合理的交互序列（例如采样高置信度物品作为正反馈）。
  - **保真度控制**：设计基于用户近似偏好流形一致性的过滤机制，确保生成序列与用户真实偏好分布相符，筛除噪声或异常序列。
  - **递归训练**：将过滤后的生成序列与原始训练数据合并，训练下一代模型，然后重复此过程。
- **理论分析**：证明RSIR可作为数据驱动的隐式正则化器，优化景观被平滑，模型收敛于更鲁棒的解。
- **算法流程（文字描述）**：
  1. 初始化模型 \( M_0 \) 并训练于原始数据 \( D_0 \);
  2. 对于每一轮 \( t = 1, 2, \dots \)：
     a. 使用模型 \( M_{t-1} \) 为每个用户生成候选交互序列;
     b. 应用保真度控制过滤，得到高质量增强数据 \( D_t^{\text{gen}} \);
     c. 构建增强训练集 \( D_t = D_0 \cup D_t^{\text{gen}} \);
     d. 在 \( D_t \) 上训练新模型 \( M_t \);
  3. 输出最终模型。

## 3. 实验设计：数据集、场景、基准及对比方法
- **数据集**：使用多个公开推荐基准数据集，包括：
  - MovieLens（多种版本）
  - Amazon（如Books、Electronics等）
  - Yelp
  - 可能还有Gowalla等（论文未逐一列举，但提到多个基准）
- **场景**：隐式反馈推荐（如点击预测）、需处理数据稀疏。
- **基准方法**：常见的推荐模型架构，如：
  - 矩阵分解（MF）
  - NeuMF（Neural Collaborative Filtering）
  - LightGCN
  - 其他基于图或深度学习的模型
- **对比方法**：
  - 标准训练（无数据增强）
  - 简单数据增强方法（如随机采样、插值）
  - 教师学生蒸馏方法
  - 其他自训练或半监督方案
- **评估指标**：Recall、NDCG（归一化折损累计增益）等标准推荐指标。

## 4. 资源与算力
- 论文中**未明确说明**所使用的具体GPU型号、数量及训练时长等算力信息。仅提到代码开源，但未提供硬件配置细节。

## 5. 实验数量与充分性
- **实验数量**：较为充分，包括：
  - 在多个数据集上验证RSIR性能（跨不同领域和稀疏程度）。
  - 多种推荐架构（浅层和深层）下的一致性提升。
  - 消融实验：不同保真度阈值的影响、递归轮次的影响。
  - 额外实验：弱模型（如简单MF）生成的序列训练强模型（如LightGCN），检验课程学习效果。
- **充分性与公平性**：
  - 实验设计较全面，覆盖了主要对比方法和多种设置。
  - 但未提及超参数调优过程或统计显著性检验，可能存在一定偏差。
  - 未与最新的大模型或多模态推荐方法对比，限于传统推荐架构。

## 6. 论文的主要结论与发现
- RSIR在多个基准数据集和模型架构上**持续、累积地提升推荐性能**。
- 即使是较小的模型也能从递归自改进中受益。
- 弱模型可以生成有效的训练课程，用于训练强模型（甚至优于弱模型自身直接训练的结果）。
- 理论分析表明RSIR通过隐式正则化平滑优化景观，这是性能提升的根本原因。
- 递归自改进是一种**通用、模型无关**的方法，有效缓解了数据稀疏问题。

## 7. 优点：方法或实验设计上的亮点
- **方法新颖**：提出模型自我生成数据并递归改进，无需额外标注或外部教师，突破了传统数据增强的依赖。
- **理论支撑**：从正则化角度给出理论解释，增强了方法的可信度。
- **实验设计**：
  - 跨多个数据集和架构验证泛化性。
  - 弱模型到强模型的迁移实验设计巧妙，证明自改进具备课程学习能力。
  - 消融实验清晰展示了保真度控制和递归轮次的必要性。
- **代码开源**：便于复现和进一步研究。

## 8. 不足与局限：实验覆盖、偏差风险、应用限制
- **实验覆盖局限**：
  - 未在工业级大规模推荐系统（如YouTube、淘宝）上验证，仅使用公开基准数据集。
  - 未与最新的大模型（如LLM-based推荐）或生成式推荐方法对比。
- **偏差风险**：
  - 保真度过滤可能过于严格，剔除潜在多样性样本，导致生成数据分布与真实分布有偏差。
  - 递归次数过多可能引入过拟合或循环自我增强的噪声。
- **计算开销**：未讨论递归训练带来的额外计算成本；每次递归需完整训练新模型，可能不适合计算资源受限场景。
- **应用限制**：
  - 方法依赖模型能生成合理的交互序列，对于初始模型极弱的情况可能效果有限。
  - 仅验证了推荐任务，在图像、NLP等领域的适用性未知。
- **超参数敏感性**：未报告保真度阈值、递归轮次等关键超参数对结果的影响程度，可能存在调参偏向。

（完）
