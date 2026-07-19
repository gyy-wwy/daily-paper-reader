---
title: "TraveLLaMA: A Multimodal Travel Assistant with Large-Scale Dataset and Structured Reasoning"
title_zh: TraveLLaMA：大规模数据集和结构化推理的多模态旅行助手
authors: "Meng Chu, Yukang Chen, Haokun Gui, Shaozuo Yu, Yi Wang, Jiaya Jia"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/37335/41297"
tags: ["query:rec-sys"]
score: 8.0
evidence: 多模态旅行助手，包含数据集和结构化推理，支持旅行规划
tldr: 现有旅行助手缺乏专业知识和情境理解。本文提出TraveLLaMA多模态语言模型，构建了包含26.5万问答对的TravelQA数据集和Travel-CoT结构化推理框架。该方法能够分解旅行查询并生成合理推荐，在旅行规划任务上表现出色。该工作为旅游推荐系统的智能化提供了基础数据和推理范式。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有旅行助手缺乏专业知识和上下文理解能力，无法提供个性化旅行推荐。
method: 构建包含文本和视觉问答的大规模数据集TravelQA，并设计Travel-CoT结构化推理框架分解旅行查询。
result: 在旅行规划任务上，TraveLLaMA比基线模型有显著提升，验证了数据集和推理框架的有效性。
conclusion: TraveLLaMA为旅游推荐系统提供了多模态数据和推理能力，有望推动个性化旅行助手发展。
---

## Abstract
Tourism and travel planning increasingly rely on digital assistance, yet existing multimodal AI systems often lack specialized knowledge and contextual understanding of urban environments. We present TraveLLaMA, a specialized multimodal language model designed for comprehensive travel assistance. Our work addresses the fundamental challenge of developing practical AI travel assistants through three key contributions: (1) TravelQA, a novel dataset of 265k question-answer pairs combining 160k text QA from authentic travel sources, 100k vision-language QA featuring maps and location imagery, and 5k expert-annotated Chain-of-Thought reasoning examples; (2) Travel-CoT, a structured reasoning framework that decomposes travel queries into spatial, temporal, and practical dimensions, improving answer accuracy by 10.8% while providing interpretable decision paths; and (3) an interactive agent system validated through extensive user studies. Through fine-tuning experiments on state-of-the-art vision-language models (LLaVA, Qwen-VL, Shikra), we achieve 6.2-9.4% base improvements, further enhanced by Travel-CoT reasoning. Our model demonstrates superior capabilities in contextual travel recommendations, map interpretation, and scene understanding while providing practical information such as operating hours and cultural insights. User studies with 500 participants show TraveLLaMA achieves a System Usability Scale score of 82.5, significantly outperforming general-purpose models and establishing new standards for multimodal travel assistance systems.

---

## 论文详细总结（自动生成）

# 详细中文总结：TraveLLaMA：大规模数据集和结构化推理的多模态旅行助手

## 1. 核心问题与整体含义（研究动机和背景）

- **研究动机**：现有数字旅行助手多为通用型多模态AI系统，缺乏对城市环境的**专业知识**和**上下文理解**。旅行规划需要同时理解视觉场景（图片、地图）、地理上下文（位置、距离）和实际约束（营业时间、预算、文化习俗），而现有模型在跨模态推理和领域知识方面存在严重不足。
- **整体含义**：开发实用的AI旅行助手面临数据缺失和推理能力薄弱两大挑战。本文旨在通过构建**大规模多模态旅行数据集**和**结构化推理框架**，训练出能够提供综合旅行建议的专门化模型，填补该领域的空白。

## 2. 方法论：核心思想、关键技术细节、流程

- **核心思想**：利用大规模、多模态、结构化数据微调视觉语言模型（VLM），并引入分解式推理（Travel-CoT）来显式处理空间、时间和实用三个维度的约束，最后通过ReAct风格的智能体实现实时交互。
- **关键技术细节**：
  - **TravelQA数据集**：包含26.5万问答对 —— 16万纯文本问答（来自26k事实单元扩展为五个问题+3万增强问答）、10万视觉语言问答（来自2万个POI，每POI4-5张街景/地图图像，GPT-4生成三种问题类型）、5千专家标注的链式思维（CoT）推理示例（每例含空间、时间、实用推理）。
  - **Travel-CoT结构化推理**：两阶段过程。
    1. 给定多模态输入(x, Q)，模型先生成推理链 `r = f_θ(x, Q), r = {r_s, r_t, r_p}`（分别对应空间、时间、实用推理）。
    2. 在输入和推理链共同条件下生成最终答案 `y ∼ P_ϕ(y | x, Q, r)`。
    3. 联合训练损失：`L = λ L_CoT(r*, r) + L_ans(y*, y)`。
  - **智能体架构**：基于ReAct，四阶段流程：
    1. **查询分析**：从文本和图像中提取约束（目的地、天数、预算、人数）并识别地标。
    2. **推理**：应用Travel-CoT组织空间、时间、实用需求。
    3. **工具调用**：调用外部API获取营业时间、价格、评论、交通信息，状态更新 `Plan_t = Update(Plan_{t-1}, Tool(π(s_t, Plan_{t-1})), r)`。
    4. **结果整合**：生成最终行程，包含日程、预算验证和约束检查。
- **流程概要**：数据构建并行处理文本和视觉语言两路 → 在TravelQA上微调VLM → 使用Travel-CoT后训练 → 部署为在线智能体。

## 3. 实验设计：数据集、基准、对比方法

- **数据集**：TravelQA数据集，划分训练集（213k QA对）和测试集（52k QA对），**POI不重叠**以防止数据泄露。测试集采用**四选一多项选择（MCQ）** 格式，干扰项来自同类别/区域语义相似的候选，答案通过标准化字符串匹配和轻量级语义匹配映射到选项。
- **评估任务**：覆盖纯文本理解（旅行知识）、地图理解、场景理解、信息提取、评论理解、时间推理等。
- **对比方法**：
  - 预训练基线：BLIP-2、InstructBLIP（7B/13B）、Shikra、Qwen-VL、Qwen-VL-Chat、LLaVA-1.5（7B/13B）。
  - 微调基线：在上述模型上使用TravelQA正常QA微调。
  - 本文完整模型：微调+Travel-CoT后训练（基于LLaVA-1.5 Vicuna-13B）。
  - 用户研究对比：Claude 3.5。
- **用户研究**：500名参与者（每组250人），采用间组设计、盲评。完成四项多步行程规划任务后填写SUS问卷（10项Likert量表转0-100）。报告95%置信区间、Cohen's d效应量、双侧t检验。

## 4. 资源与算力

- 文中明确指出使用 **8块A100 GPU** 进行训练。
- 训练数据：213k QA对。
- 图像分辨率标准化为336×336像素，文本输入最大长度512 tokens。
- **训练时长未说明**。仅提到采用标准优化技术（学习率调度、梯度裁剪、早停），未提供具体小时数或轮数。

## 5. 实验数量与充分性

- **实验组数**：
  - 表3：对比了8个预训练模型、8个对应微调模型 + 1个TraveLLaMA（共17个条件），报告纯文本、VQA、综合三项指标。
  - 表4：用户研究结果（SUS总分、10项明细、5项补充问题、人口统计）。
  - 图5：定性对比示例（3个场景：地图定位、街景识别、评论理解）。
- **充分性与公平性**：
  - **充分**：覆盖了主流VLM架构（BLIP-2、InstructBLIP、Shikra、Qwen-VL、LLaVA-1.5）及其变体规模，并报告百分比提升。
  - **公平**：所有微调模型使用相同的TravelQA训练集（POI不重叠测试集），采用统一的MCQ评估协议，消融通过对比微调前后和添加Travel-CoT后体现。
  - **不足**：未单独对Travel-CoT的三个维度（空间、时间、实用）分别做消融实验；未进行跨数据集泛化测试（如TravelPlanner）；用户研究仅对比一个通用模型（Claude 3.5），缺乏与其他旅行专用系统的对比；评估全部基于自动打分，人评仅针对SUS。

## 6. 主要结论与发现

- **域内微调显著提升**：在TravelQA上微调后，所有模型获得+6.2%~+9.4%的绝对提升（综合指标），其中Qwen-VL增益最大（+9.4%），LLaVA-1.5 13B获得最高绝对性能（综合76.0）。
- **Travel-CoT额外增益**：引入Travel-CoT后，TraveLLaMA相比预训练基线提升+10.8%（综合77.8），相比仅微调版本也有额外提升（+1.8点，从76.0到77.8）。
- **用户可用性优异**：SUS得分82.5（优秀），显著高于Claude 3.5的76.3（良好），主要在易用性、学习性和减少复杂性方面领先。
- **定性分析**：TraveLLaMA在地图定位、地标识别、多模态信息融合方面提供更准确、详细且可操作的旅行建议。

## 7. 优点

- **数据集规模与质量**：首个大规模多模态旅行QA数据集（26.5万对），包含文本、街景、地图和专家CoT标注，且通过多层验证（规则检查、语义一致性、手动抽样）保证质量。
- **结构化推理创新**：Travel-CoT将规划问题显式分解为空间、时间、实用三个可解释维度，不仅提升准确性（+10.8%），还提供透明的决策路径，增强可信度。
- **端到端系统验证**：从数据构建、模型训练到智能体部署和用户研究，形成完整闭环。500人用户研究提供了可靠的可用性证据。
- **公平评估设计**：POI不重叠划分防止数据泄露；MCQ格式保证客观可复现；采用多种架构和规模对比，结果稳健。

## 8. 不足与局限

- **实验覆盖有限**：
  - 仅验证单日城市行程规划，多日可扩展性仅作为动机提及但未充分实验。
  - 测试集为MCQ格式，可能不完全反映真实开放域生成的多样性。
  - 消融实验不完整：Travel-CoT各维度贡献未单独剥离；未见对数据集规模（如只使用文本或只使用视觉）的敏感性分析。
- **数据偏差风险**：
  - 文本和视觉问答均由GPT-4生成，可能引入生成模型的固有偏差或事实错误（尽管有三层验证）。
  - 专家CoT标注仅5k例，且来源未明确说明多样性（可能来自少数标注者）。
  - 数据集覆盖35+城市，但主要集中北美、亚洲和欧洲，对非洲、南美等地区代表性不足。
- **应用限制**：
  - 智能体依赖实时API调用，其可靠性受外部服务影响（如营业时间更新不及时）。
  - 用户研究仅针对英文用户，多语言支持未验证。
  - 训练需8×A100，对计算资源有限的团队可能构成门槛。
  - 未与公开旅行专用系统（如TravelPlanner）直接对比性能，仅对比通用模型和Claude 3.5。

（完）
