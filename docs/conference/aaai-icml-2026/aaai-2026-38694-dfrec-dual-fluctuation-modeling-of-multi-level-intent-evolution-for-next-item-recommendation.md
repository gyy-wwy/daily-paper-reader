---
title: "DFRec: Dual Fluctuation Modeling of Multi-level Intent Evolution for Next-Item Recommendation"
title_zh: DFRec：多层级意图演化的双波动建模用于下一项推荐
authors: "Nengjun Zhu, Lingdan Sun, Qi Zhang, Jian Cao, Hang Yu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38694/42656"
tags: ["query:rec-sys"]
score: 9.0
evidence: 多层级意图演化的双波动建模用于下一项推荐
tldr: 用户顺序行为背后意图的演变对下一项推荐至关重要，但现有方法难以捕捉意图变化的确切时机和幅度。本文提出DFRec框架，通过双波动建模对多层级意图演化进行精确刻画。在多个真实推荐数据集上，DFRec显著优于现有顺序推荐方法，具有较好的鲁棒性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有方法难以精确捕捉用户意图变化的时间和幅度。
method: 提出DFRec，通过双波动建模多层级意图演化。
result: 在多个顺序推荐数据集上取得最佳性能。
conclusion: 精确建模意图变化的时间和幅度能有效提升顺序推荐效果。
---

## Abstract
User sequential behaviors are driven by a variety of complex and evolving intents. Capturing the dynamic change of user intents has become critical yet challenging in the next-item recommendation. Existing studies usually model the transition relationships among multiple intents within a session or integrate temporal information to capture the dynamic evolution of user intents. However, they struggle to identify the precise timing and magnitudes of intent changes, leading to ambiguity in providing consistent or violated recommendations and ultimately yielding subpar performance. To this end, we propose a novel framework called Dual Fluctuation Modeling of Multi-level Intent Evolution for Next-Item Recommendation (DFRec) in this paper. DFRec explicitly identifies the user intent changes and further quantifies the magnitude of the changes. Specifically, we assume that a user's intent fluctuates around an inherent intent, with the magnitude of fluctuations indicating the extent of changes in user intents. Thus, we design an Emerging Intent Generation Module that employs a normal distribution with dynamic variance to capture intent fluctuations at each time step. Furthermore, we introduce a dual-layer dynamic variance update mechanism to capture fluctuation characteristics at different temporal levels, enhancing the representation of possible emergent intents. Extensive experiments on three real-world datasets verify DFRec's superiority over state-of-the-art baselines.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：在下一项推荐任务中，用户行为由复杂且不断演变的意图驱动。现有意图建模方法虽然能捕捉会话内多头意图的转换或引入时间信息，但无法精确识别用户意图发生变化的**确切时机**和**变化幅度**，导致推荐结果含糊不清、性能不佳。
- **整体含义**：本文提出 DFRec 框架，通过显式建模用户意图的波动（fluctuation）并量化波动的幅度，从而更精准地刻画用户意图的动态演化，提升下一项推荐的效果。

## 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程
- **核心思想**：假设用户的意图围绕一个固有意图（inherent intent）波动，波动的幅度反映意图变化的程度。通过生成多个连续的项目段，并利用每个时间步的意图分布来捕捉新兴意图。
- **关键技术细节**：
  - **新兴意图生成模块（EIGM）**：利用动态方差的正态分布 \( \mathcal{N}(\mu_t, \sigma_t^2) \) 建模意图波动。其中 \(\mu_t\) 由**狄利克雷分布**从项目类别中得到，作为用户固有意图；方差 \(\sigma_t^2\) 代表当前时刻意图波动的幅度。通过采样得到每个时刻的新兴意图表示。
  - **双意图波动建模（DIFM）**：
    - **短期方差**：基于当前步与上一步的残差（如物品嵌入差、隐藏状态差）更新，反映即时波动。
    - **长期方差**：通过滑动窗口机制，计算窗口内各步与固有意图的平方偏差之和，反映较长周期内的波动趋势。
    - 最终方差 \(\sigma_{it} = \sigma_{short} + \sigma_{long}\)。
  - **意图融合模块（IFM）**：将连续项目段（从第一个项开始逐步添加）与对应时刻的新兴意图表示拼接后，通过多头注意力与 Lp-pooling 生成最终会话表示 \(s_{hybrid}^i\)。
- **算法流程**（文字说明）：
  1. 输入会话序列，对物品和类别进行嵌入。
  2. 基于狄利克雷分布计算固有意图 \(\mu_0\)。
  3. 利用 GRU 编码历史交互，并计算短期方差和长期方差。
  4. 从正态分布采样得到每个时刻的新兴意图表示。
  5. 生成多粒度项目段，融合新兴意图后经注意力得到会话表示。
  6. 用会话表示计算每个候选项目的分数，使用交叉熵损失训练。

## 3. 实验设计
- **数据集**：三个真实电子商务数据集
  - Yoochoose（RecSys’15 挑战赛数据集，12 个类别）
  - Jdata（京东挑战赛数据集，79 个类别）
  - Diginetica（电子商务数据集，982 个类别）
- **基准方法**（共 9 种）：
  - 传统方法：BPR
  - 典型会话推荐：NARM、SASRec
  - 图神经网络：SR-GNN、GCE-GNN、SHARE
  - 意图导向方法：DIDN、Atten-Mixer、MiaSRec
- **评价指标**：Recall@K 和 NDCG@K（K=5,10）

## 4. 资源与算力
- **文中未明确说明**：未提及使用的 GPU 型号、数量、训练时长、内存等具体算力资源。因此无法总结具体硬件配置。

## 5. 实验数量与充分性
- **实验组数**：包括
  - 整体性能对比（3 个数据集 × 9 个基线，每个指标给出多组结果）
  - 消融实验（移除 EIGM、DIFM-L、DIFM-S、IFM 等 4 个变体，在 3 个数据集上对比）
  - 参数分析（窗口大小从 1 到 6，在 2 个数据集上展示）
  - 多样性评估（在 Yoochoose 和 Jdata 上展示多样性随训练变化）
  - 案例研究（随机抽取 3 个会话可视化意图波动曲线）
- **充分性评价**：
  - **充分**：覆盖了多种基线（9 种，含不同流派）和多个数据集（类别数量差异大），消融实验验证了每个模块的作用，参数分析讨论了关键超参数的影响，多样性分析从另一个角度说明模型效果。
  - **客观公平**：超参数按论文标准设置，所有基线调至最佳性能，使用 Wilcoxon 检验验证显著性（p<0.05），实验条件一致。

## 6. 论文的主要结论与发现
- DFRec 在所有三个数据集上均显著优于所有基线，例如在 Jdata 上 Recall@5 比最佳基线 MiaSRec 高 1.07%，比最差方法 NARM 高 12.19%。
- 消融实验表明，移除任何模块（EIGM、DIFM-L、DIFM-S、IFM）都会导致性能下降，证明每个组件均有效。
- 窗口大小对性能有影响，存在最优值（Yoochoose 为 2，Jdata 为 3）。
- 多样性指标随训练提高，表明模型能有效捕捉新兴意图信号。
- 案例研究显示，不同数据集上的意图波动曲线与类别数量相关：类别少的 Yoochoose 波动平缓，类别多的 Diginetica 波动剧烈。

## 7. 优点
- **方法亮点**：
  - 创新性地使用正态分布（动态方差）来建模意图波动的不确定性，将波动幅度显式量化为方差，这是现有方法未做到的。
  - 双方差更新机制（短期+长期）从不同时间粒度捕获意图演化，更全面。
  - 多粒度项目段生成与新兴意图表示融合，融合了时间维度上的层次信息。
- **实验亮点**：
  - 在多个类别数量差异大的真实数据集上验证，证明了模型在不同场景下的鲁棒性。
  - 进行了充分的消融和参数分析，清晰展示了各模块贡献。
  - 提供公开代码（GitHub），可复现性强。

## 8. 不足与局限
- **实验覆盖**：仅使用三个电子商务数据集，未涉及新闻、视频、社交等其他推荐场景，泛化性有待验证。
- **偏差风险**：狄利克雷分布假设固有意图稳定，可能不适用于意图频繁剧烈变化的场景（如突发兴趣转移）。
- **参数敏感**：滑动窗口大小需要手动搜索最优值，不同数据集最优值不同，实际应用需调参。
- **计算复杂度**：文中给出了复杂度分析（O(TK+Td²+T Wd+T²d+Td)），但未与基线对比具体训练/推理时间，难以判断实际效率。
- **未提及硬件资源**：缺乏对算力消耗的具体报告，不利于其他研究者复现时评估资源需求。
- **未对比大规模语言模型（LLM）**：论文在结论中提及未来可结合 LLM，但当前未进行相应实验，这也是一个未覆盖的方向。

（完）
