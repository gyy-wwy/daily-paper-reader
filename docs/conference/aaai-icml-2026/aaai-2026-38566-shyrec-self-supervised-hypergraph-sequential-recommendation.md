---
title: "S²HyRec: Self-Supervised Hypergraph Sequential Recommendation"
title_zh: "S²HyRec: 自监督超图序列推荐"
authors: "Yuchen Liu, Kunyu Ni, Zhongying Zhao, Guoqing Chao, Yanwei Yu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38566/42528"
tags: ["query:rec-sys"]
score: 9.0
evidence: 自监督超图序列推荐
tldr: 本文针对序列推荐中用户意图时间建模不足和数据噪声问题，提出S2HyRec自监督超图序列推荐框架。该框架包含全局意图倾向模块和时序上下文意图模块，分别捕获长期偏好和动态兴趣，并通过超图结构增强表示。实验表明，S2HyRec在多个序列推荐数据集上达到最优，验证了超图在序列建模中的有效性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有序列推荐未有效区分全局意图和时序上下文意图，且易受噪声影响。
method: 提出自监督超图框架，包含全局意图和时序上下文两个模块，利用超图建模复杂关系。
result: 在基准数据集上，S2HyRec在推荐准确率上优于现有序列推荐方法。
conclusion: 超图自监督学习能有效提升序列推荐的鲁棒性和准确性。
---

## Abstract
Sequential recommendation models analyze user historical behavior sequences to capture temporal dependencies and the dynamic evolution of interests, enabling accurate predictions of future behaviors. However, there are still two critical challenges that remain unsolved: i) Inadequate temporal modeling of user intent, which fails to distinguish between global intent tendency and temporal contextual intent. ii) Noise in sequential interaction data may introduce bias into the model. To address these issues, we propose a Self-Supervised Hypergraph Sequential Recommendation Framework (S2HyRec). This framework features the Global Intent Tendency module for capturing long-term preferences, the Temporal Contextual Intent module for modeling dynamic time-sensitive interests. Additionally, we develop the Sequence Dependency-Aware module that analyzes the chronological flow of interactions to uncover inherent behavioral dynamics, further enriching the comprehensive user intent representation. To mitigate noisy interactions, we employ a Cross-View Self-Supervised Learning module that enhances the model's ability to distinguish genuine preferences from noise. Extensive experiments on four benchmark datasets demonstrate the superiority of S2HyRec over various state-of-the-art recommendation methods, especially achieving average improvements of 15.13% and 14.03% in NDCG@10 and NDCG@20, respectively, across the four datasets.

---

## 论文详细总结（自动生成）

# S2HyRec 论文中文详细总结

## 1. 核心问题与整体含义（研究动机和背景）
- **问题**：现有序列推荐模型存在两个关键挑战：
  - **用户意图的时间建模不充分**：未能有效区分全局意图倾向（长期稳定偏好）和时序上下文意图（时敏性兴趣），导致推荐精度受限。
  - **交互序列中的噪声干扰**：误点击、偶发行为等噪声会引入偏差，掩盖用户的真实偏好。
- **动机**：为了同时捕捉全局意图、动态上下文兴趣，并增强模型对噪声的鲁棒性，提出自监督超图序列推荐框架 S²HyRec，以更全面地建模用户意图。

## 2. 方法论：核心思想与技术细节
- **核心思想**：将用户意图分为三个互补层面——全局意图倾向（长期稳定偏好）、时序上下文意图（时敏兴趣）、序列依赖（行为动态），并通过跨视图自监督学习对齐全局与局部表示，从而过滤噪声。
- **关键技术细节**（共四个模块）：
  - **全局意图倾向学习**：利用超图结构连接用户与相关物品，通过可学习的超边嵌入矩阵 \(H_u, H_v\) 捕捉高阶依赖；经过多层超图传播（式3）和层次映射（式4），得到全局嵌入 \(\tilde{z}_u, \tilde{z}_v\)。
  - **时序上下文意图学习**：将交互序列按时间切分为 T 个时间片，每片构建交互矩阵 \(A_t\)；使用 GNN（式6）进行消息传递，得到各时间片的用户/物品嵌入；再通过自注意力机制（式9）聚合跨时间片依赖，最终求和得到上下文嵌入 \(\bar{e}_u, \bar{e}_v\)（式10）。
  - **序列依赖感知学习**：将物品嵌入与位置编码相加（式13），通过多层自注意力（式14）提取序列中位置间依赖，最后聚合得到序列依赖嵌入 \(\tilde{e}_u\)（式15）。
  - **跨视图自监督学习**：将时序上下文嵌入线性映射后，与全局嵌入进行对比学习（式12），促使真实偏好保持一致，而噪声产生差异；损失函数包含用户和物品对比项。
- **双任务训练**：将全局嵌入和时序上下文嵌入加权融合（式16），结合序列依赖嵌入计算预测得分（式17）；优化目标包括排序损失 \(L_{rec}\)、SSL 损失 \(L_{SSL}\) 和 L2 正则化（式19）。

## 3. 实验设计
- **数据集**：Gowalla（用户48k，物品52k，交互1.8M）、MovieLens（24k/8.7k/1.8M）、Yelp（19.8k/38.4k/1.5M）、Tmall（212k/99k/3.9M），涵盖不同规模和密度（7e-4 到 8e-3）。
- **基准方法**：共 18 个 SOTA 基线，涵盖：
  - 协同过滤：BiasMF, NCF
  - 传统序列模型：GRU4Rec, SASRec, TiSASRec, Bert4Rec
  - GNN 方法：NGCF, LightGCN, SRGNN, GCE-GNN, SURGE, DGCF
  - 自监督学习：SGL, ICLRec, CoSeRec, CoTRec, CLSR, SelfGNN
- **评估指标**：Hit Rate@K (H@K) 和 NDCG@K，K=10/20。
- **对比结果**：S²HyRec 在所有数据集上全面超越所有基线，例如在 Gowalla 上 NDCG@10 提升 15.1%，在 MovieLens 上 NDCG@20 提升 18.1%。

## 4. 资源与算力
- **硬件**：单张 NVIDIA GeForce RTX 4090 24GB GPU。
- **训练时间**：未给出绝对时长，但相对比较显示：比 LightGCN 快 31-35%，比 SelfGNN 快 29-35%。
- **推理时间**：0.29-0.43 ms（所有模型中最低），适合实时推荐。
- **说明**：文中未报告完整训练轮数或总耗时，仅提供了相对效率图。

## 5. 实验数量与充分性
- **实验数量**：
  - 主表：4 个数据集 × 18 个基线，共 72 组指标（H@10/20, N@10/20）。
  - 消融实验：6 种变体（去掉全局模块、替换超图为二部图、去掉注意力、GRU替换、去掉SSL、去掉序列模块）在 3 个数据集上验证。
  - 噪声鲁棒性：注入 5%-20% 噪声，对比 4 种方法（S²HyRec, SelfGNN, SURGE, Bert4Rec）。
  - 超参数分析：对 α, Lg, La, M 进行扫描，并在两个数据集上展示趋势。
  - 效率对比：训练和推理时间对比 4 个模型（SASRec, LightGCN, SRGNN, SelfGNN）。
  - 案例研究：2 个用户的具体预测展示。
- **充分性评估**：
  - **充分**：覆盖多种场景（稀疏/密集、小/大规模）、对比基线全面、消融验证各模块必要性、噪声测试证明鲁棒性、效率分析显示实用性。
  - **客观与公平**：设置明确，超参数调优合理（网格搜索），统计显著性检验（p<0.01），未发现明显偏向。

## 6. 主要结论与发现
- S²HyRec 在四个数据集上平均提升 NDCG@10 达 15.13%，NDCG@20 达 14.03%，验证了同时建模全局意图、时序上下文和序列依赖的有效性。
- 跨视图自监督学习显著增强了模型对噪声的抵抗力（20% 噪声下仍保持约 85% 性能）。
- 超图结构优于传统二部图，能更好地捕捉高阶用户-物品关系。
- 注意力机制比 GRU/简单求和更适合跨时间片依赖建模。

## 7. 优点（方法/实验设计亮点）
- **方法创新**：
  - 首次将超图全局意图与时序上下文 GNN 结合，实现多粒度意图建模。
  - 跨视图自监督学习直接对齐全局与局部表示，创新性地利用“真实偏好跨视图一致、噪声引起差异”的原则，增强鲁棒性。
  - 序列依赖模块补充了时间顺序模式，提升表示完整性。
- **实验设计亮点**：
  - 对比基线数量多（18个），涵盖主流范式，说服力强。
  - 噪声实验注入真实比例的噪声（5%-20%），贴近实际场景。
  - 超参数分析展示了关键参数（α, Lg, La, M）的敏感性，有助于理解模型行为。
  - 效率对比证明模型复杂度可控，适合实际部署。

## 8. 不足与局限
- **局限**：
  - 未在更多元化的领域（如新闻、短视频、社交）验证，可能泛化性受限。
  - 未来的工作提到需要“自适应超图结构”，暗示当前超图结构是预先定义而非动态学习的，可能无法完全适应偏好演化。
  - 训练时间虽相对高效，但文中未给出绝对时间，难以直接复现。
  - 对比方法中部分基线（如 CLSR）在某些指标上表现较差，但未深入分析原因。
  - 未讨论模型对超参数敏感度在跨数据集上的稳定性（仅展示了 MovieLens 和 Gowalla 的结果）。
- **偏差风险**：案例研究仅展示两个用户，可能有选取偏差；需更大规模定性分析。
- **应用限制**：超图结构构建和 GNN 传播可能在大规模实时场景下仍有计算挑战（尽管推理低延迟）。

（完）
