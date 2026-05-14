[← 返回 EA-WM 主页](../README.md)

# §1 Introduction

## ¶1 · video model 成为 robotic world modeling 的自然基座

> "Recent video foundation models have rapidly improved... From latent video diffusion models... to diffusion-transformer systems such as CogVideoX and the open Wan series... Since robot manipulation is fundamentally a process of predicting how actions change visual states over time, such spatiotemporal priors make video generation models a natural foundation for robotic world modeling."

**翻译**：video foundation model 进步快（CogVideoX、Wan 系列等）。**robot manipulation 本质上就是"预测 action 如何随时间改变视觉状态"** —— 所以 video 生成模型的时空先验天然适合做 robotic world modeling。

## ¶2 · 现有 WAM 的盲区：只做正问题，不做逆问题

> "Meanwhile, recent world-action models have begun to jointly tune video generation and action modeling... However, these works predominantly treat video generation merely as an auxiliary representation to optimize action prediction. Consequently, they insufficiently explore the crucial inverse problem: how to effectively leverage control signals to guide accurate video synthesis."

**翻译**：现有 WAM 联合训 video + action，**但主要把 video 当辅助表示来优化 action 预测**。它们没充分探索**关键的逆问题** —— 怎么用控制信号引导**精确的 video 合成**。

🔥 **Uni-WAM 关联**：注意 EA-WM 把问题分成两个方向：
- **正问题**："video 帮 action"（现有 WAM 做的）
- **逆问题**："action 引导 video"（EA-WM 做的）

→ Uni-WAM 关心的"WM 能不能 follow off-expert action"其实是**第三个方向** —— 不是"action 引导 video 质量好不好"，而是"action 偏离 expert 时 video 还可不可信"。

## ¶3 · 根因：domain misalignment

> "A fundamental obstacle lies in the domain misalignment between low-dimensional control signals and high-dimensional video synthesis. Standard practices typically inject raw joint-space parameters, end-effector vectors, or abstract tokens as conditionings. While computationally compact, these representations are heavily tied to specific robot embodiments and carry limited spatial context. They force the video generator to implicitly deduce cross-domain kinematics."

**翻译**：根本障碍 = **低维控制信号和高维 video 合成之间的域错配**。标准做法注入 raw joint 参数 / 末端向量 / 抽象 token —— 虽然计算紧凑，但**强绑定特定 embodiment、空间上下文有限**，强迫 video generator 隐式推断跨域 kinematics → 经常渲染不准机器人几何或捕捉不到细微物理交互。

> "Furthermore, conventional architectures often lack an effective interaction mechanism between the action and video pathways."

**翻译**：另外，传统架构**缺乏 action 和 video 两条路径之间的有效交互机制** → generator 忽略 robot-object 之间的细粒度交互信息。

→ EA-WM 的两个 contribution 各打一个：KVAFs 打"域错配"，event-aware fusion 打"缺交互机制"。

## ¶4 · 两个贡献

> "At the representation level, EA-WM constructs KVAFs by lifting low-dimensional robot actions and kinematic states into the target camera view... At the architectural level, EA-WM augments a diffusion-transformer backbone with a dedicated KVAF branch and interval-based bidirectional fusion."

| 层面 | 贡献 |
|---|---|
| **表示层** | **KVAFs** —— 把低维 action + kinematic state lift 到相机视角，渲染 depth-aware 手臂结构 / joint landmark / gripper 几何 / 末端 heatmap / pose 线索 |
| **架构层** | **KVAF branch + interval-based 双向 fusion** —— 由 **Event-Difference Latent Supervision (EDLS)** 驱动的 event-aware 机制 |

---

## 💡 §1 整段 takeaway

EA-WM 的故事链：
1. video model 是 robotic WM 的自然基座
2. 但现有 WAM 只做正问题（video 帮 action），没做逆问题（action 引导 video）
3. 根因：低维 action 和高维 video 域错配 + 缺交互机制
4. 解法：KVAFs（打域错配）+ event-aware fusion（打缺交互）

🔥 **Uni-WAM 视角**：EA-WM 提出的"正问题 vs 逆问题"二分很有启发，但**两个方向都还在"expert 分布内"**。Uni-WAM 的 off-expert action 评估是第三个独立维度 —— 既不是"video 帮 action"，也不是"action 引导 video 质量"，而是"**action 偏离 expert 时 WM 还可不可信**"。

---

[← §0 Abstract](00-abstract.md) | [§2 Related Work →](02-related-work.md)
