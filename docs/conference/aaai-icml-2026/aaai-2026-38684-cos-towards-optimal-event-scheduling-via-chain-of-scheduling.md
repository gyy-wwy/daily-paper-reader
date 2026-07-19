---
title: "CoS: Towards Optimal Event Scheduling via Chain-of-Scheduling"
title_zh: CoS：基于调度链的优化事件调度
authors: "Yiming Zhao, Jiwei Tang, Shimin Di, Libin Zheng, Jianxing Yu, Jian Yin"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38684/42646"
tags: ["query:rec-sys"]
score: 8.0
evidence: 事件调度推荐，旅游相关约束
tldr: 事件社会网络中的行程推荐需同时满足用户偏好、时间和地理约束，属于NP难问题。本文提出CoS框架，通过分解为探索、验证和集成三个阶段，激活大语言模型的调度能力。实验表明CoS在效率和效果上优于传统方法，为旅游行程规划提供了新思路。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 事件调度推荐面临时间、地理约束，且为NP难问题，现有方法在效率与效果间难以平衡。
method: 提出Chain-of-Scheduling框架，将调度任务分解为探索、验证和集成三个原子阶段，引导大语言模型生成调度方案。
result: 在事件社会网络数据集上，CoS在推荐准确率和效率上均显著优于基线方法。
conclusion: 大语言模型通过结构化引导可有效解决事件调度推荐问题。
---

## Abstract
Recommending event schedules is a key issue in Event-based Social Networks (EBSNs) in order to maintain user activity. An effective recommendation is required to maximize the user's preference, subjecting to  both time and geographical constraints. Existing methods face an inherent trade-off among efficiency, effectiveness, and generalization, due to the NP-hard nature of the problem. This paper proposes the Chain-of-Scheduling (CoS) framework, which activates the event scheduling capability of Large Language Models (LLMs) through a guided, efficient scheduling process. CoS enhances LLM by formulating the schedule task into three atomic stages, i.e., exploration, verification and integration. Then we enable the LLMs to generate CoS autonomously via Knowledge Distillation (KD). Experimental results show that CoS achieves near-theoretical optimal effectiveness with high efficiency on three real-world datasets in a interpretable manner. Moreover, it demonstrates strong zero-shot learning ability on out-of-domain data.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究问题**：在基于事件的社交网络（EBSN，如 Meetup、Eventbrite）中，为用户推荐一组可参与的事件并形成可行的时间表（schedule），需同时满足用户偏好、时间窗口约束和地理可达性约束。这是一个 NP-hard 的组合优化问题。
- **现有挑战**：现有方法（精确搜索、启发式算法、图神经网络、大语言模型）在效率、效果、泛化性和可解释性之间存在固有 trade-off。特别是大语言模型直接用于规划时，常出现冗余推理、陷入无效循环、生成冲突时间表等问题。
- **论文目标**：提出 CoS 框架，激活大语言模型的调度能力，实现高效、高质量、可解释且具备零样本泛化能力的事件调度。

## 2. 论文提出的方法论

### 核心思想
- 将事件调度任务标准化为三个原子化阶段：**探索（Exploration）**、**验证（Verification）** 和 **集成（Integration）**，形成结构化的推理链（Chain-of-Scheduling, CoS），引导大语言模型逐步生成高质时间表。
- 通过**知识蒸馏（Knowledge Distillation）**，将精确搜索算法（如动态规划）产生的高质量调度知识和推理过程，蒸馏到小规模大语言模型（如 Qwen2.5-7B、Mistral-7B）中，使其能够自主生成 CoS。

### 关键技术细节
1. **CoS 构建**：
   - **探索**：枚举 top-k 个可行时间表（可由动态规划/网格搜索生成），作为候选解。
   - **验证**：计算每个候选时间表的总效用分数（即用户偏好之和），显式输出推理轨迹（如“2+3=5”）。
   - **集成**：选择验证后效用最大的时间表作为最终输出。
2. **知识蒸馏**：
   - 将搜索算法生成的 CoS 推理轨迹（含自然语言解释）与输入问题配对，构建监督微调数据集 \( D_{SFT} = \{(x_i, y_i^{CoS})\} \)。
   - 使用 LoRA 低秩适配进行监督微调，优化交叉熵损失，使 LLM 学会生成类似的 CoS 推理链。
3. **后处理**：
   - 微调后的 LLM 仍可能输出冲突时间表，采用轻量级局部搜索后处理：发现冲突时，用替代事件替换后继事件，消除冲突并最小化效用损失。

### 算法流程（文字说明）
- 输入：事件集合、用户偏好。
- 步骤1：利用搜索算法（教师）生成候选时间表及对应 CoS 推理文本。
- 步骤2：用上述数据对 LLM 进行监督微调。
- 步骤3：微调后的 LLM 针对新输入自主生成 CoS，包含探索、验证、集成三步。
- 步骤4：后处理校验并修正可能的冲突。

## 3. 实验设计

### 数据集与场景
- 从 Meetup 网站爬取三个城市数据：纽约（74411 事件，45854 用户）、华盛顿（81395 事件，44742 用户）、伦敦（218773 事件，22381 用户）。
- 数据时间跨度：2 年。按 4:1 划分训练/测试。测试时每个城市进行超过 1600 次评估，取平均。
- 效用分数：使用 BERT 计算事件描述与用户兴趣的语义相似度（0~1）。

### 基准方法
- **标准预训练 LLM**：DeepSeek-R1、DeepSeek-V3、Qwen-Max、Qwen-Plus。
- **组合优化**：网格搜索、动态规划（DP）、贪心算法、遗传算法。
- **深度学习方法**：GOOSE（GNN 启发式）、GNN-DRL（图神经网络+强化学习）。
- **LLM + 多步推理**：上述 LLM 加 Chain-of-Thought（CoT），以及 CoS 方法。

### 评价指标
- **效用分数**：用户偏好总和（越高越好）。
- **延迟**：推理时间（秒）。
- **冲突率**：时间表内事件间的时间/距离冲突比例（仅报告 LLM 方法，后处理前）。

## 4. 资源与算力

- **论文明确说明**：
  - 使用 LoRA 训练，rank=8，alpha=16，学习率 1e-5，batch size=2，epochs=3，最大长度 32768。
  - 训练在 **2 块 NVIDIA A800-SXM4-80GB GPU** 上完成。
- 未提及训练时长，但根据参数规模（7B 模型）和 LoRA 配置，推测训练时间在数小时以内。

## 5. 实验数量与充分性

- **主要实验**：在三个城市数据集上对比所有基准方法，报告效用、延迟、冲突率（表1）。
- **零样本泛化实验**：在纽约训练，直接测试华盛顿和伦敦（表2）。
- **消融实验**：分别去除探索、验证、集成和后处理，观察效用和冲突率变化（表3）。
- **超参数敏感性分析**：k 从 1 到 5，观察效用和延迟变化（图4）。
- **充分性评估**：
  - 对比方法覆盖全面（精确、启发式、GNN、LLM 系列）。
  - 消融验证了每个组件的重要性。
  - 零样本测试验证了泛化能力。
  - 但缺少对更大规模 LLM（如 70B）的微调对比，以及更多随机种子的统计显著性检验。

## 6. 主要结论与发现

- **CoS 在所有数据集上效用均接近最优解（>90%），延迟显著低于精确算法**（如 DP 需 8~24 秒，CoS 仅 1~5 秒）。
- **冲突率远低于其他 LLM 方法**（15~41% vs 84~99%），表明 CoS 有效避免了无效时间表。
- **零样本泛化能力突出**：在未见过的城市上，效用比 GNN 方法高 50% 以上，证明学到可迁移的时空调度语义。
- **消融实验**：探索环节最重要（效用下降 7%），验证和集成均不可或缺。
- **超参数 k 选择 3 即可兼顾效果与效率**。

## 7. 优点

- **方法论亮点**：
  - 将 NP-hard 调度问题拆解为 LLM 擅长的结构化推理步骤，提升可解释性。
  - 知识蒸馏将精确算法能力注入小模型，平衡效率与效果。
  - 后处理轻量，解决 LLM 幻觉问题。
- **实验设计亮点**：
  - 三个不同规模的城市数据集，增强了结论鲁棒性。
  - 零样本测试证明了泛化能力，这是以往方法缺失的。
  - 对 LLM 方法报告冲突率，揭示其失效根源，分析深入。
- **实用价值**：在现实 EBSN 中有部署潜力，延迟低、可解释强。

## 8. 不足与局限

- **蒸馏依赖教师模型质量**：若教师（DP）本身受限于输入规模过大而无法求解，CoS 的训练数据质量会下降。
- **LLM 幻觉仍未完全消除**：即使后处理降低了冲突率，但仍有少量冲突残留（表1 中 9.8%~41%）。
- **实验覆盖有限**：
  - 仅测试了 7B 规模模型，未探索更大参数模型的效果与速度权衡。
  - 未报告统计显著性（如 t 检验）及多次运行方差。
  - 效用分数模拟依赖 BERT 语义相似度，可能与真实用户偏好有差异。
- **应用限制**：CoS 需针对新领域重新蒸馏（如旅游行程推荐），零样本虽好但可能不如领域内精调。

（完）
