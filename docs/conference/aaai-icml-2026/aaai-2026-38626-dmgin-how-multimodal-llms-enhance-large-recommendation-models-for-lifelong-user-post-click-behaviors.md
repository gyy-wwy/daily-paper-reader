---
title: "DMGIN: How Multimodal LLMs Enhance Large Recommendation Models for Lifelong User Post-click Behaviors"
title_zh: DMGIN：多模态大语言模型如何增强大规模推荐模型用于终身用户点击后行为
authors: "Zhuoxing Wei, Qingchen Xie, Qi Liu, Jingsong Yu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38626/42588"
tags: ["query:rec-sys"]
score: 7.0
evidence: 多模态大语言模型，大规模推荐模型，终身行为
tldr: 大规模推荐模型在处理终身用户行为序列时面临计算和架构挑战，多模态嵌入的引入加剧了问题。本文提出DMGIN，通过多模态LLM增强推荐模型，有效融合多模态信息并优化计算效率。实验表明该方法在点击率预测上显著提升，同时保持了较低的计算开销。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 大规模推荐模型处理终身行为序列时存在计算开销大和多模态融合困难。
method: 提出DMGIN，利用多模态大语言模型与推荐模型协同，有效融入多模态特征。
result: 在点击率预测任务上，DMGIN降低了计算成本并提升了预测准确性。
conclusion: 多模态大语言模型能有效增强大规模推荐模型的终身行为建模能力。
---

## Abstract
Modeling user interest based on lifelong user behavior sequences is crucial for enhancing Click-Through Rate (CTR) prediction. However, long post-click behavior sequences themselves pose severe performance issues: the sheer volume of data leads to high computational costs and inefficiencies in model training and inference. Traditional methods address this by introducing two-stage approaches, but this compromises model effectiveness due to incomplete utilization of the full sequence context. More importantly, integrating multimodal embeddings into existing large recommendation models (LRM) presents significant challenges: These embeddings often exacerbate computational burdens and mismatch with LRM architectures. 
To address these issues and enhance the model's efficiency and accuracy, we introduce Deep Multimodal Group Interest Network (DMGIN). Given the observation that user post-click behavior sequences contain a large number of repeated items with varying behaviors and timestamps, DMGIN employs Multimodal LLMs(MLLM) for grouping to reorganize complete lifelong post-click behavior sequences more effectively, with almost no additional computational overhead, as opposed to directly introducing multimodal embeddings. To mitigate the potential information loss from grouping, we have implemented two key strategies. First, we analyze behaviors within each group using both interest statistics and intra-group transformers to capture group traits. Second, apply inter-group transformers to temporally ordered groups to capture the evolution of user group interests. Our extensive experiments on both industrial and public datasets confirm the effectiveness and efficiency of DMGIN. The A/B test in our LBS advertising system shows that DMGIN improves CTR by 4.7% and Revenue per Mile by 2.3%.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：在点击率（CTR）预测任务中，如何高效建模用户的终身（lifelong）点击后行为序列，并有效融合多模态信息（如文本描述、图片、视频等）。传统方法面临两大挑战：一是长序列导致计算开销巨大，模型训练和推理效率低；二是直接将多模态嵌入引入大规模推荐模型（LRM）会加剧计算负担，且与LRM架构不匹配。
- **研究动机**：用户点击后行为序列中包含大量重复或高度相关的店铺（通过一致的名称和图像可识别），且伴随不同的行为细节和时间戳。作者假设用户对店铺的兴趣由其接触的多模态信息（文本描述和视觉内容）共同塑造。因此，希望借助多模态大语言模型（MLLM）对行为序列进行重组，在几乎不增加额外计算的前提下实现高效建模。
- **整体含义**：提出DMGIN（Deep Multimodal Group Interest Network），首次在工业级LRM中实现终身用户行为序列的高效端到端建模，同时融合多模态信息，显著提升CTR预测效果。

## 2. 论文提出的方法论：核心思想、关键技术细节

### 核心思想
- 利用MLLM生成店铺的多模态嵌入，并通过K-means聚类将所有店铺映射到语义一致的“兴趣分组”（group），从而将用户原始的长达数万条的行为序列压缩为数百个分组级序列，大幅降低序列长度。
- 为弥补分组可能带来的信息损失，设计两个增强策略：组内兴趣统计 + 组内行为演化（用Transformer），以及组间演化Transformer（用HSTU）捕捉分组间的时序依赖。
- 最后通过候选感知分组注意力模块（Candidate-Aware Group Attention Module）将目标物品与分组表示对齐，得到用户长期兴趣表示。

### 关键技术细节
- **跨模态表示学习模块（CMRLM）**：预训练一个类似CLIP的双塔模型，使用多种多模态对（店铺名-店铺图片、食品类别-产品描述、业务关键词-场景图片）进行对比学习，使文本和视觉嵌入对齐到共享语义空间。
- **兴趣驱动实体聚类模块（IDECM）**：推断阶段，对所有唯一店铺（而非每次用户行为）生成多模态嵌入，再使用K-means聚类。通过负载平衡检查和t-SNE可视化保证聚类质量。输出店铺到兴趣分组的映射。
- **组内兴趣增强模块（IGIEM）**：
  - **组级兴趣统计**：统计每个分组内各行为类别（强兴趣、弱兴趣、负反馈、支付等）的计数、平均/最大时长、平均消费金额等。
  - **组内行为演化**：对分组内按时间排序的行为序列（包含时间戳、位置、行为类型嵌入），使用多头自注意力（MHSA）捕捉演化模式，再取均值池化得到`dyna_s`。
  - **Top-k索引搜索**：基于每个分组内最大时间戳（最近交互时间）选出Top-k分组，优先考虑最近活跃的分组，全部行为被激活。
- **时序分组演化Transformer模块（TGETM）**：对时间排序后的分组表示，堆叠多层HSTU（Hierarchical Sequential Transduction Units），捕捉分组间的时序演化和相互影响。HSTU采用相对位置偏置、门控机制等。
- **候选感知分组注意力模块（CAGAM）**：计算目标物品嵌入与演化后分组表示之间的注意力权重，得到用户长期兴趣表示`r_long`。
- **缓存机制**：TGETM输出与候选物品无关，可离线预计算并缓存到Redis，线上服务直接读取，满足严格延迟要求。

### 公式说明
- 组内行为表示：`e_bi = concat(e_bi_itemid, e_bi_timestamp, ..., e_bi_behavior_type)`
- 多头自注意力：`MHSA(eb) = concat(head1,...,headh)W^O`，其中`head_i = Softmax((eb W^Q_i)(eb W^K_i)^T / sqrt(d')) eb W^V_i`
- 组内动态表示：`dyna_s = mean_pool(MHSA(eb))`
- 分组表示：`g_s = concat(dyna_s, stat_s)`
- HSTU中的操作：`U,V,Q,K = Split(phi1(f1(G)))`，`A(G)V(G) = phi2(Q(G)K(G)+bias_p) V(G)`，`Y(G) = f2(Norm(A(G)V(G))⊙U(G))`
- 候选感知注意力：`r_long = exp(sim(Wt e_t, Wg g'_s)) / sum_{s'} exp(sim(Wt e_t, Wg g'_{s'}))`

## 3. 实验设计：数据集、场景、基准方法

### 数据集
- **工业数据集（Industry）**：来自美团的LBS广告平台。使用过去30天日志训练，下一天测试。每个用户历史行为序列最长10000条，行为类型76种（点击、加购、收藏、浏览菜品、查看评论、下单等）。用户数约4亿，物品数500万，字段数317，样本量66亿。
- **公开数据集（Amazon）**：使用Grocery and Gourmet Food子集。包含287209个商品，5074160条带时间戳的评论及细粒度后点击行为。每个用户历史最长100条，将最近10条作为短序列，最近90条作为长序列。用户数135.7万，物品数47.4万，字段19，样本量7217万。

### 评估指标
- AUC（Area Under ROC Curve）和GAUC（Group AUC），GAUC考虑了用户组级别偏差。

### 对比方法（baselines）
- **短序列建模**：DIN、DIEN、DSIN。
- **两阶段检索-细化模型**：SIM、ETA、SDIM、TWIN、TWIN V2（全终身序列建模）。
- **直接建模全序列**：DIN Full、DSIN Full（带层次注意力结构）。

### 实现细节
- 工业数据集：嵌入维度16，K-means聚类数10000，学习率5e-4，训练时8块80G A100 GPU，单卡batch size 1024。
- Amazon数据集：嵌入维度18，学习率1e-3，单块80G A100 GPU，batch size 1024。
- 优化器：Adam。

## 4. 资源与算力
- **工业数据集**：使用8块NVIDIA A100 80GB GPU，batch size 1024/卡。文中未提及训练时长。
- **Amazon数据集**：使用1块NVIDIA A100 80GB GPU，batch size 1024。文中未提及训练时长。
- **参数存储**：基线约2.8GB，DMGIN约4.0GB（增加聚类嵌入和HSTU层）。
- **推理延迟**：从4-5ms增加到7-8ms/请求（因更深架构）。
- 文中未明确报告完整训练时间，但提到工业场景需要满足严格延迟要求，并通过缓存机制加速线上服务。

## 5. 实验数量与充分性

### 实验组数
- **主实验**：在Industry和Amazon两个数据集上，对比11种baseline方法，报告AUC和GAUC。
- **消融实验**：
  - 对IGIEM模块的两个子组件（兴趣统计、行为演化）进行消融：逐步移除，观察性能变化。
  - 对TGETM模块的深度（1-6层HSTU）进行消融，观察性能趋势。
- **线上A/B测试**：在工业LBS广告系统中进行数周的A/B测试，对比现行生产模型（两阶段长序列CTR模型），测量CTR和RPM提升。

### 充分性与公平性
- **充分性**：覆盖多个数据规模（工业大规模和公开中等规模），对比了多种代表性方法（短序列、两阶段、全序列），并做了关键组件消融和深度缩放实验。线上部署验证了实际效果。
- **客观与公平**：文中提到所有实验运行5次取平均，并使用相同评估指标。超参数设置明确。但未说明baseline是否经过调优，可能存在的过优训练未明确。

## 6. 论文的主要结论与发现

1. **DMGIN在工业数据集和Amazon数据集上均取得最优性能**，AUC和GAUC均显著超过所有baseline（包括TWIN V2、DSIN Full等）。
2. **组内行为演化模块比组内统计信息更重要**：移除行为演化导致AUC下降幅度远大于移除统计特征（工业数据集上约5倍）。表明动态演化是性能提升的主要驱动力。
3. **TGETM深度增加带来持续性增益**：从1层增加到6层HSTU，AUC单调上升（~0.3%-0.5%），未见饱和，暗示可进一步堆叠更多层。
4. **线上A/B测试**：CTR提升4.7%，RPM提升2.3%，验证了实际业务价值。
5. **效率可观**：通过聚类压缩序列长度（数万→数百），且使用缓存机制，仅增加少量资源消耗（参数增约1.2GB，延迟增3-4ms），可部署。

## 7. 优点：方法或实验设计上的亮点

- **创新性地利用MLLM进行物品级聚类而非直接嵌入**：避免了多模态嵌入计算爆炸，且只对所有唯一店铺计算一次，重复使用，几乎无额外计算开销。
- **双层次信息补偿**：组内统计+演化、组间时序演化，全面弥补分组信息损失。
- **端到端全序列建模**：无需两阶段检索，直接利用完整序列上下文，且压缩后仍保留兴趣演化。
- **工业级有效性**：在4亿用户、500万物品的超大规模平台验证，A/B测试显示显著业务提升。
- **缓存设计**：将TGET输出预计算并缓存，满足线上低延迟要求，实际部署可行。
- **消融实验严谨**：逐步验证每个模块贡献，量化各组件重要性。

## 8. 不足与局限

- **聚类质量依赖预训练和聚类参数**：K-means聚类数和质量可能影响最终效果，文中虽检查负载平衡和人工验证，但未讨论聚类数选择的敏感性或自适应方法。
- **实验覆盖局限性**：公开数据集（Amazon Grocery）最大历史长度仅100，无法充分体现DMGIN在极长序列（数万）上的优势。工业数据集虽大但未公开，可复现性受限。
- **未讨论多模态预训练数据规模和来源的细节**：预训练所用的多模态对数量、质量、领域分布等可能影响泛化能力。
- **资源消耗分析不完整**：训练时长未报告，难以评估训练成本。推理延迟仅给出均值，未考虑峰值或分位数。
- **消融实验未对比不同聚类方法**：仅用K-means，未探索更优聚类算法（如层次聚类、GMM）或自适应聚类。
- **可能存在的偏差风险**：分组可能导致长尾店铺信息被稀释，影响个性化。文中未分析分组对低频物品或冷启动用户的影响。
- **A/B测试时间长度和流量比例未说明**：缺乏统计显著性检验细节。

（完）
