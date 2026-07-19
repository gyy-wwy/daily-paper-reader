---
title: "NP-MiSR: Neural Process-based Multi-Interest Learning for Session-Based Recommendation"
title_zh: 基于神经过程的多兴趣学习用于会话推荐
authors: "Jun Bao, Junbo Wang, Yiheng Jiang, Xiangfeng Liu, Mingyang Lv, Yuanbo Xu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38463/42425"
tags: ["query:rec-sys"]
score: 9.0
evidence: 基于会话的推荐，多兴趣学习
tldr: 现有会话推荐的多兴趣方法需要预定义兴趣数量，无法适应不同用户的偏好模式。本文提出NP-MiSR，基于神经过程自适应学习每个会话的兴趣数量，并利用相关会话的信息。实验表明该方法在多个基准上表现优异，为会话推荐提供了灵活的兴趣建模方案。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有会话推荐的多兴趣方法需要预定义兴趣数量，缺乏适应性。
method: 利用神经过程自适应学习每个会话的兴趣数量和表示，并集成相关会话信息。
result: 在多个会话推荐数据集上，NP-MiSR在推荐准确性上超越了固定兴趣数量的方法。
conclusion: 神经过程能灵活建模多兴趣，提升会话推荐对用户偏好的适应能力。
---

## Abstract
Session-based recommendation (SBR) aims to provide users with satisfactory suggestions via modeling preferences based on short-term, anonymous user-item interaction sequences. Traditional single interest learning methods struggle to align with the diverse nature of preferences. Recent advances resolved this bottleneck by learning multiple interest embeddings for each session. However, due to the pre-defining scheme of interest quantity (e.g. the number of interests), these approaches are deficient in adaptive ability towards distinctive preference patterns across different users. Moreover, these methods rely solely on the current session and ignore useful information from related ones. The short-term property of sessions would magnify the insufficient representation issue. To address these limitations, we propose a Neural Process-based Multi-interest learning framework for Session-based Recommendation, namely NP-MiSR. To be specific, our method enables adaptive multi-interest representation learning through two complementary mechanisms: 1) Neural Process-based Intra-session interest modeling: We employ Neural Processes to model the distribution of interests within a session, where the fixed interest configurations are no longer needed. 2) Cross-session context fusion: We extract interest distributions of similar sessions as contextual priors to refine the current session’s interest representation. Extensive experiments on three datasets demonstrate that our method consistently outperforms state-of-the-art SBR approaches with an average improvement of 38.8%. Moreover, the few-shot learning task reveals that NP-MiSR achieves a surprisingly favorable efficiency v.s. performance trade-off where utilizing only 10% of the training data attains 95% of the recommendation performance.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）
- **问题**：现有基于会话的推荐（SBR）方法中，多兴趣学习（Multi-Interest Learning）虽然优于单兴趣方法，但存在两个关键缺陷：
  1. 需要**预定义兴趣数量**（如固定K个兴趣），无法适应不同用户/会话中兴趣模式多样且动态变化的特点，缺乏灵活性。
  2. 仅依赖**当前会话信息**，忽略了**相似会话**的上下文信息，导致短会话中物品表示不充分。
- **动机**：提出一种能**自适应**学习多兴趣分布、并能融合跨会话上下文的方法，同时引入**不确定性校准**以应对用户意图的演化。

### 2. 论文提出的方法论
#### 核心思想
- 提出 **NP-MiSR** 框架，首次将**神经过程（Neural Processes, NPs）**引入会话推荐，实现功能分布级的多兴趣建模。
- 通过**两个互补机制**：
  - **基于神经过程的会话内兴趣建模**：用NP建模兴趣的函数分布，不需要预设兴趣数量。
  - **跨会话上下文融合**：从相似会话中提取兴趣分布作为先验，增强当前会话表示。

#### 关键技术细节
1. **会话表示学习层**：使用GCE-GNN（或其他SBR模型）提取每个会话的聚合表示 \( h_S \)。
2. **NP模型结构**：
   - **确定性编码器**：处理上下文对（当前会话的相似会话-标签对），输出确定性表示 \( r_C \)。
   - **潜变量编码器**：同时处理上下文和目标，输出潜变量分布（均值 \( \mu \)、方差 \( \sigma^2 \)），通过重参数化采样K个潜变量 \( z \)。
   - **解码器**：将目标会话表示、\( r_C \) 和 \( z \) 拼接，输出K个预测向量，经平均池化得到最终会话表示 \( h'_S \)。
   - **分类器**：用 \( h'_S \) 与物品嵌入点积+softmax得到推荐概率。
3. **训练模式**：
   - 在会话集中，对每个目标会话，选取Top-C个最相似会话（基于表示相似度）作为上下文。
   - 损失函数：ELBO（交叉熵 + KL散度），如公式(6)。
4. **推理模式**：
   - 只使用上下文编码器的先验分布采样，解码后输出推荐结果，包含不确定性（熵）。

### 3. 实验设计
#### 数据集
- **三个真实数据集**：Diginetica、RetailRocket、Nowplaying（统计信息见表1，平均长度5–7个物品）。
- **预处理**：与SR-GNN一致。

#### 基线方法（共7种）
- GCE-GNN（全局-局部图网络）
- CORE（一致表示空间）
- TAGNN（目标注意力图网络）
- A-Mixer（注意力混合器）
- MiaSRec（多意图感知）
- DMI-GNN（动态多兴趣图网络）
- Link（线性物品-物品模型）
- NP-MiSR（本文方法）

#### 评价指标
- **HR@N**、**MRR@N**、**COV@N**（覆盖率），N = 5,10,20。

### 4. 资源与算力
- 论文明确说明：所有实验在**一块 NVIDIA 3090Ti GPU（24GB VRAM）**上完成。
- 训练参数：epoch=20，batch size=100，嵌入/潜变量维度=100，每个实验重复3次以上。
- **未提及**训练总时长、GPU数量或更详细的能耗信息。

### 5. 实验数量与充分性
#### 实验种类
1. **主实验（表2）**：在3个数据集上对比7种基线的9个指标（共27个结果），全面验证优势。
2. **消融研究（表3）**：移除NP模块（w.o.NP）和会话表示层（w.o.SR），验证两个核心组件的贡献。
3. **NP模型通用性验证（图3）**：将NP集成到其他会话模型（GCE-GNN、DMI-GNN、SR-GNN、NARM），展示性能提升。
4. **少样本学习（图4）**：仅用10%训练数据，比较NP-MiSR、完整数据NP-MiSR、最优基线。
5. **超参数分析（图5）**：考察采样次数K（3,5,10）和上下文比例T（0.05,0.1,0.15）的影响。

#### 充分性评估
- **充分**：覆盖了多场景、多基线、多指标，消融和超参数研究完整。
- **客观公平**：所有基线使用相同预处理和评估设置，重复实验3次以上。
- **少量局限**：未在更多品类或长间隔会话数据集上验证。

### 6. 论文的主要结论与发现
1. **性能领先**：NP-MiSR在所有数据集全部9个指标上均优于所有基线，平均提升38.8%（表2）。
2. **少样本高效**：仅用10%训练数据即可达到全量训练最优基线的95%以上（甚至超越），展示极佳数据效率。
3. **模块有效性**：NP模块贡献平均39.58%提升；会话表示学习层不可或缺（移除后性能暴跌）。
4. **超参数敏感性**：更大的K（采样次数）通常提升性能；上下文比例T的效果因数据集分布而异。

### 7. 优点
- **创新性**：首次将神经过程用于会话推荐，解决了多兴趣数量自适应和不确定性建模的难题。
- **灵活性**：可替换任意会话表示模型（如GNN），兼容性强。
- **数据效率**：少样本学习能力突出，对冷启动场景有巨大潜力。
- **全面实验**：主实验、消融、通用性、超参数分析覆盖完整，结果稳健。

### 8. 不足与局限
- **计算开销**：NP模型在推理时需为每个会话检索相似上下文并采样K次，可能增加延迟（未详细对比耗时）。
- **上下文检索依赖**：相似会话选取基于表示相似度，未探讨更高效的检索策略（如聚类）。
- **基线与版本**：部分基线发布时间较早（2020–2023），缺少最近更强基线（如LLM增强方法）的比较。
- **数据集多样性**：三个数据集均来自电商/音乐领域，未验证在新闻、视频等领域的泛化性。
- **未公开资源消耗**：未提供完整训练时间、显存占用等工程细节，可复现性受限。

（完）
