[← 返回论文 README](../README.md) ｜ [← 上一节](00-abstract.md) ｜ [下一节 →](02-related-work.md)

# 01 · Introduction（引言）

> **来源**：Wan paper §1 INTRODUCTION（p3，full.md 行 143–192）
>
> **一句话定位**：Introduction 干两件事 —— **(1) 立靶**（指出当时开源模型的 3 大 gap）+ **(2) 摆出 Wan 的解题方案**（架构、规模、双尺寸、下游任务、开放训练全流程）。

---

## 📌 预览

Introduction 的逻辑骨架：

```
   现状：Sora 引爆视频生成 → 开源社区努力追赶
        ↓
   3 大 Gap：
      ① Suboptimal Performance（开源 vs 闭源还是有质量差距）
      ② Limited Capabilities（开源大多只做 T2V，覆盖不足）
      ③ Insufficient Efficiency（动辄 60GB+ 显存，普通人用不了）
        ↓
   Wan 的解法：
      • 架构：DiT + Flow Matching + 全时空 attention
      • 规模：14B params，O(1) trillion tokens 训练
      • 双尺寸：1.3B (8.19GB VRAM) + 14B
      • 多任务：I2V / 视频编辑 / 个性化 / 实时 / 音频
      • 开放：全流程公开（数据 / VAE / 训练 / 加速 / 评测）
```

⚠️ **关键观察**：这 3 大 gap **正好对应 Abstract 里 4 大 key features 中的前 3 个**：
| Intro 的 Gap | Abstract 的 Key Feature |
|---|---|
| Suboptimal Performance | → Leading Performance ✅ |
| Limited Capabilities | → Comprehensiveness ✅ |
| Insufficient Efficiency | → Consumer-Grade Efficiency ✅ |

第 4 个 key feature **Openness** 在这一节最后一段独立论证（公开训练全流程）。**这印证了我们 Q0.2 的修正：4 个 key features 是正交的**。

---

## 📄 原文 + 批注

### Para 1 · 立靶 —— 开源社区的三大 Gap

> "Since the introduction of Sora (OpenAI, 2024) by OpenAI, video generation technology has attracted substantial attention from both industry and academia, leading to rapid advancements in the field."

💡 **时代背景**：Sora 是 2024 年 2 月 OpenAI 公开的视频生成模型，被视为视频生成领域的"GPT-3 时刻"。从此**视频生成成为 AI 的主流方向**，类似 ChatGPT 之于 NLP。

> "These rapid advancements in video generation technology have also been greatly attributed to the development of the open-source community. Notable projects like HunyuanVideo (Kong et al., 2024), Mochi (GenmoTeam, 2024), and CogVideoX (Yang et al., 2025b) have made their video foundation model codes and weights publicly available..."

💡 **三大同期开源对手 cheatsheet**：

| 模型 | 团队 | 规模 | 特点 |
|---|---|---|---|
| **HunyuanVideo** | 腾讯 | 13B | 同期最强开源 T2V |
| **Mochi** | Genmo | 10B | 美国创业公司，强 motion |
| **CogVideoX** | 智谱 | 5B | 中国学术界路线，效率优先 |

⚠️ **小白别误读**："HunyuanVideo" ≠ Wan 主要 baseline，它只是被点名提及的同期工作。Wan 的真正比较对象在 Figure 1 里（包含 CN-TopA/B/C 这种匿名商业模型）。

> "However, it is essential to acknowledge the persistent gap between these outstanding open-source models and the latest closed-source models. This gap is primarily evident in three aspects: ..."

💡 **接下来这 3 个 gap 是论文的核心立靶**。Wan 后面所有的设计都是为了同时打掉这 3 个 gap。把这 3 个名字记住：

#### Gap 1: Suboptimal Performance（性能不足）

> "A notable performance gap remains, as the pace of development in commercial models far exceeds that of current open-source models, resulting in significantly superior capabilities."

💡 **翻译**：开源模型的进步赶不上闭源模型的进步速度。**注意原文用的是"pace of development"** —— 它说的不只是当下落后，而是**差距正在拉大**。

🤔 **为什么开源会越追越落后**？因为闭源公司（OpenAI / Runway / Kling）**有持续付费用户的现金流 → 持续投入算力 → 持续训新版本**；开源项目大多是企业 PR 投入或学术界一次性 release，没法持续迭代。

#### Gap 2: Limited Capabilities（能力受限）

> "Most foundational models are limited to general text-to-video (T2V) tasks, whereas the demands of video creation are multifaceted. Consequently, basic T2V models are insufficient to address these diverse requirements."

💡 **翻译**：大多开源模型只做 T2V（文本→视频），但实际创作需要更多任务 —— 比如：
- I2V（图像→视频，让一张图动起来）
- V2V / 视频编辑（基于现有视频改）
- 个性化（基于用户上传素材生成）
- 相机控制（指定运镜）
- 实时生成（流式输出）
- 音频生成（视频配音 / 音乐）

⚠️ **小白可能误解**：「T2V is limited」≠ T2V 不重要。T2V 是 video gen 的**核心技能**，但真正的产品需要"全家桶"。这就是为什么 Wan 要做 8 个下游任务（Abstract 提过）。

#### Gap 3: Insufficient Efficiency（效率不足）

> "Despite their impressive performance and scale, these models often prove impractical for creative teams with limited computational resources, hindering their accessibility and usability."

💡 **翻译**：开源模型动辄 13B / 60GB+ VRAM，**实际上还是只有有钱的团队能用**。这呼应了 Abstract 里的 "Consumer-Grade Efficiency" key feature。

⚠️ **关键观察**：Wan 团队在 Intro 里**主动承认"开源 ≠ 真的可用"**。这是 Q0.2 我们修正过的洞察 —— 开源和易用是两个独立维度。

> "These challenges collectively impose constraints on the continued growth and innovation within the open-source community."

💡 **结论**：3 个 gap 加起来限制了开源社区的发展。**Wan 把自己定位成"打破这 3 个 gap"的方案** —— 这是论文的论证起点。

---

### Para 2 · 解法 —— 架构 + 规模 + 训练数据

> "To address the aforementioned challenges, this report introduces and publicly releases a novel series of high-performance foundational video generation models, referred to as Wan, which sets a new benchmark in the field."

💡 用一句话宣告完成上面立的靶子。**注意 "publicly releases"** —— 直接锁定了"3 大 gap 中的 efficiency 通过开放解决"这个论证链。

> "The core design of Wan is inspired by the success of Diffusion Transformers (DiT) (Peebles & Xie, 2023) combined with Flow Matching (Lipman et al., 2022)..."

💡 **Wan 的两个核心架构选择**：

#### ① DiT (Diffusion Transformer)

```
传统 Diffusion (SD 1.5 / SDXL)         DiT (SD 3 / Sora / Wan)
─────────────────────────────────────────────────────────────────
backbone = U-Net                        backbone = Transformer
更老经典，适合中小模型                  更新主流，更适合 scale up
─────────────────────────────────────────────────────────────────
图像优势                                视频更友好（attention 天然支持时序）
```

⚠️ **小白要 get 的点**：DiT 不是新算法，是把 diffusion 的 backbone 从 U-Net 换成 Transformer。这一换让 diffusion 模型继承了 Transformer 的 scaling laws 优势。**Sora 之后 DiT 成为视频生成事实标准**。

#### ② Flow Matching

💡 **它是 diffusion 的训练目标的一种新表述**：

```
Diffusion / Score Matching (传统)      Flow Matching (新)
─────────────────────────────────────────────────────────────────
学习"如何从噪声 denoise 回数据"        学习"从噪声到数据的速度场"
公式: 优化 score function ∇log p_t(x)  公式: 优化 velocity field v_t(x)
训练相对不稳定，需要复杂噪声 schedule  训练更稳定，直接学直线插值
─────────────────────────────────────────────────────────────────
SD 1.5 / 2.1 / SDXL 用这个              SD 3 / Wan / Hunyuan 用这个
```

🤔 **不必深究数学**，记住这条：Flow Matching 是 2024 年起 diffusion 训练的**主流新选择**，更稳定、采样更快。

> "Within this architectural paradigm, cross-attention is employed to embed text conditions, while the model's design is meticulously optimized to ensure computational efficiency and precise text controllability."

💡 **Cross-attention 干什么**？让文本 prompt 影响视频生成 —— 文本 token 作为 K/V，视频 latent 作为 Q，让视频在生成时能"看到"文本要表达什么。

> "To further enhance the model's ability to capture complex dynamics, a full spatio-temporal attention mechanism is incorporated."

💡 **Full Spatio-Temporal Attention 是 Wan 的关键架构选择**：

```
方案 A: 分离时空 attention (老路线)         方案 B: Full Spatio-Temporal (Wan 选这个)
─────────────────────────────────────────────────────────────────
spatial attention (帧内)                    所有 frame 的所有 patch 一起做 attention
+ temporal attention (帧间) 分两步           计算量更大，但能捕捉复杂时空依赖
计算便宜但表达力受限                        Sora / Wan / 大型 video model 都走这条
```

⚠️ **代价**：full ST attention 的计算量是 O((T·H·W)²)，T 长一点就指数增长。这就是为什么后面 §4.3 §4.4 要花大力气讲 parallelism 和 cache 优化 —— **没有这些工程优化，full ST attention 跑不起来**。

> "Through extensive experimentation, the model is validated at scale, reaching 14 billion parameters. Subsequently, Wan has seen large-scale data comprising billions of images and videos, amounting to O(1) trillions of tokens in total."

💡 **三个数字拧在一起**：
- **14B 参数**（旗舰尺寸）
- **billions** 图像 + 视频（粗粒度数据量）
- **O(1) trillion tokens**（细粒度的训练 token 数）

⚠️ **"O(1) trillions" 是什么意思**？这是个大概数 —— "1 万亿量级"。Wan 在低估自己 —— 实际可能是 1.5T 或 2T，但用 O(1) 这种符号低调表示。这个量级对标 LLM 的话相当于 Llama 3 的 15T tokens 的某个分数。

> "This extensive training facilitates the emergence of the model's capabilities, allowing it to demonstrate robust performance across multiple dimensions, such as motion amplitude and quality, visual text generation, camera control, instruction adherence, and stylistic diversity."

💡 **5 大涌现能力**（顺手记下来）：
1. Motion amplitude and quality（动作幅度和质量）
2. Visual text generation（画面里的文字 —— Abstract 里强调过这个，Wan 是首个支持中英双语的）
3. Camera control（相机/运镜控制）
4. Instruction adherence（指令遵从性）
5. Stylistic diversity（风格多样性）

⚠️ **"emergence" 这个词很 LLM 风**。Wan 团队把视频生成往"涌现能力"框架靠，暗示这些能力不是单独训练的，而是 scale 上去后自然涌现的。这是个**值得追问的 claim** —— 后面 ablation 看是否有支撑。

---

### Para 3 · 解法 —— 双尺寸策略

> "Building upon the powerful foundational model, we have expanded its capabilities to numerous downstream tasks, including image-to-video generation (I2V), instruction-guided video editing (V2V), zero-shot personalized customization, real-time video generation, and audio generation, among other critical applications."

💡 **5 个被点名的下游任务**（注意：Abstract 说 8 个，Intro 这里点名 5 个 —— 另外 3 个是 T2V 主任务、T2I、相机控制）

> "To minimize inference costs, we also introduce a 1.3B model alongside a 14B model for T2V and I2V, both of which support 480p resolution and greatly enhance inference efficiency. Remarkably, the 1.3B model requires only 8.19G of VRAM, allowing it to run on many consumer-grade GPUs, while its performance exceeds that of many larger open-source models."

💡 **重要细节**（Abstract 里没说的）：
- **1.3B 和 14B 都同时支持 T2V 和 I2V**（不是只有 14B 才有 I2V）
- **两者都支持 480p**（论文这里没提 720p，可能 720p 仅 14B 模型 / 仅扩展应用支持）
- **1.3B 在性能上"exceeds many larger open-source models"** —— 不是和 14B 比，是和"中量级开源"比。打的是 5B-10B 范围的对手

🤔 **疑问**：1.3B 的 8.19GB VRAM 是什么精度？FP16？BF16？INT8？Intro 没说，要等 §4.4 看推理优化细节。

---

### Para 4 · 开放性论证 —— 公开训练全流程

> "Moreover, we will also publicly present the entire training process, including the large-scale data construction pipeline, video variational autoencoder (VAE), training strategies, acceleration techniques, and automated evaluation algorithms, to empower the community in developing specialized foundational video models."

💡 **"Openness" 这一段单独论证 —— 印证了 Q0.2 的修正**：Openness 是独立于 Efficiency 的 key feature。

💡 Wan 公开的"5 件套"，对应论文 4 大章节：

| 公开内容 | 对应章节 |
|---|---|
| Data construction pipeline | §3 Data Processing Pipeline |
| Video VAE | §4.1 Spatio-temporal VAE |
| Training strategies | §4.2 Model Training |
| Acceleration techniques | §4.3 + §4.4 Scaling + Inference |
| Automated evaluation algorithms | §4.6 Benchmarks + §4.7 Evaluation |

> "We are confident that these contributions will play a pivotal role in accelerating the advancement of video generation technology."

💡 **结尾标准 hype 句**。每篇大厂 paper 都会写一句 "we will accelerate the field"，作为启发性收尾。

---

## 💡 Section 总结

### 核心信息速查

| 维度 | 内容 |
|---|---|
| **三大 Gap**（开源 vs 闭源） | ① Suboptimal Performance ② Limited Capabilities ③ Insufficient Efficiency |
| **架构骨架** | DiT (Diffusion Transformer) + Flow Matching |
| **关键 attention 设计** | Full Spatio-Temporal Attention（不是分离的 spatial + temporal） |
| **文本条件注入方式** | Cross-attention |
| **训练规模** | 14B params + O(1) trillion tokens |
| **5 大涌现能力** | 动作 / 视觉文本 / 相机 / 指令 / 风格 |
| **双尺寸策略** | 1.3B (8.19GB VRAM) + 14B，都支持 T2V 和 I2V，都支持 480p |
| **公开 5 件套** | 数据 pipeline + VAE + 训练策略 + 加速 + 评测算法 |

### 核心洞察

1. **Intro 的 3 个 Gap 是论文的论证起点**：Wan 后面所有设计的 motivation 都可以追溯到这 3 个 gap。读后面章节时记得回来对照。

2. **DiT + Flow Matching 是 2024-2025 年的事实标准**：Sora、Hunyuan、Wan、SD3 都用，开源社区几乎统一了。

3. **Full Spatio-Temporal Attention 选择决定了后面要做大量工程**：因为 O((T·H·W)²) 的计算复杂度，§4.3-4.4 整两章都在解决"如何让它跑起来"。这是个**架构选择驱动工程优化**的典型例子。

4. **"O(1) trillion tokens" 是低调表达**：Wan 实际训练数据量级和 LLM 中型模型相当。

5. **Para 4 单独论证 Openness 印证了 Q0.2 的修正**：Openness 和 Efficiency 不是同一件事，Intro 用了完全独立的一段来讲 Openness（公开数据 / 训练 / 加速 / 评测全流程）。

---

## 🤔 我的 Follow-up 问题

1. **1.3B 模型的 8.19GB VRAM 是什么精度？** Intro 又提了一遍这个数字但没说精度 → §4.4 看
2. **"Emergence of capabilities" 这个 claim 是否有 scaling ablation 支撑？** → §4.2 + §4.7 看 scaling experiments
3. **Full ST Attention 的计算复杂度怎么 tame？** → §4.3 (Parallelism) + §4.4 (Inference Cache)
4. **Flow Matching 在 Wan 的具体损失函数长啥样？** → §4.2 看公式
5. **1.3B 模型也支持 I2V 吗？性能怎样？** Intro 说"both for T2V and I2V"，但 5.1 只字未提 1.3B I2V 的实测 → §5.1 看

---

## 🔥 拷打记录

### Round 2 · 2026-05-09

#### Q1.1 ｜ 三大 Gap 是否独立？

**问题**：3 大 Gap (Suboptimal Performance / Limited Capabilities / Insufficient Efficiency) 是彼此独立的还是有重叠？挑最容易混的一对，讲根本区别。

**Ricardo 答**：「Limited Capabilities 和 Insufficient Efficiency 有重叠，因为本质都源于闭源公司有钱、开源公司赶不上。」

**评分**：6/10 — 思考深度好，但**框架走偏**：把"shared cause"和"overlapping nature"混淆了。

**关键修正**：3 大 Gap 共享上游原因（资源差距），但作为**问题表现**是独立的 3 个维度：

| Gap | 衡量维度 | 测试问题 |
|---|---|---|
| Suboptimal Performance | 视频本身质量 | 「出来的视频好不好看？」 |
| Limited Capabilities | 任务覆盖度 | 「能干哪些事？只 T2V 还是 + I2V/编辑/...?」 |
| Insufficient Efficiency | 硬件资源门槛 | 「需要什么档次 GPU？」 |

**反例验证**（套用 Q0.2 方法论）：HunyuanVideo（13B 开源 T2V，60GB+ VRAM）—— 修复 Capabilities（加 I2V）不会自动修复 Efficiency；反之亦然。它们独立。

**类比记忆**：感冒咳嗽和发烧都源于病毒感染（共同原因），但治咳嗽不会自动退烧。**Shared cause ≠ Overlapping nature**。

**Take-away**：判断"两个东西是不是同一回事"的方法 —— 找两个交叉反例（A 但非 B / B 但非 A）。这套方法已用于 Q0.2 (Efficiency vs Openness) 和 Q1.1 (3 大 Gap)。

---

#### Q1.2 ｜ Cross-attention vs Full Spatio-Temporal Attention

**问题**：Wan 用了两种 attention，各自作用是什么？为什么需要两种而不是一种？

**Ricardo 答**：
- Cross-attention：「文字 token 和视频互相看，T2V 要文本 token 进来，I2V 可能图片 + 文字一起进来」
- Full ST attention：「我不是很懂。视频是持续的、空间的，所以要时空 attention」

**评分**：5/10 — 直觉对方向，但精度不够（缺 Q/K/V 框架；没抓到 "full vs separated" 的核心区别）。

**补的基础知识 — Attention 的 Q/K/V**：

类比搜索引擎：Q = 你的搜索词，K = 网页标题，V = 网页内容。Self-attention 是 Q/K/V 全来自同一组 token；Cross-attention 是 Q 来自一组，K/V 来自另一组。

**标准答案**：

| 项 | Cross-attention | Full ST Self-attention |
|---|---|---|
| Q 来自 | 视频 latent | 视频 latent |
| K, V 来自 | **文本 token**（T5 等编码器） | **视频 latent**（同一组） |
| 方向性 | 视频 → 看 → 文本（单向） | 视频 → 看 → 视频自己 |
| 解决问题 | 跨模态条件注入 | 模态内部时空一致性 |

**Full vs Separated ST Attention 的核心区别**（Q1.2 真正考的）：

```
Separated（老路线）                Full（Wan 选）
─────────────────────────────────────────────────
Spatial：只看同一帧             所有 (T·H·W) 个 patch
Temporal：只看同一空间位置跨帧   一次 attention 全看
─────────────────────────────────────────────────
计算便宜                         O((T·H·W)²) 爆炸
跨时空对角依赖抓不到             任意依赖都能抓
─────────────────────────────────────────────────
                                 ⇒ 必须配套做工程
                                   优化 (§4.3 §4.4)
```

**为什么需要两种 attention**：
- Cross-attention 处理"外部"（文本想让我做什么）
- Self ST attention 处理"内部"（视频各 patch 自己怎么协调）
- 缺 cross：视频不知道要生成什么内容
- 缺 self：视频内部前后帧不连贯

**Take-away**：
- Attention 类型 = "谁查询谁"（Self = 自查；Cross = 跨查）
- "Full" ST attention 选了就要付出 O((T·H·W)²) 的代价 → 推论：§4.3 §4.4 整两章的存在理由 = 让这个代价能扛住

---

#### Q1.3 ｜ Para 4 为什么独立成段？

**问题**：Intro 共 4 段，前 3 段讲 Gap 和解法，Para 4 单独讲公开训练全流程的 5 件套。为什么独立？这印证了 Q0.2 的什么洞察？

**Ricardo 答**：「我觉得是写作结构上的考虑。5 件套可能分别对应论文 section。Wan 团队大概想把自己的创新点再强调一下。」

**评分**：5/10 — 抓到了"路线图功能"这个真实点，但**没接上 Q0.2 的洞察**。

**标准答案 —— Para 4 同时干两件事**：

#### 第一层（题目重点，对应 Q0.2 的回响）：印证 Openness 是独立 key feature

数一下 Intro 4 段的功能分布：

```
Para 1 ── 立靶：3 大 Gap (Performance / Capabilities / Efficiency)
Para 2 ── 架构 + 规模 + 训练           → 答 Performance
Para 3 ── 多任务 + 双尺寸              → 答 Capabilities + Efficiency
Para 4 ── 公开训练全流程               → 答 Openness（独立成段）
```

🎯 **关键观察**：如果 Openness 只是 Efficiency 的延伸（"开源就够了"），它会被合并到 Para 3 末尾一句话带过。但 Wan 团队**特意单独成段**写它 —— 这是 **论文结构上的物理证据**：「Openness 是独立 key feature」。

> Para 4 独立存在 = Q0.2 修正洞察的实物证据。

#### 第二层（你抓到的）：路线图功能

| Para 4 的 "5 件套" | 对应论文章节 |
|---|---|
| Data construction pipeline | §3 |
| Video VAE | §4.1 |
| Training strategies | §4.2 |
| Acceleration techniques | §4.3 + §4.4 |
| Automated evaluation algorithms | §4.6 + §4.7 |

#### 两层合起来

> **Para 4 同时**：(1) 在论文写作结构上**实证** Openness 是独立 key feature；(2) 充当论文路线图。

**Take-away**：以后读 paper 的 Intro **要数段落数和功能分布** —— 段落怎么切是作者论证结构的物理体现。如果一个概念被独立成段，它八成是作者认为的"独立贡献维度"。

---

### Round 2 总结

| 题 | 得分 | 关键改进点 |
|---|---|---|
| Q1.1 概念区分 | 6/10 | 混淆 "shared cause" 和 "overlapping nature"；用反例法验证独立性 |
| Q1.2 架构理解 | 5/10 | 补 Q/K/V 框架；锁定 "full vs separated" 是核心区别；理解 full 选择的工程代价 |
| Q1.3 论证结构 | 5/10 | 抓到路线图但未接 Q0.2 洞察；学会"数段落看论证结构" |

**全 Round 主题**：直觉敏锐 → 但精度需要用方法论锁住。

**已建立的方法论库**（持续累积）：
1. **想反例法**（Q0.2 + Q1.1）：判断两个概念是否独立，找 (A 而非 B) 和 (B 而非 A) 两个反例
2. **Q/K/V 框架**（Q1.2）：分析 attention 时先问"谁是 Q、K、V"
3. **数段落法**（Q1.3）：读 Intro 时数段落数和功能分布，独立段 = 独立贡献维度

**下一节 (02-related-work.md) 重点关注**：related work 通常会展开 Para 1 里点名的对手（HunyuanVideo / Mochi / CogVideoX）—— 留意 Wan 用什么维度比较自己和它们，这能反向验证 Wan 自认的"差异化卖点"是哪几个。

---

[← 返回论文 README](../README.md) ｜ [← 上一节](00-abstract.md) ｜ [下一节 →](02-related-work.md)
