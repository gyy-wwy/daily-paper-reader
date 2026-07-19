---
title: Hyperbolic-Enhanced Mixture-of-Experts Mamba for Sequential Recommendation
title_zh: 双曲增强混合专家Mamba用于序列推荐
authors: "Yuwen Liu, Lianyong Qi, Xingyuan Mao, Weiming Liu, Xuhui Fan, Qiang Ni, Xuyun Zhang, Yang Zhang, Yuan Tian, Amin Beheshti"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38567/42529"
tags: ["query:rec-sys"]
score: 10.0
evidence: 双曲增强的Mamba序列推荐
tldr: 现有序列推荐方法忽略物品层次结构，且Mamba依赖静态前馈网络。本文提出双曲增强的混合专家Mamba模型，在双曲空间中学习层次依赖，同时改进Mamba的动态建模能力，显著提升序列推荐效果。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有序列推荐忽视物品层次结构，且Mamba的静态前馈网络限制了动态建模能力。
method: 提出双曲增强的混合专家Mamba，在双曲空间捕获层次依赖，并利用混合专家结构增强Mamba的动态性。
result: 在多个数据集上超越基线，尤其在长序列推荐中表现优异。
conclusion: 双曲空间和混合专家结构能有效提升序列推荐性能。
---

## Abstract
Sequential recommendation has emerged as a fundamental task in various domains, aiming to predict a user's next interaction based on historical behavior. Recent advances in deep sequence models, particularly Transformer-based architectures and the more recent Mamba, have substantially pushed the boundaries of sequential modeling performance. However, existing methods still face two critical challenges. First, many current approaches overlook the hierarchical structures and high-order dependencies among items, typically restricting representation learning to conventional Euclidean spaces, which limits their capacity to capture complex relational information. Second, although Mamba excels at long-range dependency modeling, its reliance on static Feed-Forward Networks (FFNs) hinders its ability to dynamically adapt to evolving user preferences across diverse contexts. To address these limitations, we propose a Hyperbolic-Enhanced Mixture-of-Experts Mamba recommender (HM2Rec) for sequential recommendation. HM2Rec first encodes user-item relationships through hyperbolic graph convolution to exploit hierarchical structure more effectively. Then, a Variational Graph Auto-Encoder (VGAE) is employed to reconstruct node embeddings, improving structural robustness. To further enhance sequential modeling, we integrate Rotary Positional Encoding (RoPE) into Mamba to better capture relative position dependencies, and replace the FFN with Mixture-of-Expert (MOE) module, enabling dynamic and personalized expert selection for each token. Our extensive experiments on four widely-used public datasets demonstrate that HM2Rec outperforms several advanced baseline models.

---

## 论文详细总结（自动生成）

# 论文总结：《Hyperbolic-Enhanced Mixture-of-Experts Mamba for Sequential Recommendation》

## 1. 核心问题与整体含义（研究动机和背景）

- **研究背景**：序列推荐（Sequential Recommendation）旨在根据用户历史交互行为预测下一个交互物品。近年来，基于Transformer的模型和最新的Mamba模型在序列建模上取得了显著进展，但存在两个关键挑战：
  - **忽视层次结构与高阶依赖**：多数方法将表示学习限制在欧几里得空间，难以捕捉物品间的层次关系和复杂依赖。
  - **Mamba依赖静态前馈网络（FFN）**：虽然Mamba擅长长程依赖建模，但其FFN是静态的，无法动态适应不同上下文中用户偏好的变化。
- **核心问题**：如何同时利用双曲空间捕捉层次结构，并增强Mamba的动态建模能力，以提升序列推荐性能。
- **整体含义**：提出HM2Rec框架，通过双曲图卷积、变分图自编码器（VGAE）以及混合专家（MOE）增强的Mamba，解决上述问题，并在多个数据集上超越现有最优方法。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- 使用双曲空间（Hyperbolic Space）建模物品间的层次关系，通过双曲图卷积网络（HGCN）进行特征聚合。
- 引入VGAE对图结构进行重建，增强节点表示的鲁棒性。
- 将双曲增强后的表示作为输入，结合旋转位置编码（RoPE）和MOE模块改进Mamba序列编码器，实现动态专家路由。

### 关键技术细节
1. **双曲表示学习**：
   - 使用双曲流形（Hyperboloid manifold）进行嵌入，通过指数/对数映射在流形与切空间之间转换。
   - 采用LightGCN风格的聚合方式（无特征变换和非线性激活），利用度归一化平均聚合邻域信息，并引入跨层跳跃连接。
2. **VGAE驱动的表示增强**：
   - 将双曲图卷积输出作为VGAE编码器的输入，学习节点表示的均值与方差。
   - 使用重参数化采样得到潜在表示，并用内积解码器重构邻接矩阵。
   - 优化目标包含重建损失和KL散度正则项。
3. **MOE增强的RPMamba**：
   - 在Mamba的FFN位置替换为MOE层，包含多个专家（轻量MLP）和门控路由机制，实现每个token自适应选择专家（top-k路由）。
   - 引入旋转位置编码（RoPE）以捕获相对位置依赖。
4. **训练目标**：
   - 多目标损失：主任务交叉熵损失（区分正负样本）+ VGAE重建损失 + L2正则化，通过超参数α, β, γ加权。

## 3. 实验设计

- **数据集**：四个公开数据集：
  - Amazon：Books（93,043用户，54,756物品，50.6万交互），Toys（116,429用户，54,784物品，47.8万交互）
  - Foursquare：NYC（1,083用户，9,989物品，17.9万交互），TKY（2,293用户，15,177物品，49.5万交互）
- **评价指标**：Hit Rate@K 和 NDCG@K，K=10,20。
- **对比方法**：涵盖多类基线：
  - RNN：GRU4Rec, NARM
  - Transformer：SASRec, BERT4Rec, TiSASRec
  - GNN：SRGNN, SINE, CORE, FEARec, MAERec
  - 对比学习：CL4SRec, DuoRec, DiffRec, MCLRec, AdaMCT
  - Mamba系列：Mamba4Rec, MiaSRec, TALE
- **实验设置**：PyTorch，2块NVIDIA RTX 4090 GPU；优化器Adam，学习率1e-2；GCN层数2；嵌入维度32（Books/Toys）/128（NYC/TKY）；批量大小2048（Books/Toys）/256（NYC/TKY）；专家数4（Books/Toys/NYC）/5（TKY）；其他超参数（正则化系数1e-6、dropout 0.3、n-hop=3、曲率K=1等）统一。

## 4. 资源与算力

- **文中明确**：使用2块NVIDIA RTX 4090 GPU。
- **未明确**：未提及具体训练时长（如每个epoch时间、总训练轮数）。因此无法给出精确的算力消耗描述。

## 5. 实验数量与充分性

### 实验数量
- **整体性能对比**：在4个数据集上，与14个基线方法比较，报告了HR@10,20和NDCG@10,20共16个指标，形成全面对比表。
- **消融实验**：在Books和NYC上进行了4种变体（w/o Hyperbolic, w/o VGAE, w/o RoPE, w/o MOE）的消融。
- **鲁棒性测试**：在NYC和Books上，模拟5%/15%/25%噪声替换，对比MCLRec和Mamba4Rec。
- **超参数分析**：分别测试嵌入维度、n-hop邻居数、重建阈值b、双曲曲率K、专家数，在至少2个数据集上进行。
- **成本比较**：展示NYC数据集上的运行时间和GPU内存对比。

### 充分性与公平性
- **充分性**：实验类型覆盖了性能排名、组件贡献、鲁棒性、超参数敏感性，较为全面。
- **公平性**：对比方法涵盖主流类别，且使用统一框架进行评估（未明确是否使用官方代码，但超参数设置参照了前人工作）；消融实验逐一去掉关键组件，验证了每个模块的必要性。
- **客观性**：结果表格中报告了最佳基线并加粗强调，HM2Rec在所有指标上全面最优，提升幅度明确（如HR@10提升0.33%~6.12%）。

## 6. 主要结论与发现

- HM2Rec在所有四个数据集和所有指标上均显著优于现有最优基线（最高相对提升6.12%）。
- 每个组件（双曲空间、VGAE、RoPE、MOE）的去除都会导致性能下降，证明它们各自贡献有效。
- 模型在噪声环境下比MCLRec和Mamba4Rec更鲁棒。
- 超参数建议：嵌入维度32（密集数据）或128（稀疏数据）；n-hop=3；重建阈值b=0.3~0.6（因数据集而异）；双曲曲率K=1；专家数4~5。

## 7. 优点

- **创新性**：首次将双曲空间表示、VGAE图重建与MOE增强的Mamba结合，同时解决层次结构建模和动态适应问题。
- **有效性**：实验充分证明了方法的通用性和鲁棒性，在多个指标上大幅领先。
- **效率**：虽然集成多个模块，但GPU内存和运行时间仍处于可接受范围（如图2所示），平衡了性能与效率。
- **设计合理性**：采用LightGCN式的简化图卷积、top-k路由等轻量化技巧，避免了过度复杂化。

## 8. 不足与局限

- **实验覆盖**：
  - 未在更大规模数据集（如具有百万级用户的场景）上验证，当前最大数据集TKY也仅约2万用户。
  - 未进行在线A/B测试或真实系统部署评估，仅限于离线指标。
- **偏差风险**：
  - 超参数调优可能对特定数据集有偏向（如b和K在不同数据集上采用不同最优值），泛化性需进一步验证。
  - 未讨论交互稀疏（冷启动）场景下的表现，而此类场景可能对双曲表示有更高要求。
- **应用限制**：
  - 模型包含双曲图卷积、VGAE、MOE等多模块，调参和部署复杂度较高。
  - 双曲流形计算涉及指数/对数映射，可能引入额外计算开销，尤其在嵌入维度较高时。
- **缺少明确训练时长**：未给出具体训练时间，难以完全评估实际部署成本。

（完）
