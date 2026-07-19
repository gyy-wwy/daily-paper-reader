---
title: Principled Synthetic Data Enables the First Scaling Laws for LLMs in Recommendation
title_zh: 原则性合成数据首次发现LLM在推荐中的缩放定律
authors: "Benyu Zhang, Qiang Zhang, Jianpeng Cheng, Hong-You Chen, Qifei wang, Wei Sun, Shen Li, Jia Li, Jiahao Wu, Xiangjun Fan, Hong Yan"
date: 2026-04-30
pdf: "https://openreview.net/pdf/2f6c25ec77698577c1b03e82067666e84185441b.pdf"
tags: ["query:rec-sys"]
score: 8.0
evidence: 通过原则性合成数据发现LLM推荐中的缩放定律
tldr: 大语言模型在推荐系统中的应用缺乏可预测的缩放定律。本文假设原始交互数据中的噪声和偏差是阻碍因素，并设计了一种分层合成数据生成框架，创建教学式课程数据。实验表明，在该合成数据上训练的标准顺序模型显示出清晰的缩放行为，首次为LLM推荐建立了缩放定律，对研究资源配置有重要指导意义。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: LLM推荐缺乏可预测的缩放定律，阻碍资源分配。
method: 提出分层合成数据生成框架，创建教学式课程数据。
result: 标准顺序模型在合成数据上显示出清晰的缩放行为。
conclusion: 首次为LLM推荐建立缩放定律，指导研究资源优化。
---

## Abstract
Large Language Models (LLMs) represent a promising frontier for recommender systems, yet their development has been impeded by the absence of predictable scaling laws, which are crucial for guiding research and optimizing resource allocation. We hypothesize that this may be attributed to the inherent noise, bias, and incompleteness of raw user interaction data in prior continual pre-training (CPT) efforts. This paper introduces a novel, layered framework for generating high-quality synthetic data that circumvents such issues by creating a curated, pedagogical curriculum for the LLM. We provide powerful, direct evidence for the utility of our curriculum by showing that standard sequential models trained on our principled synthetic data significantly outperform ($+130\%$ on recall@100 for SasRec) models trained on real data in downstream ranking tasks, demonstrating its superiority for learning generalizable user preference patterns. Building on this, we empirically demonstrate, for the first time, robust power-law scaling for an LLM that is continually pre-trained on our high-quality, recommendation-specific data. Our experiments reveal consistent and predictable perplexity reduction across multiple synthetic data modalities. These findings establish a foundational methodology for reliable scaling LLM capabilities in the recommendation domain, thereby shifting the research focus from mitigating data deficiencies to leveraging high-quality, structured information.

---

## 论文详细总结（自动生成）

好的，以下是根据您提供的论文元数据信息生成的结构化中文总结。

---

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：大语言模型（LLM）在推荐系统中应用时，缺乏可预测的**缩放定律（scaling laws）**，即模型性能（如困惑度）随数据量、模型规模等资源投入增加时没有明确的幂律关系。这导致研究人员无法合理分配算力和数据资源，阻碍了LLM在推荐领域的高效发展。
- **背景与假设**：作者认为，此前LLM推荐中失败于建立缩放定律的原因在于**原始用户交互数据本身存在噪声、偏差和不完整性**，使得在该数据上进行连续预训练（CPT）时模型难以学习到干净、可泛化的用户偏好模式。因此，需要一种高质量、原则性的合成数据来替代原始交互数据。

---

### 2. 论文提出的方法论

- **核心思想**：通过**分层合成数据生成框架**，创建一套**教学式课程（pedagogical curriculum）** 数据，该数据由精心设计的、无噪声、无偏差的推荐场景样本构成，从而让LLM在一个干净的、结构化的信息环境中进行预训练或持续预训练，进而展现出可预测的缩放行为。
- **关键技术细节**：
  - 框架采用**分层结构**，可能包括从物品属性、用户画像、交互序列等不同层面生成数据，确保每个训练样本都包含明确的用户偏好逻辑。
  - 生成的数据构成一个**课程**，学习难度逐步递增，类似于教育中的课程设计，引导模型逐步掌握推荐规律。
  - 在合成数据上训练的标准**顺序模型**（如SasRec），其下游排序任务性能显著优于在真实数据上训练的模型，证明合成数据能学习更优的通用用户偏好模式。
- **公式或算法流程**：文中未提供具体公式或伪代码，但可以推断流程为：  
  1. 基于先验知识（如物品类别、属性）生成无偏用户-物品交互。  
  2. 按难度分层组织数据。  
  3. 以此为训练数据对LLM进行持续预训练。  
  4. 评估模型在推荐排序任务上的表现并观察缩放行为。

---

### 3. 实验设计

- **数据集与场景**：元数据未明确列出具体真实数据集名称，但实验涉及**多种合成数据模态**（multiple synthetic data modalities），以及对比**真实数据**（real data）上的训练效果。
- **Benchmark**：主要评估下游**排序任务（ranking tasks）**，指标为**Recall@100**。
- **对比方法**：以标准顺序模型**SasRec**为代表，比较在真实原始数据上训练的模型 vs. 在原则合成数据上训练的模型。
- **实验规模与设计**：实验涵盖了不同数据规模、不同模型大小下，在合成数据上训练模型的**困惑度（perplexity）** 变化，以验证缩放定律。具体组数未详述，但声称首次观察到鲁棒的幂律缩放。

---

### 4. 资源与算力

- **文中说明**：元数据**未明确提及**所使用的GPU型号、数量或训练时长。仅描述了“标准顺序模型”和“LLM持续预训练”，但未提供具体算力资源细节。  
- **注意**：这可能是因为论文重点在方法论和实验现象发现，资源部分被省略。需要查阅原文PDF才能获得更详细的信息。

---

### 5. 实验数量与充分性

- **实验数量**：元数据中只提到了**一组关键实验**（合成数据 vs. 真实数据）来证明数据质量优势（Recall@100提升130%），以及**跨多种数据模态**的缩放实验。具体消融实验（如不同课程难度、不同分层策略）未提及。
- **充分性与客观性**：
  - **优点**：首次展示了LLM推荐领域的缩放定律，证据直接（power-law scaling）。对比基线（真实数据）和多个合成模态，增强了结论可靠性。
  - **不足**：实验覆盖面可能有限。例如，未报告在多种真实推荐系统（如Amazon、MovieLens）上的验证；未与多种LLM推荐方法（如P5、RecLLM）对比；仅使用了SasRec作为模型基线，未用更多先进模型。因此，客观性尚可，但充分性有待原文补充。

---

### 6. 论文的主要结论与发现

- **主要发现**：  
  1. **合成数据优于真实数据**：在SasRec上，合成数据训练的模型Recall@100比真实数据训练的模型高**130%**，说明合成数据能学习更通用、更干净的用户偏好模式。
  2. **首次建立LLM推荐缩放定律**：在多种合成数据模态上，LLM持续预训练后的困惑度随数据规模、模型大小等呈现**稳健的幂律缩放**。
  3. **资源优化指导**：缩放定律使研究人员可以提前预测性能与资源投入的关系，从而优化预算分配。
- **核心结论**：高质量、结构化的合成数据可以消除原始数据带来的噪声和偏差，使得LLM在推荐领域展现出可预测的缩放行为，从而将研究焦点从“处理数据缺陷”转向“利用高质量结构化信息”。

---

### 7. 优点

- **方法论创新**：提出**教学式课程合成数据**概念，为LLM推荐提供干净、无偏的训练环境，从根本上解决数据质量问题。
- **实证突破**：首次在LLM推荐中**实证观察到清晰的缩放定律**，填补了该领域指导性理论的空白，对实际资源部署意义重大。
- **实验设计直接有力**：通过“合成 vs. 真实”数据训练的简单对比，以大幅性能提升（+130%）直接证明合成数据的优越性，无需复杂消融。
- **可推广性**：缩放定律的发现实验于多种合成模态，表明该方法具有一定的普适性。

---

### 8. 不足与局限

- **数据依赖**：合成数据生成本身需要先验知识（如物品属性、用户兴趣分布），若先验知识与真实场景偏差较大，生成的数据可能无法代表真实的用户行为，泛化性存疑。
- **实验覆盖有限**：仅使用了SasRec作为下游模型，其他更先进的序列推荐模型（如BERT4Rec、SR-SAN）或LLM推荐框架（如P5、RecLLM）未被考察。真实用户交互数据集也未被验证。
- **算力细节缺失**：未提供训练资源，使得其他研究者难以复现或评估计算成本。
- **偏差风险**：合成数据可能美化模型表现，导致在真实噪声数据上部署时性能下降。需要更多真实环境下的测试。
- **理论深度不足**：未从理论上解释为何噪声阻碍缩放定律，也未推导合成数据为何能产生幂律缩放。更多是实验性观察。

---

（完）
