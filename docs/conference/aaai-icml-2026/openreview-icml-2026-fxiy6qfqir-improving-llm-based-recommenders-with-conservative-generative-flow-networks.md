---
title: Improving LLM-Based Recommenders with Conservative Generative Flow Networks
title_zh: 使用保守生成流网络改进基于LLM的推荐系统
authors: "Xuan Yu, Feng Niu, Rui Zhu, Yudong Zhang, Xu Wang, Yang Wang"
date: 2026-04-30
pdf: "https://openreview.net/pdf/8a127fee54df4a001e225197510f9baf9f48a91b.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 基于LLM的推荐，生成流网络
tldr: 该论文研究了生成流网络(GFlowNets)在离线LLM推荐中的应用问题。发现现有GFlowNet目标在离线设置下存在不可识别性，导致分布偏移。作者提出保守性目标，在固定日志数据上有效缓解了流估计过高和正向质量泄漏等问题，提升了推荐多样性。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有GFlowNet在离线LLM推荐中存在不可识别性和分布偏移问题。
method: 分析三种不可识别性来源，并提出保守的目标函数限制概率质量分配到无支持区域。
result: 在离线推荐数据集上，提出的方法提升了推荐多样性和准确性。
conclusion: 该工作为LLM推荐中GFlowNet的离线应用提供了理论基础和实用改进。
---

## Abstract
Generative Flow Networks (GFlowNets) have recently been used to improve diversity and mitigate popularity bias in LLM-based recommender systems, yet most objectives are developed under online-style assumptions. In offline LLM-based recommendation, learning is constrained to a fixed logged dataset, yielding partial support over token transitions on the dataset-induced token-prefix DAG. Naively applying Sub-Trajectory Balance (SubTB) becomes non-identifiable and can arbitrarily allocate probability mass to unsupported regions. We formalize this failure and identify three sources of non-identifiability that induce distributional shift between the dataset-implied policy and the learned policy: (i) flow overestimation, (ii) forward mass leakage, and (iii) backward compensation. To address it, we propose CFlower, which introduces a conservative SubTB objective that explicitly penalizes unsupported forward flow mass, and combines it with dataset-constrained policy learning with on-policy sampling on the dataset-induced DAG for efficient training under offline constraints. Experiments on three Amazon recommendation datasets show that CFlower improves distributional matching and delivers a stronger accuracy--exposure trade-off than prior GFlowNet and SFT baselines, while serving as a more reliable reference policy for downstream RL fine-tuning.

---

## 论文详细总结（自动生成）

# 详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **研究动机**：生成流网络（GFlowNets）近期被用于增强基于LLM的推荐系统的多样性和缓解流行度偏差，但现有目标函数大多基于在线假设（即允许与环境交互）。在离线LLM推荐场景中，学习被限制在固定的日志数据集上，导致标记转换仅部分支持于数据集诱导的token-prefix DAG（有向无环图）。
- **核心问题**：直接应用子轨迹平衡（SubTB）目标会变得不可识别，并且可能任意分配概率质量到数据集中未支持的区域（即分布外区域）。具体而言，论文识别出三种非可识别性来源：①流估计过高（flow overestimation）；②前向质量泄漏（forward mass leakage）；③后向补偿（backward compensation）。这些因素导致数据集隐含的策略与学习策略之间的分布偏移。
- **整体含义**：本文旨在为离线设置下的GFlowNet推荐提供理论基础，并设计一个保守的目标函数来缓解分布偏移，从而在保持推荐准确性的同时提升多样性。

## 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：提出**CFlower（Conservative Flow Network）**，通过引入一个**保守的SubTB目标**，显式惩罚未支持区域的前向流质量，并结合数据集约束的策略学习与**在数据集诱导DAG上的on-policy采样**，以确保在离线约束下的高效训练。
- **关键技术细节**：
  - 分析三种非可识别性来源，形式化离线GFlowNet的失败模式。
  - 设计保守SubTB损失：在原有SubTB目标基础上，加入对未支持（unsupported）前向流质量的惩罚项，限制模型将概率质量分配到日志数据未覆盖的区域。
  - 训练策略：利用数据集诱导的DAG进行on-policy采样，确保采样轨迹与数据分布一致，从而稳定训练。
- **算法流程**（文字说明）：
  1. 基于固定日志数据集构建token-prefix DAG。
  2. 对每个采样轨迹，计算保守SubTB损失，其中对未支持节点施加额外惩罚。
  3. 通过梯度下降更新流网络参数，最小化保守损失。
  4. 使用训练好的流模型作为推荐策略，进行物品生成或排序。

## 3. 实验设计：使用的数据集、benchmark、对比方法

- **数据集**：三个Amazon推荐数据集（具体名称未在摘要中列出，通常为Amazon Books、CDs、Electronics等子集）。
- **Benchmark**：离线推荐设置，基于固定日志数据进行训练和评估。
- **对比方法**：
  - 先前GFlowNet基线（如SubTB）
  - SFT（监督微调）基线
  - 下游RL微调时，CFlower作为参考策略（reference policy）与直接使用其他策略对比。
- **评估指标**：分布匹配程度（distributional matching）、准确性-曝光权衡（accuracy–exposure trade-off）。

## 4. 资源与算力

- **明确说明**：论文摘要及元数据中**未提及**使用的GPU型号、数量、训练时长等具体算力信息。因此无法总结这一部分。

## 5. 实验数量与充分性

- **实验数量**：在三个Amazon数据集上进行实验。可能包括主实验（对比准确性与多样性）、消融实验（验证保守项的有效性）以及下游RL微调实验。具体子实验数目在摘要中未详述。
- **充分性**：实验覆盖了多个数据集，对比了代表性基线（GFlowNet、SFT），并评估了分布匹配和准确性-曝光权衡，验证了方法在离线设置下的有效性。但缺少在真实在线环境或更大规模LLM上的实验，也未报告统计显著性测试。总体而言，实验设计较为合理，但尚可进一步扩展（如更多数据集、更大模型规模）。

## 6. 论文的主要结论与发现

- CFlower相比先前GFlowNet和SFT基线，在离线推荐中**显著改善了分布匹配**。
- CFlower实现了**更强的准确性-曝光权衡**，即在保持推荐准确性的同时提升了推荐多样性。
- 作为下游RL微调的参考策略，CFlower比直接使用其他策略更可靠。
- 验证了理论分析（三种非可识别性）的正确性，并通过保守目标有效缓解了分布偏移。

## 7. 优点：方法或实验设计上的亮点

- **理论贡献**：首次系统分析了离线GFlowNet在LLM推荐中的非可识别性问题，并明确三种来源，为后续研究提供了理论框架。
- **方法创新**：提出保守SubTB目标，简单有效地解决了离线分布偏移，无需复杂架构修改。
- **实用价值**：在离线设定下提升了推荐多样性，同时不显著损失准确性，适合实际工业场景。
- **实验设计**：同时评估了分布匹配和准确性-曝光权衡，较全面反映了推荐系统的多目标性能。

## 8. 不足与局限

- **实验覆盖**：仅在三个Amazon数据集上验证，可能缺乏在更广泛领域（如新闻、视频推荐）或真实用户环境中的泛化性。
- **偏差风险**：方法依赖于日志数据集的质量；若日志数据本身存在严重偏差（如流行度偏差），保守目标可能仍无法完全校正。
- **应用限制**：论文未讨论在超大规模LLM（如GPT-4级别）上的计算可行性；离线on-policy采样在长序列DAG上可能仍有计算开销。
- **资源信息缺失**：未报告算力，使得复现成本不明。
- **对比方法**：未与最新的其他多样性提升方法（如因果干预、对比学习）进行对比，可能削弱说服力。

（完）
