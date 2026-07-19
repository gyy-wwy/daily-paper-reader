---
title: "RecCocktail: A Generalizable and Efficient Framework for LLM-Based Recommendation"
title_zh: RecCocktail：一种可泛化且高效的LLM推荐框架
authors: "Min Hou, Chenxi Bai, Le Wu, Hao Liu, Kai Zhang, Weiwen Liu, Richang Hong, Ruiming Tang, Meng Wang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38504/42466"
tags: ["query:rec-sys"]
score: 9.0
evidence: 可泛化的LLM推荐框架
tldr: 现有LLM推荐方法将通用推荐和领域特定推荐分开处理。本文提出RecCocktail，结合多领域指令数据和领域增强，实现两方面的互补优势，提升推荐泛化性和领域性能。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有LLM推荐方法未能有效结合通用推荐和领域特定推荐的优势。
method: 提出RecCocktail框架，集成多领域指令数据微调和领域特定增强模块。
result: 在多种推荐场景下表现优异，尤其冷启动场景。
conclusion: 结合通用和领域特定知识是LLM推荐的有效路径。
---

## Abstract
Large Language Models (LLMs) have achieved remarkable success in recent years, owing to their impressive generalization capabilities and rich world knowledge. To capitalize on the potential of using LLMs as recommender systems, mainstream approaches typically focus on two paradigms. The first paradigm designs multi-domain or multi-task instruction data for generalizable recommendation, so as to align LLMs with general recommendation areas and deal with cold-start recommendation. The second paradigm focuses on enhancing domain-specific recommendation tasks, improving performance in warm recommendation scenarios. While most previous works treat these two paradigms separately, we argue that they have complementary advantages, and combining them can yield better results. In this paper, we propose a generalizable and efficient LLM-based recommendation framework RecCocktail. Our approach begins with fine-tuning a "base spirit" LoRA module using domain-general recommendation instruction data to align LLM with recommendation knowledge. Next, given users' behavior of a specific domain, we construct a domain-specific "ingredient" LoRA module. We then provide an entropy-guided adaptive merging method to mix the "base spirit" and the "ingredient" in the weight space. Please note that, RecCocktail combines the advantages of the existing two paradigms without introducing additional time or space overhead during the inference phase. Moreover, RecCocktail is efficient with plug and play, as the "base spirit" LoRA is trained only once, and any domain-specific "ingredient" can be efficiently mixed with only domain-specific fine-tuning. Extensive experiments on multiple datasets under both warm and cold-start recommendation scenarios validate the effectiveness and generality of the proposed RecCocktail.

---

## 论文详细总结（自动生成）

# RecCocktail 论文详细总结

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：现有基于大语言模型（LLM）的推荐方法主要分为两大范式：  
  - **广度导向**（Breadth-oriented）：通过多领域/多任务指令数据训练通用推荐知识，提升泛化能力和冷启动推荐性能。  
  - **深度导向**（Depth-oriented）：专注于特定领域的深度对齐，利用 LoRA 等参数高效微调提升领域内热启动场景性能。  
- **问题**：这两种范式各有优劣，但现有工作往往孤立对待，未能有效融合两者的互补优势。  
- **整体含义**：本文提出 **RecCocktail** 框架，将通用推荐知识与领域特定知识在参数空间内以“鸡尾酒”方式融合，实现既保持泛化性又增强领域性能的 LLM 推荐系统。目标是同时提升冷启动和热启动场景下的推荐质量，且不引入推理阶段额外开销。

## 2. 方法论：核心思想、关键技术细节

### 核心思想
- 受鸡尾酒调制启发，将通用推荐 LoRA 模块视为“基酒”（base spirit），领域特定 LoRA 视为“配料”（ingredient），通过权重空间线性合并实现知识融合，并利用测试时熵最小化自适应确定合并系数。

### 关键技术细节
1. **构建通用基酒**  
   - 收集多个领域的用户行为数据，构造指令数据集 \(D_g\)。  
   - 对预训练 LLM 进行 LoRA 微调，得到领域通用 LoRA 参数 \(\Delta\Theta_g\)。

2. **构建领域特定配料**  
   - 针对目标领域 s，构造指令数据集 \(D_s\)。  
   - 同样以 LoRA 微调得到领域特定参数 \(\Delta\Theta_s\)。

3. **LoRA 鸡尾酒合并**  
   - 线性合并两个 LoRA 模块（权重空间加法）：  
     \[
     \Delta\Theta_m = (\lambda_1 \Delta\Theta_g) \oplus (\lambda_2 \Delta\Theta_s)
     \]
     其中 \(\lambda_1 + \lambda_2 = 1\)，每个 LoRA 的低秩矩阵分别对应相加。  
   - 合并后模型参数为 \(\Theta_{\text{cocktail}} = \Theta_{\text{pre}} + \Delta\Theta_m\)，推理时仅需一个 LoRA 模块，无额外计算开销。

4. **熵引导自适应合并**  
   - 为了避免手动选择 \(\lambda_1, \lambda_2\) 的繁琐，提出在测试时利用少量未标记样本（如 50 个）最小化模型输出的香农熵：  
     \[
     \min_{\lambda_1,\lambda_2} \sum_{x_i \in D_t} H(F_{\Theta_{\text{cocktail}}}(x_i))
     \]
   - 仅需计算输出序列前几个 token 的平均熵，可快速更新合并系数，适应不同分布。

5. **效率优势**  
   - 基酒 LoRA 只训练一次，领域配料仅需轻量微调；合并后模型参数数不增加，推理时间与标准 LoRA 相同。

## 3. 实验设计

- **数据集**  
  - **电子商务场景**：Amazon 多领域数据（Clothing, Cell, Grocery, Health, Home, Pet, Tools, Videos 作为通用训练集；Beauty, Toys, Sports 作为目标域）。  
  - **电影推荐场景**：通用基酒使用 MovieLens-10M，目标域为 MovieLens-1M。  
  - 所有物品用文本标题表示，并确保通用集与目标集无重叠。

- **场景设置**  
  - **热身启动（Warm-Start）**：保留五核数据，留一法划分（最后交互为测试，倒数第二为验证）。  
  - **冷启动（Cold-Start）**：训练/验证集与热身相同，但测试集中的物品完全不在训练/验证集中出现（新物品）。

- **对比方法**  
  - **传统方法**：BPR-MF, GRU4Rec, SASRec, FMLP-Rec。  
  - **可迁移序列推荐**：UniSRec, VQ-Rec, One Model for All（跨领域）。  
  - **LLM 推荐**：Qwen2-7B-zero-shot, RecFormer, P5, TALLRec, AlphaRec。  
  - **消融变体**：RecCocktail-WA（简单权重平均）, RecCocktail-G（仅基酒）, RecCocktail-S（仅配料）。

- **评价指标**：NDCG@1 和 NDCG@3（每个用户候选集包含1正+29负）。

## 4. 资源与算力

- **论文明确说明**：使用 NVIDIA RTX 4090 GPU（具体数量未列出，文中提到“on NVIDIA RTX 4090 GPUs”，实验中 LoRA 全流程训练使用多卡，但未提及具体卡数和训练时长）。  
- **算力细节缺失**：未提供模型训练的计算时间、总 GPU 小时数、内存占用等，仅在实现细节中提及采用梯度检查点（gradient checkpointing）和 VLLM 推理加速，以及部分实验（如 LoRA 融合）使用双卡（batch size 60）。  
- **结论**：资源报告不够详细，仅知硬件类型，缺乏量化数据。

## 5. 实验数量与充分性

- **实验组数较多**：  
  - 4 个目标域数据集（Beauty, Toys, Sports, MovieLens-1M），均报告热身和冷启动两个场景下 NDCG@1 和 NDCG@3。  
  - 包含完整的消融实验（三种变体对比）。  
  - 进行了系数 λ 分析、未标记样本数量影响分析、少量训练数据场景（few-shot）实验。  
  - 使用不同的 LLM 骨干网络（Llama-3.1-8B）进行鲁棒性验证。  
  - 提供了可视化案例研究。

- **充分性评估**：  
  - **优点**：覆盖多领域、多场景、多基线，消融充分，自适应合并机制验证到位。  
  - **不足**：  
    - 未在更多样化的任务（如排序、评分预测）上验证。  
    - 未与更多近期 LLM 推荐方法（如某些基于全量微调的方法）比较。  
    - 实验仅基于两个基座模型（Qwen2-7B 和 Llama-3.1-8B），未探索更大模型（如 13B/70B）。  
    - 冷启动场景仅考虑新物品，未涉及新用户或跨域冷启动。  
  - **客观公平性**：Baseline 选择广泛，包括传统、可迁移、LLM 类，且实验设置统一（如候选集构建方式相同）。

## 6. 主要结论与发现

- **RecCocktail 在热身和冷启动场景下均取得最好性能**，在四个数据集上全面超越所有基线方法（p < 0.05）。  
- **简单合并（RecCocktail-WA）有时效果差**，印证了自适应合并的必要性。  
- **冷启动时 λ1（基酒权重）更大，热启动时 λ2（配料权重）更大**，表明熵引导自适应合并能自动平衡通用与领域知识。  
- **少量未标记测试样本（50 个）即可快速确定最优合并系数**。  
- **在 Few-shot 训练设置下，RecCocktail 远优于二次微调（避免灾难性遗忘）**。  
- **任务泛化性**：案例中 RecCocktail 可正确生成解释性推荐，而 GPT-4 错误关联。

## 7. 优点

- **方法创新**：首次将 LoRA 任务向量（task vector）思想引入 LLM 推荐，通过简单线性合并实现知识和能力融合。  
- **效率极高**：基酒一次训练，配料轻量微调，合并后零额外推理开销。  
- **泛化性强**：基酒本身就能处理未见域零样本推荐，合并后进一步提升。  
- **自适应合并设计巧妙**：熵最小化引导系数选择，无需标注数据，仅需少量未标记样本。  
- **实验设计全面**：涵盖热身、冷启动、few-shot 等实际场景，消融和多骨干验证充分。  
- **代码开源**（https://github.com/1149722739/RecCocktail），重复性好。

## 8. 不足与局限

- **实验覆盖不够广泛**：未在社交推荐、新闻推荐、跨域用户迁移等场景验证；冷启动类型单一（仅新物品）。  
- **基座模型规模有限**：仅测试 7B/8B 模型，未验证更大参数模型下的扩展性和效果变化。  
- **计算资源报告不充分**：未给出训练时间和具体能耗，影响可重复性评估。  
- **依赖 LoRA 秩的选择**：文中使用 rank=16，但未探讨不同 rank 对结果的影响。  
- **自适应合并需要少量测试样本**：在极端缺乏测试数据（如仅 1 条）时可能不稳定，未讨论。  
- **与更多最新 LLM 推荐方法对比缺失**：如基于全量参数微调的 RLHF 方法，或利用协同信号的深度对齐方法（尽管文中提到 iLoRA 等，但实验未包含）。  
- **应用限制**：仅基于指令格式的候选集排序，未拓展到其他推荐任务（如评分预测、序列推荐）。  
- **偏差风险**：未讨论模型可能存在的流行度偏差、公平性等问题。

（完）
