[← 返回 Genie Envisioner 主页](../README.md)

# 串讲 · 几张图过完 Genie Envisioner

> **arXiv 2508.05635 (v3 2025/11)** · 2025/08 首次发布 · 作者 Liao / Zhou / Huang 等 14 人 (AgiBot Genie Team + LV-NUS + BUAA)
>
> **Uni-WAM 视角的前置判断**：**a 类 AC-WM 平台** ⭐⭐ — 真具身 manipulation；**EnerVerse-AC 的同团队后续 + 统一框架**。

---

## 1. 这篇 paper 解决的是什么问题？

| 痛点 | 现状 |
|---|---|
| **现有 robot manipulation 系统是"补丁堆叠"** | data collection / training / evaluation 各自独立, 用不同 representation |
| **Manual reprogramming 难规模化** | 新任务、新模态、新场景都需要手工调整, hindering scalability |
| **现有 video gen WM 缺 closed-loop** | T2V model 能生成视频但**不能 act**, 形成不了 policy 闭环 |

**Genie Envisioner 一句话定位**：
> AgiBot 团队把**数据 / 训练 / 评估 / 模拟**全部塞进**一个 closed-loop video generative world model framework**。模式：先训一个超大 video diffusion (GE-Base)，再衍生出 policy (GE-Act) + simulator (GE-Sim) + benchmark (EWMBench)。

---

## 2. 方案全景 · Figure 1 看懂

![](../images/figure-01.png)

**Figure 1 = 整个平台四件套**：

### 中心：**GE-Base**（World Foundation Model）
- 输入：3 视角 observation + instruction + memory（历史帧）
- 输出：generated video（多视角未来视频）
- 训练数据：**AgiBot World 1M+ trajectories**（来自 AgiBot-World-Beta）

### 左下：**GE-Act**（World Action Model）
- 输入：GE-Base 的 latent features
- 输出：action chunk → 直接送 robot 执行
- 用途：从想象转成可执行 policy（goal-directed control）
- 部署任务：Pour water / Assemble box / Fold clothes 等

### 右下：**GE-Sim**（World Simulator）
- 输入：Action condition（可来自任何 action model: ACT / GR1 / Octo / π₀ / OpenVLA）
- 输出：rendered action execution video
- 用途：**作为 simulator 对任意 policy 做 closed-loop simulation**

### 底部：**EWMBench**（Embodied World Model Benchmark）
- 数据集 + 评估工具
- Metric: Scene / Motion / Semantics 三维度（你之前读过）
- 评估 video WM 的视觉保真度、物理一致性、指令对齐

🔥 **关键 insight**：
- **GE-Base 是 backbone**，所有其他模块都从它衍生
- **EVAC 演化为 GE-Sim** —— 同团队同思路, 但更大规模、更系统化
- **policy 和 simulator 共享 backbone** —— Uni-WAM proposal 里"MoT 一体化"思路的现实版本

---

## 🔄 前置认知：GE 是 EnerVerse-AC 的"宏大版"

| 维度 | EnerVerse-AC (2025/05) | **Genie Envisioner (2025/08+)** |
|---|---|---|
| 范围 | 单个 AC-WM (评估器 + 数据引擎) | **平台**: WM + Policy + Simulator + Benchmark |
| Backbone | EnerVerse VDM | **LTX-Video 2B** + 自家改造 |
| 训练数据 | AgiBot World 一部分 | **AgiBot-World-Beta 全集, 1M+ trajectories, 3000+ hours** |
| 训练计算 | 32 A100 × 8 days | **GE-Base: 16×8 = 128 A100 × 数天; GE-Act: 8 A100 × 36 hours** |
| Policy output | ❌ 不直接出 action（只是 simulator）| ✅ **GE-Act 直接出 action chunk** |
| Benchmark | 无 | **EWMBench 配套** |
| 开源 | ✅ | ✅ (https://genie-envisioner.github.io) |

→ **Genie Envisioner = EnerVerse-AC 升级 + Policy 模块 + Benchmark**。同团队（Yuxin Jiang 也在作者列表里）。

---

## 3. 核心方法 · Figure 3 + Figure 7

### Figure 3：GE-Base 架构 = autoregressive video chunk generation

![](../images/figure-03.png)

**(a) 左图**：autoregressive 工作流
- 历史 frames + noise → GE-Base → 下一段 video chunk
- 一个 chunk 一个 chunk 地预测，每个 chunk 内多视角联合

**(b) 右图**：Causal Block 内部
- **Self-Spatial Attention**（左视角 / 前视角 / 右视角各做空间 attention）
- **Cross-Attention**（跨视角信息交换）
- 总共 N+M blocks

🔥 **Action 注入方式 ≠ 在 GE-Base 里**：
- **GE-Base 是 instruction-conditioned**（接语言, **不接 action**）
- 它本质是个 **language-conditioned video gen model**
- Action conditioning **发生在 GE-Sim** 中（GE-Sim 是 GE-Base 的 action-conditioned 衍生）

→ 严格说 GE-Base 自己**不是 AC-WM**，但**整个平台是**。GE-Base + action condition adapter = GE-Sim = AC-WM。

### Figure 14：GE-Sim 的 action 注入机制（这才是真正的 AC-WM 部分）

![](../images/figure-14.png)

**§5 GE-Sim 是 paper 真正的 AC-WM 部分**，独立成章。核心创新 = **Hierarchical Action-Conditioning Mechanism**（图 a 左侧）。

#### Action 输入（7D × K 步）

每一步 action = **7D vector**：`[x, y, z, roll, pitch, yaw, gripper_openness]`
- 位置（xyz）+ 朝向（rpy）+ 夹爪状态 = 7D
- 双臂时拼成 14D（左 7 + 右 7）
- K 步合在一起 = `A ∈ R^{K × 14}`

#### 两路注入（Pose2Image + Motion Vector）

**第 1 路: Pose2Image Conditioning**（视觉 token 层注入）

每个 timestep i 的 pose `a_i` → 画成 pose image `P_i`:
1. **位置** (x_i, y_i, z_i) → 用相机内外参 project 到 2D 像素坐标
2. **朝向** (r_i, p_i, y_i) → 转 rotation matrix, 把三个正交轴 project 到 image plane（指示方向）
3. **gripper** o_i → 画在 unit circle 上，**颜色深浅代表开合**（淡色 = 开，深色 = 闭）
4. **左右臂** 用不同色区分

→ pose image `P_i` 和历史帧 `I_i` 都用**同一个 video encoder ε 编码**, 然后**element-wise add**：

$$v_i = \varepsilon(I_i) + \varepsilon(P_i)$$

合成 token `v_i` **作为 visual token 注入 generation stream**。

🔥 **这就是 EnerVerse-AC 的 "Spatial-Aware Pose RGB" 思路** —— **把 6D pose 画成 RGB 图，然后和 obs 图一起编码**。

**第 2 路: Motion Vector Conditioning**（cross-attention 注入）

计算连续 pose 的 delta：

$$\Delta a_i = a_i - a_{i-1} = [\Delta p_i, \Delta r_i]$$

→ 经过 learnable encoder → **和 reference image style token concatenate** → **通过 cross-attention 注入到每个 DiT block**

🔥 **这就是 EnerVerse-AC 的 "Delta Action Cross-Attention" 思路** —— **temporal 动作变化通过 cross-attention 注入**。

#### 训练（§5.2 简略）

- 从 **GE-Base-MR**（high-temporal-resolution variant）初始化
- 在 **full AgiBot-World-Beta** 上训
- 用 **ground-truth action trajectories** 做 conditioning input
- 训练 corpus 加入 **failure cases**（incomplete behaviors, suboptimal control）— 和 EVAC 一脉相承

#### GE-Sim vs EVAC 的关系

| | EnerVerse-AC | **GE-Sim** |
|---|---|---|
| Spatial-Aware Pose RGB | ✅ | ✅ Pose2Image Conditioning |
| Delta Action Cross-Attention | ✅ | ✅ Motion Vector Conditioning |
| Gripper magnitude RGB | ✅ | ✅（合并在 Pose2Image 里）|
| Failure data | ✅ 人工 augmented | ✅ AgiBot-World-Beta 含 failure |
| Backbone | EnerVerse VDM | GE-Base (LTX-Video 2B 或 COSMOS2 2B)|
| 规模 | 中 | 大（用 GE-Base-MR）|

→ **GE-Sim ≈ EnerVerse-AC 的升级实现** —— 同思路、同两路注入、更大 backbone、更大数据。

### Figure 7：GE-Act 3-Stage 训练

![](../images/figure-07.png)

**3 阶段 pipeline**（不同 GPU 时长）：
- **Stage 1**: Action Pre-training（54 step 序列 @ 30Hz, **AgiBot-World-Beta full data**, 3 days × 16×8 GPU）
- **Stage 2**: Task-Specific Video Adaptation（freeze GE-Base, 微调到 task data, 12 hours × 8 GPU）
- **Stage 3**: Task-Specific Action Specialization（24 hours × 8 GPU）

🔥 **GE-Act 是 VLA 范式**: vision-language-action, 但**搭载在 video world model 上**作为 backbone。

---

## 4. 实验（简略）

Paper 的实验主要 demo GE-Base 在 multi-view robot manipulation 视频生成上质量高（Figure 5 等定性结果）+ GE-Act 在 AgiBot 上能完成多种 robot task。

具体的 ranking alignment / OOD action 测试**不是 Genie Envisioner 的重点** —— 它聚焦"作为基础设施"，不是 evaluation framework。

→ **量化对比 baseline** 在 EWMBench 上（不在这篇 paper 里）。

---

## 5. Summary · 整篇 paper 一段话

> **Genie Envisioner (GE)** 是 AgiBot Genie Team 提出的 robot manipulation **统一世界基础模型平台**。它把**数据 / WM 训练 / Policy 推理 / Simulator / Benchmark** 全部塞进**一个 closed-loop video-generative framework**:
> - **GE-Base** = 大规模 video diffusion (LTX-Video 2B based)，instruction-conditioned 多视角 video gen
> - **GE-Act** = lightweight flow-matching decoder, 把 latent → action chunk（policy）
> - **GE-Sim** = action-conditioned 衍生, 当 simulator 用（**EnerVerse-AC 的演进**）
> - **EWMBench** = 配套 benchmark
>
> 训练数据：AgiBot-World-Beta **1M+ trajectories, 3000+ hours video**。模型、checkpoints、code 全开源。

### 三个最该记住的点

| # | 点 | 一句话 |
|---|---|---|
| 1 | **平台范式** | 4 个模块共享 backbone, 一体化 closed-loop（数据 / WM / Policy / Sim / Bench）|
| 2 | **GE-Base 是 instruction-conditioned, 不直接接 action** | Action 在 GE-Sim 衍生 |
| 3 | **EnerVerse-AC 的演化** | EVAC = GE-Sim 的前作, 同团队同思路, 但更小规模 |

### 一句话标签

> **Genie Envisioner = AgiBot 出品的"robot 版 Cosmos 平台"** — 不只是一个 model, 是一个 4-in-1 unified framework。

---

### 🔥 Uni-WAM 视角的简评

**对 Uni-WAM 的高价值**:
- ✅ **平台范式启发**：Uni-WAM proposal 里"MoT 一体化"思路在 GE 上有现实版本（不同模块共享 backbone）
- ✅ **数据规模参考**：AgiBot 1M+ trajectories 是真机数据上限的 reference
- ✅ **GE-Sim 作为 AC-WM 的实现**：和 EnerVerse-AC 思路一脉相承, 但更大规模, **是 Uni-WAM 的直接竞品**
- ⚠️ Paper 主要做"基础设施", **没系统对照 off-expert action 评估**

**Uni-WAM 仍能切入的空白**:
- GE 平台**不做** off-expert action 系统化诊断
- EWMBench 是从 GE 系列的"自评"角度做的, 不像 Uni-WAM 的 Action Following Fidelity 那样**主动暴露 WM 弱点**

**分类**：**a 类 AC-WM 平台**（GE-Sim 部分）+ **b 类候选**（GE-Base 是 backbone, 提供完整 finetune pipeline）。
