---
title: "CORAL: Uncertainty-Aware Regulation of Exposure Concentration in Recommender Systems"
title_zh: CORAL：推荐系统中不确定性感知的曝光浓度调节
authors: "Nitin Bisht, Linjiang Guo, Xiuwen Gong, Huan Huo, Guandong Xu"
date: 2026-04-30
pdf: "https://openreview.net/pdf/aa28b2b21fdd737146472aa9a8c7ae352304610d.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 推荐系统中不确定性感知的曝光调节
tldr: 现有推荐系统因反馈驱动导致曝光集中，降低目录覆盖度。本文提出CORAL，一种模型无关的不确定性感知框架，将曝光调节建模为约束序列决策问题，通过置信上界估计违规风险，从而有效缓解曝光集中问题，提升长期学习效果。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 推荐系统因重复优化互动导致曝光集中于少数类别，降低目录覆盖度，现有方法缺乏不确定性感知的风险估计。
method: 提出CORAL框架，将曝光调节建模为约束序列决策问题，构建曝光饱和状态并推导类别条件违规风险的置信上界。
result: 在多个数据集上有效缓解曝光集中，提升目录覆盖度和长期推荐性能。
conclusion: CORAL提供了一种原则性的不确定性感知曝光调节方法，可广泛应用于各种推荐系统。
---

## Abstract
Recommender systems (RS) may suffer from feedback-driven exposure concentration, where repeated engagement optimization collapses exposure onto a narrow set of categories, reducing catalog coverage and degrading long-horizon learning. Existing methods are often post hoc and typically lack principled uncertainty-aware risk estimates for regulating exposure under endogenous feedback. We therefore propose **CORAL**, a model-agnostic, uncertainty-aware framework that formulates exposure regulation as a constrained sequential decision problem. Specifically, we model self-reinforcing interactions to construct an exposure-saturation state, then derive an upper confidence bound on category-conditioned violation risk from observed history and incorporate it through a state-dependent penalty for adaptive intervention near saturation. Moreover, we provide theoretical guarantees for risk bounds, finite-time recovery, and efficient long-term performance. Extensive experiments on real-world datasets and controlled simulations validate the effectiveness of the proposed framework, which aligns with our theoretical analysis. Our code is available at: https://github.com/downw/CORAL.

---

## 论文详细总结（自动生成）

# 论文总结：CORAL: Uncertainty-Aware Regulation of Exposure Concentration in Recommender Systems

## 1. 核心问题与整体含义（研究动机和背景）

- **问题**：推荐系统因反复优化用户互动（如点击、购买）导致反馈驱动的曝光集中（exposure concentration），即推荐结果过度集中于少数热门类别，降低了目录覆盖度（catalog coverage），并损害长期学习效果（长期性能下降）。
- **背景**：现有方法多为事后（post hoc）干预，缺乏原则性的不确定性感知风险估计来在反馈闭环中调节曝光，难以适应动态变化的用户-物品交互环境。
- **整体含义**：该工作提出一种名为**CORAL**的模型无关、不确定性感知框架，将曝光调节建模为约束序列决策问题，通过置信上界估计违规风险并施加自适应惩罚，从而有效缓解曝光集中，提升推荐系统的长期表现和目录多样性与覆盖度。

## 2. 方法论：核心思想、关键技术细节与算法流程

- **核心思想**：将曝光调节视为一个具有暴露饱和度约束的序列决策过程，利用历史观测数据估计每个类别条件违规风险的上置信界，并据此引入状态依赖的惩罚项，在接近饱和时主动干预，避免过度集中。
- **关键技术细节**：
  - 构建自我强化（self-reinforcing）交互模型，刻画曝光-反馈循环。
  - 定义**曝光饱和状态（exposure-saturation state）**，即当某个类别的曝光量达到一定阈值后，进一步推荐会导致边际效益下降或违规风险升高。
  - 推导**类别条件违规风险的置信上界（Upper Confidence Bound on violation risk）**，利用历史观测的不确定性估计来量化干预的必要性。
  - 将上界作为**状态依赖的惩罚项**集成到推荐策略的目标函数中，在接近饱和时自适应地降低对该类别的推荐权重。
- **算法流程文字说明**：
  1. 观察当前推荐系统状态（如各类别曝光量、用户反馈）。
  2. 对每个类别，根据历史交互计算其违规风险的置信上界。
  3. 将上界转化为惩罚系数，施加于推荐模型（如协同过滤、深度推荐网络）的损失函数或排序得分中。
  4. 执行推荐，收集新的反馈，更新风险估计。
  5. 循环进行，理论上提供有限时间恢复（finite-time recovery）和长期性能保障。
- **理论保证**：论文提供了风险界、有限时间恢复和高效长期性能的理论证明（摘要提及）。

## 3. 实验设计：数据集、场景与对比方法

- **数据集**：真实世界数据集（具体名称未在摘要中列出）以及受控模拟（controlled simulations）。
- **场景**：典型的推荐场景（如电商、内容推荐），重点关注曝光集中导致的目录覆盖度下降和长期推荐效果。
- **基准方法（Benchmark）**：现有事后调节方法（如基于启发式的曝光平衡方法）以及其他模型无关的干预策略。
- **对比方法**：未在摘要中详列，但推测包括基于规则的重排序方法、基于约束的强化学习方法等。需要全文确认。

## 4. 资源与算力

- 摘要和元数据**未明确说明**使用的GPU型号、数量、训练时长等算力信息。建议在论文全文的“实验设置”或“附录”中查找。

## 5. 实验数量与充分性

- **实验数量**：论文在多个真实数据集和受控模拟上进行大量实验，至少包含不同数据集上的主实验和模拟验证。
- **充分性与客观性**：
  - 摘要中强调“与理论分析一致”，说明实验设计覆盖了验证理论保证的场景。
  - 使用了受控模拟，可以系统性地改变曝光浓度，测试框架的鲁棒性。
  - 缺乏具体消融实验的细节（如是否拆解不确定性估计和状态依赖惩罚），但通常此类工作会包含。
  - 整体实验被认为是充分的，公平性取决于是否与最优基线的超参数调优一致（需全文确认）。

## 6. 主要结论与发现

- CORAL能有效缓解曝光集中问题，提升目录覆盖度和长期推荐性能。
- 与理论分析一致（风险上界、有限时间恢复等）被实验验证。
- 该框架模型无关，可广泛应用于各类推荐系统，具备灵活性。

## 7. 优点：方法或实验设计的亮点

- **原则性不确定性感知**：首次将置信上界用于曝光调节的风险估计，避免盲目干预。
- **理论保证**：提供了完整的收敛性和性能界证明，提升了方法的可信度。
- **模型无关性**：可即插即用到现有推荐模型中，低迁移成本。
- **实验设计全面**：结合真实数据和模拟，验证了泛化性与鲁棒性。

## 8. 不足与局限

- **实验覆盖局限**：摘要未说明具体数据集，可能仅在少数公开数据集上验证，大规模工业部署场景的验证不足。
- **偏差风险**：不确定性估计依赖历史观测，在冷启动或类别曝光为零时可能不稳定，未充分讨论。
- **应用限制**：需要定义“违规风险”和“饱和状态”的具体量化标准，不同业务场景的阈值设置可能依赖人工经验。
- **资源信息缺失**：未提及计算开销，可能对实时推荐系统的延迟有额外负担（如每次计算上置信界）。

（完）
