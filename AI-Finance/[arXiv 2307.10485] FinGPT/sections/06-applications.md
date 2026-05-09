[← 上一节](05-lightweight-adaptation.md) ｜ [返回论文 README](../README.md) ｜ [下一节 →](07-conclusion.md)

# 06 · Demonstrative Applications of FinGPT（三个 Demo）

> **来源**：arXiv 2307.10485v2, §6.1–§6.3, p7–p10（含 Table 1 & Table 2）
>
> **一句话定位**：这一节是论文的"实证篇"，给出三个 application：① Robo-advisor（只给一个 demo 例子），② Sentiment for Quant（**有完整定量评测**，全篇最实在的一节），③ Low-code factor 开发（demo 例子）。

---

## 📌 预览

§6 三个 application 的强度差异**极大**，要分别看：

```
6.1 Robo-Advisor              ──  Demo 1 例（cherry-picked）→ ⚠️ 几乎没说服力
6.2 Sentiment for Quant       ──  Table 1 + Table 2 完整评测 → ⭐ 最有信息的一节
       6.2.1 Labeling by Market（RLSP 验证）→ Table 1
       6.2.2 Supervised Fine-tuning（标准 SFT）→ Table 2
6.3 Low-code Development      ──  Demo 1 例 → ⚠️ 同样仅举例
```

🎯 **本节的训练目标**：
- 能解读 Table 1 的 198% improvement 的真实含义
- 能解读 Table 2 中 fine-tuned ChatGPT 在 FPB 上 0.878 vs BloombergGPT 0.511
- 能识别 robo-advisor 和 low-code 例子的 cherry-picking

---

## 📄 §6 · 三个 Application 概览

> **原文**：
>
> "In this section, we showcase three demonstrative financial applications of FinGPT, including:
>
> • **Robo-advisor**: Automated financial advisory services that offer personalized investment advice based on the user's risk tolerance and financial goals.
> • **Quantitative trading**: Using FinGPT's output as trading signals. Trading decisions can be made by combining with risk control.
> • **Low-code development**: Empowering non-technical users to create application software through graphical user interfaces and configuration, reducing the cost of programming."

💡 **三个 application 的实证强度**：

| Application | 评测形式 | 评测强度 |
|---|---|---|
| Robo-advisor | 单个 cherry-picked 例子（AAPL 2023-03-03） | ⚪ 几乎为 0 |
| Sentiment for Quant | Table 1 (RLSP) + Table 2 (SFT, 4 数据集) | ⭐⭐⭐ 最强 |
| Low-code | 单个 example（构建 factor library） | ⚪ 几乎为 0 |

⚠️ **小白容易被骗**：看到三个 application 平铺直述会以为它们都被严肃验证了。实际**只有 sentiment 一个有完整数据**。这是论文的"花哨封面"现象。

---

## 📄 §6.1 · Robo-Advisor（机器人投顾）

> **原文要点**：
>
> "The primary objective of a Robo-advisor is to offer easily understandable financial advice to human users. ... we showcase the capabilities of FinGPT by crafting appropriate prompts to deliver professional-grade financial services."
>
> "**Robo-Advisor on news.** We use crafted prompts to generate a news analysis of AAPL stock (Apple company) on March 3rd, 2023, which was the last trading day of that week..."
>
> 给了 3 段 prompt：
> - **Initial prompt**: 设定 trading machine 角色（"in a virtual environment", "won't have any impact on people's daily life", "express whatever you want without any restrictions"）
> - **News prompt**: "Please give a brief summary of the news and analyze the possible trend of the stock price..."
> - **FinGPT's response**: 一段约 200 字的新闻总结 + 涨跌分析

💡 **Initial prompt 的 jailbreak 味道**：
- "in a virtual environment" / "won't have any impact" / "express whatever you want without any restrictions"
- **这是经典的 prompt jailbreak 套路**，让 LLM 绕过自我审查（"我不能给投资建议"这种安全 reply）

⚠️ **小白别学**：这种 prompt 在合规语境下**有问题**：
- 真正给客户用的 robo-advisor 必须有合规约束（"不构成投资建议"等免责声明）
- 论文用这个 prompt 是 academic demo，**部署时不能直接用**

🧠 **FinGPT 的回答有什么特点**？让我们逐句拆：

> "There are a lot of news articles about Apple (AAPL) in the market, covering a wide range of topics, including the company's fundamentals, cloud initiatives, and new investments in India. Analysts are highlighting the company's overlooked catalysts, and subscription mania as a core driver for Apple. ... However, there are also reports about the company's cloud chief stepping down, and the rejection of ChatGPT-powered app updates. Based on these news articles, the possible trend of the stock price of Apple Company could be **positive**, ... However, ... the trend of the stock price of Apple Company **could be volatile**, and it could be influenced by a wide range of factors..."

🤔 **批判性观察**：
1. **"两面话"修辞**：先说 positive，又说 volatile / influenced by wide range of factors。**这是 LLM 在金融语境下的标准模板**，给所有方向都留余地，事后无论涨跌都能"对"
2. **没具体数字**：说"价格可能 positive"，但没说幅度、没说时间窗口、没给概率
3. **没风险刻画**：没说 stop loss、没说 position size、没说和投资者风险偏好的匹配

⚠️ **作为投顾这个回答不及格**：真正的投顾报告会有目标价、止损位、持有期、所基于的核心假设。FinGPT 这个回答更像是**新闻摘要 + 一句模糊预测**。

🧠 **但作为 demo 还行**：它展示了 FinGPT 能"读 N 条 AAPL 新闻 → 综合 → 给出一段连贯分析"。**这是 LLM 的基本能力**（GPT-4 也能做），FinGPT 没特别凸显出"金融专长"。

⚠️ **论文没说**：FinGPT 这一段输出 vs 直接拿 GPT-4 同样 prompt 的输出，差异在哪？没有 baseline 对比。**这是 §6.1 最大的问题** —— 单个例子 + 没 baseline = **几乎没有实证强度**。

---

## 📄 §6.2 · Sentiment Analysis for Quantitative Trading（最实在的一节）

### §6.2.1 · Labeling by Market（验证 RLSP）

> **原文**：
>
> "**Experimental setting.** We use the news data from the FMP data source and the price data from yahoo finance. We apply an automatic sentiment labeling process using a threshold of 2%. It is worth mentioning that the news data exclusively pertains to the constituents of the S&P 500 index. In our experiments, we compare the performance of LLaMA with that of FinGPT."
>
> "**Results.** The results are shown in Table 1. ... Notably, when excluding the 'neural' label, FinGPT exhibits substantial improvement. The superiority of FinGPT is also reflected in the cumulative return when performing the actual quantitative trading with an improved Avg. CRR."

🧠 **Table 1 · 复现表**（论文 p8）：

| Metrics | LLaMA | FinGPT | Improvement |
|---|---|---|---|
| ACC All | 0.450 | 0.481 | +0.031 (6.8%) |
| ACC w/o neutral | 0.063 | 0.188 | +0.125 (**198.4%**) |
| F1 All | 0.091 | 0.128 | +0.037 (40.7%) |
| F1 w/o neutral | 0.0350 | 0.0712 | +0.362 (103.4%) |
| Avg. CRR | -0.1% | **+9.5%** | +9.6% |

> Table 1 caption 解释指标：
> - **ACC w/o neutral**：只看 ground truth 是 positive / negative 的样本上的 accuracy
> - **CRR**（Cumulative Return Rate）：用 sentiment 信号做模拟交易：output positive → 5 天后卖；output negative → 5 天后买回（论文原文这里是 "buy back"，逻辑像是先 short 再 cover）

⚠️ **198.4% improvement 的真实含义**（很重要！）：

```
LLaMA  ACC w/o neutral = 0.063   (基本上随机的 ⅓ 都不到，约等于乱猜)
FinGPT ACC w/o neutral = 0.188   (差不多是随机的 1.88×，仍然很差)

「198% 提升」 = (0.188 - 0.063) / 0.063

但 0.188 这个绝对水平本身就是垃圾水平（三分类随机基线 0.33）
198% 的提升听起来很美，但 from 0.063 to 0.188 是「从难看到比较难看」
```

🤔 **批判**：
1. **基数小，倍数虚**：从极低 baseline（0.063）出发，任何小绝对增量都看起来是巨大百分比
2. **绝对水平不及格**：FinGPT 的 ACC w/o neutral = 0.188 < 三分类随机基线 1/3
3. **F1 全集 0.128 也很烂** —— 论文用 macro-F1，三分类随机的 macro-F1 大约 0.22，**FinGPT 0.128 比随机还差**
4. **CRR +9.5% 是 backtest，不是真实交易**：
   - 没扣手续费（每笔 0.1-0.2%）
   - 没扣滑点（市价单成交价 vs 决策价的偏差）
   - 没考虑做空成本（短视频股做空借券费很高）
   - 测试期间 S&P 500 整体走势没披露 → **可能是 buy-and-hold 也能涨 9.5%**

🧠 **正确解读 Table 1**：FinGPT 在 sentiment 分类绝对水平上**很差**（比随机猜还差），相对 LLaMA 有提升，但提升的来源**不一定是**"模型变聪明"，可能是：
- LLaMA 在金融语境下基本不会用，FinGPT 至少懂"positive / negative" 这些词在金融里指什么 → improvement 可解释为"格式适配"
- LLaMA 输出经常是 free text 不在三分类里 → ACC 自然低
- FinGPT 微调后输出格式严格三分类 → ACC 自然高

⚠️ **这意味着**：**Table 1 的 198% improvement 主要来自"格式匹配"而非"语义理解"**。这是这篇 paper 一个被忽视但严重的弱点。

⚠️ **CRR 的解读要谨慎**：
- 论文里 CRR -0.1% (LLaMA) → +9.5% (FinGPT) **可能完全是 noise**
- 没说测试集大小、测试期长度、对照基线
- **这是论文 framing 巧妙但实证薄弱的典型例子**

---

### §6.2.2 · Supervised Fine-tuning（标准 SFT 4 数据集）

> **原文**：
>
> "We mainly focus on the comparison of four financial datasets:
>
> • **FPB**: The Financial Phrasebank entails a sentiment classification task on sentences from financial news...
> • **FiQA SA**: ... forecast sentiment in English financial news and microblog headlines...
> • **TFNS**: The Twitter Financial News Sentiment dataset is an English-language compilation of finance-related tweets... 'Bearish' / 'Bullish' / 'Neutral'
> • **NWGI**: The News With GPT Instruction dataset uses labels produced by ChatGPT. ... it offers not just seven classification labels, but also provides a rationale for each label."

🧠 **Table 2 · 数据复现**（F1 score by support）：

| Category | Models | Device | Time | FPB | FiQA-SA | TFNS | NWGI |
|---|---|---|---|---|---|---|---|
| **Pre-trained LLM** | BloombergGPT | 512 × A100 | 53 d | 0.511 | 0.751 | - | - |
| | ChatGLM2 | 64 × A100 | 2.5 d | 0.381 | 0.790 | 0.189 | 0.449 |
| | Llama2 | 2048 × A100 | 21 d | 0.390 | 0.800 | 0.296 | 0.503 |
| | ChatGPT | - | - | 0.781 | 0.730 | 0.736 | - |
| | GPT-4 | - | - | 0.833 | 0.630 | 0.808 | - |
| **Fine-tuned LLM (FinGPT)** | ChatGPT (fine-tuned) | - | 4 h | **0.878** | **0.887** | **0.883** | - |
| | Llama2 (fine-tuned) | 1 × A100 | 5.5 h | 0.850 | 0.860 | 0.894 | 0.632 |
| | ChatGLM2 (fine-tuned) | 1 × A100 | 5.5 h | 0.855 | 0.850 | 0.875 | 0.642 |
| | ChatGLM2 (8-bit) | 1 × RTX3090 | 6.5 h | 0.855 | 0.847 | 0.879 | 0.636 |
| | ChatGLM2 (QLoRA) | 1 × RTX3090 | 4 h | 0.777 | 0.752 | 0.828 | 0.583 |

> 注：BloombergGPT 在 TFNS 和 NWGI 上没数（数据集不可得 / 已 disclosed）。GPT-4 NWGI 没值（NWGI 标签来自 ChatGPT，对 GPT-4 是污染）。BloombergGPT 的 device/time 是其训练 from-scratch 的成本，不是微调成本。

🔥 **Table 2 的关键观察**：

**1) 微调一个开源 LLM > 闭源 BloombergGPT**

```
FPB:      Fine-tuned ChatGLM2 (0.855) > BloombergGPT (0.511) → +67%
FiQA-SA:  Fine-tuned ChatGLM2 (0.850) > BloombergGPT (0.751) → +13%
```

🧠 **这个对比是 FinGPT 论文最强的论据**：用一张消费级显卡微调几个小时（成本 \$2.6），就把闭源花了 53 天 + 512 A100 训出来的 BloombergGPT 打败了。**这是"data + LoRA > brute-force pretraining"** 的实证支撑。

**2) Fine-tuning 比纯 prompt 强很多**

```
ChatGPT 原版 in 0-shot/5-shot:  FPB = 0.781, FiQA-SA = 0.730
ChatGPT fine-tuned:             FPB = 0.878, FiQA-SA = 0.887

→ +12% / +21% 的提升，相当显著
```

⚠️ **小白容易误判**：以为"GPT-4 强 → 用 GPT-4 就够了"。Table 2 显示 **fine-tuned 7B-13B 模型可以打败原版 GPT-4**：

```
GPT-4 原版 FPB:        0.833
ChatGLM2 fine-tuned:   0.855  → 用一个体量小 100x 的模型，微调后超过 GPT-4
```

**这是 fine-tuning 的力量** —— 任务专精化能突破规模差异。

**3) QLoRA 性价比之王，但代价是质量下降 8-10%**

```
ChatGLM2 (full fine-tuning, A100):  0.855 / 0.850 / 0.875 / 0.642
ChatGLM2 (QLoRA, RTX3090):         0.777 / 0.752 / 0.828 / 0.583
                                    -7%   -10%   -5%    -9%
```

⚠️ **QLoRA 不是免费午餐**：用 RTX3090 + 4-bit 量化，F1 在 FPB 上掉 7%、FiQA-SA 上掉 10%。这是个 **non-trivial 的 trade-off**，论文没强调。

**4) 时间对比**

```
BloombergGPT:        53 days × 512 A100      ← 训 from scratch 的天文成本
Llama2:              21 days × 2048 A100     ← 同上
Fine-tuned model:    4–6.5 hours × 1 GPU    ← 微调一次的"几小时"
```

🧠 **这个对比是论文 framing 的核心**：从 "几千张 GPU 几十天" 到 "1 张 GPU 几小时"，**3 个数量级的成本压缩**。

---

## 📄 §6.3 · Low-code Development（代码生成 demo）

> **原文要点**：
>
> "We focus on factors, which serve as the foundation of quantitative trading. Factors are utilized not only within the development environment but also in the production environment. We consider two specific example tasks as outlined below:
>
> **Example 1: Development Factors.** ... We demonstrate that the strong code generation capability of FinGPT significantly reduces the time and effort required. **Appendix G** showcases an example of utilizing FinGPT to construct a factor library.
>
> **Example 2: Finding New Factors.** ... Our FinGPT can expedite this process through the use of tailored prompts. Further details and examples can be found in **Appendix H**."

💡 **「Factor」在量化里是啥？** 量化交易里的因子是**用一组数据计算出的、对未来收益有预测能力的指标**。比如：
- **价值因子**：P/E, P/B 比率 → 便宜的股未来更可能涨
- **动量因子**：过去 12 个月收益 → 涨过的接着涨（动量延续）
- **规模因子**：小市值 vs 大市值
- **波动率因子**：低波动率股表现更好（low vol anomaly）
- **质量因子**：高 ROE / 低杠杆的"好公司"

🧠 **Factor library** 是量化基金的核心资产 —— 通常包含几十到几百个 factor 的计算代码。**写 factor 代码是低门槛但耗时的工作**：每个 factor 的逻辑不复杂，但要写、测试、回测，加起来几个月起步。

💡 **FinGPT 在这里的卖点**：让 LLM 帮你从"自然语言描述"生成 factor 代码（"给我 Fama-French 三因子的实现"）。这是个**应用 GPT 代码能力到量化领域**的典型例子。

⚠️ **§6.3 的实证强度**：和 §6.1 一样，**只有 example，没有评测**。具体例子在 Appendix G/H，正文几乎没说细节。

🤔 **可以批的点**：
- 没说 factor library 的代码正确率 / bug 率
- 没说"找新 factor"的有效率（生成的新 factor 真的有 predictive power 吗？还是 in-sample 看起来好但 out-of-sample 失效）
- 量化界对 LLM 生成 factor 的态度其实很谨慎 → 论文这里夸大了 LLM 在这个场景的实用性

---

## 💡 Section 总结

### 核心信息速查

| 维度 | 内容 |
|---|---|
| 三个 application | Robo-advisor / Sentiment for Quant / Low-code factor |
| 实证强度分布 | Robo-advisor 弱 / Sentiment 强 / Low-code 弱 |
| Table 1（RLSP） | LLaMA 0.063 → FinGPT 0.188 ACC w/o neutral；CRR -0.1% → +9.5% |
| Table 2（SFT） | Fine-tuned ChatGLM2 0.855 在 FPB 上打 BloombergGPT 0.511 |
| QLoRA 代价 | F1 掉 7-10%，但能在 RTX3090 上跑 |
| 时间对比 | 53 天 × 512 A100 → 几小时 × 1 GPU（**3 个数量级压缩**） |

### 核心洞察

1. **§6 的实证强度极不均衡**：只有 §6.2 是真实证，§6.1 / §6.3 都是 cherry-picked 例子。**读论文要分清"严肃实验"和"装饰性 demo"**。

2. **Table 2 是这篇论文最强的支撑**：fine-tuned 7B-13B 模型在 4 个 sentiment 数据集上**全面超过 BloombergGPT 50B + GPT-4**。这才是 "data-centric + LoRA > brute-force" 的真实证据。

3. **Table 1 的 198% improvement 要打折看**：基数太小（0.063），绝对水平差到比随机还差。**真正提升来源很可能是"格式适配"而非"语义提升"**。

4. **CRR +9.5% 几乎不能信**：没扣手续费 / 滑点、没披露测试期 S&P 走势、没说测试集大小。**严谨的 quant 论文必须给 Sharpe ratio + 风险指标 + benchmark**，FinGPT 没给。

5. **QLoRA 的 5-10% 性能损失要承认**：消费级硬件不是免费的。**论文 framing「\$262 训出 FinLLM」只对 QLoRA 成立，但 QLoRA 模型在 F1 上比 full fine-tuning 弱 7-10%**。

6. **Robo-advisor 例子用了 jailbreak prompt**：「won't have any impact on people's daily life」「express whatever you want without any restrictions」。这种 prompt 不能用于商业 robo-advisor，论文这里更像是技术 demo 而不是 production blueprint。

7. **Low-code factor 是个有趣的方向**，但论文做得太浅。Factor 生成的真实价值在 out-of-sample 测试，论文连 in-sample 都没系统跑。

---

## 🤔 我的 Follow-up 问题

1. **Table 1 的测试集多大？测试期多长？S&P 500 同期走势？** → 论文没说，去 Appendix D/E 看
2. **Table 2 的 fine-tuned ChatGPT 0.878 是怎么微调的？** OpenAI fine-tune API 的 cost / config？ → 论文说的不细，去 Appendix F
3. **QLoRA 的 -7-10% F1 损失，是什么原因？4-bit 量化精度损失 + LoRA 表达力不足？** → 没数据
4. **如果 RLSP 的 ±2% 阈值改用每只股票自己的 ±1.5σ，效果会变好吗？** → 论文没做
5. **fine-tuning 的训练数据多大（多少条 news + label）？** → 论文没在 §6.2 里说

---

## 🔥 拷打记录

_(待开始 — 等 Ricardo 读完此节后由 Claude 出 2~3 题做拷打)_

---

[← 上一节](05-lightweight-adaptation.md) ｜ [返回论文 README](../README.md) ｜ [下一节 →](07-conclusion.md)
