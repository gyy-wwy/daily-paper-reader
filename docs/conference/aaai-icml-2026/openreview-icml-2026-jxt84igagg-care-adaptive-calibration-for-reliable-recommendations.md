---
title: "CARE: Adaptive Calibration for Reliable Recommendations"
title_zh: "CARE: 可靠推荐的适应性校准"
authors: "Nitin Bisht, Huan Huo, Xiuwen Gong, Guandong Xu"
date: 2026-04-30
pdf: "https://openreview.net/pdf/88606032ef417979c6224ada5b7924039993449b.pdf"
tags: ["query:rec-sys"]
score: 6.0
evidence: 面向可靠推荐的适应性校准框架
tldr: 本文针对推荐系统部署后用户行为演变导致性能下降的问题，提出CARE适应性校准框架。该方法结合损失监测和在线聚合规则，动态调整推荐集大小并提供性能保证。实验证明，CARE能在保持推荐质量的同时适应行为变化，为推荐系统的可靠性提供了理论保障。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 推荐系统离线训练后部署，用户行为变化导致推荐质量下降且缺乏保障。
method: 提出基于损失监测和动态重加权的适应性校准框架，提供有限样本性能保证。
result: 在流式推荐场景中，CARE有效维持了推荐准确率并提供了可靠性保证。
conclusion: 适应性校准能增强推荐系统在动态环境中的鲁棒性和可信度。
---

## Abstract
Modern recommender systems are typically trained offline and deployed with parameters held fixed between periodic refreshes, yet user behavior can evolve substantially during deployment. This can cause ranking utility to degrade over time and makes it difficult to provide formal guarantees about recommendation quality. We propose **CARE**, an adaptive calibration framework that wraps an arbitrary backbone recommender and outputs variable-size recommendation sets with finite-sample performance guarantees over interaction streams. CARE combines (i) a loss-based monitoring module that localizes behavioral changes and triggers threshold recalibration, and (ii) an online aggregation rule that promotes compact recommendation sets by dynamically reweighting candidate set predictors. We provide theoretical results establishing finite-sample guarantees for utility-based risk control and bounds on the expected set size relative to the best constituent predictor. Experiments across multiple datasets and backbone models demonstrate that CARE improves robustness and maintains compact recommendation sets while preserving the desired statistical guarantees. The code and implementation are available in https://github.com/kalpiree/CARE.

---

## 论文详细总结（自动生成）

# CARE: Adaptive Calibration for Reliable Recommendations 论文总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：现代推荐系统通常离线训练后部署，参数在定期刷新之间固定，但用户行为在部署期间会显著演变，导致推荐效用随时间下降，且难以提供关于推荐质量的正式保证。
- **研究动机**：现有推荐系统缺乏在动态环境中对推荐质量的统计保障，用户行为的不可预测性使得离线训练模型上线后性能退化，影响推荐系统的可靠性和用户信任。
- **整体含义**：提出一个适应性校准框架，使得推荐系统能够在交互流上提供有限样本的统计性能保证，增强推荐系统在动态变化环境中的鲁棒性和可信度。

## 2. 方法论
- **核心思想**：CARE是一个包装任意骨干推荐器的自适应校准框架，输出可变大小的推荐集合，并保证在交互流上的有限样本性能。通过组合损失监测模块和在线聚合规则，动态调整推荐集大小并保持紧凑性。
- **关键技术细节**：
  - **损失监测模块**：基于损失函数（如预测准确率相关损失）监测推荐性能，当用户行为发生变化导致损失上升时，局部触发阈值重新校准。
  - **在线聚合规则**：动态重加权候选集预测器，以促进紧凑的推荐集合（即集合大小尽可能小），同时满足统计风险控制。
  - **理论保证**：提供了基于效用的风险控制有限样本保证，以及期望集合大小相对于最佳组成预测器的上界。
- **算法流程（文字说明）**：
  1. 使用任意骨干推荐器（如协同过滤、深度学习模型）生成候选推荐集。
  2. 实时监控推荐损失（如准确率、召回率相关的损失函数）。
  3. 当损失超出预设阈值时，触发阈值重新校准，调整推荐集的尺寸或包含的项数。
  4. 在线聚合多个候选集预测器，通过动态加权得到最终推荐集，使得在满足风险控制条件下集合大小尽量小。
  5. 重复上述步骤，形成流式自适应推荐。

## 3. 实验设计
- **数据集/场景**：使用了多个数据集（论文未具体列出名称，但提及“across multiple datasets”），场景为流式推荐（interaction streams），模拟用户行为随时间变化。
- **Benchmark**：未明确说明基准模型，但对比框架为其他方法（具体方法名称在摘要中未列出，但暗示与骨干推荐器结合并与其他校准方法对比）。
- **对比方法**：未在摘要中列出具体方法名称，可能包括固定大小的推荐集、无自适应校准的基线等。

## 4. 资源与算力
- **未明确说明**：论文摘要和提供的文本中未提及使用的GPU型号、数量、训练时长等具体信息。代码仓库链接为 https://github.com/kalpiree/CARE，但未提供资源消耗细节。

## 5. 实验数量与充分性
- **实验数量**：在多个数据集和多个骨干模型上进行了实验（摘要提及“multiple datasets and backbone models”），但具体实验组数未给出。可能包括：
  - 不同数据集上的主实验。
  - 不同骨干模型（如矩阵分解、神经网络等）下的对比。
  - 消融实验可能涉及监测模块和聚合规则的贡献。
- **充分性与公平性**：
  - 充分性：覆盖了多种数据集和骨干模型，且提供了理论保证和实证结果，实验设计较为全面。
  - 公平性：比较了基线方法，但未说明是否采用相同骨干和超参数调优，摘要中没有明确给出控制变量细节。总体来看，实验设计符合顶会标准，但具体公平性需看全文。

## 6. 主要结论与发现
- CARE能够有效提升推荐系统在动态环境中的鲁棒性，保持紧凑的推荐集合（即集合大小较小）的同时，维持所需的统计保证（如准确率/召回率控制）。
- 提供了有限样本风险控制的正式证明，使得推荐质量有理论保障。
- 在多个数据集和骨干模型上，CARE比无自适应校准的方法更稳定，且能适应行为变化。

## 7. 优点
- **理论贡献**：为推荐系统提供了有限样本的性能保证，这是传统推荐系统缺乏的。
- **框架通用性**：CARE可以包装任意的骨干推荐器，即插即用，无需修改原有模型。
- **动态适应性**：通过损失监测和在线聚合，自动适应行为变化，无需人工重新训练。
- **紧凑推荐集**：在满足风险控制前提下，尽量减小推荐集大小，提升用户体验和计算效率。

## 8. 不足与局限
- **实验覆盖不明确**：摘要未列出具体数据集名称、对比方法细节，可能需要阅读全文确认。
- **资源开销未报告**：未说明计算成本，对于在线部署场景，监测和重新校准的开销可能影响实时性。
- **风险控制假设**：理论保证可能依赖于某些统计假设（如交互流的平稳性？），摘要未讨论实际场景中的违反风险。
- **应用限制**：仅针对推荐集大小的校准，未涉及排序列表的优化或个性化多样性等更复杂的指标。
- **偏差风险**：监测模块可能过度敏感或迟钝，导致频繁校准或无法及时适应；在线聚合规则可能偏向简单基线，需要进一步消融分析。

（完）
