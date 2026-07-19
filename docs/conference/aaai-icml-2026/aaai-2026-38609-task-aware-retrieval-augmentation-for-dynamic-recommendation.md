---
title: Task-Aware Retrieval Augmentation for Dynamic Recommendation
title_zh: 任务感知检索增强用于动态推荐
authors: "Zhen Tao, Xinke Jiang, Qingshuai Feng, Haoyu Zhang, Lun Du, Yuchen Fang, Hao Miao, Bangquan Xie, Qingqiang Sun"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38609/42571"
tags: ["query:rec-sys"]
score: 8.0
evidence: 动态顺序推荐的任务感知检索增强
tldr: 动态推荐系统中，预训练图神经网络微调时因时间分布差异导致泛化能力不足。本文提出TarDGR框架，通过任务感知评估机制与检索增强模块，协同提升模型对用户偏好演变的建模能力。在多个真实数据集上的实验表明，该方法显著优于现有动态推荐模型。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 预训练动态图神经网络因时间分布差异导致微调泛化差。
method: 提出TarDGR，结合任务感知评估和检索增强，提升对偏好演变的捕获。
result: 在多个数据集上显著优于现有动态推荐方法。
conclusion: 任务感知检索增强有效提升动态推荐的泛化能力。
---

## Abstract
Dynamic recommendation systems aim to provide personalized suggestions by modeling temporal user-item interactions across time-series behavioral data. Recent studies have leveraged pre-trained dynamic graph neural networks (GNNs) to learn user-item representations over temporal snapshot graphs. However, fine-tuning GNNs on these graphs often results in generalization issues due to temporal discrepancies between pre-training and fine-tuning stages, limiting the model’s ability to capture evolving user preferences. To address this, we propose TarDGR, a task-aware retrieval-augmented framework designed to enhance generalization capability by incorporating task-aware model and retrieval-augmentation. Specifically, TarDGR introduces a Task-Aware Evaluation Mechanism to identify semantically relevant historical subgraphs, enabling the construction of task-specific datasets without manual labeling. It also presents a Graph Transformer-based Task-Aware Model that integrates semantic and structural encodings to assess subgraph relevance. During inference, TarDGR retrieves and fuses task-aware subgraphs with the query subgraph, enriching its representation and mitigating temporal generalization issues. Experiments on multiple large-scale dynamic graph datasets demonstrate that TarDGR consistently outperforms state-of-the-art methods, with extensive empirical evidence underscoring its superior accuracy and generalization capabilities.

---

## 论文详细总结（自动生成）

### 论文详细中文总结

#### 1. 论文的核心问题与整体含义（研究动机和背景）
- **核心问题**：动态推荐系统中，预训练的图神经网络（GNN）在微调时面临**时间分布差异**（temporal discrepancy）导致的泛化能力不足问题。即预训练阶段和历史交互图与当前时刻的交互图存在分布偏移，使得模型难以准确捕捉用户偏好的演变。
- **研究动机**：现有基于检索增强（RAG）的方法主要依赖结构或特征相似性检索历史子图，忽略了**语义任务相关性**（task relevance），导致检索到的子图可能对当前推荐任务无益甚至有害。同时，手动标注任务相关子图在图表征学习中几乎不可行。
- **整体含义**：论文提出任务感知的检索增强框架 **TarDGR**，首次将**任务感知（task-awareness）** 引入动态图推荐，通过自动识别并融合与当前推荐任务语义相关的历史子图，增强模型在时间偏移下的泛化能力。

#### 2. 论文提出的方法论：核心思想、关键技术细节、公式或算法流程
- **核心思想**：将检索增强与任务感知结合，使模型能自动识别哪些历史子图对当前推荐“有益”，并利用这些子图丰富当前查询子图的表示。
- **关键技术细节**：
  - **Task-Aware Evaluation Mechanism（任务感知评估机制）**：利用预训练推荐模型计算查询子图与正样本的相似度，在融合候选子图后计算相似度变化量  
    \[
    \Delta REL = SIM_{after} - SIM_{before}
    \]  
    根据 \(\Delta REL\) 的正负将候选子图分为“有益”（>0）、“无关”（≈0）、“有害”（<0），从而自动构建带任务相关性分数 \(C_r\) 的训练数据集 \(\mathcal{D}_{aware}\)，无需人工标注。
  - **Graph Transformer-based Task-Aware Model**：包括：
    - **语义编码器**：对查询和候选子图分别经GCN编码后拼接，加入位置嵌入，通过多头自注意力捕捉语义关系，得到 \(h_{sem}\)。
    - **结构编码器**：对拼接后的表示进行线性投影，通过多层自注意力 + 残差归一化 + 前馈网络，再经归一化邻接传播得到结构编码 \(h_{str}\)。
    - **融合与评分**：拼接 \(h_{sem}\) 和 \(h_{str}\) 得到 \(h_{task}\)，经MLP映射为任务相关性分数 \(s_i\)。
  - **BiSCL预训练（Bi-Level Supervised Correlation Loss）**：包含：
    - 幅度拟合损失 \(L_{mtl}\)：训练预测分数与真实 \(C_r\) 的均方误差。
    - 序数对比损失 \(L_{ocl}\)：保持样本间排序一致性（分数高的子图预测分数也高）。
    - 联合损失 \(L_{BiSCL} = \rho L_{ocl} + (1-\rho)L_{mtl}\)。
  - **推理与训练**：
    - 检索：对查询子图，用训练好的任务感知模型从资源库中选出 Top-M 个分数最高的子图。
    - 融合：对检索子图进行内聚合得到 \(H_{rag}\)，通过残差门控融合 \(\tilde{h}_q = \beta h_q + (1-\beta)H_{rag}\)。
    - 推荐损失：结合 BPR 损失、基于扰动的 margin ranking 损失（\(L_{mrl}\)）和正则化损失（\(L_{reg}\)）进行联合微调。

#### 3. 实验设计：数据集、benchmark、对比方法
- **数据集**（三个真实动态推荐场景）：
  - **Taobao**：10天隐式反馈
  - **Koubei**：9周用户-商店交互（来自阿里）
  - **Amazon**：13周商品评论数据
- **Benchmark**：采用 Recall@20 和 nDCG@20 作为评估指标，划分训练/测试集。
- **对比方法**（四类共12种基线）：
  - **GNN推荐器**：LightGCN, SGL, MixGCF, SimGCL
  - **动态GNN**：EvolveGCN-H/O, ROLAND, GraphPro
  - **图提示学习**：GraphPrompt, GPF
  - **检索增强模型**：PRODIGY, RAGRAPH（其中GraphPro作为基础模型）

#### 4. 资源与算力
- **未明确说明**：论文正文及附录中均未提及 GPU 型号、数量、训练时长等算力信息。需要指出这一缺失。

#### 5. 实验数量与充分性
- **主实验**：表1 展示了三个数据集上所有方法的 Recall/nDCG，每个方法报告均值±标准差（基于多次重复？），对比充分。
- **消融实验**（表2）：分别去除语义/结构编码器、全部去除，验证各组件贡献。
- **超参数分析**（图7）：对 Top-M（检索数量）和 k-hop（邻域深度）进行敏感性实验。
- **训练资源影响**（图5）：分析任务感知模型训练时使用的历史资源比例对性能的影响。
- **时间泛化对比**（图6）：将 TarDGR 与 RAGRAPH 在每个时间步上的性能进行逐点比较。
- **总体评价**：实验设计较为完整，覆盖了主要基线、消融、超参数和泛化分析，结果客观公平（多次运行取均值、报告方差）。但未进行统计显著性检验（如t检验），且所有实验均基于离线数据集，缺少在线或大规模部署验证。

#### 6. 论文的主要结论与发现
- TarDGR 在所有三个数据集上均显著优于所有基线，尤其对基础模型 GraphPro 的提升幅度最大（如 Amazon 上 Recall 提升 14.5%，nDCG 提升 16.6%）。
- 任务感知评估机制能有效区分有益/有害子图，自动构建监督信号。
- 语义编码器比结构编码器更重要（移除语义导致更大性能下降），但两者互补。
- Top-M 和 k-hop 均存在最优值：适量检索可增强泛化，过多则引入噪声；邻域过大导致信息冗余。
- 随着训练资源比例增加，性能先快速上升后饱和，表明任务感知模型需要一定量监督但不宜过多。

#### 7. 优点
- **创新性**：首次将任务感知引入图检索增强推荐，解决了语义对齐问题。
- **自动化**：无需人工标注，自动利用预训练模型构建任务相关数据集。
- **模型设计**：Graph Transformer 融合语义与结构编码，配合 BiSCL 双层次损失（幅度+序数）有效学习相关性分数。
- **泛化能力**：通过检索历史任务相关子图，缓解时间分布漂移，多个实验表明跨时间步性能稳定优于基线。
- **实验充分**：涵盖多个数据集、多种基线、消融及超参数分析，结果可信。

#### 8. 不足与局限
- **算力资源未报告**：无法评估方法在实际部署中的计算开销（训练/推理效率）。
- **依赖预训练GNN质量**：任务感知评估依赖于预训练推荐模型，若预训练不佳将影响后续裁判准度。
- **静态资源库假设**：历史子图库在训练前固定，未考虑在线增量更新，可能无法适应长期概念漂移。
- **超参数敏感**：Top-M 和 k-hop 需调优，且最优值因数据集而异，缺乏自适应机制。
- **场景覆盖有限**：仅测试了电商、生活服务类数据集，未涉及社交网络、学术引用等其他动态图场景。
- **统计检验缺失**：未报告模型间差异的显著性（如p值），对比结论主要依赖于均值高低。
- **理论分析有限**：附录虽提供泛化理论分析，但较为简略，缺乏严格的证明或上界推导。

（完）
