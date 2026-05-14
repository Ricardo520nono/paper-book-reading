[← 返回 EA-WM 主页](../README.md)

# §2 Related Work

---

## §2.1 Robotic World Models based on Video Generation Models

> "Existing robotic video world models span action-conditioned simulators and forward models for planning, model-based RL, and interaction simulation, infrastructure methods that use generated videos as synthetic trajectories, supervision, or policy-learning proxies, and learned video environments for scalable VLA evaluation and planning. Recent world-action models further couple video and action generation in unified frameworks. Despite these advances, how to fully exploit action information to improve future robot-video generation remains insufficiently explored."

**翻译**：现有 robotic video WM 三类用途：(1) action-conditioned simulator / forward model；(2) 用生成视频当合成数据/监督/policy 代理；(3) 可规模化 VLA 评估和 planning。但**怎么充分利用 action 信息来改善 future robot-video 生成仍探索不足**。

## §2.2 World Action Models

> "World Action Models (WAMs) are unified generative models that predict both robot actions and future visual states... Recent work in this line mainly treats future video modeling as dense supervision or latent guidance for better action generation and policy learning. However, their emphasis remains largely on how video generation can improve action prediction or policy learning. The reverse direction—using action information as structured guidance to improve future robot-video generation—remains much less explored."

**翻译**：WAM = 联合预测 action + future visual state 的统一生成模型（DreamZero 那条线）。**它们主要把 future video 当 dense 监督/latent 引导来改善 action 生成** —— 重心在"video 怎么帮 action"。**逆方向（用 action 引导 video 生成）探索得少得多**。

### 🔥 关键：action 表示方式的谱系

> "Existing methods have explored raw joint-space or end-effector vectors, 7-DoF control tokens, latent actions, universal action tokens, and spatial action abstractions. VideoVLA, UVA, Motus, and ViPRA represent action information through numerical, latent, or video-conditioned action spaces, while UniAct and SpatialVLA study more transferable or spatially structured action representations... Recent visual representations lift actions into image or video space via RGB action targets, virtual robot renders, pixel-grounded multiview action videos, or multiview heatmap videos."

EA-WM 梳理的 action 表示方式谱系：

| 类别 | 代表 |
|---|---|
| 数值 / latent / video-conditioned action space | VideoVLA, UVA, **Motus**, ViPRA |
| 可迁移 / 空间结构化 action 表示 | UniAct, SpatialVLA |
| **lift action 到 image/video space** | RGB action targets, virtual robot renders, **pixel-grounded multiview action videos**（= Action Images！）, **multiview heatmap videos** |

🔥 **Uni-WAM 调研记录**：
- 这段 related work 提到了 **Motus**（我们 backlog 里翔哥/梦飞点名的 a 类候选）和 **Action Images**（我们刚读完的"pixel-grounded multiview action videos"）
- EA-WM 自己批评这些 visual representation："action channels are often intentionally compact and robot-centric... robot-object interaction information is often modeled only indirectly" —— 它认为之前的"画末端"不够，要画**整条手臂**

---

## 💡 §2 整段 takeaway

| 子节 | 一句话 |
|---|---|
| §2.1 | 现有 robotic video WM 三类用途，但"用 action 改善 video 生成"探索不足 |
| §2.2 | WAM 重心在"video 帮 action"，逆方向少；EA-WM 梳理了 action 表示谱系，自己定位在"lift action 到 image space 但画得更全（整条手臂）" |

🔥 **Uni-WAM 调研价值**：§2.2 确认了 **Motus** 和 **Action Images** 的领域定位。EA-WM 没挖出新的近期具身 AC-WM 候选，但它对"action 表示谱系"的梳理很清晰，可以引用。

---

[← §1 Introduction](01-introduction.md) | [§3 Method →](03-method.md)
