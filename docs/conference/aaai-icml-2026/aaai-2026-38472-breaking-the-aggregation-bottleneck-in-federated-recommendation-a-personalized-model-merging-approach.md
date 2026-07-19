---
title: "Breaking the Aggregation Bottleneck in Federated Recommendation: A Personalized Model Merging Approach"
title_zh: 突破联邦推荐中的聚合瓶颈：个性化模型融合方法
authors: "Jundong Chen, Honglei Zhang, Chunxu Zhang, Fangyuan Luo, Yidong Li"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38472/42434"
tags: ["query:rec-sys"]
score: 8.0
evidence: 联邦推荐系统
tldr: 联邦推荐中服务器聚合会损害客户端个性化，导致聚合瓶颈。本文提出FedEM，通过弹性融合全局和本地模型补偿个性化损失。理论分析和实验证明FedEM在多个联邦推荐场景中优于现有方法，兼顾了隐私与性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 联邦推荐中聚合过程削弱了客户端个性化能力，导致性能下降。
method: 提出FedEM，弹性融合全局与本地模型以恢复个性化。
result: 在多个联邦推荐数据集上，FedEM显著优于现有方法。
conclusion: 模型融合有效解决了联邦推荐的聚合瓶颈。
---

## Abstract
Federated recommendation (FR) facilitates collaborative training by aggregating local models from massive devices, enabling client-specific personalization while ensuring privacy. However, we empirically and theoretically demonstrate that server-side aggregation can undermine client-side personalization, leading to suboptimal performance, i.e., the aggregation bottleneck. This issue stems from the inherent heterogeneity across numerous clients in FR, which drives the global model to deviate from local optima. To this end, we propose FedEM, which elastically merges the global and local models to compensate for impaired personalization. Unlike existing personalized federated recommendation (pFR) methods, FedEM (1) investigates the aggregation bottleneck in FR through theoretical insights, rather than relying on heuristic analysis; (2) leverages off-the-shelf local models rather than designing additional mechanisms to boost personalization. Extensive experiments demonstrate that our method preserves client personalization during collaborative training, outperforming state-of-the-art baselines.

---

## 论文详细总结（自动生成）

## 论文详细中文总结

### 1. 核心问题与整体含义（研究动机和背景）

- **问题背景**：联邦推荐（Federated Recommendation, FR）通过聚合大量客户端的本地模型实现协作训练，同时保护用户隐私。然而，由于客户端间固有的数据异构性（Non-IID），传统FR方法共享同一个全局模型，导致个性化能力下降。
- **核心问题**：作者通过理论和实验证明，**服务器端聚合过程会损害客户端个性化**，即存在“聚合瓶颈”（Aggregation Bottleneck）。原因在于FR场景下客户端数量巨大（百万级），全局聚合使全局模型几乎不包含单个客户端的信息，导致优化方向偏离局部最优，最终性能次优。
- **研究动机**：现有个性化联邦推荐（pFR）方法要么忽视聚合瓶颈，要么依赖启发式设计的个性化机制（如额外训练模型、个性化微调），缺乏理论解释且兼容性有限。因此，需要一种简单、理论支撑且通用的解决方案。

### 2. 论文提出的方法论

#### 核心思想
- 基于**模型融合理论**，利用现成的本地模型来补偿聚合导致的信息损失。不同于传统FR的“静态替换”（用全局模型完全替换本地模型），提出**弹性融合（Elastic Merging）** 方案，以可学习的权重融合全局模型和本地模型，在保留协作信息的同时维持个性化。

#### 关键技术细节
1. **弹性合并模块**（Elastic Merging, EM）：
   - 对于客户端\(u\)，合并后模型：\[Q_m^{(t+1)} = \rho \odot Q_g^{(t-1)} + (1-\rho) \odot Q_l^{(t-1)}\]
     其中\(\rho \in [0,1]^m\)为逐项可学习的融合向量，由一个小型MLP适配器（Adapter）根据全局-本地差异\(Q_\Delta = Q_g - Q_l\)和本地模型\(Q_l\)生成。
   - 适配器输入为拼接向量\([Q_\Delta, Q_l]\)，通过L层MLP输出\(\rho\)，确保每个元素在0-1之间。
   - 适配器参数仅在合并步骤更新，本地训练和全局聚合时冻结。

2. **本地训练**：使用合并后的模型\(Q_m\)作为初始化，对用户嵌入和项目嵌入进行常规SGD更新。

3. **全局聚合**：采用**基于相似度的客户端特定聚合**，生成每个客户端的个性化聚合权重\(\mathbf{w}_u\)，目标函数为：
   \[
   L_g = \sum_v \big((w_{uv} - p_v)^2 + \alpha (w_{uv} - \sigma(Q_l^u, Q_l^v))^2\big)
   \]
   其中\(p_v\)为数据量权重，\(\sigma\)为相似度度量。约束满足权重归一化且非负。

4. **算法流程**（每轮\(t\)）：
   - 客户端下载全局模型\(Q_g^{(t-1)}\)，和本地模型\(Q_l^{(t-1)}\)一起输入适配器得到\(\rho\)，弹性合并得到\(Q_m^{(t)}\)。
   - 基于本地数据训练更新\(p_u\)和\(Q_l^{(t)}\)。
   - 上传\(Q_l^{(t)}\)到服务器。
   - 服务器基于相似度计算聚合权重，生成个性化全局模型\(Q_g^{(t)}\)，下发给各客户端。

### 3. 实验设计

#### 数据集
- **Filmtrust**、**ML-100K**、**ML-1M**、**LastFM-2K**：覆盖不同客户端规模和稀疏程度，稀疏度从低到高。

#### 评估协议
- **留一法**（leave-one-out evaluation），报告指标：**HR@10** 和 **NDCG@10**。

#### 对比方法
- **集中式方法**：MF、NCF、LightGCN。
- **联邦方法**：FedMF、FedNCF、FedFast、PFedRec、CoLR、GPFedRec、FedRAP、FedCIA（均属SOTA）。

#### 实现细节
- 全局回合\(T=100\)，批次大小256，嵌入维度\(d=16\)，学习率\(\eta=\beta=0.1\)，本地epoch=10，Adam优化器。

### 4. 资源与算力

- **文中未明确说明使用的GPU型号、数量及训练时长**。仅提到计算复杂度：本地骨干模型复杂度\(O((m+1)d)\)，EM模块（L层MLP）复杂度\(O(L d^2)\)，其中\(m \gg d > L\)，因此开销可忽略。未提及其他算力资源。

### 5. 实验数量与充分性

- **实验种类丰富**：
  1. **总体性能对比**（表1）：4个数据集×16种方法，全面。
  2. **收敛性分析**（图4）：在ML-1M上对比8种方法的收敛曲线。
  3. **消融实验**（表2）：从FedMF出发，逐步添加相似度聚合、静态融合、动态融合、弹性融合，验证各组件的贡献。
  4. **模型可视化**（图5）：t-SNE显示不同模型嵌入分布，定性说明弹性融合的优势。
  5. **兼容性实验**（表3）：将EM模块集成到5种现有FR/pFR方法中，观察性能提升（提升幅度大）。
  6. **超参数分析**（图6）：适配器层数L的影响；相似度系数α设为1左右。
  7. **隐私保护实验**（表4）：加入局部差分隐私（LDP）后性能轻微下降但仍优于基线。
- **充分性评价**：实验设计全面，覆盖性能、收敛、消融、可视化、兼容性、超参数和隐私，多维度验证方法有效性。对比方法均为当时SOTA，实验设置公平。不过缺少在更大规模真实推荐场景（如百万级用户）上的验证。

### 6. 论文的主要结论与发现

- **聚合瓶颈存在**：理论（引理1）和实验均证实全局聚合导致本地个性化信息丢失，静态替换方案次优。
- **弹性融合有效**：弹性合并（引理2）可补偿优化偏差，保留本地优化信号，且通过客户端自适应的\(\rho\)实现更好平衡。
- **FedEM超越SOTA**：在四个数据集上，FedEM在HR@10和NDCG@10上均优于所有对比的联邦方法，甚至超过部分集中式方法。
- **兼容性强**：EM模块作为即插即用插件，显著提升现有FR方法性能（最高提升212%）。
- **轻量高效**：复杂度低，适合设备端部署；加入LDP后仍保持良好性能。

### 7. 优点

- **理论奠基**：首次严格分析联邦推荐中的聚合瓶颈，给出引理和补偿条件，而非仅凭经验。
- **方法简洁有效**：直接利用现成本地模型，无需额外复杂机制，实现简单且通用。
- **模型无关插件**：可无缝集成到各类FR/pFR方法，提升效果显著。
- **实验充分严谨**：从多个角度验证，包括消融、可视化、兼容性、超参数、隐私，结论可靠。
- **考虑实际部署**：复杂度分析指出EM模块开销小，适合资源受限的移动设备；加入LDP保护隐私。

### 8. 不足与局限

- **大规模场景验证缺失**：实验仅在常规规模数据集（最大ML-1M约100万评分）上测试，未在数百万用户、十亿级别评分的大规模工业场景下评估。
- **稀疏场景性能有限**：在极稀疏数据集LastFM-2K上，FedEM仍优于其他联邦方法，但低于集中式LightGCN，说明高度稀疏下联邦训练的局限性。
- **适配器结构需手动设定**：MLP层数L影响性能，需根据具体任务调整（实验中L=3最佳），可能缺乏自适应能力。
- **隐私-效用权衡未深入**：加入LDP后性能下降约1-4%，论文未讨论更优的隐私预算选择或与其他差分隐私方法的对比。
- **收敛性分析限于单数据集**：仅给出ML-1M上的收敛图，其他数据集上收敛行为未展示。
- **理论假设的限制**：引理和理论分析基于平滑性和近似凸性假设，而实际推荐任务损失函数通常非凸，理论结论仅提供定性见解。

（完）
