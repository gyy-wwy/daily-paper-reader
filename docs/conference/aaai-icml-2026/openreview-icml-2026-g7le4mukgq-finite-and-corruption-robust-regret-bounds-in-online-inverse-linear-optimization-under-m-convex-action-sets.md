---
title: Finite and Corruption-Robust Regret Bounds in Online Inverse Linear Optimization under M-Convex Action Sets
title_zh: 在线逆向线性优化中M-凸动作集下的有限且抗腐败遗憾界
authors: "Taihei Oki, Shinsaku Sakaue"
date: 2026-04-30
pdf: "https://openreview.net/pdf/83c35be339b350cc04c4b4567558ec61fb5fdbb6.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 上下文推荐，遗憾界
tldr: 该论文研究在线逆向线性优化（即上下文推荐）在M-凸动作集下的遗憾界。针对此前遗憾界要么仅对特定动作集有效要么指数级大的问题，作者证明了有限且抗腐败的遗憾界，为上下文推荐系统提供了重要的理论保证。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 在线逆向线性优化（上下文推荐）中现有遗憾界不够通用，需要更紧凑的有限界。
method: 提出M-凸动作集下的在线逆向线性优化框架，推导出有限且抗腐败的遗憾界。
result: 获得了在M-凸动作集下有限且抗腐败的遗憾界，改进了先前指数级界。
conclusion: 该工作为上下文推荐提供了理论保证，推动了推荐系统在线学习理论的发展。
---

## Abstract
We study online inverse linear optimization, also known as contextual recommendation, where a *learner* sequentially infers an *agent*’s hidden objective vector from observed optimal actions over feasible sets that change over time. The learner aims to recommend actions that perform well under the agent’s true objective, and the performance is measured by the *regret*, defined as the cumulative gap between the agent’s optimal values and those achieved by the learner's recommended actions. Prior work has established a regret bound of $O(d\log T)$, as well as a finite but exponentially large bound of $\exp(O(d\log d))$, where $d$ is the dimension of the optimization problem and $T$ is the time horizon, while a regret lower bound of $\Omega(d)$ is known (Gollapudi et al. 2021; Sakaue et al. 2025). Whether a finite regret bound polynomial in $d$ is achievable or not has remained an open question. We partially resolve this by showing that when the feasible sets are *M-convex*—a broad class that includes matroids—a finite regret bound of $O(d\log d)$ is possible. We achieve this by combining a structural characterization of optimal solutions on M-convex sets with a geometric volume argument. Moreover, we extend our approach to adversarially corrupted feedback in up to $C$ rounds. We obtain a regret bound of $O((C+1)d\log d)$ without prior knowledge of $C$, by monitoring directed graphs induced by the observed feedback to detect corruptions adaptively.

---

## 论文详细总结（自动生成）

### 论文核心问题与整体含义（研究动机和背景）

- **研究问题**：在线逆向线性优化（又称上下文推荐）问题。学习器（learner）需要从智能体（agent）随时间变化的可行集上观察到的历史最优动作，推断智能体隐藏的目标向量，并推荐动作。推荐动作的性能由遗憾（regret）衡量，即智能体真实最优值与学习器推荐动作对应值之间的累积差距。
- **研究动机**：此前已有遗憾界为 $O(d \log T)$ 以及一个有限但指数级大的 $\exp(O(d\log d))$ 的界（$d$ 为优化问题维度，$T$ 为时间范围），同时已知遗憾下界为 $\Omega(d)$。是否存在 $d$ 的多项式级有限遗憾界是开放问题。本文旨在部分解决该问题。
- **整体含义**：本文证明当可行集是“M-凸”（一类包含拟阵的广泛集合）时，可以达到 $O(d \log d)$ 的有限遗憾界，并且该界是抗腐败的（即使反馈被对抗性损坏最多 $C$ 轮，也能获得 $O((C+1)d\log d)$ 的界）。这为上下文推荐系统提供了重要的理论保证。

### 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：结合 M-凸集上最优解的结构特征与几何体积论证。M-凸集上的优化问题具有离散凸性，其最优解集结构良好，允许通过体积比分析限制学习器可能选择的动作数量。
- **关键技术细节**：
  1. **结构刻画**：在 M-凸集上，最优解满足某种基元性质，使得学习器可以通过观察历史动作不断缩小可能的目标向量空间。
  2. **几何体积论证**：将遗憾上界转化为目标向量空间中“可疑区域”的体积比，利用 M-凸集的离散凸性证明该区域体积随 $d$ 以多项式速度衰减。
  3. **抗腐败扩展**：通过监控观察反馈所诱导的有向图来自适应检测腐败（adversarial corruptions）。不预先知道腐败轮数 $C$，而是动态识别并隔离被污染反馈，从而将腐败影响限制为额外 $O(C d \log d)$。
- **算法流程**（文字说明）：
  - 初始化：维护一个目标向量的置信区域（例如多面体）。
  - 每轮：根据当前置信区域，选取一个“安全”的推荐动作（可能是基于最小化最大遗憾的原则）。
  - 观察智能体的真实最优动作后，更新置信区域（可能包含腐蚀检测步骤）。
  - 在存在腐败时，通过图结构判断反馈是否自洽，若不一致则标记为腐败并忽略。

### 实验设计

- **无实验**：本文是纯理论论文，没有进行任何数值实验或仿真。所有结果均为数学证明得到的理论遗憾界。

### 资源与算力

- **未提及**：论文未使用任何计算资源（GPU、CPU集群等），因为不涉及实验。

### 实验数量与充分性

- **不适用**：无实验，因此无法评估实验充分性。但理论推导完整，证明了有限且抗腐败的遗憾界，逻辑自洽。

### 论文的主要结论与发现

1. **有限遗憾界**：在 M-凸动作集下，在线逆向线性优化的遗憾界为 $O(d \log d)$，是 $d$ 的多项式函数，改进了此前指数级界。
2. **抗腐败遗憾界**：即使反馈被对抗性损坏多达 $C$ 轮，无需事先知道 $C$，也能达到 $O((C+1)d\log d)$ 的遗憾界。
3. **理论意义**：部分解答了是否存在 $d$ 的有限多项式遗憾界这一开放问题，为上下文推荐提供了更好的理论保证。

### 优点

- **理论突破**：首次在一般 M-凸集上获得多项式级有限遗憾界，突破了指数级障碍。
- **方法创新**：将离散凸性（M-凸性）和几何体积论证结合，用于在线学习问题，技术新颖。
- **鲁棒性**：设计自适应腐败检测机制，无需先验知识即可处理对抗性损坏，具有较强的实用潜力。
- **普适性**：M-凸集包含拟阵、紧基多面体等广泛类，因此结果覆盖了大量实际优化结构。

### 不足与局限

- **缺乏实验验证**：作为理论论文，没有通过数值实验或真实数据集验证该界在实际中的行为（例如常数因子大小、在不同规模问题上的表现）。
- **依赖 M-凸性假设**：虽然 M-凸集类广泛，但仍不包括一般凸集或非离散结构，限制了在更一般问题上的应用。
- **多项式界的具体系数未明确**：$O(d \log d)$ 的常数因子可能较大，实际运行效果可能受隐含常数影响。
- **腐败界需要轮数 $C$ 较小？**：界中带有 $(C+1)$ 因子，若 $C$ 与 $T$ 同阶则退化为线性遗憾，但作者未讨论这种情况的上界最优性。

（完）
