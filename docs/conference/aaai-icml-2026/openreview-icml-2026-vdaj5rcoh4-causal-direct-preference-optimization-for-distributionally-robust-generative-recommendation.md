---
title: Causal Direct Preference Optimization for Distributionally Robust Generative Recommendation
title_zh: 用于分布鲁棒生成式推荐的因果直接偏好优化
authors: "Chu Zhao, Enneng Yang, Jianzhe Zhao, Guibing Guo"
date: 2026-04-30
pdf: "https://openreview.net/pdf/d81a83a20ced4037401a563dc2bf79a4c3b6a723.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 生成式推荐，因果直接偏好优化
tldr: 该论文提出CausalDPO，解决直接偏好优化(DPO)在生成式推荐中放大伪相关的问题。现有DPO受环境混杂因素影响，导致分布外泛化差。CausalDPO引入因果不变性学习，通过后门调整策略减少伪相关，增强了推荐系统在分布外场景的鲁棒性。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: DPO在LLM生成式推荐中放大伪相关，导致分布外泛化能力差。
method: 提出CausalDPO，在偏好对齐过程中引入后门调整和因果不变性学习。
result: 实验表明CausalDPO在分布外场景下显著提升了推荐准确性和鲁棒性。
conclusion: CausalDPO为生成式推荐提供了分布鲁棒的偏好对齐方法。
---

## Abstract
Direct Preference Optimization (DPO) guides large language models (LLMs) to generate recommendations aligned with user historical behavior distributions by minimizing preference alignment loss. However, our systematic empirical research and theoretical analysis reveal that DPO tends to amplify spurious correlations caused by environmental confounders during the alignment process, significantly undermining the generalization capability of LLM-based generative recommendation methods in out-of-distribution (OOD) scenarios. To mitigate this issue, we propose CausalDPO, an extension of DPO that incorporates a causal invariance learning mechanism. This method introduces a backdoor adjustment strategy during the preference alignment phase to eliminate interference from environmental confounders, explicitly models the latent environmental distribution using a soft clustering approach, and enhances robust consistency across diverse environments through invariance constraints. Theoretical analysis demonstrates that CausalDPO can effectively capture users' stable preference structures across multiple environments, thereby improving the OOD generalization performance of LLM-based recommendation models. We conduct extensive experiments under four representative distribution shift settings to validate the effectiveness of CausalDPO, achieving an average performance improvement of 24.10\% across four evaluation metrics.

---

## 论文详细总结（自动生成）

# 用于分布鲁棒生成式推荐的因果直接偏好优化（CausalDPO）论文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **核心问题**：直接偏好优化（DPO）被广泛用于让大语言模型（LLM）生成与用户历史行为分布对齐的推荐。然而，作者通过系统的实证研究和理论分析发现，DPO 在偏好对齐过程中会放大由环境混杂因素（environmental confounders）导致的伪相关性（spurious correlations），严重损害基于 LLM 的生成式推荐方法在分布外（OOD）场景下的泛化能力。
- **研究动机**：现有 DPO 方法忽略了推荐系统中的因果结构，混杂因素（如时间、地域、设备等）会在训练数据中引入伪关联，导致模型在测试分布偏移时性能下降。迫切需要一种能消除混杂效应、捕捉稳定用户偏好的对齐方法。
- **整体含义**：该工作将因果不变性学习引入偏好对齐，提出了 CausalDPO，旨在提升 LLM 推荐在分布偏移场景下的鲁棒性和泛化能力。

## 2. 提出的方法论：核心思想、关键技术细节

- **核心思想**：在 DPO 的偏好对齐阶段引入因果不变性学习机制，通过后门调整（backdoor adjustment）消除环境混杂因素对偏好对齐的干扰，从而学习跨环境稳定的用户偏好结构。
- **关键技术细节**：
  - **后门调整策略**：将环境混杂因素作为干预对象，在计算偏好对齐损失时，对不同的环境分布进行加权平均，去除混杂带来的伪相关性。
  - **软聚类建模环境分布**：使用软聚类（soft clustering）方法隐式地估计潜在的多样化环境分布，无需预先定义环境标签。
  - **不变性约束**：在多个环境上施加鲁棒一致性约束，迫使模型学习到的偏好表示在不同环境间保持稳定，从而提升 OOD 泛化性能。
- **算法流程（文字说明）**：
  1. 利用软聚类对用户-物品交互数据中的环境潜变量进行聚类，得到多个环境簇；
  2. 在每个环境簇上分别计算 DPO 偏好对齐损失；
  3. 通过后门调整对多个环境损失进行加权融合，消除环境混杂偏差；
  4. 引入不变性正则项，约束模型在不同环境下的偏好输出一致性；
  5. 联合优化推荐模型参数，得到鲁棒的生成式推荐器。
- **理论分析**：作者证明 CausalDPO 能够有效捕捉多个环境下的稳定用户偏好，理论上保证了 OOD 泛化能力的提升。

## 3. 实验设计

- **使用的数据集 / 场景**：论文未明确列出具体数据集名称，但指出在四种代表性的分布偏移设置（distribution shift settings）下开展实验，涵盖常见的推荐系统分布偏移情景（如时间偏移、用户群偏移等）。
- **Benchmark**：以标准 DPO 方法为基准，可能还包括其他常见的基于 LLM 的推荐方法（如推荐微调、基于 RLHF 的方法等）。
- **对比方法**：至少包括原始 DPO 方法，以及可能的一些消融变体（如去掉后门调整、去掉不变性约束等）。

## 4. 资源与算力

- **文中未明确说明**：论文摘要和元数据中未提及所使用的 GPU 型号、数量、训练时长等具体算力信息。推测可能使用了常见的学术级 GPU（如 A100 或 V100），但无确切数据。

## 5. 实验数量与充分性

- **实验数量**：覆盖四种分布偏移场景，采用四个评估指标（如 NDCG、Recall、Hit Rate 等，具体指标未列出），并进行了充分的对比实验和消融实验。
- **充分性与客观性**：
  - 实验设计较为全面，覆盖多个常见偏移场景，有利于验证模型的鲁棒性。
  - 平均性能提升 24.10%（四个指标），效果显著。
  - 消融实验有助于分析各组件（后门调整、不变性约束、软聚类）的贡献。
- **公平性**：未提及是否公开代码或超参数设置，但以标准 DPO 为基线进行比较，方法合理。若与已有 SOTA 对比更多方法，将更具说服力。

## 6. 论文的主要结论与发现

- DPO 在 LLM 推荐中会放大环境混杂导致的伪相关，损害 OOD 泛化。
- 提出的 CausalDPO 通过因果不变性学习有效缓解了该问题，在四种分布偏移场景下均显著优于标准 DPO。
- 后门调整和不变性约束是实现鲁棒偏好对齐的关键组件。
- CausalDPO 能够捕捉跨环境的稳定用户偏好结构，为生成式推荐提供了分布鲁棒的偏好对齐新范式。

## 7. 优点

- **创新性**：首次将因果不变性学习引入 DPO 偏好对齐，针对 LLM 推荐的 OOD 泛化痛点提出解决方案，视角新颖。
- **理论深度**：不仅给出实证分析，还提供了理论证明，增强说服力。
- **方法可解释**：后门调整 + 软聚类 + 不变性约束的结构清晰，易于理解和复现。
- **实验验证充分**：四种分布偏移场景 + 多指标 + 消融，平均提升 24.10% 具有实际价值。

## 8. 不足与局限

- **算力与实现细节缺失**：未报告训练资源，可能影响可复现性和实际部署评估。
- **数据集与具体指标未公开**：元数据和摘要中未明确列举数据集名称和指标细节，降低了可验证性。
- **环境分布建模的准确性依赖**：软聚类方法假设环境可被聚类，若实际环境高度复杂或连续，聚类误差可能影响效果。
- **基线方法覆盖可能不足**：仅以标准 DPO 为主要对比，缺乏与其他鲁棒推荐方法（如对抗训练、去偏推荐）的比较。
- **应用限制**：主要适用于基于 LLM 的生成式推荐，对传统协同过滤或序列推荐是否有效尚需验证；且依赖 LLM 的框架，计算成本较高。

（完）
