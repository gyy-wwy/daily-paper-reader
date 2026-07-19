---
title: "DiMA: Distinguishing Resident and Tourist Preferences via Multi-Modal LLM Alignment for Out-of-Town Cross-Domain Recommendation"
title_zh: DiMA：通过多模态LLM对齐区分居民和游客偏好的外地跨域推荐
authors: "Fan Zhang, Jinpeng Chen, Tao Wang, Huan Li, Senzhang Wang, Feifei Kou, Ye Ji, Kaimin Wei, Zhenye Yang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/38663/42625"
tags: ["query:rec-sys"]
score: 9.0
evidence: 通过多模态LLM对齐区分游客偏好，实现外地跨域推荐
tldr: 外地推荐面临跨模态偏好推理和居民/游客偏好偏差问题。本文提出DiMA框架，通过多模态LLM对齐统一图像和文本偏好信号，并设计游客偏好提取模块。在OOT推荐数据集上，DiMA显著提高了POI重排质量，尤其对游客群体效果突出。该工作推进了旅游场景下的个性化推荐。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 外地推荐中用户居民和游客偏好差异大，且多模态信号难以统一推理。
method: 利用多模态LLM对齐图像和文本偏好，并设计偏好偏差纠正模块。
result: 在真实外地推荐数据上，DiMA的NDCG和召回率均大幅超过基线方法。
conclusion: 区分居民和游客偏好并跨模态对齐是提升旅游推荐效果的关键。
---

## Abstract
Out-of-Town (OOT) recommendation aims to provide personalized suggestions for users in unfamiliar cities. However, OOT recommendation faces two fundamental challenges: the difficulty of reasoning across modalities, as preference signals in disparate formats such as images and text are hard to compare; and the preference deviation problem, since a user's resident and tourist preferences often diverge, rendering simple preference transfer ineffective. To address these challenges, we propose Distinguishing Resident and Tourist Preferences via Multi-Modal LLM Alignment for Out-of-Town Cross-Domain Recommendation (DiMA), a framework for re-ranking Points of Interest (POIs). To tackle the multimodal challenge, DiMA first leverages Multimodal Large Language Models and Large Language Models (LLMs) to transform heterogeneous POI data into unified semantic tags, enabling both cross-modal reasoning and efficient downstream processing. To address preference deviation, a ``teacher'' LLM executes a custom Chain-of-Thought (CoT) process to disentangle resident and tourist preferences from multi-city histories for re-ranking. Finally, a lightweight student model learns this CoT reasoning via Supervised Fine-Tuning and is then refined with Direct Preference Optimization to align with true user choices, with the potential to surpass the teacher. Extensive experiments on a real-world dataset demonstrate that DiMA significantly enhances the performance of baseline models in the OOT recommendation re-ranking task.

---

## 论文详细总结（自动生成）

## 论文详细中文总结

### 1. 论文的核心问题与整体含义
- **研究动机与背景**：在外地推荐（Out-of-Town, OOT）场景中，用户在新城市（目标域）面临数据稀疏和冷启动问题。传统跨域推荐通过转移知识缓解该问题，但OOT推荐存在两个根本性挑战：
  - **跨模态推理困难**：POI的吸引力由图像（反映事实）和评论（包含情感）等不同模态共同决定，但格式各异，难以直接比较和推理。
  - **偏好偏差问题**：用户的日常“居民模式”偏好与旅行时的“游客模式”偏好常存在显著差异，简单依赖居民城市历史进行偏好迁移无效。
- **整体含义**：本文提出DiMA框架，通过多模态LLM对齐将异构数据统一为结构化标签，并利用教师LLM的链式推理（CoT）从多城市历史中分离居民与游客偏好，最终通过轻量学生模型高效重排POI，提升OOT推荐准确性。

### 2. 论文提出的方法论
- **核心思想**：建立一个三阶段流水线：（1）多模态对齐模块将POI图像和评论转化为统一语义标签；（2）提示构建模块整合用户历史、候选POI及协同过滤得分，形成结构化CoT提示；（3）师生对齐模块，教师LLM生成推理轨迹和排序结果，学生模型通过SFT+DPO两阶段学习并超越教师。
- **关键技术细节**：
  - **多模态对齐**：使用MLLM（InternVL-2.5-8B-MPO）将图像转为文本描述；再使用LLM（DeepSeek-V3）提取实体标签（客观）和情感标签（主观）；最后词形还原（实体）和词干提取（情感）归一化，得到统一标签集 \( T_p \)。
  - **CoT提示模板**（算法1）：包含三步推理——① 轮廓综合：分析居民和辅助旅行历史，形成“游客模式”画像；② 候选评估：结合标签和CF得分评估每个候选POI；③ 重新排序：生成最终排序列表。
  - **协同过滤（CF）得分**：计算目标用户与其他用户的Jaccard相似度（类别和场馆两级），对候选POI的访问者相似度求和得到 \( s_{cat} \) 和 \( s_{ven} \)。
  - **学生模型训练**：
    - SFT阶段：最小化交叉熵损失，让学生模仿教师完整的输出（包括推理文本和排序列表）。
    - DPO阶段：构建偏好三元组 \((x, y_w, y_l)\)，其中 \( y_l \) 为教师排序，\( y_w \) 为根据真实交互构造的理想排序（命中POI排名高于未命中，保留教师内部顺序）。优化DPO损失函数（公式3），温度参数 \(\beta\) 控制偏差强度。
- **公式**：
  - SFT损失：\( L_{SFT}(\theta) = -\sum_{i=1}^{|y_{teacher}|} \log P(y_i | y_{<i}, x; \theta) \)
  - DPO损失：\( L_{DPO} = -\mathbb{E}_{(x,y_w,y_l)\sim D} \left[ \log \sigma\left( \beta \log \frac{\pi_\theta(y_w|x)}{\pi_{\theta_{ref}}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{\theta_{ref}}(y_l|x)} \right) \right] \)

### 3. 实验设计
- **数据集与场景**：
  - **Foursquare数据集**：从Foursquare API收集，三个美国城市（纽约NY、洛杉矶LA、旧金山SF）。保留有三城交互记录的用户和至少被访问5次的POI，最终1072用户，8478 POI（NY 3831, LA 2516, SF 2131）。生成6种实验设置（循环分配居民/辅助/目标城市角色）。
  - **Yelp数据集**：从公开Yelp数据集选取新奥尔良NO、费城PH、圣路易斯SL，要求POI至少有5张图片和5条评论，保留在三城出现的用户，最终593用户，2158 POI（NO 681, PH 1015, SL 462）。同样6种设置。
- **基准方法（Baselines）**：
  - 候选列表由两个SOTA OOT基线生成：**KDDC**（知识驱动因果度量学习）和**CAPTOR**（人群感知预游推荐）。
  - 比较的重排序器：**Teacher-2C**（教师模型不使用辅助历史）、**DiMA-2C**（对应学生模型）、**Teacher-3C**（完整教师模型使用三城历史）、**DiMA（完整）**。
- **评估指标**：NDCG@5、MAP@5，5折交叉验证报告平均值。

### 4. 资源与算力
- **实现框架**：PyTorch、LLaMA-Factory。
- **模型**：
  - 图像转文本：InternVL-2.5-8B-MPO。
  - 标签提取：DeepSeek-V3。
  - 教师模型：DeepSeek-R1。
  - 学生模型：Qwen3-4B（主要），还测试了Qwen2.5-1.5B/3B、Qwen3-1.7B。
- **GPU资源**：
  - SFT阶段：2× NVIDIA RTX 3090。
  - DPO阶段：1× NVIDIA H20。
- **训练时长**：未明确给出。推理耗时：教师模型每请求约5分钟；学生模型Qwen3-4B为10秒（RTX 3090），Qwen2.5-1.5B为1秒。
- **备注**：仅提及学生模型训练硬件配置，未说明预处理的MLLM/LLM调用成本。

### 5. 实验数量与充分性
- **实验组数**：
  - 两个数据集（Foursquare、Yelp）各6种OOT设置，共12个主要任务。
  - 在Foursquare上使用KDDC和CAPTOR两种候选生成，Yelp上同样使用两种，因此整体对比表格有4个大表（每个表针对一个数据集×候选生成器），包含多种模型。
  - 消融研究：在Foursquare-KDDC设置上平均6个任务，对比4个变体（w/o Tags, w/o CF, w/o CoT, w/o DPO）。
  - 学生模型规模分析：在Foursquare-KDDC设置上平均6个任务，测试4种学生模型。
  - 案例研究：展示跨模态推理实例。
- **充分性与客观性**：
  - 实验设计较为全面：覆盖两个不同来源的数据集、两种候选基线、多种消融、模型规模对比，并使用5折交叉验证，结果统计可靠。
  - 消融实验系统分析了各组件贡献，尤其验证了CoT和DPO的重要性。
  - 学生模型在CAPTOR候选和完全不同的Yelp数据集上零样本测试，证明了泛化性，避免了过拟合特定基线。
  - 公平性方面：所有对比模型使用相同的候选列表和评估协议；CF得分仅在训练集计算，防止信息泄露。
  - 潜在不足：所有实验均基于三个城市，城市数量有限；未报告多轮实验的标准差（但5折交叉验证本身包含稳定性信息）。

### 6. 论文的主要结论与发现
- DiMA在所有设置下一致优于基线，证明了LLM重排序范式的有效性。
- 引入辅助旅行历史（三城模型）显著优于仅使用居民历史（两城模型），验证了偏好偏差问题的存在及其解决方案的有效性。
- 学生模型（DiMA）在多数场景下超越教师模型（Teacher-3C），表明SFT+DPO对齐策略能够突破教师能力瓶颈。
- 消融显示CoT推理设计最为关键（移除后性能大幅下降），多模态标签和CF信号均提供有价值信息，DPO进一步优化。
- 框架具有良好的泛化性：在Foursquare-KDDC任务上训练的单模型可零样本应用于CAPTOR候选和Yelp数据集。
- 学生模型效率高（10秒 vs 教师5分钟），且不同规模模型均远超基线，提供了性能-效率权衡选择。

### 7. 优点
- **方法创新**：首次系统解决OOT推荐中的跨模态推理和偏好偏差问题，提出多模态对齐+CoT偏好分离的联合方案。
- **技术细节精巧**：CoT提示分三步明确引导模型综合游客轮廓、评估候选、排序，结构清晰；DPO中通过“赢家排序”构造偏好对，巧妙融合教师与真实标签。
- **实验全面且严谨**：两个数据集、两种候选生成、多种消融、零样本泛化测试，充分验证方法鲁棒性。
- **实用导向**：教师-学生蒸馏使模型推理速度从5分钟降至10秒，适合实际部署；学生模型可灵活权衡性能与效率。
- **案例直观**：展示了跨模态匹配（图像“taco”标签与评论“taco”标签）的成功案例，证明对齐的有效性。

### 8. 不足与局限
- **城市覆盖有限**：仅使用三个美国城市（Foursquare/Yelp各三城），扩展至更多城市（尤其非英语、不同文化背景）可能面临挑战。
- **对辅助旅行历史的依赖**：假设用户至少有一次其他城市的旅行历史，完全冷启动用户（无辅助城市）可能无法适用；虽可退化到两城模型，但未明确讨论。
- **数据预处理成本高**：使用MLLM和LLM为每个POI生成标签（尤其是图像描述）需要大量API调用或本地计算资源，实际应用中成本不可忽视（论文未量化）。附录或正文未说明有多少POI需要处理。
- **未报告模型训练时间**：SFT和DPO具体训练时长未给出，影响可复现性评估。
- **未讨论超参数敏感性**：如DPO的温度 \(\beta\)、历史长度限制、CF计算阈值等，未进行灵敏度分析。
- **评估指标单一**：仅采用NDCG@5和MAP@5，未包含召回率（Recall@K）或其他排序指标（如MRR）。
- **未考虑社交或实时因素**：未考虑用户社交关系、季节、天气等影响旅游行为的动态因素，简化了真实场景复杂度。

（完）
