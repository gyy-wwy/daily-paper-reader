---
title: "Membership Inference Attack Against Large Language Model-Based Recommendation Systems: A New Distillation-Based Paradigm"
title_zh: 基于知识蒸馏的成员推理攻击：针对大语言模型推荐系统的新范式
authors: "Cuihong Li, Xiaowen Huang, Chuanhuan Yin, Jitao Sang"
date: 2026-03-17
pdf: "https://ojs.aaai.org/index.php/AAAI/article/download/39449/43410"
tags: ["query:rec-sys"]
score: 6.0
evidence: 针对基于大语言模型推荐系统的成员推理攻击
tldr: 传统成员推理攻击在LLM推荐系统上效果不佳。本文提出基于知识蒸馏的攻击范式，通过蒸馏构建参考模型，对成员和非成员数据采用不同策略。实验表明，该方法在多个推荐数据集上显著提升了攻击成功率，揭示了LLM推荐系统的隐私风险。
source: AAAI-2026-Accepted
selection_source: conference_retrieval
motivation: 现有成员推理攻击在规模庞大的LLM推荐系统上性能下降。
method: 利用知识蒸馏构建参考模型，分别对成员和非成员数据设计不同策略以增强区分度。
result: 在多个推荐数据集上，攻击成功率和准确率均显著高于传统方法。
conclusion: 该工作揭示了LLM推荐系统的隐私泄露风险，需加强防御。
---

## Abstract
Membership Inference Attack (MIA) aims to determine whether a specific data sample was included in the training dataset of a target model. Traditional MIA approaches rely on shadow models to mimic target model behavior, but their effectiveness diminishes for Large Language Model (LLM)-based recommendation systems due to the scale and complexity of training data. This paper introduces a novel knowledge distillation-based MIA paradigm tailored for LLM-based recommendation systems. Our method constructs a reference model via distillation, applying distinct strategies for member and non-member data to enhance discriminative capabilities. The paradigm extracts fused features (e.g., confidence, entropy, loss, and hidden layer vectors) from the reference model to train an attack model, overcoming limitations of individual features. Extensive experiments on extended datasets (Last.FM, MovieLens, Book-Crossing, Delicious) and diverse LLMs (T5, GPT-2, LLaMA3) demonstrate that our approach significantly outperforms shadow model-based MIAs and individual-feature baselines. The results show its practicality for privacy attacks in LLM-driven recommender systems.

---

## 论文详细总结（自动生成）

## 1. 核心问题与整体含义（研究动机和背景）

- 成员推理攻击（Membership Inference Attack, MIA）旨在判断某数据样本是否被用于目标模型的训练。传统MIA方法依赖影子模型（shadow model）模拟目标模型行为，但在基于大语言模型（LLM）的推荐系统中效果显著下降，因为LLM的训练数据规模大、复杂度高，且已有研究表明传统影子模型对LLM的MIA性能接近于随机攻击。
- LLM推荐系统通过微调LLM（如T5、GPT-2、LLaMA3）获取推荐能力，但微调数据包含用户隐私，面临MIA风险。现有针对LLM的MIA方法（如基于困惑度或概率最小K%）依赖单一特征，区分能力不足。
- 本文提出基于知识蒸馏的MIA新范式，专门用于LLM推荐系统，旨在提升成员与非成员数据的区分能力，从而更有效评估隐私泄露风险。

## 2. 论文提出的方法论

- **核心思想**：利用知识蒸馏从目标模型（教师模型）构建参考模型（学生模型），并对成员和非成员数据采用**差异化蒸馏策略**，使参考模型对二者的输出行为差异最大化。随后从参考模型中提取融合特征（包括置信度、熵、损失、倒数第二层隐藏层向量），训练攻击模型进行推断。
- **关键技术细节**：
  - **蒸馏策略**：
    - 对非成员数据：侧重软标签（soft label），损失函数为 \( L_{\text{nonmem},i} = (1-\alpha) \cdot \text{HardLoss}_i + \alpha \cdot \text{SoftLoss}_i \)，旨在模仿教师模型在未见数据上的行为。
    - 对成员数据：侧重硬标签（hard label），损失函数为 \( L_{\text{mem},i} = \alpha \cdot \text{HardLoss}_i + (1-\alpha) \cdot \text{SoftLoss}_i \)，旨在提升成员数据上的性能，拉大与成员行为的差距。
  - **蒸馏流程（算法1）**：
    1. 在非成员数据上训练学生模型，强调软标签。
    2. 然后在成员数据上继续训练，强调硬标签。
  - **特征融合**：将置信度、熵、损失（一维浮点数）和倒数第二层隐藏层向量（512维）通过MLP升维至统一维度后拼接，形成融合特征，用于训练攻击模型（逻辑回归分类器）。
  - **攻击模型**：使用逻辑回归分类器（迭代次数1000），避免复杂模型在小数据上过拟合。
- **公式**：
  - 硬标签：成员为[1,0]，非成员为[0,1]。
  - 软标签：教师模型logits经温度T平滑后的softmax输出。
  - 硬损失：交叉熵；软损失：温度缩放后的KL散度。

## 3. 实验设计

- **数据集**：Last.FM（2k+原始）、MovieLens（8k）、Book-Crossing（19k+）、Delicious（114k+）。其中Last.FM和Delicious被扩展为自然语言指令格式（用户历史+偏好询问），仅含交互历史而无显式偏好标签。
- **目标模型**：T5-base（编码器-解码器）、GPT-2-medium（仅解码器）、LLaMA3-3B（仅解码器）。所有目标模型使用LoRA微调。
- **对比基线**：
  - Loss Attack（即PPL，以困惑度作为阈值）
  - min-k%（基于token概率的最小百分比）
  - min-k%++（归一化变体）
  - Zlib（基于压缩熵）
  - Shadow模型（传统影子模型范式）
- **评估指标**：准确率（Accuracy）、召回率（Recall）、F1分数、AUC。
- **设置细节**：对每个数据集，20%作为非成员（未参与训练），80%作为成员；其中成员数据的5%作为攻击者已知的成员背景知识，其余成员数据未知。非成员数据可通过公开资源获取。

## 4. 资源与算力

- 文中明确说明：所有实验在8块A3090 GPU上完成。
- 未具体报告每项实验的训练时长（如蒸馏轮数、微调时间等），仅提到蒸馏时每个学生模型在成员和非成员数据上各训练5个epoch。

## 5. 实验数量与充分性

- **主实验**（表2）：在T5-base上对比四个数据集的全部基线，每个数据集均报告Acc、Recall、F1、AUC，共4组。
- **不同LLM上的基线对比**（表3、表4）：在MovieLens和Book-Crossing上对T5、GPT-2、LLaMA3分别应用PPL、min-k%、min-k%++，共6组。
- **蒸馏模型类型选择**（表5）：考察不同教师-学生架构组合（同构/异构），在4个数据集上做3×3=9种组合，共36组结果。
- **α参数影响**（图4）：α从0到1每0.1步长，4个数据集各绘制曲线。
- **攻击模型类型**（表6）：比较逻辑回归与BERT两种攻击模型，4个数据集，训练和推理时间对比。
- **融合特征有效性**：对比融合特征 vs 单个特征（表7），以及去除每个特征后的性能（表8），各4个数据集。
- **特征融合策略**（表9）：7种融合策略在MovieLens上的比较。
- **总体评价**：实验覆盖多种模型架构、多种数据集、多种基线、多种消融场景，全面性高。结果呈现清晰，具有客观性。

## 6. 论文的主要结论与发现

- 提出的蒸馏MIA范式在所有数据集和评估指标上均显著优于基线。
- 传统影子模型MIA在LLM推荐系统中性能接近随机（F1/AUC约50%），而本文方法可大幅提升至AUC 90%以上。
- 融合特征（置信度、熵、损失、隐藏向量）比单一特征更具区分能力，其中损失特征贡献最大，向量特征贡献最小。
- 蒸馏后参考模型的成员与非成员输出特征分布差异明显（图3），而影子模型和原始模型几乎无区分。
- 简单分类器（逻辑回归）在数据量不大时优于复杂模型（BERT），且训练时间可忽略不计。
- α值对攻击性能有影响，最优值因数据集而异，但总能在0.1~0.9范围内显著超过随机攻击。

## 7. 优点（方法或实验设计的亮点）

- **方法创新**：首次将知识蒸馏引入LLM推荐系统的MIA，并设计差异化蒸馏策略（成员侧重硬标签、非成员侧重软标签），打破了影子模型的对称行为限制。
- **特征融合**：解决单个特征区分力不足的问题，通过MLP升维后拼接，保留多维度信息。
- **实验广泛**：涵盖4个数据集、3种不同架构的LLM（T5、GPT-2、LLaMA3），并扩展了Last.FM和Delicious到自然语言格式。
- **消融深入**：对蒸馏参数α、攻击模型复杂度、特征组合、融合策略等均进行了系统比较，结论稳健。
- **实用性**：灰盒攻击假设合理（仅知道模型类型），且代码开源（https://github.com/Cherie212/MIA4LLMRS.git）。

## 8. 不足与局限

- **攻击假设**：当前为灰盒攻击（知道模型类型），更严格的攻击场景应为黑盒（仅能访问输出）。作者承认未来会探索黑盒蒸馏。
- **数据语义**：扩展的Last.FM和Delicious仅含交互历史而无显式用户偏好，导致语义信息不足，影响攻击性能。作者计划未来提取更高质量语义。
- **蒸馏限制**：跨架构蒸馏（如T5→GPT-2）因词汇表不匹配需截断，造成信息损失，导致异构蒸馏性能低于同构。这限制了范式在不同架构间的直接迁移。
- **输出分布非高斯现象**：T5的成员/非成员输出分布有时呈多模态（非高斯），原因尚不明确，需进一步研究。
- **实验覆盖**：未涉及更大型LLM（如LLaMA3-8B以上）或更多推荐场景（如图文推荐、序列推荐等），泛化性有待验证。
- **防御讨论缺失**：论文未讨论如何针对该攻击设计防御措施，仅揭示了隐私风险。

（完）
