[← 返回论文 README](../README.md) ｜ [下一节 →](01-introduction.md)

# 00 · Abstract（摘要）

> **来源**：[Wan: Open and Advanced Large-Scale Video Generative Models](https://arxiv.org/abs/2503.20314), arXiv 2503.20314, p1
>
> **一句话定位**：本文是 Wan（阿里通义视频生成基座）的**技术报告**，宣告 Wan 系列在性能、覆盖度、效率、开放性四个维度上同时做到第一梯队。

---

## 📌 预览

Wan 是一个**完整的开源视频生成"套件"**（不是单一模型），核心论点四条：

1. **Leading Performance**：14B 模型超越同期开源 + 商业模型，并展示 video generation 的 scaling laws
2. **Comprehensiveness**：提供 1.3B / 14B 两档 + 8 个下游任务（T2V / I2V / 视频编辑 / T2I / 个性化 / 相机控制 / 实时 / 音频）
3. **Consumer-Grade Efficiency**：1.3B 模型只要 8.19GB VRAM，消费级 GPU 友好
4. **Openness**：代码 + 全部模型权重开源

技术底座：**DiT（Diffusion Transformer）+ 自研时空 VAE**。

---

## 📄 原文 + 批注

### 第 1 句 · 立场宣告

> "This report presents Wan, a comprehensive and open suite of video foundation models designed to push the boundaries of video generation."

💡 **关键词解读**：
- **a comprehensive ... suite**（套件，不是单一模型）—— 暗示 Wan 不是一个 model，而是**一个家族**：包含不同尺寸 + 不同任务的模型
- **video foundation models**（视频基础模型）—— 这是个"自我定位"，类比 LLM 里的 "foundation model"。意味着 Wan 想做视频界的 GPT/Llama，是底座，别人在它之上做 finetune
- **open**（开源）—— 这不是技术声明，是**战略声明**。当时的对手（Sora, Runway, Kling）都是闭源 SaaS

⚠️ **小白别误读**：「open suite」≠ 一个 GitHub 仓库丢出来。它意味着代码 + 数据策划方法 + 模型权重 + 推理代码 + 评测工具全开源。

---

### 第 2~3 句 · 技术路线

> "Built upon the mainstream diffusion transformer paradigm, Wan achieves significant advancements in generative capabilities through a series of innovations, including our novel spatio-temporal variational autoencoder (VAE), scalable pre-training strategies, large-scale data curation, and automated evaluation metrics."

💡 **「mainstream diffusion transformer paradigm」是什么意思？**

视频生成在 2024-2025 年基本统一在两个范式中，Wan 选了主流的那个：

```
Pixel-space Diffusion (传统)         Latent Diffusion + DiT (主流)
─────────────────────────────────────────────────────────────────
直接在 RGB 像素上做 denoise          先用 VAE 把视频压缩到 latent space
                                       再在 latent 上做 denoise
计算贵到爆炸                          用 Transformer 替代 U-Net
难以 scale 到长视频 / 高分辨率       天然适合 scaling laws

代表：早期 Imagen Video              代表：Sora, Wan, HunyuanVideo, CogVideoX
```

💡 **Wan 的四大创新（这句话是全文骨架）**：

| 创新 | 对应论文章节 | 解决的问题 |
|---|---|---|
| **① 时空 VAE**（spatio-temporal VAE） | §4.1 | 把视频高效压成 latent，不丢动态信息 |
| **② Scalable 预训练** | §4.2 + §4.3 | 让模型能稳定地从小到大 scale up |
| **③ 大规模数据策划** | §3 | 数据质量 = 视频生成质量天花板 |
| **④ 自动化评测** | §4.6 | 不用每次都靠人工评，自建 Wan-Bench |

⚠️ **必记口诀**：「VAE + 训练 + 数据 + 评测」—— 这四块是 Wan 的技术全景图，**对应论文 4 大章节**。

---

### Key Feature 1: Leading Performance

> "The 14B model of Wan, trained on a vast dataset comprising billions of images and videos, demonstrates the scaling laws of video generation with respect to both data and model size. It consistently outperforms the existing open-source models as well as state-of-the-art commercial solutions across multiple internal and external benchmarks, demonstrating a clear and significant performance superiority."

💡 **三个信号**：

1. **数据规模**：billions（十亿级）图像 + 视频。这是工业级训练量，学术界基本玩不起 —— 暗示 Alibaba 砸了大钱
2. **scaling laws of video generation**：这是个**技术声明** —— Wan 团队亲自验证了"模型越大、数据越多，视频生成质量单调上升"。这件事在 LLM 是定论，在视频生成 2025 年还需要重新验证
3. **outperforms ... open-source ... commercial**：同时打开源（HunyuanVideo, Mochi）和商业（Sora, Runway, Kling），这是**最强 flex**

🤔 **批判性思考**：什么叫 "across multiple benchmarks"？哪些 benchmark？是自建的 Wan-Bench 还是公开 benchmark？这关系到结果的可信度 —— 我们到 §4.6 §4.7 时要特别留意 evaluation 的公平性。

---

### Key Feature 2: Comprehensiveness

> "Wan offers two capable models, i.e., 1.3B and 14B parameters, for efficiency and effectiveness respectively. It also covers multiple downstream applications, including image-to-video, instruction-guided video editing, and personal video generation, encompassing up to eight tasks. Meanwhile, Wan is the first model that can generate visual text in both Chinese and English, significantly enhancing its practical value."

💡 **「双模型策略」**：
- **1.3B → 走效率（efficiency）**：消费级 GPU 能跑，给学术界 / 个人开发者用
- **14B → 走效果（effectiveness）**：性能旗舰，对标商业服务

这个设计哲学**贯穿到了 Wan 2.2** —— 你已经知道 Wan 2.2 也是双线策略（A14B MoE 走效果，TI2V-5B 走效率）。所以**「双模型」不是 2.2 的新发明，是 Wan 系列的一贯哲学**。

💡 **「8 个下游任务」是哪些？** 论文 §5 章会一一展开：

```
1. Text-to-Video (T2V)              ── 主力
2. Image-to-Video (I2V)             ── 你用过这个
3. Instruction-guided Video Editing ── 文字指令改视频
4. Text-to-Image (T2I)              ── 副产物
5. Video Personalization            ── 拿用户上传内容生成定制视频
6. Camera Motion Controllability    ── 控制运镜
7. Real-time Video Generation       ── 流式 / 低延迟生成
8. Audio Generation                 ── 给视频生成配音
```

💡 **「first to do bilingual visual text」是什么意思？**

视觉文本（visual text）= 视频画面里出现的文字。比如让模型生成「一个霓虹招牌写着'珍珠奶茶'」，要求画面里真的出现可读的中文。这件事 GPT-4o / Sora 在 2024 年都搞不定中文。Wan 是**第一个**能同时生成中文和英文画面文字的视频模型。

⚠️ **小白容易误解**：visual text ≠ 字幕。字幕是后期叠加的，visual text 是模型直接画在画面里的（霓虹灯/牌匾/海报），技术难度天差地别。

---

### Key Feature 3: Consumer-Grade Efficiency

> "The 1.3B model demonstrates exceptional resource efficiency, requiring only 8.19 GB VRAM, making it compatible with a wide range of consumer-grade GPUs. It also exhibits superior performance compared to larger open-source models, showcasing remarkable efficiency for text-to-video."

💡 **8.19 GB VRAM 意味着什么**：
- RTX 3060 (12GB) ✅
- RTX 3070 (8GB) ⚠️ 临界点
- RTX 4060 Ti (16GB) ✅
- M2 MacBook Air (统一内存) ✅
- 集成显卡 ❌

也就是说，**几乎任何人都能在自己电脑上跑 Wan 1.3B 做 T2V**。这是 video generation 史上的一个里程碑 —— 之前 HunyuanVideo 13B 需要 60GB+，Mochi 10B 需要 60GB+。

💡 **"superior performance compared to larger open-source models"** —— Wan 1.3B 在打的不是 14B 同量级，而是**比 1.3B 大但比 14B 小**的中量级开源模型（比如 OpenSora）。1.3B 干翻量级更大的对手，这才是 efficiency 的本意。

---

### Key Feature 4: Openness

> "We open-source the entire series of Wan, including source code and all models, with the goal of fostering the growth of the video generation community. This openness seeks to significantly expand the creative possibilities of video production in the industry and provide academia with high-quality video foundation models."

💡 **开源策略的两个目标人群**：
1. **industry**：商业用户拿去做产品（受 Apache 2.0 license 约束）
2. **academia**：学术界拿去做研究 baseline / finetune 基础

这件事的对照：Sora 至今闭源、Runway 闭源、Kling 闭源、Pika 闭源。Wan 选择开源，**部分原因是 Alibaba 想用开源生态对冲闭源服务的优势**（类似 Llama 之于 GPT-4）。

---

### Figure 1 · 性能对比图

> Figure 1: Comparison of Wan with state-of-the-art open-source and closed-source models. Following both benchmark and human evaluations, Wan consistently demonstrated superior results.

💡 **图表怎么读**：

**左半边 · Wan-Bench Score（自动化评测）**

```
模型              得分
Wan2.1-14B       0.72  ⭐ 最高
Mochi            0.64
Hunyuan          0.67
CN-TopA          0.69
CN-TopB          0.69
CN-TopC          0.70
Sora             0.69
```

⚠️ **关键观察**：Wan 2.1-14B 比 Sora 高 0.03。这是个**自评 benchmark**（Wan-Bench 是 Wan 团队自建的），所以要打个折扣 —— 但 0.03 的领先是稳定的。

**右半边 · Human Preference Win Rate（人类评测胜率）**

人类盲评对比 Wan vs 其他模型时，Wan 的胜率：

```
对手        Wan 输      平局    Wan 赢
CN-TopC     24%         3%      73%   ← 大胜
Runway      15%         3%      82%   ← 暴打 Runway
CN-TopB     24%         8%      68%   
CN-TopA     25%         6%      69%
```

💡 **CN-TopA/B/C 是谁**？论文匿名了，但根据"中国 Top 商业视频生成模型"的语境，大概率是可灵 / 即梦 / Vidu / 海螺等里面的三个。

⚠️ **Win rate 比 score 更可信**：因为 score 在自家 benchmark 上，而 win rate 是人类盲评，更难造假。Wan 在人类评测上对所有对手都是压倒性优势（68%~82%），这是文章最有说服力的数字。

---

## 💡 Section 总结

### 核心信息速查

| 维度 | 内容 |
|---|---|
| 模型类型 | Video Foundation Model（视频基础模型套件） |
| 技术范式 | Diffusion Transformer (DiT) + 自研时空 VAE |
| 模型尺寸 | 1.3B（消费级）+ 14B（旗舰） |
| 训练数据 | Billions 级图像 + 视频 |
| 下游任务 | 8 个（T2V/I2V/编辑/T2I/个性化/相机/实时/音频） |
| 1.3B VRAM | 8.19 GB（4060 Ti / 3060 都能跑） |
| 14B 对标 | Sora / Runway / 国内商业 Top |
| 评测 | Wan-Bench（自建）+ 人类盲评 |
| 开源 | 代码 + 全部模型权重 |
| 独特卖点 | **首个支持中英双语视觉文本的视频模型** |

### 核心洞察

1. **Wan 不是模型，是套件**：1.3B + 14B 双线策略 + 8 个下游任务，这种"全家桶"路线是 foundation model 的标准打法

2. **DiT + Latent Diffusion 是 2024 起视频生成的事实标准**：Wan 没在范式上革命，而是在每个组件上做精

3. **开源是战略不是慈善**：Alibaba 用 Wan 对标 Sora/Runway 的闭源服务，类似 Meta 用 Llama 对标 GPT-4

4. **8.19GB VRAM 是 framing 巧妙的数字**：1.3B FP16 大概要 2.6GB，剩下的是 VAE + 推理 buffer + KV cache。这是经过推理优化（Section 4.4）的数字，不是裸跑

5. **最强 flex 是 human win rate**：自动化评测可以刷，人类盲评不能 —— Wan 在人类评测上对所有对手 60%+ 胜率，这是最有信服力的证据

---

## 🤔 我的 Follow-up 问题

读完 Abstract 我留下的问号（带到后面章节回头看是否解决）：

1. **Wan-Bench 具体测什么？** 摘要只说"自动化评测"，但具体维度是哪些？这关系到 0.72 这个分数的可信度。→ §4.6
2. **「scaling laws of video generation」具体长什么样？** 是 loss vs FLOPs 的对数线性？还是别的？→ §4.2
3. **8 个下游任务里，哪些是从 T2V 主模型 finetune 来的，哪些是独立训练的？** → §5
4. **Wan 1.3B 真的在 RTX 3060 上能跑吗？** 还是说需要某种特殊量化？→ §4.4

---

## 🔥 拷打记录

_(待填充 — Claude 会在这一节问我 3 个问题，我用自己的话回答，然后把 Q&A 整理回写)_

---

[← 返回论文 README](../README.md) ｜ [下一节 →](01-introduction.md)
