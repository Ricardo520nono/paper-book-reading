[← 上一节](04-data.md) ｜ [返回论文 README](../README.md) ｜ [下一节 →](06-applications.md)

# 05 · Lightweight Adaptation of General-Purpose LLMs to FinLLMs（LoRA + RLSP）

> **来源**：arXiv 2307.10485v2, §5, p7
>
> **一句话定位**：这一节回答**"训出一个 FinLLM 用什么方法"**。两个核心技术：① LoRA / QLoRA（解决 cost），② RLSP（解决 label）。**RLSP 是这篇论文 catchy 但争议最大的"创新点"**。

---

## 📌 预览

§5 一节解决两个挑战：

```
挑战 A: 全量微调贵      ────►  解法 A: LoRA / QLoRA
挑战 B: 标注数据贵      ────►  解法 B: RLSP（用市场涨跌当 label）
```

**这一节的训练目标**：
- 能说清 LoRA / QLoRA 在做什么、节省了什么
- 能说清 RLSP 的机制、它的隐患（confounding）
- 能识别出"RLSP 名字像 RL，本质是 supervised"

---

## 📄 原文 + 批注

### Para 1 · 双挑战陈述

> "The financial market is highly dynamic, necessitating frequent fine-tuning of the model. Leveraging pre-existing LLMs and fine-tuning them specifically for finance offers an efficient and cost-effective alternative to the expensive and time-consuming process of retraining models from scratch. However, there are two key challenges in enabling efficient fine-tuning."
>
> "**Firstly, LLMs consist of a large number of trainable parameters**, making the fine-tuning of all parameters a costly endeavor."
>
> "**Secondly, it is hard to directly obtain high-quality fine-tuning datasets in real-time.** The most commonly used method, Reinforcement Learning from Human Feedback (RLHF), requires human annotations, which, unfortunately, are difficult to obtain in real-time."

🧠 **两个挑战要分清**：

| 挑战 | 是什么 | 类比 |
|---|---|---|
| **A. 参数多 → 训不起** | 50B 参数全更新 → 几十 GB 显存 + 大量 GPU 时间 | "整本书重写"贵 |
| **B. 标签缺 → 不知道训什么** | 没人给金融文本打 ground truth 情绪标签 | "重写但不知道往哪改" |

⚠️ **小白容易把这两个挑战合并理解** —— 实际是**正交的**两件事：
- A 是「**怎么更新参数**」（计算 / 显存）
- B 是「**朝哪个方向更新**」（监督信号）
- 两个都要解决才能 fine-tuning 跑起来

---

### Para 2 · LoRA / QLoRA 解决挑战 A

> "To tackle the first challenge, FinGPT adopts Low-rank Adaptation (LoRA) and its quantized version QLoRA, which can significantly reduce the number of trainable parameters, and the training cost (see Appendix C for the detailed training cost analysis)."

🧠 **LoRA 的原理速记**：

```
原始 Transformer 层的某个 weight matrix W (大小 d × k):
    forward: h = W @ x

Full fine-tuning：
    把 W 整个更新 → 要算 W 的梯度 → 要存 W 的 optimizer 状态
    显存爆炸，参数量 = d × k

LoRA：
    把更新拆成 W + ΔW，其中 ΔW = B @ A
       A: r × k     (r ≪ d, k；通常 r = 8 或 16)
       B: d × r
    forward: h = (W + B @ A) @ x = W @ x + B @ (A @ x)
    
    训的时候 W 冻结，只训 B 和 A
    可训参数 = d×r + r×k = r(d+k) ≪ d×k
```

💡 **数字感受**：LLaMA-7B 中某个 attention 矩阵 d=k=4096，full fine-tuning 要更新 16M 参数。LoRA r=8 只要 65K 参数 → **降了 250 倍**。

🧠 **QLoRA 多干了一件事**：把冻结的 W 量化到 4-bit（NF4 量化），显存又省 4 倍。综合下来：

```
Full fine-tuning  LLaMA-7B：  ~80 GB GPU 显存（A100 80GB 勉强够）
LoRA              LLaMA-7B：  ~16 GB GPU 显存
QLoRA             LLaMA-7B：  ~6 GB GPU 显存（消费级 RTX 3060 都够）
```

**这就是为什么 FinGPT 能在 RTX 3090 上跑** —— Table 2 里 ChatGLM2 (QLoRA) 的硬件就是 1 × RTX3090。

⚠️ **小白别误解**：LoRA 不是"白送的"。代价：
- **拟合能力下降**：低秩 ΔW 表达能力 < 全秩
- **复杂任务上 underfit**：在简单任务（情绪三分类）上 LoRA 几乎无损；在复杂任务（多轮对话 / 复杂推理）上可能落后 full fine-tuning 2-5%
- **多 LoRA 不能简单叠加**：训了一个情绪 LoRA + 一个交易 LoRA，合起来用结果不可预测

🧠 **「适合 FinGPT 的任务」**：FinGPT 主要做情绪三分类、文本生成（投顾），任务相对简单 → LoRA 够用。如果是数学推理 / 长链条规划，full fine-tuning 优势会更明显。

---

### Para 3 · RLSP 解决挑战 B（这是这篇 paper 的 signature 创新）

> "To tackle the second challenge, FinGPT leverages the market's inherent labeling capacity, dubbed Reinforcement Learning with Stock Prices (RLSP). Specifically, we prompt the model to select one from the positive, negative, and neutral output, given an input text. Then, we use the relative stock price change percentage as the output label to instruct the LLMs."

🧠 **RLSP 的工作流**（重要！）：

```
1. 拿一条新闻（关于某只股票），比如「AAPL 发布 Q3 财报，营收超预期」
2. 看新闻发布后 N 天（论文里没明说 N，从 Table 1 推测是 5 天）股价变化
3. 用 ±2% 阈值打 3 分类标签：
       涨 > +2%   →  Positive
       跌 < -2%   →  Negative
       中间       →  Neutral
4. 用 (新闻文本, 标签) 对训 LLM 做三分类
```

⚠️ **「Reinforcement Learning」是个 misnomer**：
- 真正的 RL 涉及：state, action, reward, trajectory, policy gradient (PPO 等)
- RLSP **没有 reward model、没有 trajectory、没有 PPO**
- 它实际是**"用股价当弱监督标签的 supervised fine-tuning"**

🤔 **为什么这么命名？** 三个可能：
1. **致敬 RLHF**：让人立刻 grasp "用反馈代替人工标签"
2. **修辞效果**：RL 听起来比 supervised 高级
3. **市场反馈被框架性地视为"环境奖励"**：可以 hand-wave 说"市场是 environment, label 是 reward signal"

⚠️ **小白别被名字骗了**：**RLSP 在实现层面是 supervised fine-tuning。** 看代码（GitHub）就懂。

---

### Para 4 · 数据隐私 + LoRA 的"plug-and-play"

> "The application of LoRA within our framework not only enhances performance but also maximizes the protection of our users' data privacy. Users are empowered to utilize our FinGPT framework to train their own LoRA weights, which can be used in a straightforward 'plug-and-play' manner. Essentially, our FinGPT framework does not offer direct financial advice but instead equips end users with data sources and tools to train their own LoRA weights and integrate them with LLMs."

💡 **"Plug-and-play LoRA" 是 LoRA 的一个非常实用属性**：训出来的 LoRA 权重通常很小（几十 MB），可以独立分发、热插拔到 base model 上。这意味着：
- 用户可以**只在自己机器上训自己的 LoRA**，不用上传数据到云
- 一个 base model 可以加载**不同的 LoRA**（情绪 LoRA / 投顾 LoRA / 因子 LoRA）来切换任务
- 可以**多人共享**：你训了情绪 LoRA，别人下载就能用，base model 不动

🧠 **数据隐私的论证**：
- 自己训自己的 LoRA → 数据不出本地
- FinGPT 不提供 LLM-as-a-service → 不接触用户数据
- 这种 "self-hosted" 设计是开源 FinLLM 的天然优势

⚠️ **批判**：这个隐私论证**只对懂技术的用户有效**。普通用户没法自己 host LLM + 训 LoRA。普通用户最终还是要用某个 LLM-as-a-service（OpenAI / Anthropic / 国内厂商），数据隐私问题没解决。**FinGPT 把隐私问题**外包**给了用户的工程能力**。

---

### Para 5 · 实现细节（重要！）

> "**Implementation.** In this work, we implement this idea by applying specific thresholds to gauge fluctuations in the stock price. We categorize company-related texts into three groups: 'Positive' when the stock price exhibits an increase of more than 2%, 'Negative' when the stock price shows a decrease of over 2%, and 'Neutral' when the relative change falls within the range of -2% to 2%. Notably, this automated labeling process does not require human participation. We used the following prompt for fine-tuning, **'What is the sentiment of this news? {sentence} Please choose an answer from strong negative/moderately negative/mildly negative/neutral/mildly positive/moderately positive/strong positive, then provide some short reasons.'**, where {sentence} is the input text."

💡 **细节 1：±2% 阈值**

```
为什么是 2%？
- 股票日均波动率（VIX 平静时）：1-1.5%
- 2% ≈ 1.5σ，捕获明显事件，过滤随机噪声
- 但论文没做 sensitivity analysis（换 1% / 3% 会怎样）
```

⚠️ **2% 是个看似合理但没系统验证的拍脑袋数字**。这是 RLSP 的一个隐患：
- 不同股票波动率差异巨大（meme stock 日均 5%, 大盘股 1%）
- 用统一阈值会让小盘股全是 noise label，大盘股全是 neutral

**改进方向**：用「standardized return」（除以历史波动率）当阈值，而不是固定 2%。论文没做这件事。

💡 **细节 2：prompt 不是三分类，是 7 分类！**

注意 prompt 里：

```
strong negative / moderately negative / mildly negative / neutral / 
mildly positive / moderately positive / strong positive
```

**7 个 bucket，不是 3 个！**

但 RLSP 的 ground truth label 只有 3 类（pos/neg/neu）。**这是个不一致**：
- 训练时 supervision 信号是 3 分类
- 模型输出时却要 7 分类
- 7 分类的细粒度从哪来？

🤔 **可能的解释**：
1. 用 7 分类作 prompt template，但 loss 只在 "positive / negative / neutral" 这 3 个桶上算（聚合）
2. 用 base model 自己的语言能力把 7 类映射回 3 类（弱推断）
3. 论文写 sloppy，实际 fine-tuning 只用 3 类，prompt 是 inference 时的额外要求

**论文这一段写得不清楚**。要看 GitHub 代码才能搞清楚实际怎么实现的。

⚠️ **如果是情况 3，那 prompt 里 7 类其实是装饰品**，模型实际只学到 3 类的判别能力。**这是论文写作的一个含糊点，可作为追问点**。

---

### Para 6 · 转 Appendix

> "We provide more discussion of the dynamic datasets and the fine-tuning methods in Appendix A."

💡 **§5 整节正文极短**（不到 1 页），多数细节在 Appendix。这是 NeurIPS Workshop 论文的常见模式：正文限页 → Appendix 自由发挥。

---

## 💡 Section 总结

### 核心信息速查

| 维度 | 内容 |
|---|---|
| 双挑战 | A. 参数多（训贵） + B. 标签缺（找不到 ground truth） |
| 解法 A | LoRA / QLoRA（参数 -250×, 显存 -10×） |
| 解法 B | RLSP（用 ±2% 股价波动当弱监督标签） |
| RLSP 阈值 | ±2%（论文没做 sensitivity） |
| Prompt | 7 类粒度（strong neg / mod neg / mild neg / neu / mild pos / mod pos / strong pos） |
| Label | 3 类（论文实际打的标签） |
| 实现位置 | 主体在 §5，细节在 Appendix A |

### 核心洞察

1. **LoRA + QLoRA 是 FinGPT 经济性的真正根基**。\$262 这个数字大半要归功于 LoRA。如果没有 LoRA，FinGPT 的"民主化"叙事就垮了。

2. **RLSP 是 misnomer，本质是 supervised fine-tuning**。它不是 RL，没有 reward model、没有 PPO、没有 trajectory。论文起这个名字主要为了 framing。

3. **RLSP 的核心隐患是 confounding**：股价涨不一定是因为这条新闻。可能：
   - 同时有大盘行情拉动
   - 同时有更大的负面新闻被这条对冲
   - 政策面 / 宏观因素影响占主导
   - **论文完全没讨论这个问题**

4. **±2% 阈值是个拍脑袋数字**：没做 sensitivity analysis，没考虑股票间波动率差异。**用 standardized return 是更合理的改进**。

5. **Prompt 7 类 vs label 3 类的不一致**：论文写得不清楚，可能是"装饰性 prompt + 实际 3 类训练"，可能要看代码确认。**这是这篇 paper 在表达精确度上的一个失分点**。

6. **数据隐私论证只对工程师友好**：FinGPT 鼓励用户"自己训自己的 LoRA"，听起来隐私好，但**普通用户做不到 self-host**，所以最终隐私问题被外包给了用户工程能力。

---

## 🤔 我的 Follow-up 问题

1. **±2% 换成 ±1% / ±3% / standardized return 会怎么样？** → 自己跑实验
2. **Prompt 7 类 vs label 3 类的实现细节** → 看 GitHub
3. **RLSP 的命名争议在 OpenReview 上有没有 reviewer 提？** → 论文是 NeurIPS Workshop 不知道公开 review 没
4. **multi-LoRA 叠加（情绪 LoRA + 投顾 LoRA）效果如何？** → 论文没做，是个 future work
5. **如果 base model 切换（LLaMA → Qwen → Gemma），LoRA 是不是要重训？** → 是的，LoRA 与 base 强绑定

---

## 🔥 拷打记录

_(待开始 — 等 Ricardo 读完此节后由 Claude 出 2~3 题做拷打)_

---

[← 上一节](04-data.md) ｜ [返回论文 README](../README.md) ｜ [下一节 →](06-applications.md)
