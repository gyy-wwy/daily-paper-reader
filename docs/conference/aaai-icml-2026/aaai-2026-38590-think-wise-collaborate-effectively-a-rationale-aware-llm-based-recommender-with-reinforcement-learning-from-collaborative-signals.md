---
title: "Think Wise, Collaborate Effectively: A Rationale-Aware LLM-Based Recommender with Reinforcement Learning from Collaborative Signals"
title_zh: 智慧思考，高效协作：基于协作信号强化学习的理性感知大语言模型推荐系统
authors: "Chung Park, Taesan Kim, Hyeongjun Yun, Dongjoon Hong, Junui Hong, Kijung Park, MinCheol Cho, Min Sung Choi, Jihwan Seok, Jaegul Choo"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38590/42552"
tags: ["query:rec-sys"]
score: 7.0
evidence: 融合协作信号和强化学习的大语言模型推荐系统
tldr: 本文针对大语言模型推荐缺乏协作信号的问题，提出TWiCE-Rec。该方法通过强化学习将协同过滤信号融入LLM推理过程，生成基于用户-物品交互的解释性推荐。实验表明，TWiCE-Rec在推荐准确性和解释质量上均优于仅使用LLM或CF的方法，实现了推理与行为的有效统一。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 大语言模型推荐性能受限于缺乏用户-物品交互的协作模式。
method: 使用强化学习融合协同过滤信号，构建理性感知推荐器。
result: 在推荐任务上，TWiCE-Rec在准确率和解释性上均取得提升。
conclusion: 结合协作信号能显著增强LLM推荐系统的表现。
---

## Abstract
Large Language Models (LLMs) have recently emerged as powerful reasoning engines in recommender systems, generating natural-language explanations that foster user engagement.
However, their recommendation performance remains limited, as they lack exposure to collaborative user-item interaction patterns.
In contrast, collaborative filtering (CF) models achieve strong performance by learning from these behavioral patterns at scale.
To unify the strengths of both paradigms, we propose TWiCE-Rec (Think Wise, Collaborate Effectively), a rationale-aware LLM-based recommender that incorporates collaborative user-item interactions.
In the first stage, we construct a rationale dataset by applying in-context learning with self-annotated curation. 
A state-of-the-art LLM is guided to generate persuasive rationales that explain the causal relationship between the user’s interaction sequence and the ground-truth next item, resulting in a curated post-hoc training dataset.
In the second stage, we perform multi-task instruction-tuned adaptation—based on the rationale-augmented training dataset—comprising item description generation and both non-reasoning and reasoning-based sequential recommendation, to equip the LLM with the ability to generate rationales that reflect how user preferences align with item characteristics.
Finally, we aim to enhance the LLM’s recommendation performance by incorporating user-item interaction patterns derived from the CF-Rec model.
To achieve this, we propose a confidence-weighted reinforcement learning strategy that adjusts rewards in proportion to both the LLM’s prediction alignment with the ground-truth and the confidence from the pretrained CF-Rec model.
Our method outperforms both CF- and LLM-Rec models on Amazon datasets in terms of recommendation performance and rationale quality. 
In an online A/B test, it achieved about 8% higher click-through rate than existing models, demonstrating practical value.

---

## 论文详细总结（自动生成）

# 论文中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **问题**：大语言模型（LLM）在推荐系统中能够生成自然语言解释，但推荐性能受限，因为缺乏对用户-物品协作交互模式的接触。相比之下，协同过滤（CF）模型通过大规模学习行为模式取得了强性能，但无法提供可解释的理由。
- **目标**：融合LLM的推理能力与CF的协作信号，实现既准确又可解释的推荐。
- **整体含义**：提出TWiCE-Rec框架，通过三个阶段（理性数据构建、知识对齐、协作偏好对齐）将协作信号注入LLM，提升推荐准确率和解释质量。

## 2. 方法论：核心思想、技术细节、公式或算法流程

### 核心思想
- 将LLM与CF-Rec结合：先通过指令微调让LLM学会生成推荐理性，再通过强化学习（GRPO）利用CF模型提供的协作信号进一步优化。

### 关键技术细节

#### 阶段1：自标注理性构建
- 基于用户交互序列 $S_u$ 和真实下一物品 $i^+$，通过上下文学习（in-context learning）引导LLM生成后验理性 $r_{i^+}$。
- 使用自标注筛选：LLM自动评估因果逻辑，过滤掉弱相关或不合理的理性，确保数据集质量。

#### 阶段2：理性感知知识对齐
- 多任务指令微调（LoRA），包含三项任务：
    1. **物品描述生成**：从标题生成描述，学习物品语义。
    2. **非推理顺序推荐**：仅预测下一物品，学习物品共现模式。
    3. **基于推理的顺序推荐**：同时生成理性和推荐物品（使用阶段1的数据），采用“理由优先”解码（先输出`<think>`再输出`<item>`），注入链式思维。

#### 阶段3：协作偏好对齐
- 采用基于可验证奖励的强化学习（RLVR），具体使用**组相对策略优化（GRPO）**。
- 奖励函数设计：
    - **协作引导奖励**：结合物品匹配奖励（$\rho_{\text{match}}$ 二进制）和协同过滤奖励（$\rho_{\text{cf}}$ 来自预训练CF模型如SASRec的置信度）。公式：
      \[
      \rho_{\text{cg}}^k = 
      \begin{cases}
      1 + \lambda \cdot \rho_{\text{cf}}^k & \text{if } \rho_{\text{match}}^k=1 \text{ and } \rho_{\text{cf}}^k \ge \tau \\
      1 & \text{if } \rho_{\text{match}}^k=1 \text{ and } \rho_{\text{cf}}^k < \tau \\
      \rho_{\text{cf}}^k & \text{if } \rho_{\text{match}}^k=0 \text{ and } \rho_{\text{cf}}^k \ge \tau \\
      0 & \text{otherwise}
      \end{cases}
      \]
    - **格式奖励**：检查输出是否包含正确标签（`<think>`, `</think>`, `<item>`, `</item>`）和结构正确性。
- 最终奖励为两者之和，再通过组内标准化得到优势，用于策略更新。

## 3. 实验设计

- **数据集**：Amazon Review 2023的8个领域，分为两类：
    - Type A（高频需求驱动）：Grocery and Gourmet Food, Industrial and Scientific, Musical Instruments, Office Products。
    - Type B（低频偏好驱动）：Amazon Fashion, Baby Products, Clothing Shoes and Jewelry, Health and Household。
- **评价指标**：HR@1, HR@5, NDCG@5，采用留一法评估。
- **基准方法**：约20个基线，包括CF类（SASRec, BERT4Rec, TransRec等）和LLM类（InstructRec, Rec-R1, Rec-SAVER等）。
- **实验设置**：LLM骨干使用Gemma-3-1B-it；对比了1B、4B、12B规模趋势（附录J）。

## 4. 资源与算力

- **文中明确提及**：未详细说明训练使用的GPU型号、数量或训练时长。仅在致谢中提到SK Telecom提供GPU集群，以及IITP和NRF的资助。
- 未给出具体算力消耗，仅说明在线推理延迟约为138ms/call。

## 5. 实验数量与充分性

- **数量**：共进行多组实验：
    - 主实验：8个领域，对比约20个基线，覆盖HR@1/5和NDCG@5。
    - 理性质量评估：从8个领域各采样300个样本，共2400个样本，使用GPT-4o进行4级评分。
    - 消融实验：比较5种模型变体（Base、+Stage2、+Stage3、+Stage2+基本GRPO、完整模型），在两个域展示图表（附录D涵盖全部）。
    - 超参数敏感性分析：对$\lambda$（0~5）进行实验。
    - 在线A/B测试：两周内对比随机控制和传统ML模型，指标为点击率（CTR）。
    - 理性影响实验：对比有无理性生成（训练/推理阶段）的效果。
- **充分性与公平性**：
    - 覆盖多领域、多基线，采用标准评估协议（留一法，候选集配对负样本），结果表格清晰。
    - 消融实验证明各阶段贡献，敏感性分析验证超参数影响。
    - 在线测试验证实际效果。
    - 总体较为充分、客观。

## 6. 主要结论与发现

- **推荐性能**：TWiCE-Rec在HR@1上全面超越所有基线，在8个域上相比最佳基线提升约2%~63%；在HR@5和NDCG@5上接近最优，基本在1%p以内。
- **理性质量**：在2400样本中，75.71%达到最高分（3分），高于基线（约4%p）。
- **各阶段贡献**：完整模型（Stage2+Stage3）最优，仅Stage2或仅Stage3不足；加入协作信号（Stage3）显著优于仅使用物品匹配奖励。
- **超参数$\lambda$**：$\lambda=3$附近最优，过高会过度模仿CF。
- **在线结果**：CTR 11.49%，优于随机（9.30%）和ML模型（10.67%），提升约8%。
- **理性作用**：在训练和推理中引入理性均能提升推荐准确性。

## 7. 优点

- **方法创新**：首次将CF置信度作为强化学习奖励信号，结合物品匹配和格式约束，设计精细的奖励函数。
- **多阶段训练框架**：自标注理性 → 多任务知识对齐 → 协作偏好对齐，逻辑清晰，逐步增强模型能力。
- **实验结果全面**：覆盖多个域，对比大量基线，消融实验和敏感性分析完整，并有在线验证。
- **可解释性与性能统一**：同时提升推荐准确性和解释质量，具有实际部署价值。

## 8. 不足与局限

- **实验覆盖**：仅使用Amazon数据集，未在更多平台（如MovieLens、Yelp）验证，泛化性存疑。
- **算力信息缺失**：未报告训练所需的资源（GPU型号、数量、时间），难以复现资源需求。
- **CF模型选择单一**：仅使用SASRec作为CF模型，未探索其他CF模型（如BERT4Rec）的奖励信号影响。
- **超参数敏感性**：$\lambda$和$\tau$在不同域可能需要调整，文中仅对$\lambda$做部分敏感性分析，未深入讨论$\tau$。
- **在线测试规模**：随机组仅30K印象，模型组15K印象，样本量较小，置信区间未报告。
- **理性评估依赖GPT-4o**：自动评估可能存在偏差，缺乏人工评估。
- **理性质量评估的公平性**：对比基线中仅包含Rec-SAVER和Rec-R1，未比较其他LLM-Rec（如RecExplainer）。

（完）
