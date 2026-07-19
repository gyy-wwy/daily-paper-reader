---
title: Data-Centric Sequential Recommendation with Relation-Augmented Generation
title_zh: 基于关系增强生成的数据中心化序列推荐
authors: "Yichen Li, Yichen Tan, Yijing Shan, Haozhao Wang, Rui Zhang, Imran Razzak, Ruixuan Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38540/42502"
tags: ["query:rec-sys"]
score: 9.0
evidence: 序列推荐，数据增强
tldr: 该论文提出RaSR框架，用于序列推荐的数据中心化增强。现有方法要么高成本学习物品关系，要么生成模式缺乏可解释性。RaSR通过关系引导的生成模型，在保证可解释性的同时有效增强数据集质量，提升了序列推荐性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 序列推荐中数据质量提升面临关系挖掘困难，现有方法要么成本高要么可解释性差。
method: 提出关系引导的数据集增强与再生框架RaSR，利用生成模型自适应学习交互模式并注入关系信息。
result: 实验表明RaSR在多个序列推荐数据集上显著提升了推荐准确率。
conclusion: RaSR为数据中心化序列推荐提供了一种高效且可解释的解决方案。
---

## Abstract
Data-Centric Sequential Recommendation (DaCSR) has emerged as a promising technique that enhances dataset quality to better capture user preferences without increasing training complexity. However, mining item relations to improve data quality remains challenging due to the intricate nature of interaction sequences. Existing methods predominantly either: 1) optimize models to learn such item relations from fixed datasets at significant training cost, or 2) employ generative models to adaptively learn only interaction patterns, which lack interpretability and cannot guarantee effective data quality enhancement. In this paper, we pioneer a relation-guided dataset augmentation and regeneration framework for sequential recommendation called \textbf{RaSR}. This framework can significantly improve model performance on original datasets while maintaining training efficiency without modifying the model architecture. Specifically, we first preprocess user interactions to construct standardized sequential data and extract semantic representations via a Large Language Model (LLM). We then build a multi-relation graph with manually predefined metrics and semantic representations to generate augmented datasets. Finally, a relation-aware generator can produce regenerated datasets with both the multi-relation graph and the augmented dataset. To verify the effectiveness of RaSR, we conduct experiments on various backbone models and datasets, and achieve significant performance improvement compared to training the model only on the original dataset.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **研究背景**：序列推荐系统通过建模用户历史交互序列中的时序依赖来捕获动态偏好。现有方法主要分为两类：一是模型中心（MoCSR），即优化模型在固定数据集上的表现，但假设数据高质量；二是数据中心（DaCSR），旨在提升数据质量而不增加训练复杂度。然而现有数据中心方法要么高成本学习物品关系，要么仅用生成模型学习交互模式，缺乏可解释性且无法保证数据质量。
- **核心问题**：如何高效且可解释地利用物品关系对序列推荐数据集进行增强和再生，从而提升下游模型性能。
- **整体含义**：论文提出一种关系引导的数据集增强与再生框架（RaSR），在不修改任何骨干模型架构的前提下，通过构建多关系图并利用关系感知生成器生成高质量数据集，显著提升推荐准确率，为数据中心序列推荐提供了新思路。

### 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：利用物品之间的多种关系（共现、时序、语义）指导数据增强和再生，从而提供更高质量的训练数据。
- **技术流程**：
    1. **预处理与语义提取**：对原始交互序列进行清理（保留多于5条交互的用户），并利用大语言模型（LLM）提取每个物品的语义嵌入。
    2. **多关系图构建与数据增强**：
        - 共现矩阵（Co-occurrence）：滑动窗口统计物品对共现频率。
        - 时序矩阵（Temporal）：考虑物品先后位置，引入衰减因子λ加权。
        - 语义矩阵（Semantic）：基于LLM嵌入的余弦相似度。
        - 基于该多关系图生成增强数据集：通过语义相似性插入/替换物品；通过共现和时序分数指导替换。
    3. **关系感知序列再生**：
        - 对每个物品通过Transformer编码器获取嵌入，分别聚合共现邻域特征、时序邻域特征，并将语义嵌入投影。
        - 通过位置感知加权融合三种特征。
        - 使用编码器-解码器生成模型（如Transformer），以原始序列为输入、增强序列为目标训练，再生成最终数据集（保留前K个原始物品以保证稳定性）。
        - 生成的再生数据集用于训练任意骨干模型。

- **公式**：论文给出嵌入提取、矩阵计算、特征融合、生成损失等核心公式，但无需复现，仅需文字说明即可。

### 3. 实验设计

- **数据集**：四个公开数据集——Amazon子集（Beauty、Sports、Toys）和MovieLens-1M（ML-1M）。稀疏度均极高（>99.9%）。
- **基准方法**：唯一的数据中心基线为DR4SR（Yin et al. 2024）；此外对比了直接在原始数据集上训练（Original）。
- **骨干模型**：五种经典序列推荐模型：GRU4Rec、SASRec、FMLP、GCE-GNN、CL4Rec。
- **评价指标**：Recall@10/20、NDCG@10/20。
- **实验设置**：
    - 优化器：Adam，学习率1e-3，batch size 256，嵌入维度64，序列最大长度50。
    - 增强阶段：每条序列最多生成4个变体（2个语义、2个显式关系），窗口大小2，衰减因子0.8。
    - 生成模型训练100 epoch，多样性因子5。

### 4. 资源与算力

- **明确提及的硬件资源**：论文在实验配置中指出：“All experiments are run on 8 RTX 4090 GPUs and 16 RTX 3090 GPUs.”（共24块GPU）。
- **未说明的部分**：未给出单个实验的平均训练时长、再生数据生成的耗时、或总体算力消耗。因此无法具体评估计算成本。

### 5. 实验数量与充分性

- **实验数量**：
    - 主实验（Table 1）：4个数据集 × 5个骨干模型 × 三种设置（原始、DR4SR、RaSR），共60个性能指标记录（含提升百分比）。
    - 收敛趋势实验（Fig 2）：展示GRU4Rec在Beauty上的收敛曲线。
    - 消融实验（Fig 3）：移除三个模块（语义增强、显式关系增强、关系感知生成器），比较性能变化。
    - 参数敏感性分析（Fig 4 & Fig 5）：分析再生数据比例、窗口大小、衰减因子的影响。
- **充分性与公平性**：
    - 数据集覆盖稀疏（Amazon）和相对密集（ML-1M）场景，骨干模型涵盖RNN、Transformer、MLP、GNN、对比学习等主流架构，对比基线唯一但合理（该领域尚无人研究）。
    - 消融实验验证了每个模块的必要性；参数分析展示了超参数影响；收敛曲线显示RaSR收敛更快更稳定。
    - 不足：未在更多基线（如其他数据中心方法）上对比，但当前领域只有DR4SR一个相关工作。此外未分析生成数据的统计特性（如多样性、分布偏移等）。

### 6. 论文的主要结论与发现

- **核心发现**：RaSR框架在所有四种数据集和五种骨干模型上均一致优于原始数据训练和DR4SR基线，尤其在稀疏数据集上提升显著（如Beauty上Recall@10提升最高达32.88%）。
- **模块贡献**：消融实验表明关系感知生成器是最大贡献者，其次是语义增强和显式关系增强；三者结合效果最佳。
- **收敛优势**：RaSR使模型更快达到更高性能平台，且训练过程更稳定。
- **参数影响**：再生数据比例、窗口大小、衰减因子均存在最优区间，需要根据数据集特性调整。

### 7. 优点

- **模型无关性**：无需修改任何骨干模型架构，可作为插件使用，实用性强。
- **可解释性**：显式构建物品关系（共现、时序、语义），生成过程具有可解释性，区别于纯黑盒生成模型。
- **数据效率**：通过关系引导的增强和再生，有效缓解稀疏性和长尾问题，提升模型泛化能力。
- **实验全面**：覆盖多种骨干、多稀疏度数据集，并进行了充分的消融和参数分析。
- **性能显著**：在多个场景下相对于原始数据和DR4SR均有大幅提升，且收敛更快。

### 8. 不足与局限

- **实验覆盖**：仅对比了一个基线（DR4SR），缺少与其他数据增强方法的比较（如随机增强、图增强等），可能导致说服力不足。
- **计算开销**：需要先训练LLM提取语义嵌入、构建多关系图、再训练生成模型，总体预处理代价较高，论文未量化。
- **生成数据比例敏感性**：实验显示过度生成会导致性能下降，这意味着需要针对每个数据集调优比例，增加了应用复杂度。
- **生成模型选择**：论文仅使用了一种生成模型架构（encoder-decoder），未探讨其他生成模型的影响。
- **潜在偏差**：多关系图基于预定义指标和LLM语义，可能引入人为偏差或噪声，尤其在冷门物品上关系质量难以保证。
- **应用限制**：仅在四个数据集上验证，未涉及更大型、更多样化的推荐场景（如多模态推荐、跨域推荐）。

（完）
