---
title: Interest-Shift-Aware Logical Reasoning for Efficient Long-Sequence Recommendation
title_zh: 兴趣漂移感知的逻辑推理用于高效长序列推荐
authors: "Fei Li, Qingyun Gao, Enneng Yang, Jianzhe Zhao, Guibing Guo"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38531/42493"
tags: ["query:rec-sys"]
score: 9.0
evidence: 高效长序列推荐
tldr: 逻辑推理推荐方法在长序列场景下难以捕捉兴趣漂移且计算复杂度高。本文提出ELECTOR，通过次序列感知逻辑推理建模兴趣迁移，并优化推理效率。在长序列推荐数据集上，ELECTOR在准确性和效率上均优于现有方法。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有逻辑推理方法在长序列中无法建模子序列兴趣漂移且计算开销大。
method: 提出ELECTOR，利用次序列兴趣漂移感知逻辑推理，降低复杂度。
result: 在长序列推荐上兼顾精度与效率，优于基线。
conclusion: 兴趣漂移感知优化了长序列逻辑推理推荐。
---

## Abstract
Logical reasoning-based recommendation methods formulate logical expressions to characterize user-item interaction patterns, incorporating regularization constraints to ensure consistency with logical rules. However, these methods face two critical challenges: (1) As sequence length increases, they cannot effectively capture the dynamic transfer of user interests across subsequences (i.e., subsequence interest drift), thereby degenerating logical expressions to single-subsequence inference. (2) The time complexity of logical reasoning and rule learning scales quadratically with the sequence length, severely constraining computational efficiency in long-sequence recommendation. To address these challenges, we propose ELECTOR, an intErest-shift-aware long-sequence Logical reasoning for EffiCienT lOng-sequence Recommendation method. Specifically, we design a Subsequence Interest Learning Module (SIL) to model cross-subsequence interest drifts in long sequences. SIL employs a local attention mechanism to extract subsequence interests effectively and a global attention mechanism to capture the correlations among subsequence interests. Subsequently, we propose an Interest-aware Logical Reasoning (ILR) mechanism that performs logical reasoning using a limited set of subsequence and short-term interests, rather than reasoning over the entire sequence, significantly reducing time complexity. Additionally, ILR employs interest logical reasoning contrastive loss to ensure the model simultaneously considers multiple interests. Experiments on four real-world datasets demonstrate that our method significantly outperforms all baselines regarding computational efficiency and recommendation accuracy, confirming its effectiveness.

---

## 论文详细总结（自动生成）

好的，以下是根据您提供的论文内容生成的中文总结。

## 论文总结：兴趣漂移感知的逻辑推理用于高效长序列推荐 (ELECTOR)

### 1. 核心问题与整体含义（研究动机和背景）
- **核心问题**：现有基于逻辑推理的推荐方法在**长序列**场景下面临两个关键挑战：
    1.  **兴趣漂移**：用户的兴趣会随着时间在多个子序列间动态转移（例如从网球兴趣到商务兴趣），但现有方法将用户兴趣视为静态，无法有效捕捉这种子序列兴趣漂移，导致逻辑表达式退化为单一子序列推理，推荐精度下降。
    2.  **计算复杂度高**：逻辑推理和规则学习的时间复杂度随序列长度呈**二次方增长**，严重限制了在长序列推荐中的计算效率。
- **整体含义**：提出一种新的方法ELECTOR，旨在同时解决长序列推荐中的兴趣漂移建模和计算效率问题，实现更准确、更高效的推荐。

### 2. 方法论
- **核心思想**：通过将长用户交互序列划分成子序列，分别提取子序列兴趣和短期兴趣，并仅利用这些少量的兴趣嵌入（而非全部交互物品）进行逻辑推理，从而降低复杂度并感知兴趣漂移。
- **关键技术细节**：
    1.  **子序列兴趣学习模块 (SIL)**：
        - **子序列提取**：将历史序列均匀划分为多个子序列。
        - **局部注意力机制**：在每个子序列内部使用线性复杂度的注意力（低秩矩阵近似LRM或核特征映射KFM）提取子序列兴趣，避免跨子序列干扰。
        - **全局注意力机制**：使用自注意力建模子序列兴趣与近期交互物品之间的相关性，增强兴趣表示。
    2.  **兴趣感知逻辑推理机制 (ILR)**：
        - **简化逻辑表达式**：用子序列兴趣和短期兴趣替代所有交互物品作为逻辑变量，构建霍恩子句（Horn clause），将变量数从 nL 降至 nsh+nlh（远小于nL），显著降低推理复杂度。
        - **逻辑操作网络**：使用轻量级 MLP 实现逻辑运算（¬, ∧, ∨），并通过逻辑正则化保证一致性。
        - **兴趣逻辑推理对比损失 (ILR Contrastive Loss)**：强制模型在推理时**同时考虑多个兴趣**（子序列兴趣和短期兴趣），避免单一子序列主导，增强逻辑表达的完备性。
- **时间复杂度**：从骨干方法NCR的 O(16 nL² d²) 降低到 O(16 nh² d²)，其中 nh ≪ nL，大幅提升效率。

### 3. 实验设计
- **数据集**：ML-1M, MoTV, Amazon-500, Amazon-1000（四个真实世界公开数据集）。
- **Benchmark 对比方法**：共对比了13种基线方法，涵盖四大类：
    - 传统序列方法：STAMP, GRU4Rec, NARM, SASRec
    - 高效序列方法：LinRec, Mamba4Rec, SIGMA, GLINT-RU, RecBLR
    - 多兴趣方法：MIND, ComiRec-SA
    - 逻辑推理方法：NLR, NCR
- **评价指标**：主要使用归一化折损累计增益 @5 和@10 (N@5, N@10) 和 命中率@5和@10 (H@5, H@10)。评估推荐准确性和计算效率（训练时间）。

### 4. 资源与算力
- **文中未明确说明**所使用的GPU型号、数量、以及具体的训练时长（论文仅给出了每个epoch的训练时间或总训练时间，但未提及硬件配置）。因此无法总结具体算力资源。

### 5. 实验数量与充分性
- **实验数量**：进行了多组实验，包括：
    - **主实验**：在4个数据集上对比15种方法（含2种变体），使用两个指标。
    - **效率对比**：在3个数据集上报告了训练时间。
    - **消融实验**：包含5个变体（逐步移除SIE, LIC-S, LIC-RI, ILR等组件），验证每个模块的有效性。
    - **敏感性分析**：研究了子序列长度、子序列兴趣数量、短期兴趣数量对性能的影响。
- **充分性与公平性**：实验设计较为充分，覆盖了主流基线、多个数据集和指标。消融实验设计合理，能清晰归因各组件贡献。敏感性分析为参数选择提供了依据。但可能存在**超参数设置偏差**（如λ1=1, λ2=10, λΘ=0.0001）未说明调优过程，对结论的泛化性有一定影响。

### 6. 主要结论与发现
- **准确性优势**：ELECTOR在所有数据集上显著优于所有基线方法。尤其在Amazon(1000)长序列数据集上，N@5和H@5分别提升**288.02%** 和**238.80%**。
- **效率优势**：相较于基于物品推理的NLR和NCR，ELECTOR的训练时间大幅降低（在Amazon(1000)上比NCR减少70%~81%），同时保持或提升准确率。
- **消融证实**：子序列兴趣提取、相关性建模、多兴趣对比损失等组件均对性能有正面贡献。
- **参数敏感性**：子序列长度、兴趣数量对性能有影响，数据集不同最优设置不同。例如，序列长度100时，子序列兴趣数和短期兴趣数都设为7效果最佳；长度更长时，短期兴趣数设为9更优。

### 7. 优点
- **问题创新性**：清晰识别并定义了长序列推荐中逻辑推理方法面临的两个关键瓶颈（兴趣漂移与二次复杂度），并针对性地提出解决方案。
- **方法论亮点**：
    - 通过子序列压缩和兴趣级推理，巧妙地将推理复杂度从O(nL²)降至O(nh²)，实现了理论与实际的效率提升。
    - 引入的ILR对比损失强制模型进行多兴趣推理，有效缓解了单一子序列主导的问题，提升了逻辑表达的完整性。
- **实验设计全面**：覆盖多种类型基线、多个公开数据集、丰富的消融和敏感性分析，论证充分。

### 8. 不足与局限
- **计算资源未明确**：缺乏对实验硬件和GPU型号的详细描述，影响结果可复现性。
- **预定义子序列长度**：需要手动设定子序列长度和个数，最优值随数据集变化，缺乏自适应机制。
- **静态的分割策略**：采用均匀划分历史序列的方式，未考虑用户行为时间间隔或语义变化的非均匀性，可能无法完美适应所有场景。
- **仅对比逻辑推理方法**：虽对比了多类方法，但主要优势体现在与NLR/NCR等逻辑推理基线的对比上。与最先进的Transformer或Mamba类序列模型（如SIGMA）的差距在部分数据集上不够显著（但ELECTOR仍领先）。
- **潜在偏差风险**：兴趣数（nh）的设定依赖于经验分析，若在工业场景中用户兴趣数量差异巨大，固定设置可能导致欠拟合或过拟合。

（完）
