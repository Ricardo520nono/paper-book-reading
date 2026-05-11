# Ctrl-World: A Controllable Generative World Model for Robot Manipulation

**作者**：Yanjiang Guo, Lucy Xiaoyang Shi, Jianyu Chen, Chelsea Finn
**机构**：Tsinghua + Stanford
**链接**：[arXiv 2510.10125](https://arxiv.org/abs/2510.10125) · 发表 2025/10（v3 2026/03）
**Uni-WAM 调研分类**：**a 类（原本就是 AC-WM）**

> **本笔记从 Uni-WAM 调研视角阅读**。读完后**提炼到 Uni-WAM 视角**单独写在 [`../Uni-WAM-调研/notes/Ctrl-World.md`](../Uni-WAM-调研/notes/Ctrl-World.md)（待写），并补充到 [`../Uni-WAM-调研/README.md`](../Uni-WAM-调研/README.md) 的调研主表。

---

## 📌 一句话总结

> Ctrl-World 是一个**可控、多视角**的 generative world model，把 generalist robot policy 的 action chunk 当条件输入，用来在 closed-loop 里**评估 / 改进 policy** —— 在 DROID 数据上训，**作为 policy evaluator** 时 ranking 与真机一致，**作为 data synthesizer** 时把 π0.5 在新任务的成功率从 38.7% → 83.4%。

---

## 🎯 Uni-WAM 视角的学习目标

读完这篇我必须能回答：

### Q1：Ctrl-World 的 action 是怎么注入的？（关键，决定它是不是真正的 AC-WM）
> 答案在 §4.1 "Frame-level Action Conditioning"。
>
> **答**：Action 表示是 **Cartesian-space 6D pose**（不是 joint angle，不是 latent action）。注入机制是 **frame-level cross-attention**：在 spatial transformer 内部，**每一帧的 visual token 通过 cross-attention attend 到该帧对应的 pose embedding**。Policy 输出的原始 action 序列 `[a_{t+1:t+H}]` 经过 forward kinematics 等变换转成 Cartesian pose `[a'_{t+1:t+H}]`，再和过去的真实 pose `[q_{t-km}, ..., q_t]` concatenate 喂进 cross-attention。
>
> **新增参数**：只有一个 **action-projection MLP**，backbone 是 **Stable Video Diffusion 1.5B**（其他参数 inherit）。
>
> → **结论**：是真正的 AC-WM。

### Q2：Ctrl-World 训练数据来自哪？只有 expert 吗？
> 答案在 §5.1 Experiment Setups（DROID dataset）。
>
> **答**：**不全是 expert**（比预判稍好），但**仍远不够 Uni-WAM 的标准**。
>
> 数据集 = **DROID**：95,599 条真机遥操作轨迹，564 个场景。**76k success + 19k failure**。Paper 自己强调 "diverse actions and failure data is crucial"。
>
> ⚠️ **关键 nuance**：DROID 的 failure 是 **human-attempted failure**（人类操作时不小心失败的轨迹），仍在 "human-feasible action distribution" 内。
>
> 对应 Uni-WAM 5 类的覆盖：
> - ✅ 部分覆盖 **Perturbed expert** + **Exploratory**（人类失败案例）
> - ❌ 完全不包含 **Counterfactual / Random-feasible / Adversarial**

### Q3：Ctrl-World 测过 off-expert / OOD action 吗？
> 答案在 §5.3 + §6。**这是 Uni-WAM 最关心的 gap**。
>
> **答**：**❌ 没测过 action 维度的 OOD**。
>
> Ctrl-World 测过的 OOD 都是别的维度：
> - ✅ **视觉 OOD**：新相机位姿、新场景（§5.3）
> - ✅ **语言 OOD**：novel instructions（§5.4 + §6）
> - ❌ **行为 OOD**：测的 3 个 policy（π₀ / π₀-FAST / π₀.₅）都是 **expert-quality VLA**，输出的 action 都在 expert 分布内
>
> 🔥 **§5.3 那句关键自承认**：
> > "some failure trajectories are included in the DROID dataset, there are still **many failure modes outside the data distribution**."
>
> → Ctrl-World 自己**明确点出**了这个 gap，但**归为 data engineering 问题**（"收集更多数据填补"），而**不是 method 问题**。§6 Conclusion 里这个 gap **甚至没被升华为 limitation**。
>
> → 这就是 Uni-WAM 接住的位置：把 data engineering 问题升级为方法论问题（5 类 off-expert benchmark + IDM 反向正则化）。

### Q4：Ctrl-World 怎么处理"循环论证"问题？（policy 训练数据 → 评估的 WM 也是这个数据）
> 关键看 §4.2 + §5.3 如何论证 "WM evaluation ↔ real evaluation" 的一致性。
>
> **答**：Ctrl-World **通过 §5.3 的 ranking alignment 实验间接论证**，但**有重大局限**。
>
> **实验设定**：
> - 3 个 policy（π₀ / π₀-FAST / π₀.₅）+ 7 个 task
> - 真机和 WM 用**相同初始 obs**，分别 rollout
> - 比较 instruction-following rate 和 success rate
>
> **结果**（§5.3 Figure 7 回归方程）：
> - Instruction-following: **y = 0.87x − 0.04**
> - Success rate: **y = 0.81x − 0.11**
>
> → WM 排名和真机**正相关**，但 WM 偏**悲观**（斜率 < 1，截距 < 0）。
>
> ⚠️ **循环论证的"漏洞"**：
> - 实验用的 3 个 policy 都是 **expert-quality**（行为在 DROID 分布内）
> - **没有验证**当 policy 偏离 expert 时（如早期 RL checkpoint / sub-optimal policy）alignment 还成立吗
> - 也就是：**Ctrl-World 只证明了"在 ID setting 下 WM eval ≈ real eval"**，**没证明 "OOD setting 下 WM 仍可信"**
>
> → 这是 Uni-WAM 关心的核心质疑：当 policy 跑到 expert 分布外时，循环论证就崩塌了。Ctrl-World 没碰这个角落。

---

## 📖 批读导航

| # | Section | 内容 | Uni-WAM 相关性 | 状态 |
|---|---|---|---|---|
| - | [**summary.md**](sections/summary.md) | 🌟 **串讲速读**（图+几句话读懂全篇）| 🔥 入口 | 🟡 逐点积累中 |
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | 🔥 必读 | ✅ 完成 |
| 1 | [01-introduction.md](sections/01-introduction.md) | Introduction：要解决的问题 + Ctrl-World 自己的 pitch | 🔥 必读 | ✅ 完成 |
| 2.1 | ~~02-related-work-videogen.md~~ | Video Generation Models | 选读 | ⏭️ **跳过** |
| 2.2 | [02-related-work-acwm.md](sections/02-related-work-acwm.md) | Action-Conditioned World Models（看它怎么定位自己 vs 其他 AC-WM）| 🔥 必读 | ✅ 完成 |
| 3 | [03-problem-formulation.md](sections/03-problem-formulation.md) | Problem Formulation（W 的输入输出形式定义）| 🔥 必读 | ✅ 完成 |
| 4.1 | [04-method-learning.md](sections/04-method-learning.md) | Multi-View + Pose Memory + **Frame-level Action Conditioning** | 🔥🔥 **核心** | ✅ 完成 |
| 4.2 | [04-method-policy-eval.md](sections/04-method-policy-eval.md) | Using Ctrl-World for Policy Evaluation and Improvement | 🔥🔥 **核心** | ✅ 完成 |
| 5.1 | [05-experiment-setup.md](sections/05-experiment-setup.md) | Setups（DROID 数据 / 训练细节）| 🔥 必读 | ✅ 完成 |
| 5.2 | ~~05-quality-analysis.md~~ | 视觉质量评估 | 选读 | ⏭️ **可跳**（基础指标）|
| 5.3 | [05-policy-evaluation.md](sections/05-policy-evaluation.md) | 实测：WM 评估 vs 真机评估 排名一致性 | 🔥🔥 **核心** | ✅ 完成 |
| 5.4 | ~~05-policy-improvement.md~~ | 用 WM 造合成数据训 policy（在 π0.5 上做了实验）| 选读 | ⏭️ **可跳**（不影响 a 类判断）|
| 6 | [06-conclusion.md](sections/06-conclusion.md) | Conclusion + Limitations | 🔥 必读 | ✅ 完成 |

**Push 策略**：只 push 上面 🔥 标记的 section，跳过的不写。

---

## 🚨 标记说明（看笔记时关注）

| 标记 | 含义 |
|---|---|
| 🔥 | Uni-WAM 强相关的段落 / 句子（直接影响 a/b/c/d 分类、或揭示 off-expert gap）|
| 💡 | Claude 批注（背景补充 / 对比 / 解释）|
| 🎯 | 必须能回答的问题 |
| 🤔 | 留下的疑问 |
| ⚠️ | 容易踩的坑 |

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第一篇
