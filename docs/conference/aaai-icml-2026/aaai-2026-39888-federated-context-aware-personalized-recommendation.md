---
title: Federated Context-Aware Personalized Recommendation
title_zh: 联邦上下文感知个性化推荐
authors: "Zhihao Wang, Xiaoying Liao, Wenke Huang, Bingqian Liu, Tian Chen, Jian Wang, Bing Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39888/43849"
tags: ["query:rec-sys"]
score: 9.0
evidence: 联邦上下文感知推荐
tldr: 该论文提出FedCAR，一种联邦上下文感知推荐框架。现有联邦推荐系统使用静态用户嵌入，忽略了行为模式且需大量数据调优。FedCAR动态利用用户近期交互作为上下文构建表示进行预测，在保护隐私的同时提升了推荐效果。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 联邦推荐系统中静态用户嵌入忽略行为模式，可解释性差且需大量数据。
method: 提出FedCAR，利用用户近期交互动态构建上下文表示，替代静态嵌入。
result: 实验表明FedCAR在多个联邦推荐场景下优于基线方法。
conclusion: FedCAR为隐私保护下的上下文感知推荐提供了一种有效方案。
---

## Abstract
Federated recommender system is emerging as a new paradigm for providing personalized services while preserving user data privacy. Most existing personalized federated recommender systems predict the user's next item by discretely training user and item embeddings. However, this training approach overlooks the user's behavioral patterns, suffers from low interpretability, and requires a substantial amount of data and meticulous fine-tuning to achieve stable and accurate embeddings. To address these limitations, we propose Federated Context-Aware Personalized Recommendation (FedCAR), a novel framework that leverages users’ recent interactions as behavioral context to guide prediction. Instead of static user embeddings, FedCAR dynamically constructs context representations by aggregating and weighting recently interacted item embeddings. Additionally, we incorporate a contrastive learning strategy that enables the model to capture shared behavioral structures across clients while maintaining personalized preferences, enhancing both generalization and robustness in heterogeneous settings. Experiments on 5 benchmark datasets show that FedCAR consistently outperforms state-of-the-art methods and provides interpretable recommendations by explicitly modeling context dependencies.

---

## 论文详细总结（自动生成）

# 中文总结：Federated Context-Aware Personalized Recommendation (FedCAR)

## 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：现有联邦推荐系统（Federated Recommendation）大多使用静态的用户和物品嵌入（discrete user/item embeddings），通过离散训练进行预测。这种方式忽略了用户的动态行为模式，可解释性差，且需要大量数据和精细调参才能获得稳定精准的嵌入，在数据异构和交互稀疏场景下表现不佳。
- **研究动机**：在保护用户隐私的联邦学习框架下，如何利用用户近期交互行为作为上下文（behavioral context），动态构建表示来替代静态嵌入，从而提升推荐的准确性、可解释性以及对异构客户端的泛化能力。
- **整体含义**：提出一种联邦上下文感知个性化推荐框架 FedCAR，通过本地上下文编码器和个性化对比学习，在保持数据本地性的同时捕捉用户短期偏好，提供隐私保护且可解释的推荐。

## 2. 方法论

### 核心思想
- 替代传统离散训练用户嵌入的方法，利用用户最近交互的物品序列作为上下文，通过注意力机制动态生成上下文表示，再与目标物品嵌入结合进行评分预测。
- 引入针对联邦设置的个性化对比学习，使模型既能学习跨客户端的共享行为结构，又能保持个性化偏好。

### 关键技术细节
1. **上下文感知本地建模**
   - 对每个客户端 \(k\)，取最近 \(L\) 个交互物品作为上下文序列 \(\text{context}_k\)。
   - 通过本地物品嵌入层 \(e_k\) 获取上下文物品嵌入矩阵 \(E_k \in \mathbb{R}^{L \times d}\)，加上可学习的位置编码 \(P\) 得到 \(Z_k\)。
   - 使用单层 Transformer 编码器（TransEnc）对 \(Z_k\) 进行编码得到 \(H_k\)。
   - 以当前目标物品的嵌入 \(q_k = e_k(i_{\text{current}})\) 作为 query，对 \(H_k\) 执行缩放点积注意力（Attn），得到上下文表示 \(c_k\)。
   - 将 \(c_k\) 与 \(q_k\) 拼接后通过客户端评分函数 \(S_k\) 得到预测分数 \(\hat{r}_{ki}\)。
   - 推荐损失采用交叉熵：\(\mathcal{L}_{\text{rec}}^k = -\sum_{i \in D_k} \log(\hat{r}_{ki}) - \sum_{i' \in D_k^-} \log(1 - \hat{r}_{ki'})\)，负样本从全部未交互物品中均匀采样。

2. **个性化对比适应**
   - 对每个客户端 \(k\)，当前轮上下文表示 \(c_{\text{curr}}^k\) 作为 anchor。
   - 正样本：同一客户端上一轮模型生成的上下文表示 \(c_{\text{pos}}^k = C_k^{(t-1)}(E_k^{(t-1)})\)。
   - 负样本：随机选取另一客户端 \(k'\) 的上轮上下文表示，并添加 Laplace 噪声 \( \epsilon \sim \text{Laplace}(0, \lambda_{\text{neg}})\)，得到 \(c_{\text{neg}}^k\)。
   - 对比损失（边际损失）：\(\mathcal{L}_{\text{cl}}^k = \max(0, \|c_{\text{curr}}^k - c_{\text{pos}}^k\|^2 - \|c_{\text{curr}}^k - c_{\text{neg}}^k\|^2 + \delta)\)。
   - 该损失仅在上下文编码器（TransEnc、注意力层、位置编码）上优化，不干扰推荐主目标，无需额外超参数平衡。

### 算法流程（文字描述）
- **全局通信轮次**：每轮随机选取部分客户端参与。
- **客户端本地更新**：
  1. 接收全局物品嵌入和上下文编码器参数。
  2. 利用本地数据计算推荐损失 \(\mathcal{L}_{\text{rec}}\) 更新所有参数。
  3. 利用上一轮本地上下文编码器生成正负样本，计算对比损失 \(\mathcal{L}_{\text{cl}}\) 仅更新上下文编码器。
- **服务端聚合**：对所有参与客户端的物品嵌入和上下文编码器分别取平均，更新全局模型。

## 3. 实验设计

### 数据集（5个公开基准）
- **MovieLens-100K**：943用户，1682物品，10万评分
- **MovieLens-1M**：6040用户，3952物品，100万评分
- **Amazon-Video**：11830用户，8072物品，6.4万交互
- **Lastfm-2K**：1600用户，12454物品，18.6万交互
- **HetRec2011**：2113用户，10109物品，85.6万交互

### 对比方法
- **中央方法**：MF (Matrix Factorization), NCF (Neural Collaborative Filtering)
- **联邦方法**：FedMF, FedNCF, FedRecon, PFedRec, GPFedRec

### 评价指标
- Hit Ratio @10 (HR@10)
- Normalized Discounted Cumulative Gain @10 (NDCG@10)

### 实现细节
- 过滤交互少于6个的用户，上下文长度 \(L=5\)；负样本采样方式：每个正样本配4个负样本；通信轮次 \(E=100\)，本地轮次 \(T=1\)；嵌入维度 \(d=32\)，批次大小256；评分函数为单层MLP；优化器SGD；固定随机种子；超参数 \(\lambda_{\text{neg}}=0.1, \delta=1\)。

## 4. 资源与算力

- 文中提到所有实验在 **NVIDIA 3090 GPU** 上使用 **PyTorch** 框架进行。
- **未明确说明** GPU 数量、模型参数量、具体训练时长等细节。

## 5. 实验数量与充分性

- **主实验**：5个数据集 × 7种方法（其中中央方法2个，联邦方法5个）→ 共35组条件，每个条件报告HR@10和NDCG@10。
- **消融实验**：在MovieLens-100K和Lastfm-2K上，分别移除上下文编码器和对比学习，共3种配置（仅有对比、仅有上下文、全无），检验各组件贡献。
- **可解释性案例**：选取Lastfm-2K中两个用户，展示正负样本的注意力分布，验证上下文对齐语义标签。
- **嵌入可视化**：使用t-SNE展示两个客户端的正负物品嵌入分离情况。
- **隐私噪声鲁棒性**：在Lastfm-2K上测试不同LDP噪声系数（0.1~0.5）下的HR@10表现，并与FedNCF比较。
- **充分性评价**：实验覆盖了常见联邦推荐基准，消融设计合理，超参数统一固定，种子固定确保可复现，对比方法包括最新强基线。但缺少不同上下文长度、嵌入维度、本地轮次等超参数敏感性分析，也未在显式评分数据集或跨域场景验证。

## 6. 主要结论与发现

- FedCAR 在所有5个数据集上，在联邦设置下均达到 **最优或次优** 的 HR@10 和 NDCG@10，尤其在 Lastfm-2K 上提升显著（HR 0.8236, NDCG 0.7222）。
- 消融实验表明 **上下文编码器** 和 **个性化对比学习** 均对性能有正向贡献，两者结合效果最好。
- 案例研究显示：模型能够将高注意力分配给语义相关的上下文物品（如同一音乐风格），从而产生可解释且准确的预测。
- t-SNE 可视化证明 FedCAR 学会区分正负物品嵌入，保留用户偏好信号。
- 引入本地差分隐私（Laplacian噪声）后，FedCAR 性能下降缓慢，显著优于对照方法，表明对随机噪声具有良好鲁棒性。

## 7. 优点

- **动态上下文建模**：替代静态嵌入，实时捕捉用户短期行为变化，增强适应性。
- **可解释性强**：注意力机制可直接显示哪些历史交互影响当前预测，提供语义对齐。
- **轻量级隐私设计**：避免显式用户嵌入，减少隐私泄露风险；可无缝集成差分隐私。
- **个性化对比学习**：利用时间一致性（正样本来自自身历史轮次）与跨客户端差异性（负样本加噪声）增强个性化，无需额外平衡权重。
- **模块化优化**：对比损失仅更新上下文编码器，不影响推荐主任务，实现简单。

## 8. 不足与局限

- **稀疏问题**：在极端稀疏交互场景下，所有方法性能仍偏低，FedCAR 虽优于基线但未彻底解决。
- **可扩展性验证不足**：框架基于单层Transformer，理论上轻量，但未在不同客户端容量、大型工业数据集或跨域推荐任务上测试。
- **实验覆盖有限**：
  - 未分析超参数（如上下文长度、对比损失边际值、Laplace噪声尺度）的敏感性。
  - 仅使用隐式反馈（评分二元化），未考虑显式评分或评分预测任务。
  - 未与基于图的方法（如FedPerGNN）充分对比（文中未列出FedPerGNN结果）。
- **资源细节缺失**：GPU数量、训练时间、模型参数量等未提供，影响复现与效率评估。
- **隐私分析不足**：虽提及LDP效果，但未从理论角度分析隐私预算，也没有与Secure Aggregation等更严格机制进行比较。

（完）
