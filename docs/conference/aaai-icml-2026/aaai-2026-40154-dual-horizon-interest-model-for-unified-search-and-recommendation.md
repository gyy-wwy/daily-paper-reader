---
title: Dual-Horizon Interest Model for Unified Search and Recommendation
title_zh: 面向统一搜索与推荐的双视角兴趣模型
authors: "Wenhao Zhu, Yuxin Li, Shuo Wang, Hao Wang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/40154/44115"
tags: ["query:rec-sys"]
score: 6.0
evidence: 统一搜索与推荐，长短时兴趣
tldr: 现有统一搜索推荐框架未能区分用户的长短时兴趣，忽略了其在不同场景下的不同动态。本文提出DHIM，从搜索和推荐中独立提取长短时兴趣并整合。实验表明该模型在联合搜索推荐任务上的性能优于现有统一方法。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 搜索与推荐统一框架中，用户长短时兴趣的动态差异被忽视。
method: "提出DHIM，从搜索和推荐中独立提取长短时兴趣,并通过统一策略整合。"
result: 在统一搜索推荐基准上，DHIM在多个指标上取得了领先性能。
conclusion: 显式建模长短时兴趣能有效提升搜索推荐的统一性能。
---

## Abstract
Search and recommendation are pivotal for information access and are increasingly unified to exploit shared user-item interactions. Both tasks suffer from data sparsity, which joint modeling can mitigate by integrating behavioral data with or without explicit queries. However, existing unified frameworks rarely distinguish between users’ long- and short-term interests, despite their divergent temporal dynamics in search and recommendation. In this work, we propose a novel model, DHIM, which explicitly disentangles and integrates users' long- and short-term interests across both the search and recommendation scenarios. First, long- and short-term interests are independently extracted from search and recommendation using a unified extraction strategy. These interests are then adaptively integrated via a cross-scenario fusion module. A self‐supervised contrastive loss supervises the learning of both interest types within and across scenarios. The resulting representations are fed into downstream search and recommendation models for prediction. Extensive experiments on two public benchmarks demonstrate that our approach consistently outperforms single-scenario and state-of-the-art joint models, achieving superior accuracy and generalizability. To our knowledge, this is the first work to incorporate explicit dual-horizon interest modeling into a unified search and recommendation framework with self-supervised contrastive learning.

---

## 论文详细总结（自动生成）

## 1. 论文的核心问题与整体含义

- **研究动机**：搜索与推荐是信息获取的主要方式，现代平台（如YouTube、TikTok）越来越多地将两者统一，以利用共享的用户-物品交互数据。但两者均面临数据稀疏问题，联合建模可以缓解。然而，已有统一框架很少区分用户的长期（稳定）兴趣与短期（动态）兴趣，而这两类兴趣在搜索和推荐场景中具有不同的时间动态特性（例如，用户长期偏好电子商品，短期可能关注汽车；推荐行为可能触发后续搜索）。
- **核心问题**：如何在一个统一的搜索与推荐框架中显式地解耦和整合用户的长期与短期兴趣，从而提升两个任务的预测准确性和泛化能力。
- **整体含义**：本文提出DHIM模型，首次将双视角兴趣（长期vs短期）显式建模到统一搜索推荐框架中，并引入自监督对比学习来增强兴趣表征的一致性。实验表明，该方法显著优于单场景模型和现有联合模型。

## 2. 论文提出的方法论

- **核心思想**：从搜索和推荐行为序列中分别独立提取长期和短期兴趣，然后通过跨场景融合模块自适应整合，并利用自监督对比学习（包括场景内和场景间的约束）来监督兴趣学习。
- **关键技术细节**：
  - **嵌入层**：用户、物品、查询词的嵌入；查询表示为词嵌入的平均；采用查询-物品双向对比对齐损失 \(L_Q\)。
  - **跨场景行为融合模块**（基于UniSAR）：使用两个场景独立的Transformer编码器提取场景内特征，再使用一个全局Transformer（带二元掩码限制跨场景注意力）提取跨场景特征；通过多头交叉注意力融合；并加入双向对比对齐损失 \(L_{Align}\)（分别对推荐和搜索）。
  - **双视角兴趣编码器**：
    - **长期兴趣编码器**：将用户嵌入投影为长期查询向量，对历史行为表示进行注意力加权（使用MLP融合差值、点积等交互特征），得到长期兴趣表示 \(u_{r\ell}, u_{s\ell}\)。
    - **短期兴趣编码器**：使用GRU编码行为序列，取最后隐藏状态作为短期查询，再对GRU隐藏状态进行注意力加权，得到短期兴趣表示 \(u_{rs}, u_{ss}\)。
  - **跨场景双视角兴趣融合**：通过两个轻量级门控网络融合推荐和搜索场景下对应的长期/短期兴趣，得到统一表示，再解码回各场景空间。
  - **自监督对比学习**：
    - **场景内对比**：对每个场景，将长期兴趣与完整序列平均嵌入（正例）靠近，与近期平均嵌入（负例）拉远；短期兴趣反之。使用三元组损失 \(L_{con}^{intra}\)。
    - **场景间对比**：将搜索侧兴趣投影到推荐空间，对齐相同类型兴趣（长期-长期，短期-短期），分离不同类型。使用三元组损失 \(L_{con}^{inter}\)。
  - **预测层**：采用渐进式分层提取（PLE）多任务学习框架，共享专家网络并输出搜索和推荐两个任务的预测分数。先对候选物品进行长短兴趣自适应融合（门控网络），再输入PLE。
  - **训练目标**：包含点击预测的交叉熵损失、对齐损失、对比损失、查询-物品损失，以及L2正则化。总损失为推荐损失 + \(\lambda_1\)×搜索损失 + \(\lambda_2\)正则项。
- **公式/算法流程（文字说明）**：
  1. 输入用户行为序列（含搜索和推荐两种类型），嵌入后得到推荐行为表示 \(H_r\) 和搜索行为表示 \(H_s\)。
  2. 跨场景行为融合模块输出融合后的 \(H_r, H_s\)。
  3. 双视角兴趣编码器分别提取推荐场景的长期兴趣 \(u_{r\ell}\)、短期兴趣 \(u_{rs}\)，以及搜索场景的 \(u_{s\ell}, u_{ss}\)。
  4. 跨场景融合门控网络产生统一的长期表示 \(u_\ell\) 和短期表示 \(u_s\)，再解码回各场景。
  5. 自监督对比损失（场景内+场景间）监督兴趣学习。
  6. 对候选物品，通过门控自适应融合长短兴趣得到 \(u_{ti}\)，结合目标条件注意力得到最终特征，输入PLE多任务预测层得到点击概率。

## 3. 实验设计

- **数据集**：
  - **KuaiSAR**：真实世界用户行为，包含搜索和推荐场景的跨场景交互。
  - **Amazon（Kindle Store子集）**：半合成基准，根据先前工作生成搜索行为。
- **基准方法**：
  - 单场景推荐模型：DIN、FMLP-Rec、BASRec。
  - 单场景个性化搜索模型：AEM、ZAM、MAI。
  - 联合搜索推荐模型：SESRec、UnifiedSSR、UniSAR。
- **评价指标**：Hit Ratio (HR@1,5,10) 和 NDCG@5,10。负采样：每个正例配99个随机负例；为避免位置偏差，测试时随机化正负样例顺序。
- **实现细节**：嵌入维度64；序列长度30；PLE使用4个共享专家+4个任务特定专家；批次1024；短期窗口大小：搜索K=2，推荐K=4；对比损失超参数γ=0.001，α=β=0.01，κ和λ1在{0.001,0.01,0.1}和{0.01,0.1}中调优。

## 4. 资源与算力

文中**未明确**提及所使用的GPU型号、数量或训练时长。仅在实现细节中给出批量大小等设置，未汇报计算资源消耗。

## 5. 实验数量与充分性

- **主要结果**：在KuaiSAR和Amazon两个数据集上，对推荐任务和搜索任务分别报告HR@1,5,10和NDCG@5,10，并标注统计显著性（p<0.01）。对比了10种基线方法，覆盖单场景和联合模型。
- **消融实验**：在KuaiSAR上进行了详细的消融研究（表2），包括：
  - 删除长期/短期兴趣（推荐分支、搜索分支共4种变体）
  - 删除跨场景融合模块（w/o CSF）
  - 删除场景内对比损失（推荐/搜索各一次）
  - 删除场景间对比损失
  - 删除查询-物品对齐损失
  - 删除PLE框架（改为独立MLP）
  - 仅使用单场景（只推荐或只搜索）
- **充分性评价**：实验较为充分，数据集涵盖了真实和半合成场景；对比了主流方法；消融实验覆盖了所有关键模块。但仅在2个数据集上验证，尤其是在Amazon上搜索任务是半合成的，可能存在一定偏差。未提供跨域或更大规模数据集的结果。实验公平性方面，统一了负采样顺序以避免位置偏差，超参数调优遵循原论文。总体而言，实验设计与分析较客观。

## 6. 论文的主要结论与发现

- DHIM在推荐和搜索两个任务上均显著优于所有基线，尤其在KuaiSAR上HR@1提升2.06%，NDCG@10提升2.49% vs UniSAR。
- 联合建模统一搜索推荐优于单场景方法，验证了跨任务互补信号的价值。
- 消融表明：长期兴趣和短期兴趣均不可或缺；跨场景融合模块、自监督对比损失（尤其是场景间对比）和PLE多任务学习框架对性能贡献显著。
- 自监督对比学习能有效解耦长短兴趣并实现跨场景对齐，提升泛化能力。

## 7. 优点

- **首次将双视角兴趣建模引入统一搜索推荐**，思路创新，填补了现有联合模型的空白。
- **方法设计系统且模块化**：从行为融合、兴趣编码、跨场景融合到自监督对比学习，每一步都有明确动机和对应损失函数，结构清晰。
- **综合利用多种损失**（对比、对齐、点击预测）协同优化，并通过PLE解决多任务冲突。
- **实验设置规范**：统一了评估中的位置偏差问题；消融实验全面，验证了各组件的必要性。
- **代码与可复现性**：虽未明确提供代码链接，但给出了详细超参数设置。

## 8. 不足与局限

- **数据集限制**：仅使用两个公开数据集（其中一个为半合成），缺乏工业级大规模真实数据验证；搜索场景在Amazon上是生成的，可能与真实搜索行为有差距。
- **计算资源未报告**：无法评估训练成本与可扩展性。
- **对比基线覆盖有限**：虽然比较了多种联合模型，但未对比最近基于LLM的方法（如LREF等），可能失去最新对比参考。
- **短窗口大小固定**：短期兴趣窗口K值人为设定（推荐K=4，搜索K=2），未进行敏感度分析，可能依赖特定数据集特性。
- **未讨论负迁移风险**：虽然消融中单场景性能下降，但未深入分析不同用户群体或冷启场景下的效果差异。
- **方法复杂度**：模型包含多个Transformer、GRU、门控网络等，参数量较大，且需要预训练或端到端优化，对资源要求较高。
- **理论分析欠缺**：未从理论上解释为什么双视角兴趣建模能提升统一性能，仅凭实验验证。

（完）
