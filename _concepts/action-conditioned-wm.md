[← 返回概念词典](README.md)

# Action-Conditioned World Model (AC-WM) & Action Injection

> **一句话定义（Ricardo 版）**：
> AC-WM = **action 也能作为输入的 world model**；至于 **action 怎么喂进模型 + 喂进去能不能 work，要仔细看**。

## 📌 速查

| 关键词 | 一句话 |
|---|---|
| **World Model (WM)** | 学了"环境动力学"的模型，能预测"动作执行后变成什么样" |
| **Conditioned on X** | "以 X 为输入条件" |
| **Action-Conditioned WM (AC-WM)** | **action** 作为 conditioning input 之一的 WM |
| **Action Injection** | 工程层面：把 action 这个低维信号"塞进"模型内部高维处理流的具体方式 |
| **架构层"能喂"** ≠ **训练层"能消化"** | 两个独立维度，AC-WM 调研判分类时两者都要看 |

---

## 1. WM 是什么？

### 1.1 直观定义

WM = **学了"环境动力学"的模型，能预测"动作执行后环境变成什么样"**。

人脑里就内置了 WM：
- "如果我把右手伸过去抓桌上那杯水" → 脑子里有一段画面：手伸过去、握住杯子、提起来
- "如果我现在猛踩油门" → 脑子里有一段画面：车加速冲出去

→ 这个 **"想象 + 预演"** 的能力，就是 WM 在做的事。

### 1.2 数学定义

最朴素形式：
$$W: (s_t, a_t) \to s_{t+1}$$

更通用（输入观测、输出观测序列）：
$$W: (o_t, a_{t:t+H}) \to o_{t+1:t+H}$$

含义：**给我当前画面 + 接下来要做的动作，我能预测接下来一段视频长啥样**。

---

## 2. "Conditioned on X" 是啥意思？

ML 里 "conditioned on X" = **"以 X 为输入条件"**。举例就懂：

| 模型类型 | conditioned on 什么 | 含义 |
|---|---|---|
| Text-to-Image (T2I) | **文本** | 给文本，输出图 |
| Class-conditional GAN | **类别 label** | 给类别 ID，输出该类的图 |
| Image-to-Video (I2V) | **一张图** | 给一张图，输出视频 |
| **Action-Conditioned WM** | **动作（action）** | **给动作，输出执行后的画面** |

**所以 "Action-Conditioned" = 模型接受 action 作为输入条件之一**。

---

## 3. AC-WM vs T2V WM 的核心区别

这两个名词容易混淆，但**控制粒度根本不同**：

| 维度 | T2V WM | AC-WM |
|---|---|---|
| 输入信号 | **语言**（"把杯子拿过来"）| **数值 action**（"末端 pose [0.1, 0.2, 0.3, ...]"）|
| 控制粒度 | **High-level**（不告诉具体怎么动）| **Low-level**（每一步具体怎么动）|
| 谁来 specify 动作 | 用户用自然语言 | Policy 输出的数值序列 |
| 例子 | EWMBench 测的 7 个 model | Ctrl-World / DreamerV3 / Genie 3 / Cosmos-Predict |

**类比**：
- T2V WM = **"导演"** —— 你告诉它"演员去抓那个杯子"，它自己决定演员怎么动
- AC-WM = **"动捕系统"** —— 你给它 30 fps 的关节角度数据，它给你渲染对应画面

---

## 4. 为什么 AC 是工程难题？

**直觉**：把 action 当输入塞进 model 不就行了？

**实际困难**：
- Action 是**几个数字**（比如 7 维 joint angle vector，或 6D Cartesian pose）
- 模型内部的 feature 是**几百万维 tensor**（图像/视频的 latent representation）
- 两者**维度、空间结构完全不同**
- 怎么"融合"两者 = 工程问题

→ 所以才有 **"action injection（action 注入）"** 这个概念 —— 把 action 这个低维信号"塞进"模型内部高维处理流。

---

## 5. Action 注入的 5 种主流方式

不同 paper 用不同的 injection 方式，**决定它的控制精度**。

### 方式 1：Channel concatenation（最朴素）

把 action 当作图像的"额外 channel"直接拼上：
```
原 RGB:    [3, H, W]
加 action: [3 + 7, H, W]   ← 7 是 action 维度
```

- ✅ 实现简单
- ❌ Action 信号容易被淹没在视觉 feature 里
- 用过：早期 video prediction model

### 方式 2：Cross-attention

让 visual token 通过 attention 主动"查询" action：
```
Visual tokens (Query) → 看 Action embedding (Key, Value) → 融合后的 Visual tokens
```

- ✅ 灵活，每帧能独立 attend 对应 action；信号清晰
- ❌ 增加计算量
- 用过：**Ctrl-World**、Zhu et al. 2024

### 方式 3：AdaLN modulation（DiT 系常用）

把 action embed 之后，**用它调制 LayerNorm 的 scale 和 shift**：
```
LayerNorm(x) → scale × LayerNorm(x) + shift
其中 scale 和 shift 由 action 决定
```

- ✅ 精细到每一层 normalization
- ❌ 参数较多
- 用过：DiT 系（Wan2.2 用同样原理注入 timestep）

### 方式 4：Action prefix token（LLM 范式）

把 action 编码成 token，拼在 visual token 序列**前面**：
```
[action_token_1, action_token_2, ..., visual_tokens...]
```

- ✅ 和 LLM autoregressive 范式 align
- ❌ 长序列下效率低
- 用过：Decision Transformer、有些 RL transformer

### 方式 5：Frame-level action conditioning（Ctrl-World 的精髓）

和方式 2 cross-attention 配套使用。关键 idea：
- 每个 video frame 都对应一段 action
- 让每帧的 visual token **只 attend 到该帧对应的 action**（不是整个序列）

```
Frame 1 visual ← cross-attn ← Action at frame 1
Frame 2 visual ← cross-attn ← Action at frame 2
...
Frame H visual ← cross-attn ← Action at frame H
```

- ✅ 时间精确对齐，每帧 visual dynamics 严格对应该帧 action
- ❌ 需要时间对齐的 action label
- 用过：Zhu et al. 2024（起源）→ **Ctrl-World**

---

## 6. ⭐ 核心 nuance：架构层 vs 训练层（两个独立维度）

这是 Ricardo 第二个总结的精确化：

| 维度 | 决定因素 | 失败时的症状 |
|---|---|---|
| **架构层"能喂"** | Action injection 机制（5 种之一）| model 直接报错 / 完全 ignore action |
| **训练层"能消化"** | 训练数据 action 分布广度 | model 接受 action 但**输出垃圾视频** |

### 直观例子：喂 random-feasible action 给 Ctrl-World

| 维度 | 结果 |
|---|---|
| **架构层** | ✅ Cross-attention 接受任意 6D pose，**不会报错** |
| **训练层** | ❌ DROID 没见过 random-feasible action，**输出可能崩坏** |

### 这就是为什么 Uni-WAM 必须存在

- 现有 AC-WM 在**架构层**都能"接受 off-expert action"
- 但在**训练层**普遍只在 expert 分布上训过
- → 它们没见过 → 输出不可信
- → Uni-WAM 用 5 类 off-expert × Gated 架构系统化暴露这个问题

---

## 7. Uni-WAM 调研用这个分类

判断一个 paper 在 Uni-WAM 5 分类（a/b/c/d/e）里属于哪一类时：

| 分类 | 架构层 | 训练层 | 是否开源 finetune |
|---|---|---|---|
| **a** | ✅ AC 架构 | ✅ 在 AC 数据上训 | N/A |
| **b** | ❌ 原本是 T2V 等 | (没 AC 训练) | ✅ 提供 AC finetune pipeline |
| **c** | ❌ 原本是 T2V 等 | (没 AC 训练) | ❌ 不提供 |
| **d** | 任意 | 任意 | 不开源整个 model |
| **e** | 不是 WM 或不相关 | - | - |

**注意 nuance**：
- Ctrl-World 自己**做了** AC finetune（拿 SVD 改造），属于 a 类
- 但如果它**公开了 finetune 代码**，从"工具能让别人用"的角度看，**也算 b 类的现成案例**
- → 实际 a/b 边界有时模糊，要看 paper 的定位 + repo 公开程度

---

## 8. 在 Ricardo 的论文里出现的位置

| 论文 | 哪一节 | 怎么用 |
|---|---|---|
| **Wan paper** | §4.5 (Wan-Bench 评测) | Wan 本身是 **T2V/I2V**（不是 AC-WM），但 Wan 2.2 backbone 可以被 finetune 成 AC-WM（如 Motus、未来的 Uni-WAM）|
| **Ctrl-World** | §4.1 Frame-level Action Conditioning | **直接案例**：SVD → AC-WM 的改造，action = Cartesian 6D pose，注入方式 = frame-level cross-attention |
| **Uni-WAM proposal** | Model architecture | 提出的 MoT + Wan2.2-TI2V-5B + Qwen3-VL-4B + Shared Attention 架构，action 也是 frame-level injection |
| **EWMBench** | (不涉及，是 T2V benchmark)| 反例：T optional → 不是 AC-WM |

---

## 9. 相关概念链接

- [Encoder-Decoder 架构](encoder-decoder.md)
- [DiT (待写)](待写)
- [Flow Matching (待写)](待写)
- [Cross-attention vs Self-attention (待写)](待写)

---

## 10. 一句话存档（送 Ricardo）

> **AC-WM = action 是 conditioning input 的 WM**。**架构层"能喂"≠ 训练层"能消化"**。Uni-WAM 调研的本质就是**揪出在训练层没真的消化 off-expert action 的 AC-WM**。
