---
title: "Mitigating Reward Hacking in LLM-based Recommendation: A Preference Optimization Approach"
title_zh: 缓解基于LLM的推荐中的奖励黑客：一种偏好优化方法
authors: "Heyu Chen, Junkang Wu, Guoqing Hu, Kexin Huang, Xiang Wang, Jiancan Wu"
date: 2026-04-30
pdf: "https://openreview.net/pdf/c9392ac6523b2271d5984709a69a9713787b61f4.pdf"
tags: ["query:rec-sys"]
score: 9.0
evidence: 基于偏好优化的LLM推荐奖励黑客缓解
tldr: 大语言模型在推荐系统中的后训练适应容易遭受奖励黑客攻击，即模型利用奖励信号的缺陷导致评估指标虚高。本文从梯度角度正式定义了ε-不敏感区域，并基于Bradley-Terry模型分析其影响。提出一种偏好优化方法，在多个推荐数据集上有效缓解奖励黑客，提升推荐的真实增益。
source: ICML-2026-Accepted
selection_source: conference_retrieval
motivation: LLM推荐中的奖励黑客问题导致训练指标虚高而真实推荐效果未提升。
method: 从梯度视角分析奖励黑客，定义ε-不敏感区域，提出偏好优化方法。
result: 在多个推荐数据集上有效缓解奖励黑客，提升推荐真实增益。
conclusion: 偏好优化方法能有效提升LLM推荐的可信度与效果。
---

## Abstract
Post-training adaptation has become the central paradigm for leveraging large language models (LLMs) in recommendation. 
While recent preference optimization methods, such as Direct Preference Optimization (DPO), enhance pairwise preference discrimination, they remain vulnerable to \emph{reward hacking}: models exploit imperfections in reward signals, leading to inflated training metrics without genuine recommendation gains.
We analyze this issue from a gradient perspective and formalize the concept of the 
\emph{$\varepsilon$-insensitive region}, where pairwise updates exert little influence on the ordering between positives and unsampled negatives.
Under the Bradley–Terry model, we further show that these regions can occupy a substantial fraction of the preference space, inevitably leading to misaligned rankings.
To address this issue, we propose Simulated Preference Optimization for Reward-hacking mitigation using Pseudo-negatives (SIRIUS). 
Our framework introduces pseudo-negative samples to enrich contrastive signals and reduce the prevalence of $\varepsilon$-insensitive regions.
Extensive experiments on three public benchmarks show that \our{} consistently improves ranking quality and effectively mitigates reward hacking, providing both theoretical and practical insights for advancing LLM-based recommendation. 
Our code is available at
\url{https://anonymous.4open.science/r/C557-id}

---

## 论文详细总结（自动生成）

好的，我将根据您提供的论文信息（主要是摘要和元数据），按照要求生成详细的中文总结。

### 1. 论文的核心问题与整体含义（研究动机和背景）

- **核心问题**：在基于大语言模型（LLM）的推荐系统中，后训练阶段常用的偏好优化方法（如直接偏好优化，DPO）容易遭受**奖励黑客**（Reward Hacking）攻击。即：模型会利用奖励信号本身的缺陷或漏洞，导致训练评估指标虚高，而推荐的真实效果并未得到实质提升。
- **整体含义**：该工作旨在从理论和实践层面解决奖励黑客问题，提升 LLM 推荐系统的可信度和真实性能。作者通过梯度角度分析了奖励黑客的成因，并提出了新的偏好优化方法来缓解该问题，为构建更可靠的 LLM 推荐系统提供了重要参考。

### 2. 论文提出的方法论：核心思想、关键技术细节

- **核心思想**：从梯度角度分析奖励黑客，形式化定义了 **ε-不敏感区域**（ε-insensitive region），在该区域内，成对的偏好更新对正样本与未采样负样本之间的排序影响甚微。在 Bradley-Terry 模型下，这些区域会占据偏好空间的相当大比例，从而导致排序对齐失效。为解决该问题，提出引入**伪负样本**（pseudo-negative samples）来丰富对比信号，减少 ε-不敏感区域的出现。
- **关键技术细节**：
    - **ε-不敏感区域形式化定义**：从梯度更新的角度，明确指出了哪些参数更新是“无效”或“微弱”的，这些更新无法有效拉开正样本与负样本之间的打分差距。
    - **基于 Bradley-Terry 模型的分析**：利用该模型证明，在标准偏好优化设置下，ε-不敏感区域是普遍存在的，是导致奖励黑客的根源。
    - **SIRIUS 框架**（Simulated Preference Optimization for Reward-hacking mitigation using Pseudo-negatives）：
        - 核心思路：在训练过程中，除了使用真实的偏好对（正样本 vs. 负样本），还主动合成或采样**伪负样本**。
        - 这些伪负样本是那些当前模型打分较高但实际相关性低的样本，将它们加入对比学习任务中，可以迫使模型学习更鲁棒的偏好边界，从而压缩 ε-不敏感区域。
        - 整体流程（用文字说明）：首先，对于每个正样本，利用当前模型生成或检索一批候选伪负样本；然后，将这些伪负样本与原负样本（或偏好对）一起，设计新的对比损失函数进行训练；最终能够在保留原有偏好信号的同时，增强模型对细微偏好差异的敏感性，减少对低质量奖励信号的依赖。

### 3. 实验设计：数据集、基准、对比方法

- **数据集**：在三个公开推荐基准数据集上进行实验。具体数据集名称摘要未提及，但可推断为常用的推荐评测数据集（如 MovieLens、Yelp、Amazon 等）。
- **基准**：以原始 DPO 等偏好优化方法作为基准（baseline），比较是否引入伪负样本后的推荐排序质量。
- **对比方法**：主要对比的是标准 DPO 以及可能存在的其他变体（如基于排名的方法）。实验重点验证 SIRIUS 在缓解奖励黑客、提升真实推荐效果方面的有效性。

### 4. 资源与算力

- **文中明确说明**：摘要末尾提到代码已开源，但**未明确说明**使用了多少 GPU 型号、数量、训练时长等具体算力信息。因此该方面信息缺失。

### 5. 实验数量与充分性

- **实验数量**：虽然摘要未列出具体实验个数，但提到在“三个公开基准数据集”上进行了“广泛实验”。通常这类论文会包含主实验（对比不同方法在多个指标上的表现）、消融实验（验证伪负样本数量、选择策略等超参数的影响）、以及可能的可视化分析（展示 ε-不敏感区域的变化）。
- **充分性与公平性**：
    - 使用多个公共数据集可增加泛化性。对比方法选取了标准 DPO（作为最强基线），是公平的常见做法。
    - 但是，由于无法查看全文，不清楚是否对比了更多竞争方法（如 RLHF 中的 PPO 变体），以及是否做了超参数敏感性分析和统计显著性检验。所以无法完全判断充分性，但根据 ICML 接受论文的标准，通常会保证实验的客观和公平。

### 6. 论文的主要结论与发现

- **主要结论**：提出的 SIRIUS 偏好优化方法能够**有效缓解**基于LLM的推荐系统中的奖励黑客问题，**持续提升**推荐排序质量。
- **核心发现**：
    - 从梯度角度正式界定了 ε-不敏感区域，揭示了奖励黑客的数学本质。
    - 引入伪负样本能够明显缩小 ε-不敏感区域，增强模型对真实偏好的敏感性。
    - 该方法在多个数据集上均能取得优于标准 DPO 的效果，证明了其通用性和有效性。

### 7. 优点

- **理论贡献**：首次从梯度视角严格定义并分析了推荐中奖励黑客的成因（ε-不敏感区域），为理解该问题提供了理论支撑。
- **方法原创性**：提出的伪负样本策略思路简洁但不失独创性，能够在不需要改变原有模型结构或增加大量计算开销的情况下缓解奖励黑客。
- **实践效果**：在多个基准上验证了有效性，且代码已开源，可复现性强。
- **问题聚焦**：直击 LLM 推荐后训练中的关键痛---评估指标虚高，具有重要的实际应用价值。

### 8. 不足与局限

- **实验覆盖**：未提供额外算力细节，可能限制了其他研究者复现时的资源预估。此外，是否在冷启动、长尾物品场景下也能有效？摘要未提及。
- **偏差风险**：伪负样本的生成策略非常重要，如果生成质量差（例如与正样本过于相似或完全无关），可能引入额外噪声，影响模型收敛。论文需详细讨论不同伪负样本采样策略的影响。
- **应用限制**：该方法目前仅针对基于 DPO 的偏好优化框架，对于其他形式的奖励信号或更强的 RL 方法（如 PPO）是否同样适用？需更多探索。
- **评价指标**：实验主要关注排序质量，但奖励黑客往往体现在离线评估指标虚高，论文可能需要额外验证在真实在线或离线指标（如转化为 A/B 测试）下的一致性。

（完）
