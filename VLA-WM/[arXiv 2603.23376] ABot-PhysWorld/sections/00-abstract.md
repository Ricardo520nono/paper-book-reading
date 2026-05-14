[← 返回 ABot-PhysWorld 主页](../README.md)

# §0 Abstract

## 原文 + 翻译

> "Video-based world models offer a powerful paradigm for embodied simulation and planning, yet state-of-the-art models often generate physically implausible manipulations—such as object penetration and anti-gravity motion—due to training on generic visual data and likelihood-based objectives that ignore physical laws."

**翻译**：基于视频的 world model 是具身仿真和 planning 的强范式，但 SOTA 模型经常生成**物理不合理**的操作 —— 比如**物体穿模、反重力运动** —— 原因是训练在通用视觉数据上 + 用基于似然的目标（忽略物理定律）。

> "We present ABot-PhysWorld, a 14B Diffusion Transformer model that generates visually realistic, physically plausible, and action-controllable videos. Built on a curated dataset of three million manipulation clips with physics-aware annotation, it uses a novel DPO-based post-training framework with decoupled discriminators to suppress unphysical behaviors while preserving visual quality."

**翻译**：ABot-PhysWorld 是一个 **14B DiT**，生成"视觉真实 + 物理合理 + 动作可控"的视频。基于 **3M 操作 clip 的策划数据集**（带物理感知标注），用一个**新颖的 DPO 后训练框架 + 解耦判别器**来压制非物理行为，同时保持视觉质量。

> "A parallel context block enables precise spatial action injection for cross-embodiment control. To better evaluate generalization, we introduce EZSbench, the first training-independent embodied zero-shot benchmark combining real and synthetic unseen robot-task-scene combinations."

**翻译**：**并行 context block** 实现精确的空间动作注入，支持跨 embodiment 控制。为了更好地评估泛化，提出 **EZSbench** —— 第一个 training-independent 的具身零样本 benchmark，结合真实和合成的"未见过的 robot-task-scene 组合"。

> "ABot-PhysWorld achieves new state-of-the-art performance on PBench and EZSbench, surpassing Veo 3.1 and Sora v2 Pro in physical plausibility and trajectory consistency."

**翻译**：在 PBench 和 EZSbench 上达到新 SOTA，在物理合理性和轨迹一致性上**超过 Veo 3.1 和 Sora v2 Pro**。

---

## 💡 Abstract takeaway

| 维度 | 信息 |
|---|---|
| 模型 | 14B Diffusion Transformer（基于 Wan2.1-I2V-14B）|
| 三个核心 | 数据策划 / DPO 物理对齐 / 并行 context block 动作注入 |
| 数据 | 3M manipulation clips（5 个公开数据集）+ 物理感知标注 |
| Benchmark | EZSbench（training-independent 零样本）|
| 对手 | Veo 3.1, Sora v2 Pro（通用 video 巨头）|

🔥 **Uni-WAM 视角**：注意它的核心卖点是 **physical plausibility**（物理合理性），不是 policy evaluation。这是和 Ctrl-World / GE-Sim 不同的切入角度。

---

[§1 Introduction →](01-introduction.md)
