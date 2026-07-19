---
title: "Dual-Perspective Disentanglement: Learning Symmetric Group-Aware Representations for Cross-Domain Recommendation"
title_zh: 双视角解耦：学习对称组感知表示用于跨领域推荐
authors: "Borui Wu, Yuanbo Xu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38629/42591"
tags: ["query:rec-sys"]
score: 7.0
evidence: 跨领域推荐，解耦表示
tldr: 跨领域推荐面临用户偏好和物品吸引力的异质性挑战。本文提出DPGCDR，从用户和物品双视角动态聚类组，并对称地解耦表示。该方法在多个跨领域数据集上取得了最优性能，验证了考虑组感知解耦的有效性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 跨领域推荐中用户偏好和物品吸引力存在异质性，现有解耦方法忽略了组级信息。
method: 提出DPGCDR模型，动态将用户聚类为组、物品聚类为主题，并对称解耦共享和特定成分。
result: 在多个跨领域推荐数据集上，DPGCDR显著优于现有跨领域推荐方法。
conclusion: 双视角组感知解耦能有效缓解跨领域推荐中的数据稀疏和异质性。
---

## Abstract
Cross-Domain Recommendation (CDR) transfers user preferences from a source domain to alleviate data sparsity in a target domain. While disentangling representations into domain-specific and shared components is a common method, existing methods overlook user preference heterogeneity and item appeal heterogeneity. To this end, we propose DPGCDR, a Dual-Perspective Group-aware CDR method that learns symmetric group-aware representations from both user and item. Conceptually, DPGCDR dynamically clusters users into groups and items into themes, then symmetrically disentangles user preferences into group-specific and cross-group shared components, and item attributes into theme-specific and cross-theme shared components. We propose a two-stage training scheme: 1) an initial warm-up stage learns preliminary representations to dynamically cluster users and items into group and theme structures which generalize cross-domain scenarios into multi-group disentanglement analogous to multi-domain settings; 2) a fusion-based aggregation stage integrates these group/theme-specific components into unified global representations. Additionally, an information-theoretic alignment regularizer further ensures consistency and discriminability between global shared and group/theme-specific representations, facilitating effective knowledge transfer by explicitly modeling and preserving the inherent multi-group structure within cross-domain interactions. Extensive experiments show DPGCDR achieves state-of-the-art performance, with significant gains of up to 25% in HR@10 over baselines on datasets with heterogeneous interaction structures. Further analyses confirm our dynamic clustering mechanism effectively adapts to underlying data patterns, enabling fine-grained cross-domain knowledge transfer.

---

## 论文详细总结（自动生成）

# 论文总结：Dual-Perspective Disentanglement: Learning Symmetric Group-Aware Representations for Cross-Domain Recommendation

## 1. 核心问题与整体含义（研究动机和背景）
- **研究动机**：跨领域推荐（CDR）通过迁移源域知识缓解目标域数据稀疏问题。现有解耦方法将表示分离为域特定和共享成分，但忽略了用户偏好异质性和物品吸引力异质性，导致知识迁移不均匀、负面迁移。
- **背景问题**：用户和物品内部存在潜在的子群结构（如用户偏好分组、物品主题分组），但大多数方法假设用户同质，未显式建模这种组级异质性。作者通过预实验（Figure 1）发现用户组间行为差异显著且推荐性能波动，说明统一建模会损害细粒度迁移效果。
- **核心挑战**：如何从用户和物品双视角对称地发现并利用这些子群结构，实现更精细的解耦迁移。

## 2. 方法论：核心思想、关键技术细节、算法流程
- **核心思想**：提出DPGCDR（Dual-Perspective Group-aware Cross-Domain Recommendation），动态将用户聚类为组、物品聚类为主题，然后对称解耦：用户偏好分解为组特定+跨组共享，物品属性分解为主题特定+跨主题共享。通过两阶段训练实现。
- **关键技术细节**：
  - **变分二分图编码器（VBGE）**：基于VAE框架，对用户-物品二部图进行2跳变分传播，生成潜变量表示。每个节点聚合同类型邻居信息（通过GCN），再经VAE head输出后验均值和对数方差，通过重参数化采样。
  - **两阶段框架**：
    - **Stage 1（预热与动态聚类）**：先用VBGE训练获得初步嵌入；对用户和物品分别用K-Means聚类（轮廓系数选择簇数k），得到用户组g(u)和物品主题t(i)分配。
    - **Stage 2（双视角解耦）**：构建四个并行编码分支（用户特定、用户共享、物品特定、物品共享）。特定路径使用梯度掩码（只更新本组节点参数）；共享路径通过注意力融合所有组/主题的视角分布，得到全局共享表示。最后通过门控拼接特定和共享表示形成最终嵌入。
  - **优化目标**：综合损失 \( L = L_{rec} + \lambda_{KLD} L_{KLD} + \beta_{align} L_{align} \)。
    - \(L_{rec}\)：二元交叉熵重建损失（正负样本）。
    - \(L_{KLD}\)：各VBGE路径的KL散度正则化。
    - \(L_{align}\)：对齐损失——使共享分布靠近各组视角分布，同时与特定分布区分（通过KL散度聚合）。
- **公式与算法流程**：文本给出了详细的数学定义（式1-13），包括邻接矩阵、逐层聚合、变分参数、重参数化、KL散度闭合形式、对齐损失等。整体流程：图3展示了完整框架。

## 3. 实验设计
- **数据集与场景**：使用Amazon真实数据集，预处理后形成四个CDR场景（每场景两个领域）：
  - Elec & Phone
  - Elec & Cloth
  - Sport & Phone
  - Sport & Cloth
- **基准与对比方法**：
  - **单域方法**：BPRMF（矩阵分解）、LightGCN（图卷积）、DPGCDR*（仅预热阶段，作为本模型变体）。
  - **跨域方法**：PPGN（双GCN+共享初始化）、BiTGCF（双向迁移GCN）、DisenCDR（解耦VAE，当前SOTA基线）。
- **评价指标**：HR@10（命中率）和NDCG@10（归一化折损累计增益），采用leave-one-out评估（1个正例+999个负例）。

## 4. 资源与算力
- 文中在“Implementation Details”明确说明：训练在**单张NVIDIA 3090 GPU**上进行。
- 超参数：嵌入维度d=64，2层VBGE，dropout 0.3；预热20个epoch（lr=1e-3），动态阶段30个epoch（lr=5e-4）；Adam优化器（\(\beta_1=0.9,\beta_2=0.999\)），权重衰减5e-4；batch size=1024。
- **未明确说明**：训练总时长（小时/分钟）、是否使用多卡并行、数据预处理耗时等。

## 5. 实验数量与充分性
- **实验数量**：
  - 主要性能对比：4个场景×每个场景两个领域，共8个HR/NDCG条目（Table 1）。
  - 消融实验：3个场景（Elec&Phone, Elec&Cloth, Sport&Phone）下比较用户侧、物品侧、双视角三种设置（Figure 4）。
  - 聚类与迁移有效性分析：用户聚类可视化（Figure 5a）、共享嵌入t-SNE（Figure 5b）、每个用户组的每个epoch性能曲线（Figure 6）。
  - 超参数敏感性：两个超参数λ（KLD权重）和β（对齐权重）在不同域上的HR/NDCG热图（Figure 7）。
- **充分性评价**：
  - 对比基线全面（涵盖单域和跨域经典方法），且包含本模型变体。
  - 每个结果平均3次独立运行，降低随机性。
  - 消融和可视化分析深入剖析了组件贡献和聚类效果。
  - **不足**：仅在两个领域组成的跨域场景上验证，未测试多领域（>2）或跨平台场景；Cloth域作为目标域时出现小幅下降，作者归因于同质/稀疏，但未进一步分析原因或提出稳健策略。

## 6. 主要结论与发现
- DPGCDR在大多数场景下达到SOTA，尤其在异质交互结构数据集上提升显著（如Elec域上HR@10提升17.4%，NDCG@10提升25.1%）。
- 双视角解耦相比单视角（仅用户或仅物品）持续更优，且当跨域差异越大时增益越明显；当差异小时主要贡献为更快收敛。
- 动态聚类机制能自动识别有意义的用户组和物品主题（如Figure 5显示三个用户组分离良好），帮助冷启动组用户性能大幅提升（约40%），均衡组和高迁移组也有增益。
- 超参数λ和β在中等范围内鲁棒，过大/过小均导致性能下降。

## 7. 优点
- **对称双视角设计**：不仅考虑用户组，还对称考虑物品主题，首次提出用户和物品双层面的组感知解耦，更具一般性。
- **动态聚类**：自动从数据中发现子群结构，无需预定义域标签，可推广到多组场景（类似于多域）。
- **信息论对齐正则化**：通过KL散度强制共享表示与各组视角对齐，同时与特定表示区分，增强解耦效果。
- **实验充分且可视化丰富**：不仅报告最终指标，还通过用户组性能曲线、嵌入可视化等直观证明聚类和迁移的有效性。
- **开源实现**：附录提供源代码，可复现。

## 8. 不足与局限
- **部分场景性能下降**：在Cloth域作为目标域时（Elec→Cloth, Sport→Cloth），DPGCDR不如DisenCDR，表明方法对稀疏/同质域存在过拟合或负迁移风险。
- **仅限双域场景**：论文只验证了两个领域组成的CDR，未扩展到多域（>2）或跨系统场景，通用性待验证。
- **聚类依赖预热质量**：K-Means聚类依赖于第一阶段嵌入的质量，若预热阶段表示不理想可能导致错误分组，影响后续性能。
- **计算复杂度**：双视角解耦需构建多个编码分支，相比单模型训练时间增加，文中未给出具体训练时间对比。
- **缺失更多异质性分析**：未深入探讨用户组和物品主题之间交互（例如哪些组/主题组合最受益），以及冷启动用户的跨领域行为模式。

（完）
