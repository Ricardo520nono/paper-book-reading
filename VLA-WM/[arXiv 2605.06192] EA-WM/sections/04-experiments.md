[← 返回 EA-WM 主页](../README.md)

# §4 Experiments

---

## §4.1 Experimental Details

> "We train EA-WM in two stages: the first stage freezes the event-aware fusion modules and trains the main DiT LoRA, KVAF-branch LoRA, and KVAF head to stabilize the two streams; the second stage unfreezes the fusion modules to learn cross-stream event-aware interaction."

**两阶段训练**：
- **Stage 1**：冻结 event-aware fusion，训主 DiT LoRA + KVAF-branch LoRA + KVAF head（稳定两路）
- **Stage 2**：解冻 fusion 模块，学 cross-stream event-aware 交互

**配置**：LoRA rank 32，lr 8e-5，32 H100 GPU，batch size 32。

**评估**：用 **WorldArena** benchmark（翔哥 proposal 已收录），RoboTwin 数据 + 同样的 segmentation 方法。三个维度：**Physics Adherence / 3D Accuracy / Controllability**，综合成 **P3CScore**。

---

## §4.2 Experimental Results

### Table 1：主结果（vs video world models）

| Model | P3CScore |
|---|---|
| CogVideoX | 71.08 |
| WoW | 61.83 |
| Wan 2.2 | 60.83 |
| TesserAct | 62.02 |
| GigaWorld-0 | 59.32 |
| Vidar | 62.47 |
| Cosmos-Predict 2.5 (text) | 50.15 |
| **Genie Envisioner** | **45.40** |
| **EA-WM (Ours)** | **76.60** |

> "EA-WM achieves the best overall P3CScore of 76.60, outperforming the strongest baseline, CogVideoX, by 5.52 points."

🔥 **关键发现**：
- EA-WM SOTA 76.60，比最强 baseline CogVideoX 高 5.52 分
- 提升最明显的是 **action-induced motion 和 interaction 相关指标**：Interaction Quality 0.594→0.682，Trajectory Accuracy 0.353→0.430，Instruction Following 0.727→0.792
- ⚠️ **我们读过的 Genie Envisioner 在 WorldArena 上只有 45.40** —— EA-WM 把 GE 也比下去了（注意：GE 是 instruction-conditioned，可能不是为 WorldArena 这种 action-conditioned 评估优化的）

### Qualitative Analysis（Figure 3）
4 个随机 RoboTwin 任务，对比 GT / Wan2.2 baseline / EA-WM。EA-WM 在 gripper-object 接触关系、物体 identity、局部几何、物理合理 robot pose 上都更接近 GT。

---

## §4.3 Ablation Study（Table 2）

| Variant | P3CScore | 说明 |
|---|---|---|
| Wan2.2（baseline）| 60.83 | 纯 backbone |
| w/o KVAFs（换数值 action）| 70.97 | 去掉 KVAFs |
| w/o EAF（去 event-aware fusion）| 74.80 | 保留双向 cross-attn 但去 event 机制 |
| **EA-WM（full）** | **76.60** | 完整 |

**两个组件的互补作用**：
- **KVAFs 贡献 +5.63**（vs w/o KVAFs）—— 主要在 Trajectory Accuracy / Depth Accuracy。说明相机对齐的 KVAFs 提供了数值 action 给不了的几何运动线索
- **EAF 贡献 +1.80**（vs w/o EAF）—— 主要在 Interaction Quality / Perspectivity。说明 EDLS 引导的 event-aware fusion 帮模型关注运动变化和交互区域

**互补失败模式**（Figure 4）：
- w/o KVAFs：能保住局部 robot-object 关系，但抓取轨迹和 action-induced motion 不准
- w/o EAF：能从 KVAFs 推抓取路线，但保不住物体一致性（形状/大小/排列）
- → **KVAFs + EAF 结合才能两全**

---

## §4.4 Additional Analysis

### Action recovery（Table 3）—— 一个诚实的 limitation

从生成的 KVAF 视频里 heuristic 恢复数值 action：

| Method | Translation Err | Rotation Err | Gripper Err |
|---|---|---|---|
| Raw-action baseline | 0.004 | 0.009 | 0.013 |
| KVAF recovery | 0.0155 | 0.110 | 0.039 |

Heuristic KVAF recovery 检测率仅 **~0.45**。

> "numerical action prediction still performs better on translation, rotation, and gripper errors. This gap is expected, since KVAFs are designed as image-domain action fields rather than direct numerical action outputs."

💡 **批注**：Paper 诚实承认 —— **KVAFs 是 image-domain action field，从 KVAF 反解回精确数值 action 仍是 challenging problem**。但 Figure 5 的 overlay 显示预测的 KVAFs 和生成的 robot motion **对齐良好** —— 说明网络学到了 KVAF 里的空间物理信息，没把它当抽象条件。

🔥 **Uni-WAM 关联**：这个"action 画成图后难反解回数值"的问题，和 Action Images 的 decoding（ray casting + side-view matching）面对的是同一个难题。**EA-WM 没解决，Action Images 用多视角几何解决了一部分**。

### KVAF-conditioned video generation（Table 4）⭐ 直接 baseline 对比

| Model | P3CScore |
|---|---|
| **CtrlWorld** | 74.03 |
| **IRASim** | 69.75 |
| Cosmos-Predict 2.5 (action) | 66.12 |
| RoboMaster | 61.63 |
| **KVAF-conditioned (Ours)** | **78.13** |

🔥🔥 **关键**：EA-WM 直接和 **Ctrl-World、IRASim** 在 WorldArena 上对比 —— **EA-WM 全面领先**（78.13 vs Ctrl-World 74.03 vs IRASim 69.75）。

→ 证明"把 action 变成 structured visual field"比"raw action / trajectory 条件"是更强的 conditioning interface。

---

## 💡 §4 整段 takeaway

| 实验 | 结论 |
|---|---|
| Table 1 主结果 | P3CScore 76.60 SOTA，超 CogVideoX 5.52 分，超 Genie Envisioner（45.40）|
| Table 2 Ablation | KVAFs +5.63 / EAF +1.80，两者互补 |
| Table 3 Action recovery | 诚实承认：KVAF 反解回数值 action 仍难（检测率 0.45）|
| Table 4 baseline 对比 | **直接打败 Ctrl-World（74.03）和 IRASim（69.75）**，EA-WM 78.13 |

🔥 **Uni-WAM 视角**：
- Table 4 是 EA-WM 和 **Ctrl-World / IRASim** 的**直接对决** —— Uni-WAM 可以照搬这个评估设置（WorldArena + 同样 baseline）
- 但注意：所有评估的 action 都来自 **RoboTwin 的 expert / GT trajectory** —— **没有 off-expert action 测试**
- EA-WM 的 P3CScore 测的是"physics adherence / 3D accuracy / controllability"，**全是"生成质量"维度，不是"off-expert action 下的可靠性"**

---

[← §3 Method](03-method.md) | [§5 Conclusion →](05-conclusion.md)
