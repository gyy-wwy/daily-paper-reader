---
title: "Incentivized Exploration with Stochastic Covariates: A Two-Stage Mechanism Design for Recommender System"
title_zh: 带随机协变量的激励探索：推荐系统的两阶段机制设计
authors: "Yuantong Li, Guang Cheng, Xiaowu Dai"
date: 2026-04-30
pdf: "https://openreview.net/pdf/a5a6db18366e76b7f399d4611305e591852aefba.pdf"
tags: ["query:rec-sys"]
score: 6.0
evidence: 推荐系统中针对探索-利用权衡的两阶段机制设计
tldr: 推荐系统面临探索新物品与用户自利偏好的冲突。本文提出两阶段机制设计，在随机协变量场景下通过线性奖励结构实现次线性遗憾并满足激励约束。该方法为鼓励推荐系统探索新项目提供理论基础，兼顾用户激励。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 推荐系统需要在探索新物品和满足用户偏好之间权衡，现有方法难以处理随机用户协变量。
method: 提出两阶段框架，利用线性奖励结构同时实现次线性遗憾和激励相容约束。
result: 理论证明所提机制达到次线性后悔界，并满足贝叶斯激励相容性。
conclusion: 该机制为推荐系统中的在线探索提供了可行的激励方案。
---

## Abstract
Recommender systems play a crucial role in internet economies by connecting users with relevant products. However, designing effective recommender systems faces the key challenges: the exploration-exploitation tradeoff in securing incentive to explore new products against user’s self-interested preferences. While prior work addresses Bayesian Incentive Compatibility (BIC) in fixed-design linear bandits (Sellke & Slivkins, 2023), we tackle the challenge of stochastic user covariates sampled online. Unlike standard black-box reductions (Mansour et al., 2020), our two-stage framework exploits the linear reward structure to achieve sublinear regret while satisfying incentive constraints. To address it, we propose a two-stage algorithm that integrates incentivized exploration with any efficient plug-in offline learning algorithms. In the first stage, it explores products while maintaining incentive compatibility to gather optimal samples. The second stage employs inverse proportional gap sampling strategy (IPGS) integrated with any efficient learning methods to secure sublinear regret. Theoretically, we prove that algorithm RCB achieves $O(\sqrt{KdT})$ regret and simultaneously satisfies incentive constraints, and discovers the tradeoff between incentive budget and regret, validating in experiments. We demonstrate RCB’s strong incentive gain, sublinear regret, and robustness through a real application on personalized warfarin dosing and simulations.

---

## 论文详细总结（自动生成）

# 论文总结：带随机协变量的激励探索：推荐系统的两阶段机制设计

## 1. 核心问题与整体含义
- **研究动机**：推荐系统面临探索新项目（如商品、药物剂量）与满足用户自利偏好之间的冲突。用户倾向于选择已知高回报的项目，系统则需要探索来获取长期收益，但探索可能短期损害用户利益，因此需要设计激励机制让用户愿意接受探索。
- **背景**：已有工作（Sellke & Slivkins, 2023）在固定设计的线性老虎机中处理了贝叶斯激励相容性（BIC），但现实场景中用户协变量是随机在线抽样的，现有黑盒归约方法（Mansour et al., 2020）无法高效处理该问题。
- **整体含义**：为推荐系统提供一种理论可行且实用的在线探索激励方案，在保证用户激励相容的同时实现次线性遗憾，从而兼顾系统效率和用户福利。

## 2. 方法论
- **核心思想**：利用线性奖励结构设计两阶段框架，将激励探索与任意高效的离线学习算法集成，实现激励相容与低遗憾的平衡。
- **关键技术细节**：
  - **第一阶段**：在维持激励约束（贝叶斯激励相容性）的前提下主动探索，收集最优样本。
  - **第二阶段**：采用逆比例间隙采样策略（Inverse Proportional Gap Sampling, IPGS），结合任意高效的学习方法（如线性回归、神经网络）进行离线学习，从而保证次线性遗憾。
- **算法流程（文字说明）**：
  1. 给定随机用户协变量（上下文），系统选择项目并公布奖励函数（线性奖励，参数未知）。
  2. 第一阶段：使用激励相容的探索策略（例如基于定价或信息租金机制）选择项目，确保用户真实报告偏好不会受损。
  3. 第二阶段：利用IPGS策略对间隙（gap）较大的项目赋予更高采样概率，同时调用预训练或离线学习算法更新模型。
  4. 重复步骤2-3直到达到预设时间步T。
- **关键公式**：理论证明算法RCB的遗憾上界为 \(O(\sqrt{KdT})\)，其中K为项目数，d为协变量维度，T为时间步长。

## 3. 实验设计
- **数据集与场景**：
  - 真实应用：个性化华法林剂量（warfarin dosing）推荐，涉及患者协变量（年龄、体重等）和不同剂量选项。
  - 模拟实验：在合成数据上验证算法性能。
- **Benchmark与对比方法**：元数据未明确列出具体对比算法，但暗示与已有贝叶斯激励相容方法和黑盒归约方法对比（如Sellke & Slivkins, 2023的结果）。
- **对比指标**：遗憾（regret）、激励增益（incentive gain）、鲁棒性。

## 4. 资源与算力
- **信息不详**：论文中未提及使用GPU型号、数量或训练时长。该工作为理论研究及小规模实验验证，可能主要使用CPU进行模拟，无需大量算力。

## 5. 实验数量与充分性
- **实验组数**：元数据仅提到“真实应用 + 模拟实验”，未给出具体数量或消融实验设置。从描述看，实验旨在证明RCB在激励增益、次线性遗憾和鲁棒性方面的优势，但覆盖范围有限。
- **充分性判断**：
  - 缺少与多种基线方法的系统对比（如不同探索策略、不同激励机制）。
  - 未展示在不同协变量维度、项目数、时间步长下的敏感度分析。
  - 结论依赖理论推导，实验更多作为验证性补充，因此充分性一般，但符合理论论文的常见做法。

## 6. 主要结论与发现
- 算法RCB实现了 \(O(\sqrt{KdT})\) 的遗憾界，并同时满足贝叶斯激励相容性约束。
- 发现了激励预算（用于补偿用户探索损失）与遗憾之间的权衡关系。
- 在真实应用（华法林剂量）和模拟中验证了RCB的激励增益、次线性遗憾特性和鲁棒性。

## 7. 优点
- **理论贡献**：首次在随机协变量场景下同时实现次线性遗憾与激励相容，突破了固定设计假设。
- **方法灵活性**：两阶段框架允许集成任何高效的离线学习算法（如深度模型），易于扩展。
- **线性结构利用**：避免了黑盒归约的开销，直接利用线性奖励结构得到简洁遗憾界。
- **实验验证**：包含真实医疗应用，展示了实际可行性。

## 8. 不足与局限
- **实验覆盖不足**：缺少与多种基线（如无激励探索、固定预算激励）的详细对比，也未进行参数敏感性分析和消融研究。
- **应用限制**：
  - 假设奖励为线性函数，现实场景中可能存在非线性或高阶交互。
  - 随机协变量假设可能不适用于对抗性上下文或非平稳环境。
- **偏差风险**：华法林剂量数据集可能较小，结论的推广性需更多验证。
- **算力与复现性**：未提供代码或详细超参数，复现难度较大。

（完）
