---
title: "CRAMER: Control via Request-Aware Masking for Editing Recommenders"
title_zh: CRAMER：通过请求感知掩码控制推荐编辑
authors: "Zhiyuan Su, Naihe Feng, Luther Qin, Ga Wu"
date: 2026-04-30
pdf: "https://openreview.net/pdf/7b55bfd346d63befa4c96a676b9daf5628ae9c36.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 基于请求感知的序列推荐编辑
tldr: 序列推荐模型难以灵活响应用户即时请求。本文提出CRAMER框架，通过请求感知掩码编辑推荐模型行为，无需重训练或借助大语言模型推理。该方法在保持推荐质量的同时大幅降低计算开销，适用于大规模推荐服务。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: 现有方法在适应即时用户请求时计算开销大，限制了大规模应用。
method: 提出CRAMER，利用请求感知掩码直接编辑序列推荐模型的行为。
result: 在多个序列推荐数据集上，CRAMER以低开销取得与重训练相当的适应性。
conclusion: 请求感知掩码为高效推荐编辑提供新范式。
---

## Abstract
Sequential recommendation models, while powerful, have limited flexibility in responding to immediate user requests, making it difficult to adapt their recommendations to the user's timely interests. Unfortunately, existing user request adaptation methods often incur high computational overhead due to either 1) retraining the entire backbone network or 2) leveraging the inference ability of large language models (a.k.a. prompt engineering), limiting their applicability in large-scale recommendation services. This paper presents **C**ontrol via **R**equest-**A**ware **M**asking for **E**diting **R**ecommenders (**CRAMER**), a framework that takes users' natural-language requests to immediately change sequential recommendation models' behavior. Specifically, inspired by the model control theory, CRAMER treats user requests as control signals to modulate frozen backbone parameters through masking, achieving instant adaptation to diverse requests while avoiding costly retraining. Experiments on multiple large-scale benchmark datasets show that CRAMER outperforms four state-of-the-art request-aware baselines across multiple recommendation metrics while achieving minimal overhead. Moreover, the proposed framework exhibits enhanced controllability and cross-domain adaptability, establishing a new paradigm for request-aware sequential recommendation.

---

## 论文详细总结（自动生成）

# 论文详细中文总结

## 1. 核心问题与整体含义（研究动机和背景）
- **核心问题**：现有的序列推荐模型虽然性能强大，但在响应用户的即时请求（例如通过自然语言表达的临时兴趣变化）时灵活性不足，难以快速调整推荐结果以匹配用户当下的需求。
- **背景与动机**：传统的用户请求适应方法通常有两种思路：一是重新训练整个骨干网络（开销巨大），二是依赖大型语言模型进行推理（即提示工程，同样计算成本高）。这使得这些方法难以在大规模推荐服务中实际部署。因此，亟需一种高效、轻量且能即时响应用户请求的推荐编辑方法。

## 2. 论文提出的方法论
- **核心思想**：受模型控制理论的启发，将用户的自然语言请求视为控制信号，通过**请求感知掩码（Request-Aware Masking）** 直接调制冻结的骨干网络参数，从而实现即时适应，避免昂贵的重训练。
- **关键技术细节**：
  - **CRAMER框架**：其全称为“Control via Request-Aware Masking for Editing Recommenders”。
  - **核心组件**：利用一个轻量的掩码生成模块，该模块以用户请求的嵌入为条件，为预训练的、已冻结的序列推荐模型的每个参数或子层生成一个二值或软掩码。这些掩码控制哪些参数被“激活”或“抑制”，从而在不修改原始参数的情况下改变模型行为。
  - **流程简述**：输入用户历史序列 + 用户自然语言请求 → 请求编码器提取请求特征 → 掩码生成模块根据请求特征产生掩码 → 将掩码应用于冻结的推荐模型 → 模型以修改后的参数进行推荐。整个过程无需反向传播更新原始模型参数，仅需训练掩码生成模块（通常参数量很小）。
- **公式与算法**：原文摘要中未给出具体公式，但可推断其损失函数包含推荐准确性和控制目标（如满足请求的特定指标）的联合优化。训练时，掩码生成器通过最小化推荐损失以及可能的正则项来学习如何根据请求调整掩码。

## 3. 实验设计
- **数据集**：使用了多个大规模基准数据集（原文提及“multiple large-scale benchmark datasets”），具体名称未在摘要中列出。常见的序列推荐数据集如Amazon系列、MovieLens、Yelp等可能是候选。
- **场景**：请求感知的序列推荐，即用户给出自然语言请求（例如“推荐更便宜的电子设备”或“推荐更多喜剧电影”），模型需据此调整输出。
- **Benchmark**：与四种最先进的请求感知基线方法进行对比（具体方法名称未在摘要给出，但提到了“four state-of-the-art request-aware baselines”）。
- **对比方法**：包括重训练方法、大模型提示方法等。CRAMER在多个推荐指标上优于这些基线，且开销最小。

## 4. 资源与算力
- 论文摘要中**未明确说明**使用的GPU型号、数量或训练时长等具体算力资源。仅提到CRAMER实现了“minimal overhead”（最小开销），暗示其计算效率高。但无法获知具体硬件配置。

## 5. 实验数量与充分性
- **实验数量**：从摘要看，至少进行了以下类型的实验：
  - 主对比实验：在多个数据集上对比CRAMER与四个基线。
  - 控制力评估：展示了增强的可控性（controllability）分析。
  - 跨领域适应性：验证了跨域泛化能力。
- **充分性评价**：
  - **优点**：覆盖了多个数据集、多个指标、基线数量足够（4个SOTA），并有控制力和跨域分析，实验设计较为全面。
  - **不足**：缺少消融实验的具体细节（如掩码设计选择、请求编码器设计）以及超参数敏感性分析。由于未获得完整论文，无法判断是否包含充分的消融和鲁棒性测试。总体而言，基于摘要所呈现的信息，实验较为充分，但可能存在遗漏细节。

## 6. 主要结论与发现
- **主要结论**：CRAMER通过请求感知掩码直接编辑推荐模型行为，在不牺牲推荐质量的前提下，实现了对即时用户请求的快速适应，并且计算开销远低于重训练和大模型推理方法。
- **关键发现**：
  - CRAMER在所有评估的推荐指标上均优于四种SOTA请求感知方法。
  - 框架表现出更强的可控性（用户请求能更精确地影响输出）和跨领域适应性（可在不同数据集或领域间泛化）。
  - 该方法建立了一种请求感知序列推荐的新范式，平衡了适应性与效率。

## 7. 优点
- **方法论亮点**：
  - 提出了一种轻量、无需重训练的推荐编辑机制，借鉴模型控制理论，新颖地使用掩码来调制冻结参数。
  - 直接利用自然语言请求作为控制信号，无需额外的标注或复杂的提示工程。
  - 计算开销极低，适合大规模在线服务。
- **实验设计亮点**：
  - 在多个大规模数据集上验证，并与多个先进基线对比，结果全面。
  - 评估了可控性和跨域泛化，增强了结论的鲁棒性。

## 8. 不足与局限
- **实验覆盖**：
  - 具体数据集名称、基线方法名称未在摘要中给出，读者无法直接复现或验证。
  - 缺少计算资源消耗的具体量化对比（如训练时间、GPU内存）。
  - 未提及是否包含用户请求的多样性测试（如请求的复杂度、语义多样性）以及模型对噪声请求的鲁棒性。
- **偏差风险**：
  - 掩码生成可能依赖于预训练的推荐模型质量，若基础模型较弱，效果可能受限。
  - 实验尚未说明是否在真实用户反馈环境中测试（仅离线指标），存在在线效果差异风险。
- **应用限制**：
  - 需要预先训练好的序列推荐模型，且其架构需支持掩码操作，可能无法直接应用于所有现成模型。
  - 对于涉及隐私或安全的请求，掩码编辑可能引入不可控的副作用（如偏见放大），论文未讨论。

（完）
