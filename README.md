# Paper & Book Reading 📚

Ricardo 的论文 + 读书笔记仓库。每篇内容采用「批读 + 拷打」双轨学习：

- **批读（Batch Reading）**：原文完整保留 + 内嵌批注，重点 paper 走完整流程
- **拷打（Quiz）**：每读完一节，由 Claude 反向提问直到我能用自己的话讲清楚，问答记录回写到笔记里

## 📚 概念词典

跨论文通用的基础概念沉淀在这里：

> [`_concepts/`](_concepts/) — VAE / Encoder-Decoder / DiT / Flow Matching / Q-K-V / ...

读 paper 第一次遇到某个概念就**链接到这里**，不在 section 里重复解释。

## 📂 课题列表

### 🤖 VLA-WM（Vision-Language-Action + World Models）

视频生成基座 / 机器人操作策略 / 世界模型

| 论文 | 会议 / 来源 | 状态 | 一句话 |
|---|---|---|---|
| [Wan](VLA-WM/%5BarXiv%202503.20314%5D%20Wan/) | arXiv 2503.20314 | 🟡 批读中 | 阿里通义全开源视频生成基座（1.3B / 14B），Wan 系列论文底座 |

### 💹 [AI-Finance](AI-Finance/)（金融与 AI 交叉）

金融大模型（FinLLM） / 量化交易 NLP / 市场情绪分析 / 金融数据工程

| 论文 | 会议 / 来源 | 状态 | 一句话 |
|---|---|---|---|
| [FinGPT](AI-Finance/%5BarXiv%202307.10485%5D%20FinGPT/) | arXiv 2307.10485 · NeurIPS 2023 Workshop | 🟡 批读中 | 用 data-centric 开源框架把 FinLLM 训练成本从 267 万美刀压到 262 美刀（34 数据源 + LoRA + RLSP） |

### 💰 [Wealth-Growth](Wealth-Growth/)（财富增长）

商业模式 / 营销与销售 / 定价与现金流 / 个人财富。**纯读书笔记栏目**，不走拷打流程，详见栏目内 README。

| 书 | 作者 | 状态 | 一句话 |
|---|---|---|---|
| [$100M Money Models](Wealth-Growth/%5BAlex%20Hormozi%5D%20%24100M%20Money%20Models/) | Alex Hormozi | 🟡 待读 | 用 4 类「money model」让一家公司在 30 天内回本现金、解锁无限广告投放 |

---

## 🛠️ 仓库使用约定

### 文件夹命名

```
[会议 年份] 论文名/   或   [arXiv ID] 论文名/
```

### 单篇论文结构

```
[会议 年份] 论文名/
├── README.md         ← 论文概览 + 学习目标 + Section 导航
├── sections/         ← 批读笔记（原文 + 内嵌批注 + 拷打）
│   ├── 00-abstract.md
│   ├── 01-introduction.md
│   └── ...
├── full.md           ← PDF 抽出的完整文本
├── images/           ← 论文关键图片
└── paper.pdf         ← 原始 PDF
```

### 批注语义约定

| 标记 | 含义 |
|---|---|
| `💡` | Claude 的批注：术语解释 / 背景补充 / 对比 / 重组 |
| `🎯` | 学习目标 / 我必须回答的问题 |
| `🤔` | Follow-up：读完留下的问号 |
| `🔥` | 拷打 Q&A：Claude 的提问 + 我的回答 + 总结 |
| `⚠️` | 容易搞混 / 容易踩的坑 |

### 图片硬性约定（重要）

**只要批读里提到了 Figure / Table，就必须把图本身嵌到 markdown 里**，不能光用文字描述。

工作流：
1. 用 `pdftoppm` 把 PDF 渲染成 PNG（每页一张）
2. 用 PIL 裁剪到具体 Figure 的区域
3. 保存到该论文的 `images/` 目录，命名 `figure-NN.png` / `table-NN.png`
4. markdown 里用 `![](../images/figure-NN.png)` 嵌入

**裁剪 + 图注铁律**：
- ✂️ **裁剪要紧**：只包含图本身 + 它自己的英文 caption，**不要带前一节末尾或后一节开头的 prose**。裁完一定 Read 一遍验证
- 🚫 **正文不重复英文 caption**：caption 已经在图里了，markdown 里不要再用 `> Figure X: ...` 引用块复述一遍
- 🇨🇳 **如果需要补充图注**，写**中文概要**而不是英文原文复读（避免冗余）
- 🔁 **裁剪不准就重裁**：裁剪是迭代的，不要凑合保留烂图

### 拷打规则（重要）

| 规则 | 内容 |
|---|---|
| **R1 目的** | 拷打的目的是让我**真正理解当前 section**，不是为了出题而出题 |
| **R2 范围** | 问题**只能**基于当前 section 的已读内容，不跳到后面没读的章节 |
| **R3 闭环** | **任何拷打 Q&A 必须同步写到 section 文件的 `🔥 拷打记录` + 推到 GitHub，不能只在 chat 里飘**。详见两种格式 ↓ |
| **R4 节奏** | 一轮拷打 = 一组 2~3 题 → 一次 commit/push（不要每题一推，太碎） |

#### 两种拷打格式

**格式 A（老）·"我先答"**：适合有基础时强化记忆
1. Claude 出题（不给答）
2. Ricardo 用自己的话答
3. Claude 评分 + 标答
4. **Q&A 写回 section 🔥 拷打记录 + push**

**格式 B（新）·"题 + 答同时给"**：适合打地基阶段，速度更快
1. Claude **同时**给出题和标答（chat 里 + section 文件里）
2. **立刻 commit + push**
3. Ricardo 阅读 → 有不懂的追问 Claude
4. Claude 解答追问 → 必要时更新 section + 再 push

⚠️ **不论哪种格式，规则是一样的**：**Q&A 内容不能只存在 chat 里**，必须落到 section 文件并 push 上 GitHub。

### 好题 vs 烂题（Claude 出题前自检）

| 题型 | 价值 |
|---|---|
| 「X 在第几章 / 第几页？」 | ❌ 烂题：查目录就行，不考理解 |
| 「X 是什么名字？」 | ⚠️ 中等：纯记忆 |
| 「X 解决了什么问题？为什么这个设计 work？」 | ✅ 好题：考理解 |
| 「X 和 Y 比，谁更适合 Z 场景？」 | ✅ 好题：考应用 |
| 「这个方法有什么弱点？哪里值得追问？」 | 🌟 满分题：考批判 |

---

由 Ricardo + Claude 协作整理 🤝
