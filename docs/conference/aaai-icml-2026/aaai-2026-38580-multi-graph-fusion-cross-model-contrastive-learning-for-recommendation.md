---
title: Multi-graph Fusion Cross-model Contrastive Learning for Recommendation
title_zh: 多图融合跨模型对比学习推荐方法
authors: "Shengjun Ma, Yuhai Zhao, Fenglong Ma, Baoyin Liu, Zhengkui Wang, Wen Shan"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38580/42542"
tags: ["query:rec-sys"]
score: 9.0
evidence: 知识图谱增强的推荐系统
tldr: 为了解决知识图谱支持的推荐系统中双图语义对齐不足和关系噪声问题，本文提出多图融合跨模型对比学习模型MFCCL。该模型通过跨模型对比学习对齐用户-物品二分图和知识图谱表示，并自动筛选重要关系。实验表明MFCCL在多个数据集上显著提升了推荐精度，缓解了数据稀疏性挑战。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有知识图谱推荐模型忽略二分图与知识图谱的语义差异，并引入噪声关系，导致性能次优。
method: 提出MFCCL，通过跨模型对比学习融合二分图和知识图谱，并自适应筛选重要关系。
result: 在多个推荐数据集上，MFCCL准确率优于基线模型。
conclusion: 多图融合与对比学习有效提升推荐性能。
---

## Abstract
Knowledge Graph (KG)-supported Graph Neural Network models are becoming crucial in recommendation systems due to their ability to mitigate the data sparsity challenge. However, these models remain suboptimal because they overlook the representation differences between the inherent user-item Bipartite Graph (BG) and the external head-relation-tail KG, leading to semantic misalignment. Moreover, they indiscriminately incorporate various types of relations from the KG, which may introduce noise information into the model, ultimately degrading recommendation performance. To address these challenges, we propose an end-to-end model named Multi-graph Fusion Cross-model Contrastive Learning (MFCCL). To uncover users' interest in items and explore the associations between items, we first construct a user-interest graph by integrating information from both the BG and KG, and an item-association graph derived from the BG. We devise a multi-graph representation learning module that incorporates rich semantics into user and item representations in parallel. Simultaneously, a classical collaborative filtering module is introduced to fully leverage user-item collaborative signals. Additionally, we design a novel free data-augmentation cross-model contrastive learning to facilitate the exchange of complementary information between different models. Empirical evaluations on three widely used benchmarks demonstrate that our MFCCL method achieved significant improvements over the baselines.

---

## 论文详细总结（自动生成）

# 多图融合跨模型对比学习推荐方法（MFCCL）详细总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：知识图谱（KG）增强的图神经网络推荐模型虽然能缓解数据稀疏性，但现有方法存在两个关键缺陷：
  1. **语义不对齐**：忽略用户-物品二分图（BG）与外部知识图谱（KG）的表示差异，导致两种图的嵌入空间不一致，如图1(c)所示，CKE在Last-FM上的嵌入值差异显著。
  2. **关系噪声**：不加区分地引入KG中的所有关系类型，这些关系不一定对推荐有益，反而引入噪声，增加参数规模并导致优化困难。
- **背景**：传统两步法（如CKE）先编码KG嵌入再补充物品表示，但未解决表示差异；端到端方法（如KGIN）将BG和KG拼接成协作知识图（CKG），但CKG训练成本高且关系复杂；对比学习方法（如KGCL、KGRec）依赖数据增强，可能引入额外噪声。
- **目标**：提出一种端到端模型MFCCL，解决表示差异和关系噪声问题，提升推荐精度。

## 2. 方法论：核心思想、关键技术细节
### 2.1 核心思想
- **多图生成**：不是直接将BG和KG拼接，而是显式构造两个辅助图：
  - **物品关联图（IaG）**：基于BG中物品共现关系，通过余弦相似度选择top-k最相似物品构建边，捕捉物品内在关联（如啤酒-尿布现象）。
  - **用户兴趣图（UiG）**：连接用户和物品属性（来自KG），依据用户历史交互中属性的出现频率，剪枝弱关联后保留强兴趣边。
- **多图表示融合**：采用双塔架构并行学习用户和物品表示：
  - **物品端**：从KG学习属性语义（使用虚拟关系替代原始关系，通过注意力融合），从IaG学习物品关联语义（LightGCN聚合），再通过注意力机制融合。
  - **用户端**：从BG学习长期习惯语义（LightGCN），从UiG学习短期偏好语义，再注意力融合。
- **协同过滤（CF）模块**：独立于多图表示的LightGCN分支，充分捕捉用户-物品协作信号。
- **跨模型对比学习（CMC）**：将同一ID的多图表示和CF表示作为正对，不同ID作为负对，使用InfoNCE损失最大化一致性；无需数据增强（free data-augmentation），避免引入噪声。
- **模型优化**：联合BPR损失（预测用户对正负样本的排序）和CMC对比损失，带L2正则化。

### 2.2 关键技术细节
- **虚拟关系机制**：在KG表示学习中，用一组可学习的虚拟关系（数量P为超参数）替代原始所有关系，通过注意力得分β(p,i)加权聚合，减少关系噪声并提升效率。
- **公式摘要**：
  - IaG相似度计算：\( w_{i,j} = \frac{A_{i,j}^{co} - \frac{Deg(i)}{s}}{\sqrt{Deg(i)Deg(j)}} \)
  - UiG剪枝：\( A_{i,j}^{ui} = 1 \text{ if } A_{i,j}^{ua} > c \text{ else } 0 \)
  - 物品KG聚合：\( e_{i,kg}^{(l)} = \sum_{p} \beta(p,i) e_p \odot \frac{1}{|N_i^{kg}|} \sum_{t \in N_i^{kg}} e_t^{(l)} \)
  - 用户/物品多图融合：注意力加权求和（如 \( e_{u,mg}^{(l+1)} = att(e_{u,bg}^{(l)}) e_{u,bg}^{(l)} + att(e_{u,ui}^{(l)}) e_{u,ui}^{(l)} \)）
  - 对比损失：InfoNCE，负采样来自同batch内其他ID。
  - 预测分数：\( y_{ui} = a \cdot e_{u,mg}^\top e_{i,mg} + (1-a) \cdot e_{u,cf}^\top e_{i,cf} \)

### 2.3 算法流程（文字说明）
1. 离线构建IaG（物品共现top-k）和UiG（用户-属性强关联）。
2. 初始化多图表示模块（用户BG、用户UiG、物品KG、物品IaG）和CF模块（用户BG、物品BG）的嵌入。
3. 每层：
   - 多图模块：用户端从BG和UiG分别聚合后融合；物品端从KG（含虚拟关系）和IaG分别聚合后融合。
   - CF模块：从BG进行LightGCN聚合。
4. 所有层叠加得到最终多图表示和CF表示。
5. 计算BPR损失（基于预测分数\( y_{ui} \)）和跨模型对比损失。
6. 联合优化。

## 3. 实验设计
### 3.1 数据集与场景
- **三个公开数据集**：
  - **Alibaba-iFashion**：电商服装，用户11.4万，物品3万，交互178万，KG三元组27.9万。
  - **Last-FM**：音乐，用户2.3万，物品4.8万，交互303万，KG三元组46.4万。
  - **Yelp2018**：本地商户，用户4.5万，物品4.5万，交互118万，KG三元组185万。
- 密度极低（5e-4 ~ 3e-3），数据稀疏性高。

### 3.2 基准方法（Baselines）
- **传统CF**：BPR, NFM
- **GNN-CF**：GC-MC, LightGCN
- **图对比学习**：SGL, SimGCL, XSimGCL
- **KG增强推荐**：CKE, KTUP, KGCN, KGAT, KGIN
- **KG+CL**：KGCL, KGRec

### 3.3 评估指标
- recall@20, ndcg@20（全排序策略，所有未交互物品作为负样本）

### 3.4 参数设置
- 嵌入维度64，Adam优化器，batch size 1024，网格搜索学习率、层数、dropout等。

## 4. 资源与算力
- 论文未明确说明使用的GPU型号、数量或训练时长。
- 仅提到使用PyTorch实现，Adam优化器。
- 从时间复杂度分析看，每epoch复杂度为 \( O(L(3|U| + (P+2)|I|)d) \)，与基线模型（如KGCL、KGRec）可比。

## 5. 实验数量与充分性
### 5.1 实验组数
- **整体性能对比**（RQ1）：表2，三个数据集上对比13种基线方法，每个报告recall和ndcg。
- **多图表示消融**（RQ2）：表3，在Alibaba-iFashion和Last-FM上，移除UiG、BG、IaG、KG、虚拟关系（MFCCL-vr）以及CMC模块（MFCCL-cmc），共6个变体×2数据集×4指标=48个结果。
- **CMC模块分析**：图3，展示了不同λ1权重的影响（3个值）和训练动态（loss和recall曲线）。
- **超参数敏感性**：图4，对虚拟关系数量P、融合参数α、温度τ进行变化（约4-5个值），在Alibaba-iFashion和Last-FM上观察recall@20和ndcg@20变化。

### 5.2 充分性与客观性
- **充分性**：覆盖了所有关键模块的消融，验证了每个组件的贡献；对比基线全面，包含CF、GNN、CL、KG等多种类型；超参数分析提供了合理范围。
- **客观性**：使用了所有ranking策略，避免采样偏差；多次独立实验取均值；代码和参数设置有一定说明（但未提供开源代码）。
- **公平性**：嵌入维度统一，优化器相同，数据集处理遵循先前工作。

## 6. 主要结论与发现
1. **MFCCL在三个数据集上均显著优于所有基线**：recall@20提升2.38%~4.81%，ndcg@20提升更明显。
2. **多图表示融合有效**：去除任何图组件（BG、UiG、IaG、KG）都会导致性能下降（1.26%~8.90%），验证每个图贡献互补语义。
3. **虚拟关系机制优于原始关系**：替换为虚拟关系后，ndcg@20提升7.21%（Alibaba-iFashion）和8.28%（Last-FM），证明其能有效减轻关系噪声。
4. **跨模型对比学习是核心**：移除CMC模块后性能大幅下降13.85%~22.29%，且训练收敛更快、更稳定。
5. **超参数敏感性可控**：虚拟关系数量P取中等值（如4）、融合参数α约0.2、温度τ约0.2时性能最佳，符合直觉。

## 7. 优点（方法或实验设计亮点）
- **无数据增强的对比学习**：跨模型对比学习直接利用两个独立模块的输出作为正对，避免图扰动或特征掩码引入的噪声，属于创新性设计。
- **多图构造策略**：通过构建IaG和UiG，将BG和KG的信息解耦为更细粒度的语义，且离线构造不增加训练成本。
- **虚拟关系替代真实关系**：不仅简化KG表示，还能自动过滤噪声关系，提升鲁棒性。
- **双模块融合预测**：预测时同时使用多图表示和CF表示，并引入可调参数a，实现动态平衡。
- **实验全面**：消融实验从用户侧、物品侧、对比学习等多个维度验证，尤其对虚拟关系机制的单独消融（MFCCL-vr）证明了其有效性。

## 8. 不足与局限
- **计算资源未明确**：缺乏GPU型号、训练时间等详细信息，难以复现计算成本。
- **实验局限性**：
  - 仅评估了top-20指标，未报告recall@10、ndcg@10或更高K值（如recall@50）等常见指标。
  - 未在冷启动场景或交叉域推荐上验证，对于新用户/新物品的鲁棒性未知。
  - 未对比基于大语言模型（LLM）的方法（虽然未来工作提到将结合LLM）。
- **超参数依赖**：虚拟关系数量P和融合参数a需要针对每个数据集调整（如P最佳值在Alibaba-iFashion和Last-FM上不同），可能影响泛化性。
- **潜在偏差**：UiG剪枝阈值c固定为2，但数据分布差异可能影响效果，未进行消融分析。
- **对比学习负采样**：仅在一个batch内采样，可能不充分，未探索更复杂的负采样策略。
- **可解释性**：虚拟关系缺乏语义含义，难以解释模型学到了哪些关联。

（完）
