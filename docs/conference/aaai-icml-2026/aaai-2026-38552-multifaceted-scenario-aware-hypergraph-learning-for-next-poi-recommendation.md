---
title: Multifaceted Scenario-Aware Hypergraph Learning for Next POI Recommendation
title_zh: 多面场景感知超图学习用于下一兴趣点推荐
authors: "Yuxi Lin, Yongkang Li, Jie Xing, Zipei Fan"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38552/42514"
tags: ["query:rec-sys"]
score: 9.0
evidence: 利用场景感知超图学习进行下一个兴趣点推荐
tldr: 现有顺序推荐方法忽略不同场景（如游客与本地人）下的移动模式差异，导致推荐效果不佳。本文提出多面场景感知超图学习（MSAHG），采用场景分离范式分别建模不同场景特征并解决冲突。在真实LBSN数据上，该方法在下一POI推荐任务中显著优于现有方法。该工作强调了场景感知在顺序推荐中的重要性。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有顺序推荐方法未能区分不同场景下用户的移动模式，导致推荐性能下降。
method: 设计场景分离范式，为每个场景构建超图，并通过冲突解决机制融合多场景信息。
result: 在多个LBSN数据集上，MSAHG在下一POI推荐准确率上超过现有最佳模型。
conclusion: 场景感知超图学习能有效提升顺序推荐的个性化和准确性。
---

## Abstract
Among the diverse services provided by Location-Based Social Networks (LBSNs), Next Point-of-Interest (POI) recommendation plays a crucial role in inferring user preferences from historical check-in trajectories. However, existing sequential and graph-based methods frequently neglect significant mobility variations across distinct contextual scenarios (e.g., tourists versus locals). This oversight results in suboptimal performance due to two fundamental limitations: the inability to capture scenario-specific features and the failure to resolve inherent inter-scenario conflicts. To overcome these limitations, we propose the Multifaceted Scenario-Aware Hypergraph Learning method (MSAHG), a framework that adopts a scenario-splitting paradigm for next POI recommendation.
Our main contributions are:
(1) Construction of scenario-specific, multi-view disentangled sub-hypergraphs to capture distinct mobility patterns;
(2) A parameter-splitting mechanism to adaptively resolve conflicting optimization directions across scenarios while preserving generalization capability.
Extensive experiments on three real-world datasets demonstrate that MSAHG consistently outperforms five state-of-the-art methods across diverse scenarios, confirming its effectiveness in multi-scenario POI recommendation.

---

## 论文详细总结（自动生成）

### 1. 论文的核心问题与整体含义（研究动机和背景）

*   **研究背景**：位置社交网络（LBSN）中的下一个兴趣点（POI）推荐是预测用户下一步可能访问的地点，对个性化服务、城市移动分析等有重要价值。现有方法（序列模型、图模型、超图模型）通常假设用户行为单一，忽略了现实世界中不同情境（如本地用户与游客、工作日与周末、市区与郊区）下移动模式的显著差异。
*   **核心问题**：这些差异导致两个关键局限：①无法捕获场景专属特征；②无法解决跨场景间的优化冲突（如梯度方向相反）。作者认为这是现有方法性能次优的根本原因。
*   **整体含义**：论文首次明确将“多场景下一POI推荐”形式化，并强调场景感知建模在LBSN中的必要性，提出**MSAHG（Multifaceted Scenario-Aware Hypergraph Learning）**框架，采用“场景分割”范式来分别建模不同情境下的移动模式。

### 2. 论文提出的方法论

#### 核心思想
- **场景分割**：沿三个正交维度（用户类型：本地/游客；时间上下文：工作日/周末；空间区域：市区/郊区）将每条轨迹划分到具体场景组合（如“本地&周末&郊区”）。
- **双层面解耦**：
  - **数据层面**：为每个场景构造**多视图解耦子超图**，捕获场景内迁移模式。
  - **参数层面**：设计**自适应参数分裂机制**，动态解决跨场景间的梯度冲突。

#### 关键技术细节
1. **子超图构建**（共8个子超图 + 1个共享过渡超图）：
   - **用户类型视图**（协作子超图）：节点为POI，超边为单个用户轨迹，分本地/游客两组。
   - **时间视图**（时间-用户子超图 + 时间-POI子超图）：将一天分为48个时段作为超边，分别以用户或POI为节点，分工作日/周末。
   - **地理视图**（地理子超图）：节点为POI，超边为地理邻近POI集合（距离阈值内），分市区/郊区。
   - **过渡视图**（共享过渡超图）：节点为POI，超边为连续POI间的有向转移，不按场景分裂。
2. **子超图卷积网络**：
   - 采用节点→超边→节点的两阶段消息传播，聚合函数用平均池化，残差连接防止过平滑。
   - 对无向子超图：聚合所有节点到超边，再将超边消息传播回节点。
   - 对有向过渡超图：仅源节点到超边，再传播到目标节点。
   - 用户嵌入通过转置关联矩阵从POI嵌入推导（协作、过渡、地理视图），或直接从时间-用户视图获得，并通过门控机制融合：
     \[
     E_U^F = \lambda_{Tem} E_U^{Tem} + \lambda_C E_U^C + \lambda_T E_U^T + \lambda_G E_U^G
     \]
     其中 \(\lambda_X = \sigma(E_U^X W_X)\)，\(\sigma\) 为Sigmoid。
3. **对比学习**：对多视图嵌入计算InfoNCE损失，促进视图间协作。用户和POI分别有对比损失 \(L_U^{con}\) 和 \(L_P^{con}\)，最终每个场景的损失为：
   \[
   L_{final} = \lambda (L_U^{con} + L_P^{con}) + (1-\lambda) L_{Rec}
   \]
4. **自适应参数分裂**：
   - 对每个参数 \(\theta\)，计算各场景损失梯度的余弦相似度 \(s_{\theta}^{i,j} = g_\theta^i \cdot g_\theta^j\)（归一化后）。
   - 若 \(s_{\theta}^{i,j} < 0\) 表示冲突，则将 \(\theta\) 复制为 \(\theta'\)，按梯度相似性聚类场景，各自使用不同参数副本。
   - 周期性检查（每若干epoch），分裂后参数专属固定。模型大小适度增长，通常不超过原参数两倍。

### 3. 实验设计

#### 数据集
- **Gowalla**：2009年2月–2010年10月，覆盖加州和内华达州。
- **NYC & TKY**：来自Foursquare，分别对应纽约和东京。
- 每个数据集都按三个场景维度（用户类型、时间、空间）划分为多个子场景进行评测。

#### Benchmark & 对比方法
- 5个最先进（SOTA）基线：
  - **HST-LSTM** (Kong & Wu, 2018)：层次化空间-时间LSTM。
  - **DeepMove** (Feng et al., 2018)：RNN + 注意力。
  - **GETNext** (Yang et al., 2022)：Transformer + 全局轨迹流图。
  - **STHGCN** (Yan et al., 2023)：超图卷积 + 时空Transformer。
  - **DCHL** (Lai et al., 2024)：多视图超图 + 对比学习。
- 评估指标：Acc@1,5,10,20 和 MRR（平均倒数排名）。

#### 消融实验
- 比较完整模型 vs. 两个变体：①无参数分裂；②无场景特定子超图。
- 还计算了效率对比（训练/测试时间、内存）。

### 4. 资源与算力

文中明确提及：
- **硬件**：Linux服务器，503 GB RAM，160核 Intel Xeon Gold 6230（2.10 GHz），**A40 GPU**（未说明具体数量，可能单卡或多卡）。
- **软件**：PyTorch 2.4.1。
- **训练配置**：
  - 嵌入维度128，超图卷积层数3，距离阈值2.5 km。
  - 对比损失权重 \(\lambda=0.1\)。
  - 参数分裂检查：NYC和Gowalla在20 epoch后开始，TKY在30 epoch后；每隔10个batch计算一次梯度冲突，相似度阈值-0.5时分裂。
  - 训练100 epoch，Adam优化器，学习率1e-3，权重衰减5e-4，batch size 200，early stopping patience=10。
- **效率数据**（NYC数据集）：MSAHG训练每epoch约10.69秒，内存1292 MB；测试12.47秒，内存225 MB。相比STHGCN训练慢（73.29秒）但更省显存。文中指出训练内存高源于梯度相似度缓存而非参数增加（测试时下降明显）。

**注意**：未说明GPU数量、具体单次训练总时长（仅给每epoch时长），也未说明多卡是否并行。

### 5. 实验数量与充分性

- **数量**：在 **3个真实数据集 × 5种基线 × 6个指标 × 6个常见子场景**（Local, Tourist, Downtown, Suburban, Workday, Weekend）共约 **90个情境-指标组合** 上进行对比。此外还有：
  - 消融实验（2个变体）对比。
  - 效率对比（2个基线和1个变体）。
  - 超参数敏感性分析（\(\lambda\)、层数、温度\(\tau\)、分裂epoch）。
  - 额外分析：预测POI类别分布差异、距离分布差异（定性图）。
- **充分性与客观性**：
  - 覆盖多种情境，且每个场景单独测评，避免全局指标掩盖场景差异。
  - 基线复现严格遵循原论文设置。
  - 消融实验证明两个核心组件均有效。
  - 超参数敏感度分析显示稳健性。
  - **公平性**：所有方法在同环境下运行，但基线未针对多场景优化；作者承认这是首个显式解决多场景问题的框架，因此基线的对比本身可能对MSAHG有利（因为基线未做场景分割）。不过实验仍展示了MSAHG在几乎所有场景下的优势，且消融验证了组件必要性。

### 6. 论文的主要结论与发现

- **性能优势**：MSAHG在53.3%（48/90）的评估场景-指标组合中取得第一，33.3%（30/90）取得第二，合计86.7%进入前两名。
- **场景捕获能力**：通过预测POI类别分布和距离分布的分析，MSAHG比DCHL更接近真实分布，说明其能有效保留不同场景的独有特征（如游客倾向酒店、郊区短距离移动）。
- **参数分裂的必要性**：消融实验显示移除参数分裂后性能下降显著（ACC@1从0.2268→0.2026），说明跨场景梯度冲突确实存在且MSAHG能有效解决。
- **子超图有效性**：移除场景特定子超图后性能下降（ACC@1→0.2164），证明多视图解耦建模的必要性。
- **效率**：MSAHG训练时间远低于STHGCN和GETNext，测试内存也较小，具备实际部署潜力。

### 7. 优点（亮点）

1. **问题定义新颖**：首次将“多场景”显式引入下一POI推荐，更贴近真实LBSN用户行为多样性。
2. **数据+参数双重解耦**：不仅从数据层面拆分子超图，还从优化层面动态分裂参数，避免了传统联合训练中的梯度冲突。
3. **可解释性强**：通过预测类别和距离分布分析，直观展示了模型对不同场景的差异性建模能力。
4. **模块化设计**：子超图构造与参数分裂可以独立消融，便于未来扩展。
5. **计算效率相对较高**：尽管增加了子超图和分裂机制，但训练速度优于多个SOTA基线（如STHGCN、GETNext），测试内存也低。
6. **开源代码**：提供GitHub仓库，可复现。

### 8. 不足与局限

1. **场景划分的局限性**：
   - 用户类型仅基于“住宿类POI比例”简单二分（阈值5%），可能遗漏更细粒度的用户特征（如频繁出差者）。
   - 空间区域仅按离市中心10 km硬分割，未考虑多中心大城市或地理边界。
   - 时间只分了工作日/周末，未考虑节假日或特殊事件。
2. **参数分裂的假设与开销**：
   - 梯度冲突检测依赖余弦相似度阈值-0.5，该值手动设置，可能需针对不同数据集调整。
   - 训练内存较高（因缓存梯度相似度），虽然测试时降低，但训练资源需求仍比无分裂变体大。
   - 只支持分裂一次（一旦分裂不再合并），可能无法适应训练中动态变化的冲突关系。
3. **实验局限性**：
   - 仅对比了5种基线，且全是面向传统单场景推荐的方法，未与已有的多场景推荐方法（如STAR、M2M、SAR-Net，这些来自推荐系统而非POI领域）进行对比（作者在相关工作部分介绍了它们，但实验中未纳入）。
   - 超参数敏感性分析只在附录中简要提及，未在主文展示详细图表。
   - 未报告方差（标准误差）或重复实验次数，难以评估结果稳定性。
4. **应用限制**：
   - 方法依赖位置、时间、用户类型等标注，在缺乏这些元信息的数据集上难以直接应用。
   - 未讨论冷启动用户/新POI情况。
   - 未分析在不同城市规模、文化背景下的泛化能力（仅用了北美/东亚城市）。

（完）
