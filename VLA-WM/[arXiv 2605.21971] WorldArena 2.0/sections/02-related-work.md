[← 返回主页](../README.md)

# §2 Related Work

## §2.1 Embodied World Models（具身世界模型）

> "Existing approaches fall into three broad categories."

**翻译**：现有具身 WM 方法分三大类：

| 类 | 做法 | 代表 |
|---|---|---|
| **① 紧凑 latent 动力学** | 学低维 latent dynamics 做控制/RL，提升仿真采样效率 | **Dreamer 家族**（DreamerV3 等）|
| **② 大规模生成模型** | 用大生成模型做 action-conditioned 视频预测（diffusion / 自回归），条件是 agent 动作 + 语言指令 | 各类 video WM |
| **③ 物理/交互保真增强** | 用 physics-informed 目标 + 联合视觉-动作建模，强化物理和交互保真度 | physics-informed WM、WAM |

> "Together, these directions reflect a shift from low-dimensional dynamics to visually grounded, action-aware prediction."

**翻译**：三个方向合起来，体现了从「低维动力学」到「视觉扎实、动作感知的预测」的转变。

> "Despite these advances, visual realism does not guarantee physical validity. EWMs can produce plausible-looking rollouts yet violate basic physical rules and accumulate errors over long horizons. Limited human demonstration data further reduces robustness under closed-loop sequential interactions."

🔥 **翻译（关键）**：尽管有进步，**视觉逼真不等于物理有效**。具身 WM 能产出看起来合理的 rollout，却违反基本物理规则、并在长程下累积误差。**有限的人类示教数据进一步削弱了它在闭环序列交互下的鲁棒性。**

> 所以评估 EWM 不能只看生成质量 —— 要看它能否捕捉**物理扎实、对具身决策有用的多模态动力学**。这就引出了「需要更丰富感官模态 + 交互/闭环评估 + 真机平台」的 benchmark。

## §2.2 Benchmarks for Embodied World Models（具身 WM 的 benchmark）

> "Standard video generation metrics emphasize perceptual quality but largely overlook physical realism and action-relevant fidelity."

**翻译**：标准视频生成指标重感知质量，**忽略物理真实性和「动作相关的保真度」**。

近期 benchmark 分三型：

| 型 | 代表 | 测什么 | 短板 |
|---|---|---|---|
| **① 感知/时空质量** | **EWMBench** | 场景一致性、运动正确性、语义对齐 | 不评估「生成动力学能否支撑具身决策」|
| **② 功能效用** | WorldSimBench（用 inverse dynamics 把视频转控制信号）/ WorldEval / WoW-World-Eval | policy 评估、action planning | 评估限于固定 policy / 单步 planning |
| **③ 统一框架** | **WorldArena（1.0）** | 联合测感知质量 + data engine / policy evaluation / action planning | 仍以视觉仿真为中心、下游用途有限 |

> "Despite these advances, existing benchmarks have important limitations. They rely exclusively on visual inputs, ignoring tactile feedback... Their functional evaluations are largely restricted to fixed policies or single-action planning... Furthermore, most assessments remain simulator-only, leaving the sim-to-real gap largely unexamined."

**翻译**：现有 benchmark 三个老问题（再次呼应 §1 的三局限）：① 只靠视觉输入，忽略触觉；② 功能评估限于固定 policy / 单步 planning，没测「连续交互式 policy 训练且不累积误差」；③ 大多 simulator-only，sim-to-real gap 没查。

> "WorldArena 2.0 addresses these gaps by incorporating visuotactile modalities, enabling interactive reinforcement learning, and evaluating performance across both simulated and real robotic platforms."

**翻译**：2.0 通过纳入 visuotactile 模态、支持交互式 RL、跨仿真+真机评估，来补这三个洞。

---

## 💡 §2 批注（Uni-WAM 视角）

- §2.1 那句 **"Limited human demonstration data further reduces robustness under closed-loop sequential interactions"** 几乎是我们 proposal 论点的另一种说法 —— **训练数据全是示教（expert），导致闭环下不鲁棒**。可以直接引用。区别在于：它把锅归到「数据量有限」和「闭环累积误差」，我们把锅归到「数据 action 分布偏窄（只有 expert）+ 从没在 off-expert action 下验证过」。
- §2.2 的三型 benchmark 分类，可直接搬进我们 proposal 的「相关工作」表，把 WorldArena 1.0/2.0 都归到「③ 统一框架」，然后指出**它们都缺第四型：off-expert action 下的可靠性评估**。

---

[← §1 Introduction](01-introduction.md) | [§3 Method →](03-method.md)
