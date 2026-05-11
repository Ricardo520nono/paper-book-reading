[← 返回 Ctrl-World 主页](../README.md)

# 串讲 · 一张图 + 几句话读懂 Ctrl-World

> **目的**：用最少的字数 + 几张关键 figure 把这篇 paper 串起来。不重复 sections/ 里的逐段细节，**只抓骨架**。

---

## 1. 这篇 paper 解决的是什么问题？

**两个痛点**（都属于 generalist robot policy 的研发循环）：

| 痛点 | 现状 | 为什么麻烦 |
|---|---|---|
| **Policy Evaluation** | 评估 VLA policy 要在**真机上反复跑 rollout** | 慢、贵、难规模化，阻碍迭代 |
| **Policy Improvement** | policy 出问题后**只能收集更多 expert 数据**再训 | 同样慢、贵；且很多 failure 模式无法靠加数据自然出现 |

**一句话**：研究员手里没有一个**快 + 便宜 + 反馈驱动**的工具来评估和改进 VLA policy。

**Ctrl-World 的解法定位**：
> 把"真机 rollout"换成"在 imagination 里 rollout" —— **用 world model 当 simulator**。

→ 后面我们会看到，它怎么搭这个 simulator（§4.1）、怎么用它评估（§4.2 + §5.3）、怎么用它改进（§5.4）。

---

## 2. 这篇 paper 的解决方法全景 · 一张图看懂

![](../images/figure-01.png)

**按 Figure 1 从左到右读**：

### 输入（左侧）
- **Instru**（紫块）：language instruction，比如 "place the blue block on the plate"
- **Cam 1 / 2 / 3**（绿块）：3 个相机视角的当前 obs（2 个 third-person + 1 个 wrist-view）
- **N step action chunk**（灰块）：policy 输出的 N 步数值 action（具体到 6D pose）

### 核心循环（中间，重点！）

```
Generalist Policy → 给我未来 N 步该怎么动 (action chunk)
        ↓
World Model    → 给我执行完这 N 步后画面变啥样 (Pred 1/2/3 三个视角)
        ↓
Pred 喂回 Policy → 下一个 N 步 action chunk
        ↓
循环 ×N 次
```

**关键名字 = "policy-in-the-loop world model rollout"**。两个 agent **互相喂数据**：
- Policy 决定动作 → WM 预测后果 → 后果喂回 policy → 再决定动作

### 输出（右侧）

整个 rollout 跑完 → 拿到一条 **synthetic trajectory**（合成轨迹）。这条轨迹有两种用法：

| 用例 | Figure 1 右侧展示 | 后面在哪节验证 |
|---|---|---|
| **Policy Evaluation** | 散点图（y=0.87x-0.04）—— WM 里的 policy 排名 ≈ 真机 ranking | §5.3 |
| **Policy Improvement** | 柱状图（+44.7%）—— 拿合成的 successful trajectory 做 SFT 提升 policy | §5.4 |

### 还有一个小细节：Memory

WM 旁边的 **Memory**（绿块）是 Ctrl-World 的关键工程组件 —— **稀疏历史帧 + pose 检索**，防止长 rollout 时漂移。详见 §4.1。

---

**一句话总结这张图**：
> Ctrl-World = **policy 和 WM 自回归交替运行的 simulator**，跑出来的合成 trajectory 既能用来给 policy 排名（evaluation），也能挑成功的来 SFT（improvement）。

🔥 **Uni-WAM 视角的小预警**：
- 这张图的 loop 流程**完全假设 policy 输出的 action 在 WM 训练分布内**
- 如果换一个早期 sub-optimal policy（输出 off-expert action）→ WM 还能正确预测吗？Figure 1 没回答
- 这是 Uni-WAM 关心的核心 gap

---

## 3. Ctrl-World 的核心方法 · 一张架构图看懂

![](../images/figure-02.png)

> ⚠️ **先看一个 notation 警告，免得读着读着懵**：
> Figure 2 里有**两个 "N"**，含义完全不同！
> - 左下 `N: Camera Views` 的 N = **相机视角数**（这里 N=3：2 个 third-person + 1 个 wrist）
> - 中间 transformer block 旁边 `×N` 的 N = **transformer block 堆叠数**（每个 block = Spatial + Temporal 串起来）
>
> 后面我会把第二种叫 "× N blocks"，避免混淆。

---

### 怎么看这张图（4 块结构）

#### 块 1：左侧 + 顶部时间轴 → 输入是什么

**左侧 3 张图**：3 个相机视角的当前 obs（第 1 个组件 **Multi-View Joint Prediction** 的体现）。

**顶部 Timeline**：横轴是时间。两种 frame：
- **稀疏历史帧**（深绿色实心）：$o_{t-km}, ..., o_{t-2m}, o_{t-m}, o_t$ —— 不是连续每一帧都拿，是**每隔 m 步取一帧**，共 k 个
- **未来要预测的帧**（虚线方块）：$o_{t+H}$，长度为 H

→ **这就是 Memory 在 Figure 1 里所代表的实质内容**：稀疏历史帧序列。

**每帧的 token 数**：$P = N \times H \times W$（N 个相机 × Latent 高度 × Latent 宽度）

#### 块 2：中间 transformer 主干

**输入打包**：稀疏历史帧 + Noised Future（加了噪声的未来帧）拼起来，shape `(B×T, P, C)`
- B = batch size
- T = 总 frame 数（历史 + 未来）
- P = 每帧 token 数
- C = channel 数

**两个 transformer 交替**：
- **Spatial Transformer**：处理每帧内的空间关系 (B×T 个 frame，每个 P 个 token)
- **Temporal Transformer**：处理跨帧的时间关系 (B×P 个 token 位置，每个有 T 个时间点)
- 串起来 = **1 个 block**，重复 **× N blocks** 次

**输出**：底部白色 + 绿色 tokens —— 绿色就是新预测的未来 frame，用虚线箭头**指回 timeline 上的未来位置** $o_{t+H}$。

#### 块 3：两类条件信号（喂给 transformer 主干）

模型怎么知道"该往哪走 / 该理解什么场景"？靠两类条件信号：

**条件 1：CLIP Semantic Tokens**（蓝色方块）
- 1 个 token，shape `(B, 1, C_t)`
- 用 CLIP 提取**整段视频的语义** —— 让模型理解"这是什么任务"
- 输入到 Temporal Transformer

**条件 2：History Poses + Action Chunk Poses**（绿色+红色方块）
- shape `(B×T, 1, C_a)` —— 每个 frame 对应 1 个 pose token
- **绿色 = 历史 pose** $[q_{t-km}, ..., q_t]$：过去机械臂真实在哪
- **红色 = action chunk pose** $[a'_{t+1:t+H}]$：policy 让机械臂未来去哪（把 raw action 转成 Cartesian pose）
- 输入到 Spatial Transformer

→ **这里就是 action 注入 model 的入口！** 它不是直接拼到 visual token，而是通过 cross-attention 让 visual token 主动 attend 到 pose。

#### 块 4：右侧 Frame-Level Cross-Attention（核心机制 zoom-in）⭐

右侧那个细节图是**整个 paper 最关键的机制**，把它讲清楚：

```
上方（Query 端）:    [o_{t-km}, ..., o_{t-m}, o_t]  [x_{t'}]
                      历史 visual tokens          未来 noised tokens
                      ↕   ↕   ↕   ↕               ↕   ↕   ↕
                     cross-attention（每对独立）
                      ↕   ↕   ↕   ↕               ↕   ↕   ↕
下方（K/V 端）:      [q_{t-km}, ..., q_{t-m}, q_t]  [a'_{t+1:t+H}]
                      历史真实 pose                未来要去的 pose
```

🔥 **关键点：每一帧的 visual token 只 attend 到 "它自己对应的那一帧 pose"**：
- 第 t-2m 帧的 visual token → 只看第 t-2m 帧的真实 pose $q_{t-2m}$
- 第 t+5 帧的 noised visual token → 只看第 t+5 帧的 action pose $a'_{t+5}$
- 不是"所有 visual tokens 都 attend 整个 pose 序列"
- 是 **"一帧 visual ↔ 一帧 pose"严格对齐**

**这就是 frame-level 的意思** —— 注入的颗粒度是**每一帧**，不是整段。

---

### 三大组件回看（这张图上分别在哪）

| 组件 | Figure 2 里在哪 |
|---|---|
| **1. Multi-View Joint Prediction** | 左侧 3 张相机图 + 顶部时间轴每帧都包含多视角 |
| **2. Pose-conditioned Memory Retrieval** | 顶部时间轴的**稀疏深绿历史帧** + 绿色 history pose tokens |
| **3. Frame-level Action Conditioning** | 右侧 cross-attention zoom-in 的**红色 action pose tokens** |

---

### 🔥 Uni-WAM 视角的关键观察

**Action 注入的具体路径已经清晰**：

1. Policy 输出 raw action $a_{t+1:t+H}$
2. 通过 forward kinematics 转成 Cartesian pose $a'_{t+1:t+H}$
3. 进入 Spatial Transformer 的 cross-attention 模块
4. 每一帧的 noised future visual token 通过 cross-attention 拿到它对应的 pose 信息
5. transformer 主干根据这些信号去噪 → 生成未来 frame

**对 Uni-WAM 的两个判断**：

| 角度 | 结论 |
|---|---|
| **架构能不能喂 off-expert action？** | ✅ 完全可以 —— 直接替换右下红色 pose token 内容就行 |
| **训练有没有见过 off-expert action？** | ❌ 没怎么见过 —— DROID 训练数据里全是 human teleoperation 的"自然"轨迹 |

→ 这就是 [`_concepts/action-conditioned-wm.md`](../../../_concepts/action-conditioned-wm.md) 里讲的 **"架构层能喂 ≠ 训练层能消化"** 的活生生例子。

---

> 后续点位等 Ricardo 指定再补 ⏳
