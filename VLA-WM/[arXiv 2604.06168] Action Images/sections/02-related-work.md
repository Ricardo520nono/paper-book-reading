[← 返回 Action Images 主页](../README.md)

# §2 Related Work

> 三个子领域：robotics world models / generalist robot policy models / 4D generation models。重点看它怎么定位自己 vs 已有 AC-WM。

---

## §2.1 Robotics World Models

> "These video-based approaches typically adopt a two-stage pipeline, where future observations are first predicted and actions are then generated based on these predictions. More recently, joint video-action generation has been explored to unify modeling and control. In particular, DreamZero demonstrates strong zero-shot generalization and cross-embodiment transfer. However, these methods encode actions with additional action modules, leaving much of the pretrained video knowledge underused; we instead use multi-view action images so the backbone itself is a zero-shot policy."

**翻译**：基于 video 的 WM 通常是**两阶段 pipeline**（先预测未来观测，再从预测里生成 action）。最近有 joint video-action generation 来统一建模和控制，**DreamZero** 展示了强 zero-shot 泛化和跨 embodiment 迁移。

⚠️ **Action Images 的 self-positioning**：但这些方法都用**额外的 action module** 编码 action → 没充分用上预训练 video 知识。Action Images 用 **multi-view action image**，让 backbone 自己就是 zero-shot policy。

🔥 **Uni-WAM 关联**：
- 这里提到 **DreamZero** —— 我们 backlog 里的 b 类候选之一（NVIDIA 14B World Action Model）
- "两阶段 pipeline"（先预测 obs 再生成 action）就是 Ctrl-World / GE / EnerVerse-AC 那一派
- Action Images 自己定位成"消除 action module"那一派 —— 这是 AC-WM 设计哲学的一个分支

## §2.2 Generalist Robot Policy Models

> "While multiple advances in Vision-Language-Action (VLA) models, Diffusion Policy, and Reinforcement Learning have greatly promoted the generalizability of policy models, their diversity is still limited to relatively narrow task distributions and they struggle to zero-shot generalize to new environments. ... However, how to turn video prediction into transferable control remains nontrivial; our action-frame representation bridge this gap by making action native to the video space."

**翻译**：VLA / Diffusion Policy / RL 大幅提升了 policy 泛化性，但**任务分布仍相对窄，难 zero-shot 泛化到新环境**。video generation foundation model 启发了 policy learning，但**怎么把 video prediction 转成可迁移的控制仍非平凡**。Action Images 用 action-frame 表示桥接这个 gap —— **让 action 成为 video space 的 native 表示**。

💡 **批注**：这一节区分了两条路线 —— "policy 模型"（VLA/DP/RL）和"video model 启发的 policy"。Action Images 属于后者，但走得更彻底（action 直接进 video space）。

## §2.3 4D Generation Models

> "'4D' here refers to 3D plus time. ... Close to our method, [4, 61] leverage multiview generation to produce complex dynamic 4D scenes that can be replayed at any specified camera pose and timestamp. However, for robotic tasks, 4D generation is typically limited to a fixed single view."

**翻译**："4D" = 3D + 时间。和 Action Images 接近的工作用 multiview generation 产生复杂动态 4D 场景（可在任意相机位姿/时刻 replay）。**但在 robotic 任务上，4D generation 通常局限于固定单视角**。

💡 **批注**：这一节解释了"multi-view"的技术血统 —— Action Images 把 4D generation 的多视角思路带进了 robot manipulation。对 Uni-WAM 调研价值不大，**可略读**。

---

## 💡 §2 整段 takeaway

| 子节 | Action Images 的 positioning |
|---|---|
| §2.1 Robotics WM | 区别于"两阶段 + action module"派，主打"backbone 自己当 policy" |
| §2.2 Generalist Policy | 区别于 VLA/DP/RL，走"video model 启发的 policy"路线但更彻底 |
| §2.3 4D Generation | 借鉴 multi-view 4D generation 的技术血统 |

🔥 **Uni-WAM 调研记录**：§2.1 提到的 **DreamZero** 已在 backlog（b 类候选），其他引用没挖出新的近期具身 AC-WM 候选。

---

[← §1 Introduction](01-introduction.md) | [§3 Method →](03-method.md)
