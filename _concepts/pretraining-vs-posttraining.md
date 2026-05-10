[← 返回概念词典](README.md)

# Pre-training vs Post-training（预训练 vs 后训练）

## 🎯 一句话定义

| | 一句话 |
|---|---|
| **Pre-training（预训练）** | 用**海量、多样、便宜**的数据让模型学到**广博的基础能力** |
| **Post-training（后训练）** | 用**少量、高质量、贵**的数据让模型**对齐到具体任务/品味** |

**类比**：上大学读通识 → 读研深造。

---

## 📊 对比表

| 维度 | Pre-training | Post-training |
|---|---|---|
| **目标** | 学通识 / 基础能力 | 对齐到具体应用 / 提升品味 |
| **数据规模** | 海量（trillion 级） | 小（千~百万级） |
| **数据质量** | 不太挑（噪声多） | 高度筛选 |
| **数据成本** | 便宜（爬 / 抓） | 贵（人工标注） |
| **算力占比** | 90%+ | 5-10% |
| **时长** | 几个月 | 几天-几周 |
| **架构是否变** | — | **不变**（直接接续 pre-trained） |
| **优化器是否变** | — | **不变** |
| **关键技术** | 自监督 / 大模型 scaling | SFT / RLHF / DPO / curriculum |

---

## 🤖 LLM 中的实例（GPT 范式）

```
Pre-training:                          Post-training:
─────────────────────                   ─────────────────────
任务：预测下一个 token                   任务：让模型听话 + 安全 + 优雅
数据：互联网海量文本（10T+ tokens）       数据：人类筛选的指令对话（10K-1M）
计算：上千 GPU、几个月                   100-1000 GPU、几天到几周
产出：GPT-base（什么都懂但不听话）       GPT-Chat（听话且优雅）
```

**没有 post-training 的 GPT-2**：知识有但乱说话
**经过 RLHF post-training 的 ChatGPT**：变成可用的产品

---

## 🎬 视频生成中的实例（Wan）

```
Pre-training (Wan §4.2.2):             Post-training (Wan §4.2.3):
─────────────────────                   ─────────────────────
3 阶段渐进：                             架构和优化器不变
  256px image                          直接从 pre-trained checkpoint 初始化
  → 480px joint (image+video)          在 §3.2 高质量数据集上继续训
  → 720px joint                        分辨率：480px + 720px

数据：billions of images + videos       数据：精选高质量视频（人工筛选）
                                        数量小但每条质量高

学到：通用视觉建模能力                   学到：电影感 / 高美学 / 高保真
```

---

## 🤔 为什么 2 阶段范式那么流行？

### 1. 数据效率
- Pre-training 数据：便宜、海量、可爬
- Post-training 数据：贵、精挑、需人工
- → **用便宜的让模型学通识，用贵的让模型学品味，总成本最低**

### 2. 计算效率
- Pre-training: 90% 算力，几个月
- Post-training: 5-10% 算力，几天-几周
- → 一个 pre-trained 模型可以**多次 post-train** 出不同产品（Llama 3-base → Llama 3-Instruct, Llama 3-Code, ...）

### 3. 模块化复用

```
   Pre-trained 基座
         │
         ├── Post-train A: 高美学视频
         ├── Post-train B: 卡通风格
         ├── Post-train C: 行业特化
         └── Post-train D: 安全过滤
   
   1 个基座 + N 套 post-training = N 个产品
```

---

## 🔑 关键观察：架构 / 优化器 / 数据格式通常都不变

> Wan paper §4.2.3 第一句：
> "we **maintain the same model architecture and optimizer configuration** from the pre-training stage"

**不只 Wan，几乎所有现代模型都这样**：
- Post-training 用**同一个架构** + **同一个优化器**
- **只换数据 + 调整 LR**
- 直接从 pre-trained checkpoint 初始化继续训

**好处**：
1. 不浪费已学知识（continuity）
2. 工程简单（同一套 pipeline）
3. 稳定（不引入新的架构 bug）

---

## ⚠️ 关键认知：质量 > 数量（在 post-training 阶段）

LLM 实证：
- 1000 条人工筛选的高质量对话 + RLHF **>** 1M 条网络对话直接训练

视频生成同理：
- Wan post-training 数据虽小，但每条经过严格美学/质量筛选
- 模型学到"**好视频长什么样**"，比 pre-training 的"**视频是什么**"更精细

---

## 🔗 在我读的论文里

| Paper | Section | 用法 |
|---|---|---|
| Wan (2503.20314) | §4.2.2 / §4.2.3 | 3-stage pre-training (256→480→720) + 同架构同优化器的 post-training |

---

## 🔗 相关概念

- [Training vs Inference](training-vs-inference.md) — pre/post-training 都是训练，推理是另一回事
- [Diffusion 基础](diffusion-basics.md)
- _(待补) SFT / RLHF / DPO 等 post-training 算法_
- _(待补) Curriculum Learning（渐进式难度）_
