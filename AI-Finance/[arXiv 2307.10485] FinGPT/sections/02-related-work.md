[← 上一节](01-introduction.md) ｜ [返回论文 README](../README.md) ｜ [下一节 →](03-framework.md)

# 02 · Related Work（相关工作）

> **来源**：arXiv 2307.10485v2, §2, p3
>
> **一句话定位**：这一节短，但任务很关键 —— **划清 FinGPT 在三层文献版图中的位置**：① 早期金融 NLP（用文本辅助交易）；② BloombergGPT（first FinLLM）；③ 同期与之后的 FinLLM 工作。

---

## 📌 预览

Related Work 是论文里**说话最克制**的一节，但**信息密度极高**。读这一节的训练目标不是记每个引用，而是**学会把一个 paper 放进文献版图**。

FinGPT 把"金融文本 + ML"分成 3 个时代：

```
Era 1（早期）         Era 2（BloombergGPT）       Era 3（FinGPT 同期 / 之后）
─────────────         ────────────────────         ───────────────────────────
2010s 早期             2023-03                       2023-07 起
用文本做特征 → 预测     第一个 FinLLM                 同期还有 [26] / [7] 等
任务专用模型           50B from-scratch             Open-source 路线兴起
[3, 4, 24, 18, 25]    [1]                           [26, 27, 28, 8, 9, ...]
```

FinGPT 把自己放在 Era 3，主动用「open-source data pipeline」当差异点。

---

## 📄 原文 + 批注

### Para 1 · 早期金融文本工作 → BloombergGPT

> "Financial text data is indispensable for training FinLLMs. Early research efforts have focused on utilizing financial text data for stock price prediction and the development of algorithmic trading strategies [3, 4, 24]. Recent studies adopt reinforcement learning to learn trading strategies with financial text data as features [18, 25]. The most recent effort, BloombergGPT [1], trains a FinLLM on a mixture of general text data and financial text data."

🧠 **早期金融 NLP 的 4 种范式**（按时间顺序）：

| 时期 | 范式 | 代表工作 | 文本的作用 |
|---|---|---|---|
| 2010-2015 | Bag-of-words / TF-IDF + 线性模型 | [4] (Zhang & Skiena 2010) | 词频→情绪极性→交易信号 |
| 2015-2018 | Word2Vec / GloVe + LSTM | [3] (Peng & Jiang 2016) | 词向量→序列模型→预测涨跌 |
| 2018-2022 | BERT / FinBERT + 监督任务 | [24], FinBERT (Yang et al. 2020) | 微调小模型做情绪 / NER |
| 2022-2023 | RL on text features | [18], [25] | 文本作为 RL agent 的状态特征 |
| 2023+ | LLM 时代 | [1] BloombergGPT, FinGPT | LLM 直接生成 / 推理 |

⚠️ **小白看这一段会被 reference 数字劝退**。技巧：**记范式，不记 reference number**。下次需要查具体哪一篇时再去 reference list 找。

💡 **「mixture of general text data and financial text data」是 BloombergGPT 的关键设计**：
- 总共 708B tokens
- 其中 **363B 是金融数据**（Bloomberg 内部 45 年的归档）
- **345B 是通用数据**（Pile, C4 等公开语料）
- 一半一半混着训，避免模型只懂金融、丢了基本语言能力

---

### Para 2 · 从「特征工程」到「LLM」的范式跃迁

> "While these studies shed light on the importance of financial text data in the financial domain, they lack an open-sourced data collection and curation pipeline, which is crucial for practical applications in the time-sensitive financial market, especially in training FinLLMs. Furthermore, previous text data have primarily been used either to train models for specific tasks or to build LLMs from scratch [1]. In contrast, our FinGPT utilizes text data for efficient fine-tuning, incorporating real-time market feedback efficiently."

💡 **这段在做"差异化"声明**，三个对比：

| 之前的工作 | FinGPT |
|---|---|
| 没有 open data pipeline | ✅ 开源 + 实时 |
| 特定任务专用模型 | ✅ 通用 → 通过 fine-tuning 适配多任务 |
| From scratch（贵） | ✅ Fine-tuning（便宜） |

🧠 **范式跃迁的本质**：以前是「文本是特征，喂进模型预测涨跌」；现在是「文本是知识，让模型自己理解金融」。

⚠️ **「real-time market feedback」是 RLSP 的伏笔** —— Related Work 把 RLSP 包装成"和先前工作的差异点之一"，再次强化它在 Abstract / Intro 中提过的 contribution。这是论文写作的**重复-强化技巧**。

---

### Para 3 · 与同期工作的关系

> "A contemporary work [26] has also focused on financial text data. What sets our endeavor apart is our commitment to delivering not only high-quality datasets but also a streamlined data pipeline. The vision paper [7] has outlined the vision of FinGPT and discussed the future directions. However, in contrast to [7], the current paper centers on the datasets, with the intention of empowering users to harness our data sources to train their own FinLLMs. Additionally, we provide evaluations to showcase the potential of our data sources, an aspect not addressed in the vision paper [7]."

💡 **这一段在做 self-clarification** —— 同一个团队 (Yang, Liu, Wang) 的另一篇 [7] 在同期发布 (FinLLM Symposium @ IJCAI 2023)。两篇关系：

| [7] FinGPT vision paper | 当前 paper（数据 paper） |
|---|---|
| 描绘整个 FinGPT 的愿景 / 路线图 | 聚焦在数据这一层 |
| 没具体跑实验 | 跑了 sentiment 实验 |
| 综述性 | 工程性 |

⚠️ **小白别被绕晕**：「FinGPT」既指**这篇 paper 描述的数据框架**，也指**整个项目（[7] 描述的更大愿景）**。"FinGPT 是一个项目，本 paper 是这个项目里的一篇"，不是孤立成果。

🧠 **同期工作 [26]**：是另一个团队也在做金融文本数据集。论文没点名，但 [26] 是 PIXIU (Xie et al. 2023)。FinGPT 和 PIXIU 是 2023 年并行的两个开源 FinLLM 项目，**FinGPT 主打数据管线，PIXIU 主打 instruction-tuned model + benchmark**。两者其实互补而非竞争。

---

### Para 4 · 审稿期间出现的相关工作

> "During the reviewing process, we saw several relevant works [27, 28, 8, 9, 29, 30, 31, 32, 33]. We have provided additional related work in Appendix J."

💡 **这是个学术写作的「自保动作」** —— 论文从投稿到见刊往往隔几个月，这期间会冒出一堆相关工作。如果不提，审稿人会质疑「你为什么不引这些」。把它们扔进 Appendix J 是常见操作。

⚠️ **小白阅读策略**：这些 [27, 28, ...] 不需要逐个查，**只在你做综述 / 自己写 paper 时再去看**。**当下读 paper 不要被 reference 拖住节奏**。

---

## 💡 Section 总结

### 核心信息速查

| 维度 | 内容 |
|---|---|
| 文献版图划分 | Era 1 早期 NLP / Era 2 BloombergGPT / Era 3 同期 FinLLM |
| FinGPT 的位置 | Era 3 中的「open-source data-centric」一支 |
| 同期对照 | PIXIU [26]（开源 model + benchmark），FinGPT [7] vision paper（路线图） |
| 关键差异点 | 开源 + 数据管线 + 实时 + LoRA 微调 + 用市场反馈 |

### 核心洞察

1. **Related Work 的真正功能：文献版图定位**。不是为了"显得读得多"，是为了**告诉读者"我和谁竞争、和谁互补"**。FinGPT 把自己放在 Era 3，明确点名 [1] BloombergGPT 是对手、[26] PIXIU 是友军、[7] 是同团队的姊妹篇。

2. **同名概念的歧义**：「FinGPT」既是项目名（[7] 里的愿景），也是当前 paper 的名字。读论文时遇到 "FinGPT" 要 disambiguate 是指**愿景项目**还是**这篇数据 paper**。

3. **Appendix J 是个"过期防御工事"**：审稿期出现的论文都堆在那。**第一次读这篇 paper 时不必读 Appendix J**；如果以后做综述，再回来翻。

4. **范式跃迁的简短故事**：
   - 早期：文本 → 特征 → 模型 → 信号
   - 现在：文本 → LLM 微调 → 直接做任务（情绪 / 投顾 / 代码）
   - 差异：从「特征工程」转向「指令微调」

5. **「open-source data pipeline」作为差异点的强弱**：作为学术贡献，纯工程的 data pipeline 是**弱**的；作为开源社区贡献，是**强**的。FinGPT 自觉走的是后者 —— 它不是 SOTA paper，是 community paper。

---

## 🤔 我的 Follow-up 问题

1. **PIXIU 和 FinGPT 的具体差别是什么？** → 论文外查 PIXIU 原文 [26]
2. **FinGPT vision paper [7] 的"愿景"具体描绘了什么？** → 同上
3. **「real-time market feedback」是不是和 [18, 25] 的 RL-on-text 有重叠？** → 论文没说清这两类 RL 的差异，可能是后续要追的点
4. **2024-2026 年 FinLLM 领域是不是又冒出新一批论文，FinGPT 的 framing 还成立吗？** → 这是我作为 2026 年读者要自己评估的

---

## 🔥 拷打记录

_(待开始 — 等 Ricardo 读完此节后由 Claude 出 2~3 题做拷打)_

---

[← 上一节](01-introduction.md) ｜ [返回论文 README](../README.md) ｜ [下一节 →](03-framework.md)
