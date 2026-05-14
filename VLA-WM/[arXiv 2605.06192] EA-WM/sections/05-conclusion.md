[← 返回 EA-WM 主页](../README.md)

# §5 Limitations + §6 Conclusion

## §5 Limitations（paper 自己承认的）

> "EA-WM requires robot kinematics, camera calibration, and synchronized action-state logs to construct KVAFs. While this yields accurate, well-aligned action representations in simulation, real-world deployment may be affected by calibration errors, occlusions, sensor noise, or embodiment shifts. Current KVAFs mainly encode robot-side geometric motion cues, while object-state changes and robot-object interactions are captured indirectly."

**翻译**：paper 自己承认的 2 个 limitation：
1. **构造 KVAFs 需要 robot kinematics + 相机标定 + 同步的 action-state 日志** —— 仿真里准，但真机部署会受标定误差/遮挡/传感器噪声/embodiment shift 影响
2. **KVAFs 主要编码 robot-side 几何运动线索** —— 物体状态变化和 robot-object 交互只能通过 event-aware fusion + EDLS **间接**捕捉。未来方向：扩展 KVAFs 加 object-centric / contact-aware 视觉场

## §6 Conclusion

> "Unlike current WAMs that mainly optimize action prediction or policy learning, EA-WM focuses on preserving robot spatial motion and robot-object interaction dynamics in generated videos. It maps actions and kinematic states into Structured Kinematic-to-Visual Action Fields (KVAFs)... On WorldArena, EA-WM achieves the best P3CScore among compared models."

**翻译**：和现有 WAM（主要优化 action 预测或 policy learning）不同，EA-WM 聚焦**保住生成视频里的机器人空间运动 + robot-object 交互动态**。在 WorldArena 上 P3CScore SOTA。

🔥🔥 **Uni-WAM 视角 —— 关键对照**

EA-WM 自己承认的 limitation：**依赖相机标定 + KVAFs 只编码 robot-side 几何**。

**完全没提**：
- ❌ off-expert action 评估
- ❌ 训练/评估数据的 action 分布偏倚（WorldArena 的 RoboTwin 数据 action 都是 expert / GT trajectory）
- ❌ counterfactual / random-feasible / adversarial action 测试

→ **第九篇了，依旧没有一篇碰 off-expert action 这个轴**。

---

## 📌 整篇 paper 读完的高层 takeaway

### 是不是 AC-WM？
✅ 是真正的 AC-WM —— action（通过 KVAFs）是核心 conditioning input。

### 分类（a/b/c/d/e）？
**a 类 AC-WM**（真具身 robot manipulation），基于 Wan2.2-TI2V fine-tune（LoRA）。

### Action 注入方式？
**KVAFs** —— forward kinematics + camera projection 把 action + kinematic state 渲染成"整条手臂骨架 + joint landmark + gripper 几何 + 末端 heatmap + pose 轴"的相机对齐视觉场，然后用**独立 KVAF branch**（DiT full-depth copy）处理 + **event-aware 双向 fusion** 和 video branch 交互。

### 训练数据？
WorldArena 的 RoboTwin 数据（仿真），action 都是 expert / GT trajectory。

### 核心创新角度？
**用 action 引导精确 video 生成（逆问题）** —— 保住机器人几何 + robot-object 交互动态。KVAFs（画整条手臂）+ EDLS（帧差监督）是两个核心。

### 测过 off-expert / OOD action 吗？
**❌ 没有**。WorldArena 的评估 action 全来自 RoboTwin expert / GT trajectory。

### 这篇对 Uni-WAM 的价值？
🔥🔥 **极高 —— 是 Uni-WAM 最强的架构参照**：
1. **同 backbone（Wan2.2-TI2V）** —— 和 Uni-WAM proposal 完全一致
2. **双分支 + event-aware 双向 fusion ≈ Uni-WAM 的 MoT + Shared Attention** —— EA-WM 在 WorldArena 上 SOTA，**证明这个架构哲学 work**
3. **直接 baseline 对比**（Table 4：Ctrl-World / IRASim）—— Uni-WAM 可照搬评估设置
4. **WorldArena 是 proposal 已收录的 benchmark** —— EA-WM 的实验设置可直接复用
5. **EDLS（帧差 latent 监督）思路** —— 强迫模型关注"状态转换区域"，对 Uni-WAM 的"WM 真学 dynamics"目标有启发

---

## 🔥 拷打（1 题，格式 B）

### Q-EAWM.1

**问**：EA-WM 的"双分支（video branch + KVAF branch）+ event-aware 双向 fusion"和 Uni-WAM proposal 的"MoT（生成分支 + 动作分支）+ Shared Attention"看起来是同一个架构。**EA-WM 已经在 WorldArena 上证明这个架构 SOTA 了，那 Uni-WAM 的架构还有什么新意？Uni-WAM 是不是只是 EA-WM 的复制？**

**答**：

> **架构骨架确实高度相似，但 Uni-WAM 在三个层面有本质不同 —— 而且 EA-WM 的成功恰恰是 Uni-WAM 的"可行性证明"，不是"撞车"。**
>
> **1. 两个分支的"模态"不同**
> - EA-WM：video branch 和 KVAF branch **都是视觉模态**（KVAF 是渲染出来的 RGB 图，用同一个 VAE 编码）—— 本质是"两路视觉流的 fusion"
> - Uni-WAM：生成分支（Wan2.2-TI2V-5B，视觉）+ 动作分支（**Qwen3-VL-4B，是 VLM**）—— 本质是"视觉流 + 语言/动作理解流"的 fusion
> - → EA-WM 的 KVAF branch 只是"另一路视频"，Uni-WAM 的动作分支带**语义理解能力**。这是 MoT 真正"Mixture-of-Transformers"的含义
>
> **2. fusion 的"监督信号"不同**
> - EA-WM 的 event-aware fusion 由 **EDLS（帧差 latent 监督）** 驱动 —— 帧差监督让模型关注"哪里在变"
> - Uni-WAM 的 Shared Attention 没有 EDLS 这种东西，但 Uni-WAM 有 **IDM 反向正则化** —— 用 inverse dynamics 强迫 WM 学真实 action→obs 映射
> - → EDLS 关注"视觉变化区域"，IDM 反向正则化关注"action 和 obs 的因果一致性" —— **两者目标不同**。EA-WM 没有 IDM 这层
>
> **3. 评估的"action 分布"不同 —— 这是最根本的区别**
> - EA-WM 在 WorldArena 上 SOTA，但 WorldArena 的 action **全是 RoboTwin 的 expert / GT trajectory**
> - Uni-WAM 的核心是 **Action Following Fidelity benchmark** —— 主动喂 5 类 off-expert action（counterfactual / random-feasible / adversarial...）测 WM
> - → **EA-WM 证明的是"这个架构在 expert 分布内 work"**，Uni-WAM 要回答的是"**这个架构在 off-expert action 下还可不可信**" —— 这是 EA-WM 完全没碰的问题
>
> **结论**：Uni-WAM **不是 EA-WM 的复制**。EA-WM 的成功是 Uni-WAM 架构哲学的**正面背书**（"双分支 + 双向 fusion 在 Wan2.2 上能做到 WorldArena SOTA"），但 Uni-WAM 的独特性在于：(1) 动作分支是带语义的 VLM，不是另一路视频；(2) 有 IDM 反向正则化这个 EA-WM 没有的机制；(3) **最关键** —— Uni-WAM 评估的是 off-expert action 下的可靠性，而 EA-WM（和我们读过的所有 paper）都只在 expert 分布内评估。Uni-WAM 应该**把 EA-WM 当 baseline**，用 Action Following Fidelity benchmark 去测它 —— **EA-WM 在 WorldArena 上 78.13，但喂它 counterfactual action 会怎样？没人知道**。

---

[← §4 Experiments](04-experiments.md) | [返回主页 →](../README.md)
