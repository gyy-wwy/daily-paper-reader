---
title: Cross-Scale Collaboration between LLMs and Lightweight Sequential Recommenders with Domain-Specific Latent Reasoning
title_zh: LLM与轻量顺序推荐器的跨尺度协作及领域潜在推理
authors: "Yipeng Zhang, Xin Wang, Hong Chen, Junwei Pan, Qian Li, Jun Zhang, Jie Jiang, Hong Mei, Wenwu Zhu"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38680/42642"
tags: ["query:rec-sys"]
score: 9.0
evidence: LLM与轻量顺序推荐器的跨尺度协作及领域潜在推理
tldr: 顺序推荐中利用大语言模型的推理能力面临推理路径数据稀缺和语义表示脱节等问题。本文提出跨尺度协作框架，LLM负责领域特定潜在推理，轻量顺序模型进行高效推荐，二者协同工作。实验表明，该方法在多个顺序推荐基准上达到最佳性能，且推理效率较高。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: LLM在顺序推荐中推理能力受限于推理路径数据稀缺和语义表示脱节。
method: 提出跨尺度协作框架，LLM负责潜在推理，轻量模型高效推荐。
result: 在多个顺序推荐基准上达到最佳性能，推理效率高。
conclusion: 跨尺度协作有效利用LLM推理能力提升顺序推荐效果。
---

## Abstract
Sequential recommendation aims to predict the next item based on historical interactions. To further enhance the reasoning capability in sequential recommendation, LLMs are employed to predict the next item or generate semantic IDs for item representation, given LLMs' extensive domain knowledge and reasoning ability. However, existing LLM-based methods suffer from two limitations. (i) The scarcity of recommendation data with reasoning paths makes it challenging to design suitable chain-of-thought prompting templates, and the full potential of LLMs' reasoning abilities remains underutilized. (ii) Upon obtaining semantic IDs, the LLMs and their representations are excluded from the subsequent recommendation model training, preventing downstream models from fully utilizing the rich semantic information encoded within these IDs. To address these issues, we propose a novel CoderRec framework, which is capable of fully exploiting the information encoded in semantic IDs to guide the recommendation process. Specifically, to address the problem of scarcity in reasoning path-augmented data, we introduce latent reasoning into sequential recommendation and treat the representation captured by the downstream model as domain-specific latent thought, enabling implicit logical inference without requiring explicit CoT annotations. To ensure that the downstream recommendation models are able to deeply leverage the semantic information within IDs, we propose a novel cross-scale model collaboration strategy, which employs cross-scale IDs and a two-phase approach to align LLM-derived semantics with recommendation objectives. Extensive experiments have shown the effectiveness of our proposed CoderRec framework.

---

## 论文详细总结（自动生成）

好的，以下是对该论文的详细中文总结，按照您要求的8个要点组织。

---

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究背景**：顺序推荐（sequential recommendation）旨在根据用户历史交互预测下一项物品。大语言模型（LLM）凭借其丰富的领域知识和推理能力，被引入推荐系统以增强性能，例如通过思维链（CoT）提示生成语义ID或直接预测物品。
- **核心问题**：现有方法存在两大局限：
  - **推理路径数据稀缺**：推荐领域缺乏带有显式推理链的标注数据，难以设计通用的CoT模板，导致LLM的推理潜力未被充分利用。
  - **语义表示脱节**：通过RQVAE等压缩方法获得语义ID后，LLM和其编码的丰富语义信息被排除在下游推荐模型的训练之外，导致下游模型无法深度利用这些信息。
- **整体目标**：提出一种能**充分利用语义ID中编码的信息**来指导推荐过程的方法，同时在不依赖显式推理路径的情况下激发LLM的推理能力。

## 2. 论文提出的方法论：核心思想与关键技术

- **核心思想**：提出**CoderRec框架**，包含两大支柱：
  - **跨尺度模型协作（Cross-Scale Model Collaboration）**：将LLM提取的语义信息通过RQVAE压缩成语义ID，并创新性地将**原始物品ID**与**语义ID**作为**跨尺度ID（Cross-Scale IDs）**联合建模，设计**嵌入融合器（Embedding Fuser）**和**交叉尺度表示重构辅助损失（Cross-Scale Representation Reconstruction Auxiliary Loss）**，使下游推荐模型能够深层理解语义ID中的领域知识。
  - **领域特定潜在推理（Domain-Specific Latent Reasoning）**：受Quiet-STaR启发，在物品序列的token之间引入**潜在思维（latent thought）**，利用LLM并行生成潜在推理，并将下游推荐模型的表示作为**领域特定的潜在思维**，通过**思维融合器（Thought Fuser）**指导LLM的推理方向，从而在不需显式CoT数据的情况下实现隐式逻辑推理。

- **关键技术细节**：
  - **跨尺度ID建模**：对每个物品同时使用原始item ID和K+1维的语义ID（K层RQVAE码本索引+一个消歧维度），分别嵌入后通过线性层融合成最终表示。
  - **辅助损失**：除了推荐主损失（交叉熵），还设计了**交叉尺度表示重构损失**，强制下游模型预测的语义ID经过RQVAE解码后能接近LLM的原始表示，从而对齐语义空间。
  - **两阶段训练**：先进行**预热阶段**（仅下游推荐模型训练，损失为L_rec + λ1 L_aux），再**联合训练**推荐模型和LLM（损失为L_rec + λ1 L_aux + λ2 L_LLM）。
  - **潜在推理机制**：对每个序列位置，通过并行注意力掩码高效生成多个潜在思维token，并利用下游模型的表示（Domain-Specific Latent Thought）通过思维融合器加权融合，最终用于下一物品预测。

## 3. 实验设计

- **数据集**：使用Amazon数据集中的三个子类（Beauty、Sports、Instruments），均采用5-core设置。数据集规模：用户数2.2万~3.5万，物品数0.99万~1.8万，交互数19.8万~29.6万，稀疏度均很低（~0.0007~0.0008）。
- **对比方法**：包括传统方法（SASRec、Bert4Rec、S^3-Rec）、引入推理的小模型方法（ReaRec）、以及LLM-based方法（LLM-ESR、TIGER、LC-Rec、EAGER-LLM）。共8种基线。
- **评估指标**：Recall@K和NDCG@K，K∈{5,10}。
- **消融实验**：在Beauty数据集上对三大模块（w/o item ID、w/o L_aux、w/o reasoning）分别进行剥离实验，共4组对比（含完整模型）。
- **超参数敏感性**：对辅助损失权重λ1、λ2进行了网格搜索（取值0.1,0.01,0.001,0.0001,0.0）。

## 4. 资源与算力

- 论文正文中**未明确说明**使用的GPU型号、数量或训练时长。
- 代码仓库（https://github.com/defineZYP/CoderRec）可提供更详细环境配置，但本次分析未涉及。通常此类工作可能在单卡或少量GPU（如1~4张A100）上训练，具体待作者补充。

## 5. 实验数量与充分性

- **实验数量**：共在**3个数据集**上进行完整对比，每组报告4个指标（R@5、R@10、N@5、N@10），共形成4个表（表1为主结果、表2为数据集统计、表3为消融、表4为权重敏感性）。实验数量虽不多，但覆盖了主要维度的分析。
- **充分性与公平性**：
  - 对比方法涵盖了传统、轻量推理、LLM-based三类，较为全面。
  - 消融实验明确验证了每个模块的贡献，且权重分析展示了超参数的影响。
  - 数据集预处理遵循前人工作（TIGER等），保证了公平对比。
  - 但**未报告标准差**，可能结果存在波动。且仅使用了Amazon三个子集，未在更多领域（如新闻、视频）验证，泛化性有待确认。

## 6. 论文的主要结论与发现

- 提出的**CoderRec在三个数据集上全面超越所有基线**，在Beauty上R@5达到0.0607（第二优EAGER-LLM为0.0548），提升约10~15%。其他指标类似。
- **潜在推理对大小模型均有效**：ReaRec（纯小模型推理）性能接近甚至超过部分LLM方法；CoderRec结合LLM潜在推理后达到最佳。
- **跨尺度协作**（联合使用item ID与语义ID）和**辅助重构损失**均对性能有显著贡献，剥离后性能明显下降（如w/o item ID降幅达10%）。
- 损失权重λ2不宜过大，过大会导致LLM语义空间主导梯度，反而降低效果；预热阶段对对齐语义空间至关重要。

## 7. 优点

- **创新性**：首次在顺序推荐中将LLM潜在推理引入下游任务，无需显式CoT数据，解决数据稀缺问题。
- **架构设计**：提出跨尺度ID融合和双阶段训练，有效解决了语义ID信息损失问题，使LLM知识被深度利用。
- **效率与效果平衡**：通过并行注意力掩码和两阶段策略，在不过度增加计算开销的前提下提升性能。
- **实验设计**：消融实验和权重分析清晰验证了每个组件的必要性，对比基线全面，逻辑严谨。

## 8. 不足与局限

- **实验覆盖有限**：仅使用Amazon三个子领域（美妆、运动、乐器），未在更广泛推荐场景（如新闻、视频、电商全品类）验证，泛化性存疑。
- **缺乏统计显著性**：未报告多次实验的标准差或置信区间，结果波动性未知。
- **算力与效率考量**：文中未讨论LLM推理阶段的推理速度、显存占用等实际部署成本。潜在推理的并行机制虽高效，但整体模型仍涉及LLM多次前向，可能延迟较大。
- **依赖RQVAE预训练**：需要先训练RQVAE压缩LLM表示，增加了系统复杂度，且RQVAE的质量直接影响下游性能。
- **理论分析不足**：为什么潜在推理在推荐中有效？缺乏理论支撑，主要依赖实验验证。

（完）
