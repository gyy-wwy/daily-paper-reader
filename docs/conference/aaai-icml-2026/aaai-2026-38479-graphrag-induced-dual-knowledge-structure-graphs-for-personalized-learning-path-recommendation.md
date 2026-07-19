---
title: GraphRAG-Induced Dual Knowledge Structure Graphs for Personalized Learning Path Recommendation
title_zh: 图RAG诱导的双知识结构图用于个性化学习路径推荐
authors: "Xinghe Cheng, Zihan Zhang, Jiapu Wang, Liangda Fang, Chaobo He, Quanlong Guan, Shirui Pan, Weiqi Luo"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38479/42441"
tags: ["query:rec-sys"]
score: 4.0
evidence: 知识图谱用于学习路径推荐
tldr: 学习路径推荐依赖先序关系，但标注成本高且单结构易导致学习阻塞。本文利用图RAG构建双知识结构图，融合概念与练习关系，在无需完整先序标注下生成个性化学习路径。实验表明该方法提高了学习效率并缓解了阻塞问题。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有学习路径推荐依赖难以获取的单一先序关系，且易导致学习阻塞。
method: 利用图RAG构建包含概念和练习关系的双知识结构图，生成学习路径。
result: 在多个教育数据集上，该方法优于传统先序依赖的方法。
conclusion: 双知识结构图有效缓解数据稀缺和学习阻塞问题。
---

## Abstract
Learning path recommendation seeks to provide students with a structured sequence of learning items (e.g., knowledge concepts or exercises) to optimize their learning efficiency. Despite significant efforts in this area, most existing methods primarily rely on prerequisite relations, which present two major limitations: (1) Prerequisite relations between knowledge concepts are difficult to obtain due to the cost of expert annotation, hindering the application of current learning path recommendation methods. (2) Relying on a single sequentially dependent knowledge structure based on prerequisite relations implies that a confusing knowledge concept can disrupt subsequent learning processes, which is referred to as blocked learning. To address these two challenges, we propose a novel approach, GraphRAG-Induced Dual Knowledge Structure Graphs for Personalized Learning Path Recommendation (KnowLP), which enhances learning path recommendations by incorporating both prerequisite and similarity relations between knowledge concepts. Specifically, we introduce a knowledge structure graph generation module EDU-GraphRAG that constructs knowledge structure graphs for different educational datasets, significantly improving the applicability of learning path recommendation methods. We then propose a Discrimination Learning-driven Reinforcement Learning (DLRL) module that utilizes similarity relations as fallback relations when prerequisite relations become ineffective, thereby alleviating the blocked learning. Finally, we conduct extensive experiments on three benchmark datasets, demonstrating that our method not only achieves state-of-the-art performance but also generates more effective and longer learning paths.

---

## 论文详细总结（自动生成）

# 论文结构化总结

## 1. 核心问题与整体含义（研究动机和背景）

学习路径推荐（LPR）旨在为学生提供结构化的学习项目序列（如知识概念或练习题），以优化学习效率。现有方法大多依赖**先序关系**（prerequisite relations），存在两大局限：  
- **先序关系获取困难**：依赖专家标注，成本高，难以大规模应用。  
- **单一结构导致学习阻塞**（blocked learning）：当学生混淆某个知识概念时，仅靠先序关系无法应对，后续学习会被打断。

本文提出 **KnowLP 框架**，通过**同时利用先序关系与相似关系**构建双知识结构图，自动生成知识图谱并采用区分学习驱动的强化学习来缓解阻塞，提升推荐效果。

---

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- 利用 **GraphRAG** 自动从知识概念名称中生成高质量的先序与相似关系图。  
- 基于生成的图，设计**区分学习驱动的强化学习（DLRL）模块**，当先序关系失效时，使用相似关系作为“回退”机制，模拟人类区分学习过程。

### 关键技术细节

#### 2.1 知识结构图生成模块（EDU-GraphRAG）
- **KC解释生成器**：基于 TextGrad 框架，通过迭代生成、评估、优化过程，为每个知识概念生成可靠的自然语言解释，作为后续图构建的源文档。  
- **EDU-GraphRAG**：将解释文档分块，用 LLM 提取实体和关系，构建全局实体知识图，再通过查询生成最终的先序关系图 \( G_A = (C, P, S) \)，其中 \(C\) 为概念集，\(P\) 为先序关系，\(S\) 为相似关系。

#### 2.2 学习路径生成模块（DLRL）
- **知识追踪（KT）**：使用 DIMKT 模型跟踪学生知识状态，并结合异构图表征（HGNN）融合练习和难度特征。  
- **三个智能体模拟区分学习**：
  - **先序智能体（P-Agent）**：基于先序关系顺序选择知识概念，使用 PPO 优化。状态为知识状态与学习目标拼接，奖励为学习目标 mastery 的提升。
  - **相似智能体（S-Agent）**：当先序智能体评估当前概念 mastery 提升低于阈值 \(\tau\) 时激活，利用相似关系搜索混淆概念并推荐子路径。
  - **难度智能体（D-Agent）**：根据学生 mastery 水平，选择难度最匹配的练习题（公式 \( m^* = \arg\min_m |\text{diff}(e_m) - h_{c_i}| \)）。
- **路径构建机制**：
  - **节点初始化**：从目标概念反向遍历先序关系，找到最合适的起始节点（基于学生已掌握的先修概念）。
  - **智能体切换**：若先序智能体检测到 mastery 提升不足，则切换到相似智能体，完成后恢复先序选择。

---

## 3. 实验设计

### 数据集
| 数据集 | KCs 数 | 学生数 | 记录数 | 说明 |
|--------|--------|--------|--------|------|
| Junyi | 835 | 525,061 | 21,460,249 | 提供完整先序关系图 |
| MOOCCubeX (MCX) | 443 | 629 | 17,447 | 先序关系不完整 |
| ASSISTments2009 (ASS09) | 167 | 4,217 | 346,860 | 无先序关系图 |

所有数据集均无相似关系。

### 对比方法
KNN、GRU4Rec、Actor-Critic、RL-Tutor、CSEAL、SRC、GEHRL、DLPR（SOTA）。

### 评估指标
学习效率提升 \( E_i = \frac{S_k^i - S_0^i}{S_{sup}^i - S_0^i} \)，其中 \(S_k^i\) 为学习结束时的 mastery 水平，\(S_0^i\) 为初始水平。

---

## 4. 资源与算力

- **硬件**：单张 NVIDIA GeForce RTX 4090 GPU，256 GB 内存。  
- **运行时间**（以 Junyi 为例）：
  - EDU-GraphRAG 模块：约 12 分 53 秒。
  - DLRL 模块：约 8 小时 23 分 26 秒（较 DLPR 的 7 小时 37 分 49 秒稍高，但性能提升显著）。
- 文中未明确说明训练轮次、batch size 等细节。

---

## 5. 实验数量与充分性

实验覆盖以下方面，整体较充分：
- **主实验**（表2）：在三个数据集、四个推荐步长（5/10/15/20）上与9个基线模型全面对比，KnowLP 均获最优。
- **消融实验**（表3）：移除相似智能体后性能显著下降，验证其必要性。
- **图质量对比**（图3-4）：在 Junyi 和 MCX 上可视化了原始图与生成图，并对比了使用原始图与生成图的推荐性能，两者接近。
- **案例研究**（RQ4）：展示 TextGrad 改善 KC 解释质量（区分混淆概念），并提供可解释的路径推荐理由。
- **仿真实验**（图7）：基于 KES-Junyi 模拟器与 SRC、GEHRL、DLPR 对比，KnowLP 表现更优。
- **阈值分析**（表5）：对比 \(\tau = 0.1, 0.01, 0.001, 0.0001\)，确定最优值 0.001。
- **运行时间分析**（表4）：给出模块耗时。

实验设计公平，基线均使用公开代码或原始设置，p 值 < 0.05 表明统计显著。

---

## 6. 主要结论与发现

1. **KnowLP 在所有数据集上均达到 SOTA**，尤其在更长路径（步长 20）上提升明显，证实了相似关系对缓解阻塞学习的作用。
2. **EDU-GraphRAG 生成先序图的质量接近人工专家标注**，且能覆盖缺失概念，极大降低了标注成本。
3. **相似智能体是关键创新**：消融实验显示去除后性能大幅下降；加入后长路径推荐效果更优。
4. **TextGrad 有效提升 KC 解释质量**，区分易混淆概念，提高了后续图构建的准确性。
5. **仿真实验验证了方法在动态学生场景下的适应性**。

---

## 7. 优点

- **创新性**：首次在 LPR 中同时建模先序与相似关系，模拟人类区分学习，理论基础扎实（Gagné 学习层级理论）。
- **自动化**：EDU-GraphRAG 无需人工标注，可适应不同数据集，增强实用性。
- **可解释性**：基于社区摘要可生成每条推荐路径的解释，提升用户信任。
- **鲁棒性**：在无先序图或图不完整的数据集上仍能构建有效图并取得优异性能。

---

## 8. 不足与局限

- **运行时间**：DLRL 模块训练耗时较长（数小时），对实时推荐场景可能不够友好。
- **依赖 LLM 质量**：EDU-GraphRAG 对 LLM 的解释能力要求高，若 LLM 存在幻觉可能影响图准确性。
- **阈值敏感性**：\(\tau\) 需手动调节，不同数据集最优值可能不同，缺少自适应策略。
- **数据集局限**：仅评估3个公开数据集（其中 ASS09 较老，KC 数少），未在更大规模或不同领域（如编程、医学）验证泛化性。
- **未讨论冷启动**：对于新学生或新 KC，基于历史日志的知识追踪可能失效，论文未涉及。
- **可重复性**：代码已开源，但未提供完整的超参数配置和随机种子，可能影响复现。

（完）
