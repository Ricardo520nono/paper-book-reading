[← 返回 Paper-Book-Reading 主页](../../README.md)

# Uni-WAM 调研库

> **调研目标**：为翔哥的 Uni-WAM proposal 广撒网调研所有"具身视频生成 WM"，按 a/b/c/d 分类填表，找漏。**目标会议：CoRL**。

> **Proposal 全文**：[Uni-WAM-proposal.pdf](proposal/Uni-WAM-proposal.pdf) | [文本版](proposal/Uni-WAM-proposal.txt)

---

## 🎯 任务来源（翔哥微信原文）

> 调研目前所有的 action-conditioned world model，比如 ctrl-world，cosmos-predict2.5 也算（因为它提供了 action-conditioned 的 finetune pipeline），大概需要做的流程就是：
> 1. 在飞书里列一个列表
> 2. 调研所有的流行的具身视频生成 WM
> 3. 对于这些 WM 分类
> 4. AC-WM 梦飞那边只找到 cosmos-predict 和 motus，你可以广撒网筛选一下，check 一下有没有漏的

---

## 📐 分类定义

| 分类 | 定义 | 例子 |
|---|---|---|
| **a** | 原本就是 AC-WM | Ctrl-World, Motus |
| **b** | 原本是 WM，**但开源代码提供 AC-WM finetune pipeline** | Cosmos-Predict 2.5 |
| **c** | 原本是 WM，开源但**没**提供 AC-WM | (待调研) |
| **d** | 整个就**没开源** | Sora, Kling（如果属于） |
| **e** | **不是 AC-WM**（Ricardo 加的兜底类） | 调研时**不重点关注**，只标记排除 |

### 范围

- **核心**：具身智能 / 机器人操作场景的视频生成 WM
- **包含**：有 robotics 应用的视频 WM（如 Cosmos，虽然主打 driving 但也有 robot 用例）
- **次要**：纯通用视频生成模型（Sora 类）—— 只标 e 类
- **可能排除**：纯自动驾驶 WM 且无 robot 用法（GAIA / DriveDreamer 系列）—— 待确认

---

## 📋 调研主表

> 沿用翔哥的列：分类 / 工作名称 / 做了什么 ✅ / 没做到 ❌

### 翔哥已填（WM Benchmark 类，参考用）

> 每篇做 focused 阅读（只过翔哥点的 ✅/❌），笔记在 `notes/0X-名字/notes.md`

| # | 工作 | 做了 ✅ | 没做到 ❌ | 笔记 |
|---|---|---|---|---|
| 01 | EWMBench (2025/05) | 三维评估 + HSD/NDTW/DYN 指标 + Agibot 上 7 个模型 | 仅来自训练分布；从未测 random / sub-optimal action | [✅ 完成](notes/01-EWMBench/notes.md) |
| 02 | WorldArena (2026/02) | 感知质量(16) + 功能效用(3) + EWMScore | "可控性"是 T2V 不是数值 action；功能评估仍 ID action | [待读](notes/02-WorldArena/) |
| 03 | MIND (2026/02) | 记忆一致性 + action 控制 + 双视角 + action space 泛化 | 不是 robotics 场景 | [待读](notes/03-MIND/) |
| 04 | ACT-Bench (2024/12) | IEC + TA 指标 | 仅自动驾驶 | [待读](notes/04-ACT-Bench/) |
| 05 | RoboWM-Bench (2026/04) | 视频 → action → sim 执行 + 任务完成率 | 仅评测 WM 生成视频；没考虑数值 AC-WM | [待读](notes/05-RoboWM-Bench/) |
| 06 | WorldGym (2025/05) | 测了 OOD language + OOD initial image | **没考虑 OOD action** | [待读](notes/06-WorldGym/) |

### Ricardo 待填（AC-WM 模型本身，按分类排序）

#### 分类 a：原本就是 AC-WM

| 工作 | 做了 ✅ | 没做到 ❌ | 链接 | 备注 |
|---|---|---|---|---|
| Ctrl-World (2025/10) | _(待填)_ | _(待填)_ | _(待补)_ | 翔哥已点名 |
| Motus | _(待填)_ | _(待填)_ | arXiv 2512.13030 | 梦飞已找到，Wan paper 待读清单也提过 |
| _(broad search 后补)_ | | | | |

#### 分类 b：WM 但开源提供 AC-WM finetune

| 工作 | 做了 ✅ | 没做到 ❌ | 链接 | 备注 |
|---|---|---|---|---|
| Cosmos-Predict 2.5 | _(待填)_ | _(待填)_ | _(待补)_ | 翔哥点名 + 梦飞已找到 |
| _(broad search 后补)_ | | | | |

#### 分类 c：WM 开源但无 AC-WM

| 工作 | 做了 ✅ | 没做到 ❌ | 链接 | 备注 |
|---|---|---|---|---|
| _(待调研)_ | | | | |

#### 分类 d：整个没开源

| 工作 | 做了 ✅ | 没做到 ❌ | 链接 | 备注 |
|---|---|---|---|---|
| _(待调研)_ | | | | |

#### 分类 e：非 AC-WM（不重点）

| 工作 | 为什么排除 | 链接 | 备注 |
|---|---|---|---|
| _(扫到 false positive 时记录)_ | | | |

---

## 🔍 调研约定

### 阅读深度（和 Wan paper 不一样）

```
Wan paper 流程:                 Uni-WAM 调研流程:
─────────────────────────       ────────────────────────────
完整批读 + 拷打                  Focused 阅读
原文逐段 + 内嵌批注              只看与 AC-WM 调研相关的部分
9 个概念词典深度展开             不深挖底层架构
适合: 深度学习一篇核心 paper     适合: 广撒网, 横向比较
```

### 每篇候选要提取的信息

1. **分类**（a / b / c / d / e）—— 核心
2. **是否在具身/机器人场景训练 / 评估**
3. **action 是怎么注入的**（numeric joint / 6D pose / latent action / language / image）
4. **训练数据来源**（是不是只有 expert demo？）
5. **是否提供 AC finetune pipeline**（如果是 b 类）
6. **是否在 OOD action 上有任何测试 / claim**

### 候选获取来源

- arXiv 关键词搜索: `action-conditioned world model`, `world model robot`, `embodied video generation`
- 引用图谱: 从 Ctrl-World / Motus / Cosmos-Predict 的 related work 节顺藤摸瓜
- GitHub topic: `world-model`, `action-conditioned`, `embodied-ai`
- 相关 benchmark paper 的对比表（EWMBench, WorldArena, MIND）扫漏
- Wan paper 待读清单（已经有几个 VLA-WM 候选）

### 工作流程（每个候选）

```
1. 找到 candidate (arxiv / paper page / github)
2. 在 papers/ 放 PDF (或者只保存链接)
3. 在 notes/ 写 focused 笔记 (1-2 页 markdown)
4. 决定分类 a/b/c/d/e
5. 在 README 主表填一行
6. push
```

---

## 📚 已知候选 backlog

> 持续更新。每读一篇 paper，把它 related work 里发现的新候选**只筛 2023+**（太老不看）加进来，并标注来源。

### 高优先级 / 翔哥点名 + 梦飞先发现

**a 类候选**:
- ✅ Ctrl-World (2025/10) ⭐ 翔哥点名 · [已读完](notes/Ctrl-World.md)（待写浓缩）
- Motus (arXiv 2512.13030) ⭐ 梦飞已找到，Wan 待读清单
- Cosmos Policy (Wan 待读清单提过)
- DreamGen (NVIDIA, 与 Cosmos 系)

**b 类候选**:
- Cosmos-Predict 2.5 ⭐ 翔哥点名 + 梦飞已找到
- DreamZero (NVIDIA, Wan 待读清单, 14B World Action Model)

---

### 通过 Ctrl-World §2.2 Related Work 发现的新候选（2023+，筛掉太老）

> 来源：[`VLA-WM/[arXiv 2510.10125] Ctrl-World/sections/02-related-work-acwm.md`](../%5BarXiv%202510.10125%5D%20Ctrl-World/sections/02-related-work-acwm.md)

**🌟 最高优先级**（Ctrl-World method 直接源头）:
- **Zhu et al. 2024** —— **frame-level action conditioning 起源** ⭐ Ctrl-World 的 action 注入机制就来自这里
- **He et al. 2025** —— frame-wise cross-attention（Ctrl-World 也直接复用）

**Diffusion-based AC-WM**（近期热门方向）:
- Quevedo et al. 2025
- Chen et al. 2024
- Gao et al. 2025
- Ren et al. 2025
- Hafner et al. 2025 —— DreamerV3 latest（前几代 V1/V2 是 2019/2020 太老不读，但 V3 2025 值得读）

**其他 AC-WM 候选**（2023+）:
- Daydreamer (Wu 2023)
- Wu 2024（同作者后续工作？待查）
- Yang 2023（可能是 UniSim，待确认）
- Huang 2025

**Ctrl-World 也引用、但太老不读**（记录在此防止后面又被引出来重复评估）:
- ❌ Nagabandi 2020（低维 state space MPC）
- ❌ DreamerV1 (Hafner 2019) / DreamerV2 (Hafner 2020)
- ❌ TD-MPC (Hansen 2022)
- ❌ Oh 2015 ATARI predictive
- ❌ Finn & Levine 2017, Ebert 2018, Xie 2019, Dasari 2019

---

### c/d 类候选 / 待判断

- Genie 2 / Genie 3 (DeepMind, 多大概率非 AC，但要查清楚)
- Pandora
- 1X World Model
- WHALE (DM)
- LDA-1B (北大 EPIC, Wan 待读清单)
- VAW (Wan 待读清单, arXiv 2602.12063)
- WoVR (Wan 待读清单, arXiv 2602.13977)
- Interactive World Sim (Columbia+TRI, Wan 待读清单)
- HumanWorld
- VideoWorld
- WorldDreamer
- IWM (Instruction-driven World Model)

### 自动驾驶 WM（待定 —— 看是否有 robot 用例 / generalize）

- GAIA-1 / GAIA-2 (Wayve)
- DriveDreamer / DriveDreamer-2
- Vista
- ADriver-I

### 通用 video gen / 大概率 e 类

- Sora
- Wan 2.1 / Wan 2.2 (没 numeric AC 直接接口，但 actcon 变体属于 b)
- HunyuanVideo
- Open-Sora / Open-Sora Plan
- Mochi
- Kling
- Runway Gen-3

---

## 🔄 进度

- [x] 建立调研库骨架
- [ ] 等翔哥/Ricardo 进一步指示
- [ ] 广撒网搜索（arXiv + Google Scholar + GitHub）
- [ ] 候选去重 + 优先级排序
- [ ] 逐篇 focused 阅读 + 填表
- [ ] 漏检 self-check（对照 EWMBench/WorldArena/MIND 对比表）
- [ ] 最终输出（飞书 + 这里同步）

---

## 📁 目录结构

```
Uni-WAM-调研/
├── README.md                    ← 本文件，主索引 + 主表
├── proposal/
│   ├── Uni-WAM-proposal.pdf    ← 翔哥的原始 proposal
│   └── Uni-WAM-proposal.txt    ← 抽取的纯文本版（方便引用）
├── papers/                      ← 候选 paper PDF 暂存
│   └── README.md
└── notes/                       ← 每篇 paper 的 focused 笔记
    └── README.md
```
