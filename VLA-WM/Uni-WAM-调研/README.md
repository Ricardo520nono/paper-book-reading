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
| **Ctrl-World (2025/10)** | • 提出 frame-level cross-attention 注入 action（Cartesian 6D pose + 1D gripper）<br>• 多视角联合预测（3 cam, 含 wrist-view）+ pose-conditioned memory 防漂移<br>• 验证 WM 作为 policy ranker: 3 个 SOTA VLA × 7 task, 排名对齐真机（y=0.87x-0.04）<br>• 验证 WM 作为合成数据源: π0.5 在 novel object/instruction 上 +44.7% | • 训练数据仅 DROID 真机遥操作（76k success + 19k failure 但仍在 human-attempted 范围内）<br>• 测的 3 个 policy 都是 expert-quality VLA, 没测 sub-optimal / 早期 RL checkpoint 的 policy<br>• 没直接喂 off-expert action（counterfactual / random-feasible / adversarial）评估 WM<br>• 合成数据只筛 successful trajectory, 跳过 failure 分布 | [📄 summary](../%5BarXiv%202510.10125%5D%20Ctrl-World/sections/summary.md) | 翔哥已点名<br>a 类（也提供 b 类 SVD → AC-WM finetune pipeline）|
| **Dreamer 4 (2025/09)** | • AC-WM（Causal Tokenizer + 2D 时空 Transformer）+ Agent（policy/value head）联合系统<br>• Action 注入方式: action 作为 token 与 latent / time / step-size token 拼接喂入 dynamics（≠ cross-attention）<br>• 3-Phase pipeline: unlabeled video pretrain → labeled finetune → imagination RL<br>• shortcut forcing 目标蒸馏 flow matching（64 → 4 步）<br>• 首次纯 offline 在 Minecraft 拿到 diamond, 比 OpenAI VPT 用 100× 更少数据 | • 场景是 Minecraft game, 不是 robot manipulation<br>• Action 是 121 类离散键鼠 + binary, 不是数值 robot action<br>• 训练数据来自 contractor gameplay（human expert demo）, 未覆盖 off-expert action 分布<br>• 无 off-expert / OOD action 评估设计 | [📄 summary](../%5BarXiv%202509.24527%5D%20Dreamer%204/sections/summary.md) | Hafner 团队 · DeepMind<br>场景偏离（Minecraft）|
| **AdaWorld (2025/03)** | • AC-WM（autoregressive video gen + latent action conditioning）<br>• Action 注入方式: 从 video 帧间用 information bottleneck autoencoder unsupervised 抽 latent action, 作为 condition 输入 WM（不需 action label）<br>• Latent Action Autoencoder + Action-Aware Pretraining 两步范式<br>• 跨 Habitat / Minecraft / DMLab / nuScenes 4 个 unseen 环境通用<br>• Few-shot adaptation: 800 步 / 30 秒 / 单 GPU 适配新环境 | • 场景是 game / 导航 / 驾驶 mixed, 不是 robot manipulation<br>• Action 是 learned latent vector（连续, unsupervised）, 不是数值 robot action<br>• 训练数据是 unlabeled video, 没有显式 action 分布概念, 不存在 off-expert 测试设计<br>• 关注跨环境泛化（visual / dynamics OOD）, 不关注 OOD action | [📄 summary](../%5BarXiv%202503.18938%5D%20AdaWorld/sections/summary.md) | latent action 路线<br>不需 action label |
| **WorldGym / WPE (2025/06)** | • AC-WM（autoregressive diffusion video gen）<br>• Action 注入方式: chunk-wise bidirectional attention + causal cross-chunk（16 帧 chunk）<br>• 用 VLM（GPT-4o）作 reward model 自动判 success<br>• 测 3 个 VLA（RT-1-X / Octo / OpenVLA）在 Bridge / Open X-Embodiment 上, 真机 vs WM 成功率相关性 r=0.78<br>• 能改 image / 改 language 测 OOD generalization | • 没考虑 OOD action（OOD 维度只在 image 和 language）<br>• 训练数据是真机 expert demo（Bridge）, 未覆盖 off-expert action<br>• 方法贡献偏弱, 主要是 evaluation framework 整合 | [📄 summary](../%5BarXiv%202506.00613%5D%20WPE/sections/summary.md) | 可能与翔哥已填 "WorldGym (2025/05)" 是同一篇<br>Ctrl-World §5.2 baseline |
| **EnerVerse-AC (2025/05)** ⭐ | • AC-WM（多视角 robot diffusion model, 真具身 manipulation）<br>• Action 注入方式: **Multi-Level** = Spatial-Aware Pose RGB + Delta Action Cross-Attention + Gripper Magnitude RGB（多层级多管齐下）<br>• 5 相机联合预测（含 dynamic wrist），长时 chunk-wise inference<br>• 训练数据含**人工 augmented failure trajectories**（关键创新）<br>• 4 task × 3 training step 上, EVAC 评估和真机评估**逐 task + 逐 step 完全对齐** | • 训练数据仍是 expert + human-attempted failure（不是 systematic off-expert）<br>• 评估的 policy 都是公司内部 VLA-MLLMs, 没测 sub-optimal / 早期 RL checkpoint<br>• 没有 counterfactual / random-feasible / adversarial 类型的 OOD action 测试<br>• 没有 OOD action 维度的指标设计 | [📄 summary](../%5BarXiv%202505.09723%5D%20EnerVerse-AC/sections/summary.md) | AgiBot + SJTU + CUHK<br>Ctrl-World §5.2 baseline<br>真具身 manipulation ⭐ |
| **Genie Envisioner (2025/08)** ⭐⭐ | • 4-in-1 统一平台: **GE-Base**（语言条件 video diffusion, LTX-Video 2B based）+ **GE-Act**（policy, flow-matching decoder）+ **GE-Sim**（action-conditioned simulator）+ **EWMBench**（benchmark）<br>• Action 注入方式: 在 **GE-Sim** 衍生时注入（GE-Base 本身只接 instruction）<br>• 训练数据: **AgiBot-World-Beta 1M+ trajectories, 3000+ hours video**（数据规模 ×10 vs EVAC）<br>• EnerVerse-AC 的统一框架升级版（同团队）<br>• 全平台开源（model / checkpoint / code） | • Paper 主要做"基础设施"，没系统对照 off-expert action 评估<br>• EWMBench 是自评 benchmark（GE 同系列），不主动暴露 WM 弱点<br>• Policy 测试都在 AgiBot 内部 task 范围内, 没测 sub-optimal policy<br>• 没 OOD action 评估的 metric 设计 | [📄 summary](../%5BarXiv%202508.05635%5D%20Genie%20Envisioner/sections/summary.md) | AgiBot + LV-NUS + BUAA<br>EnerVerse-AC 升级版<br>**Uni-WAM 直接竞品** ⭐⭐ |
| Motus | _(待填, 跳过 - 翔哥已知)_ | _(跳过)_ | arXiv 2512.13030 | 梦飞已找到<br>翔哥已知, 跳过 |

#### 分类 b：WM 但开源提供 AC-WM finetune

| 工作 | 做了 ✅ | 没做到 ❌ | 链接 | 备注 |
|---|---|---|---|---|
| Cosmos-Predict 2.5 | _(待填, 跳过 - 翔哥已知)_ | _(跳过)_ | _(待补)_ | 翔哥已知, 跳过 |
| DreamZero | _(待调研)_ | _(待调研)_ | NVIDIA, 14B World Action Model | Wan 待读清单 |

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

### 通过 Ctrl-World §2.2 Related Work + 实验对照基线 发现的新候选（2023+，筛掉太老）

> 来源：[`VLA-WM/[arXiv 2510.10125] Ctrl-World/sections/02-related-work-acwm.md`](../%5BarXiv%202510.10125%5D%20Ctrl-World/sections/02-related-work-acwm.md)

**🌟 最高优先级**（Ctrl-World method 直接源头）:

| Paper | 作者 / 年 | arXiv | 一句话 / 价值 |
|---|---|---|---|
| **IRASim** | Zhu et al. 2024 | [2406.14540](https://arxiv.org/abs/2406.14540) | **frame-level action conditioning 起源** ⭐ Ctrl-World 直接继承 |
| **Pre-trained Video Generative Models as World Simulators** | He et al. 2025 | [2502.07825](https://arxiv.org/abs/2502.07825) | frame-wise cross-attention（Ctrl-World 复用）|

**Diffusion-based AC-WM**（近期热门方向）:

| Paper | 作者 / 年 | arXiv | 一句话 |
|---|---|---|---|
| **WPE**（World-model-based Policy Evaluation）| Quevedo et al. 2025 | [2506.00613](https://arxiv.org/abs/2506.00613) | Ctrl-World §5.2 的对照 baseline，AC-WM policy eval |
| **Diffusion Forcing** | Chen et al. 2024 | NeurIPS 2024 | next-token + full-sequence diffusion 范式 |
| **AdaWorld** | Gao et al. 2025 | [2503.18938](https://arxiv.org/abs/2503.18938) | latent action AC-WM |
| **DreamerV3 latest** | Hafner et al. 2025 | [2509.24527](https://arxiv.org/abs/2509.24527) | DreamerV3 最新版（V1/V2 太老跳过）|
| ⚠️ **Cosmos-Drive-Dreams** | Ren et al. 2025 | [2506.09042](https://arxiv.org/abs/2506.09042) | **driving 不是 manipulation** —— 优先级降低 |

**其他 AC-WM 候选**（2023+）:

| Paper | 作者 / 年 | 出处 | 一句话 |
|---|---|---|---|
| **Daydreamer** | Wu et al. 2023 | CoRL 2023 | 真机 robot 学习的 WM |
| **iVideoGPT** | Wu et al. 2024 | NeurIPS 2024 | autoregressive interactive video WM |
| **UniSim** | Yang et al. 2023 | [2310.06114](https://arxiv.org/abs/2310.06114) | 通用 real-world simulator |
| **Particleformer** | Huang et al. 2025 | [2506.23126](https://arxiv.org/abs/2506.23126) | 3D point cloud WM for manipulation |

**通过 Ctrl-World §5.2 实验对照基线 发现**:

| Paper | 作者 / 年 | arXiv | 一句话 |
|---|---|---|---|
| **EnerVerse-AC** | Jiang et al. 2025 | [2505.09723](https://arxiv.org/abs/2505.09723) | Ctrl-World 实验对照之一，envisioning embodied environments with action |

**Ctrl-World 也引用、但太老不读**（记录在此防止后面又被重复评估）:

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
