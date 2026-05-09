# FinGPT: Democratizing Internet-scale Data for Financial Large Language Models

**作者**：Xiao-Yang Liu\*, Guoxuan Wang\*, Hongyang (Bruce) Yang, Daochen Zha (Columbia / JHU / Rice · AI4Finance Foundation)
**链接**：[arXiv 2307.10485 (v2)](https://arxiv.org/abs/2307.10485) · [GitHub: AI4Finance-Foundation/FinGPT](https://github.com/AI4Finance-Foundation/FinGPT) · [GitHub: AI4Finance-Foundation/FinNLP](https://github.com/AI4Finance-Foundation/FinNLP)
**会议**：NeurIPS 2023 Workshop on Instruction Tuning and Instruction Following
**版本说明**：v1 提交于 2023-07-19，本仓库批读以 v2（2023-11-14）为准

---

## 📌 一句话总结

FinGPT 是一个**以「数据」为中心的开源 FinLLM 框架**：它不和 BloombergGPT 比谁的模型大、不从零训，而是把「34 个金融数据源 + 实时数据清洗管线 + LoRA / QLoRA 轻量微调 + 用股价波动作为监督信号（RLSP）」打包开源，让训练成本从 267 万美刀（BloombergGPT）降到 **262 美刀**，同时给出 robo-advisor / 情绪分析做量化 / low-code factor 三个 demo。

---

## 🎯 学习目标（明天组会拷打 ready）

读完这篇我必须能流利回答这 4 题：

### Q1: BloombergGPT 和 FinGPT 在路线上的根本差别是什么？

> 应能讲清：训练范式（from-scratch vs 微调）、数据可得性（私有 vs 公开）、模型规模、成本量级、谁在更新这件事上更有优势。
> 答案分布：§3.3 + §5 + 自己脑补的成本对比表。

### Q2: 「Data-centric」在这篇文章里具体落实成了哪些环节？

> 应能讲清：data sources（§4.1, 4 类共 34 个源） → data interface（§4.2, date range / streaming） → cleaning（§4.4, 4 步） → filtering（§4.5, 5 类策略） → tokenization（§4.6） → 用作 LoRA 微调输入。
> 答案分布：整个 §4。

### Q3: RLSP 是什么？它解决了 RLHF 的什么问题？为什么可能不靠谱？

> 应能讲清：RLHF 要人工打标且贵；RLSP 用 ±2% 股价波动当 positive/negative/neutral 的弱监督；它的隐患是混淆相关与因果（股价涨不一定是因为这条新闻）。
> 答案分布：§5。

### Q4: FinGPT 在 quant sentiment 上对 LLaMA 的 198% 提升 / 9.5% CRR，应该信几分？

> 应能讲清：accuracy w/o neutral 198% 是从 0.063 → 0.188 的低基数翻倍（不是绝对水平的飞跃）；CRR 9.5% 是 backtest 不是真实交易，没扣手续费、滑点、做空成本；S&P 500 在测试期间整体走势会污染 baseline。
> 答案分布：§6.2.1 + Table 1 + Appendix D/E（本仓库不读 Appendix）。

---

## 📖 批读导航（按论文章节顺序）

按部就班，慢慢来比较快。每节 4 步循环：批读 → 我读 → 拷打 → Q&A 回写。

| # | Section | 内容 | 重要性 | 状态 |
|---|---|---|---|---|
| 0 | [00-abstract.md](sections/00-abstract.md) | 摘要：3 个挑战 + FinGPT 的 4 个应对 | 🔥 必读 | 🟡 已批读，待拷打 |
| 1 | [01-introduction.md](sections/01-introduction.md) | 为什么不能直接用 GPT-4 / 为什么 BloombergGPT 不够 / FinGPT 的 4 个贡献 | 🔥 必读 | 🟡 已批读，待拷打 |
| 2 | [02-related-work.md](sections/02-related-work.md) | 与 BloombergGPT、blueprint paper、当代同期工作的差异 | ⭐ 重点 | 🟡 已批读，待拷打 |
| 3 | [03-framework.md](sections/03-framework.md) | 训练 FinLLM 的 3 大挑战 + FinGPT 四层架构 + BloombergGPT 三大短板 | 🔥 必读 | 🟡 已批读，待拷打 |
| 4 | [04-data.md](sections/04-data.md) | 4 类数据源 / 数据接口 / 实时管线 / 清洗 / 过滤 / 分词（§4.1–4.6） | 🔥 必读 | 🟡 已批读，待拷打 |
| 5 | [05-lightweight-adaptation.md](sections/05-lightweight-adaptation.md) | LoRA / QLoRA + RLSP + 「为什么 ±2% 当三分类阈值」 | 🔥 必读 | 🟡 已批读，待拷打 |
| 6 | [06-applications.md](sections/06-applications.md) | Robo-advisor / Sentiment for quant（Table 1 & 2 重点拷打）/ Low-code factor | 🔥 必读 | 🟡 已批读，待拷打 |
| 7 | [07-conclusion.md](sections/07-conclusion.md) | 结论 + 我的 follow-up | ⭐ 重点 | 🟡 已批读，待拷打 |

---

## 🔑 核心贡献（先记这个）

1. **数据策划管线**：34 个数据源（19 news + 8 社交 + 3 filing + 4 research dataset），统一 API 用 date range / streaming 两种方式取
2. **应用上验证有效性**：在 robo-advisor、sentiment for quant、low-code factor 三个任务上跑通
3. **开源开放**：代码 + 数据接口 + 微调 pipeline 全部 open，对照 BloombergGPT 的全闭源
4. **轻量微调路线**：LoRA / QLoRA + RLSP（用市场价格波动当反馈，免人工打标）

---

## 📊 关键数字

| 指标 | FinGPT | BloombergGPT |
|---|---|---|
| 训练成本 | **\$262**（用 RTX3090 微调一次） | \$2.67M（512 × A100 × 53 天） |
| GPU hours | ~6 h（1 × A100 / 1 × RTX3090） | 0.65M GPU hours |
| 参数量级 | 用现成 LLM（ChatGLM2 / LLaMA2 / GPT-4 等）+ LoRA | 50B from-scratch |
| 训练数据 | 34 数据源 实时拉，按需微调 | 708B token 私有混合数据 |
| 是否开源 | ✅ 全开源 | ❌ 模型 / 数据 / API 全不开 |
| 量化 sentiment Avg. CRR (Table 1) | +9.5% | — |
| FPB / FiQA-SA F1（fine-tuned ChatGPT vs BloombergGPT） | 0.878 / 0.887 | 0.511 / 0.751 |

---

## 🤔 我读这篇时持续盯的 5 个问号

1. **「34 个数据源」是真都活着、都在维护，还是论文发表时刻的快照？** → 看 GitHub 仓库 issue 区比看论文有用
2. **RLSP 的 ±2% 阈值是怎么定的？换成 ±3% / ±5% 会怎样？** → 论文没做 sensitivity analysis（潜在弱点）
3. **Table 2 里 fine-tuned ChatGPT 在 FPB 上 0.878 真的能复现吗？** → ChatGPT 微调用的是 OpenAI 的 finetune API，价格、版本都没说清
4. **CRR 9.5% 没说时间窗口、是否扣交易成本、test set 大小** → §6.2 没回答的几个最重要问题
5. **它和后续 BloombergGPT-style "private LLM" 路线的对抗，到 2026 年到底谁赢了？** → 时代已经变了，但这篇放在 2023-07 的语境下读，依然是基本盘

---

## 🔥 拷打记录

每读完一个 section 都会在这里追加 Q&A 摘要。

_(待开始)_
