[← 返回主页](../README.md)

# §5 Conclusion and Future Work

> "We introduce WorldArena 2.0, a comprehensive benchmark built on WorldArena that extends the evaluation of embodied world models across three key dimensions: modality, functionality, and platform. It incorporates visuotactile sensory inputs, interactive reinforcement learning tasks, and real-world robotic testbeds. Experiments across 12 state-of-the-art models reveal substantial sim-to-real gaps and identify critical areas for improving embodied world model design."

**翻译**：提出 WorldArena 2.0 —— 基于 WorldArena 的综合 benchmark，沿模态/功能/平台三轴扩展具身 WM 评估，纳入**视触觉感官输入、交互式 RL 任务、真机 testbed**。在 12 个 SOTA 模型上的实验**揭示了显著的 sim-to-real gap**，并指出了改进具身 WM 设计的关键方向。

> "Looking ahead, we plan to expand the benchmark to include additional sensory modalities, increase task complexity and diversity, and explore more challenging real-world scenarios."

**翻译（未来工作）**：未来计划 —— 纳入更多感官模态、增加任务复杂度与多样性、探索更具挑战的真实场景。

---

## 📌 整篇 paper 读完的高层 takeaway

### WorldArena 2.0 是什么？
不是新模型，是**新 benchmark**。在 WorldArena（1.0）基础上沿三轴扩张：模态（+触觉）/ 功能（+在线 RL 环境）/ 平台（+真机）。

### 和 1.0 的核心区别？
1. **模态**：UniVTAC 触觉管线（Tactile VAE + 双流 WM + Action Diffusion Head，即插即用）
2. **功能**：把 WM 当 RL 环境（POMDP proxy + GRPO 闭环训 policy）
3. **平台**：RoboTwin 2.0 + LIBERO（仿真）+ AgileX ALOHA（真机），构成 sim-to-real 梯度
4. **16 指标**：基本沿用 1.0，唯一实质改动是 `Action Following` → **`Action Response Sensitivity`**

### 最重要的实验发现？
- **sim-to-real usability gap 巨大**：data engine 没一个 WM 超真实示教数据；task success 跨仿真器相关、跟真机一比就崩（Spearman 0.348）。
- **视觉保真 ≠ 动力学建模**：商用模型视觉强但 physics adherence 不占优。
- **通用大模型触觉预测反超专用具身模型**（Wan2.2）。
- **长程力控仍是 WM 通病**（Lift Bottle 全 0%）。

### 测过 off-expert / OOD action 吗？
**❌ 没有**。模态/功能/平台三轴都扩了，但评估 action 仍全是 expert/GT trajectory。「off-expert action 下的可靠性」依旧是空白轴。

### 对我们 proposal 的价值？
🔥🔥 **极高 —— 是我们 benchmark 的直接上游 + baseline 来源**：
1. proposal 已收录 WorldArena，2.0 必须跟踪
2. ⭐ `Action Response Sensitivity` 可能和我们「Action Following Fidelity」撞概念 → **必须查清定义**（决定是对手还是 baseline 组件）
3. sim-to-real gap 给「分布内好 ≠ 部署可靠」官方背书
4. WM 当 RL 环境（闭环、累积误差、action-consistency）是同源痛点的另一切面
5. 明确区分 WM vs WAM，背书我们的分类轴
6. 12+2 模型 baseline 名单 + 三平台评估设置可照搬

---

## 🔥 拷打（2 题，格式 B）

### Q-WA2.1

**问**：WorldArena 2.0 已经把评估扩到了「真机 + 在线 RL 环境」，而且 §4.2 里 policy 在 WM 环境中做 RL 探索时，**显然会产生很多非专家动作（off-expert action）**。那是不是说 2.0 其实已经隐式测了 off-expert action 下 WM 的可靠性？我们 proposal 的「Action Following Fidelity」是不是被它覆盖了？

**答**：

> **没有被覆盖。2.0 在 RL 环境里确实让 policy 探索出了 off-expert action，但它从未「直接量化」WM 在这些 action 下的保真度 —— 它只看了一个被层层稀释的间接信号。**
>
> **1. 它测的是「最终 policy 成功率」，不是「WM 在 off-expert action 下的保真度」**
> §4.2 的核心指标是「用 WM 当 RL 环境训出来的 policy，拿到真 RoboTwin 里跑的成功率」。这个数字里掺了太多东西：reward model 准不准、GRPO 收敛得好不好、policy 容量、SFT 初始化质量……WM 在 off-expert action 下「画得对不对」只是其中一个被稀释的因子。**就算 WM 在某些 off-expert action 下画崩了，只要 policy 最终没往那些区域收敛，成功率照样可以很高** —— 崩坏被掩盖了。
>
> **2. 它没有「系统性地」喂 off-expert action**
> RL 探索产生的 off-expert action 是**policy 自己采样出来的、围绕当前策略的局部扰动**，分布完全由 policy 决定，不可控、不全面。我们 proposal 要的是**主动、系统、分类地**喂 off-expert action（counterfactual / random-feasible / 大幅扰动），覆盖 expert 流形之外的区域 —— 这是 RL 探索永远到不了的地方（policy 没理由去采那些动作）。
>
> **3. 它的诊断粒度是「任务级」，我们的是「转移级」**
> 2.0 给的是「这个 WM 当 RL 环境好不好用」的一个总分。我们要的是「喂这条具体的 off-expert action，WM 这一步的视频/轨迹保真度是多少」—— 可定位、可解释、可画失败模式图。
>
> **结论**：2.0 的 RL 环境评估和我们的 Action Following Fidelity 是**正交**的。它问「WM 当环境整体好不好用」，我们问「WM 在 off-expert action 下这一步可不可信」。前者是黑盒任务级、后者是白盒转移级。2.0 反而给了我们一个论据：**既然它都强调 WM 当 RL 环境要『action-consistent、不累积误差』，那就更应该有人专门去量化『WM 对 action 到底跟随得准不准』—— 而这恰恰没人做。**

### Q-WA2.2

**问**：2.0 把 `Action Following` 改名成了 `Action Response Sensitivity`，而且数值很小、商用强生成模型（Veo3.1/Wan2.6）反而最高。这个改名是不是说明 WorldArena 团队也意识到了「动作跟随」该测，已经在抢我们的坑了？

**答**：

> **不能下定论 —— 这正是必须立刻去 GitHub 查实现的原因。但从名字和数值能做两个判断：**
>
> **1. 名字从「Following」变「Response Sensitivity」是语义偏移**
> 「Following（跟随）」测的是「生成视频是否符合给定 action 的语义」—— 是个对齐度（越高越好，CLIP-based）。「Response Sensitivity（响应敏感度）」字面是「输出对 action 变化的敏感程度」—— 听起来是个**变化量/方差**（改 action，输出变多少）。这俩根本不是一回事。
>
> **2. 数值特征支持「敏感度」解读，但方向存疑**
> 全表 0.006~0.10 这种小数值，不像 0-1 对齐分，更像归一化后的「响应幅度」。而且 **Veo3.1/Wan2.6 这种文本生成主导的强模型反而最高** —— 如果它测的是「输出对 action 的敏感度」，高分可能恰恰是坏事（说明输出乱动、不稳定），也可能是好事（说明 action 真的有效）。**方向都不确定，必须看代码。**
>
> **3. 对我们的影响：不是抢坑，是给了参照系**
> 即使它真在测某种「动作响应」，也几乎肯定还是**在 expert 分布内**算的（所有 action 来自 GT trajectory），没有我们的 off-expert action 分类体系、没有 Gated 框架（先过 Visual Integrity Gate 再算 TA）、没有 counterfactual/random-feasible 的主动采样。**它最多是个单点指标，我们是一整套评估协议。** 但我们必须：(a) 查清它定义，(b) 在 proposal 里明确「我们和 Action Response Sensitivity 的区别」，(c) 可能把它当 baseline 指标之一纳入对照。
>
> **行动项**：去 WorldArena GitHub（1.0 的 code-reference 已 clone 在 `[arXiv 2602.08971] WorldArena/code-reference/`）找 `Action Response Sensitivity` 或 `action_following` 的实现，确认：输入是什么、是否扰动 action、扰动来自哪、分数公式、方向（高好还是低好）。

---

[← §4 Experiments](04-experiments.md) | [返回主页 →](../README.md)
