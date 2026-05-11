[← 返回 Ctrl-World 主页](../README.md)

# §4.2 Using Ctrl-World for Policy Evaluation and Improvement 🔥🔥

> **本节阅读重点**：Ctrl-World 的**两个核心用例**。`Algorithm 1` 是 paper 最关键的算法 —— Uni-WAM 视角下要看里面**有没有 off-expert action 的影子**。

---

## ¶1 · Use Case 1：Policy Evaluation within World Model

> "Once a controllable and consistent world model is trained, we can conduct **policy-in-the-loop rollouts in imagination space**. Given an initial observation $o_0$ and instruction $l$, a policy π together with the world model W can generate a synthetic trajectory τ. The initial observation can be sampled from the validation dataset or recorded as a snapshot from a real-world setup. In our experiments, we **label each trajectory as a success or failure based on human preference judgments**. While recent works explore the use of Vision-Language Models as general-purpose reward models, we leave such extensions to future work."

**翻译**：

训练好 WM 后，可以做 imagination 里的 policy-in-the-loop rollout：
1. 给定初始观测 $o_0$ + 指令 $l$
2. policy π + world model W 共同生成合成轨迹 τ
3. $o_0$ 可以来自 validation set 或真实场景 snapshot
4. **每条 trajectory 用人类偏好判断打 success/failure 标签**

→ Reward 用 VLM 作为通用 reward model 是 future work。

🔥 **Uni-WAM 关联**：

| 维度 | Ctrl-World | Uni-WAM |
|---|---|---|
| 评估对象 | policy 的成功率 / 指令跟随率 | WM 本身的 action following 精度 |
| 评估方式 | **人类判 success/failure** | **GPR + TA 自动指标**（DINO/SAM3/NDTW...）|
| 评估场景 | **完整 task rollout**（policy 自由发挥）| **单步 / 短序列 action 跟随** |

**关键区别**：
- Ctrl-World 的 evaluation **是 task-level 的**（看 task 最终成功了没）
- Uni-WAM 的 evaluation **是 dynamics-level 的**（看 WM 对每段 action 的预测对不对）

→ 两者**测的不是同一件事**。Ctrl-World 假设 "task 成功 ↔ WM 准"，Uni-WAM 拆解了这个假设。

---

## ¶2 · Use Case 2：Policy Improvement with Synthetic Data

> "Beyond evaluation, the world model enables **searching for successful synthetic trajectories** to improve policy performance."

🔥 **Uni-WAM 关联**：注意"successful synthetic trajectories"。**只用合成成功的，丢掉失败的** —— 这就是 §1 提到的"只用 expert-like data 改 policy"的范式。

---

## ¶3 · Algorithm 1：World Model Rollout and Policy Improvement

> 原文（伪代码）：

```
Algorithm 1 World Model Rollout and Policy Improvement
Given: policy πθ, action perturbation function εa, world model W, task instructions [l0, ..., lM]
       with initial obs [o0_0, ..., o0_M], synthetic dataset Ds, interaction step N, action horizon H.

1: for i = 0 to M do
2:     τ = [o0_i]
3:     for j = 0 to N do
4:         Current observation: o_t = τ[t] where t = j * H
5:         Sample action from perturbed policy:
              a_{t+1:t+H} = πθ(o_t, l, εa)              ▷ For diverse rollouts
6:         Prepare history context: h = [o_{t-km}, ..., o_{t-2m}, o_{t-m}]
7:         Make predictions with world model: o_{t+1:t+H} = W(h, o_t, a_{t+1:t+H})
8:         Add predictions into trajectory: τ = τ ∪ o_{t+1:t+H}
9:     end for
10:    Judge success of τ based on human-preference. Add τ into Ds if success.
11: end for
12: Finetune πθ with Lθ = E_{o_t, a_{t:t+H} ~ Ds} ||πθ(o_t, l) - a_{t:t+H}||^2
```

**翻译 + 拆解**：

| 步骤 | 内容 | Uni-WAM 视角 |
|---|---|---|
| 5 | **"perturbed policy"** $\pi_\theta(o_t, l, \epsilon_a)$ 输出 action | 🔥🔥 **关键**：有"perturbed policy"出现，但**扰动加在 policy 上**，不是直接扰动 action |
| 6 | 准备稀疏历史 context | 工程细节 |
| 7 | WM 预测 future obs | 标准 AC-WM 用法 |
| 10 | **"based on human-preference"** 打标 + **只留 success** | 🔥 **完全跳过失败 trajectory**，只用 successful 做 SFT |
| 12 | 用 success trajectory 做 behavior cloning loss | 标准 SFT，不是 RL |

---

## ¶4 · 关于 "structured perturbations"（Ctrl-World 提到的"加扰动"是什么意思）

> "Specifically, we can (i) **rephrase the instructions**, since VLA policies tend to be steerable, exhibiting different behaviors in response to different instructions; or (ii) **reset the policy to random initial states within the world model**, which leads to diverse initial observations. Starting from a set of downstream tasks with language instructions $[l_0, ..., l_M]$, we collect synthetic rollouts and score them based on human preference. To improve the policy performance, we fine-tune the policy on successful trajectories."

**翻译**：

为了产生多样化 rollout，Ctrl-World 引入两种**结构化扰动**：
1. **Rephrase the instructions**：改写语言指令（用 LLM API）
2. **Reset policy to random initial states**：重置机械臂到随机初始位置

→ 然后跑 rollout、用人类偏好打分、留 success 来 SFT。

🔥🔥🔥 **Uni-WAM 视角 —— 极其关键的发现**

**Ctrl-World 的"扰动"加在 input 上，不是 action 上**：

| 扰动维度 | Ctrl-World | Uni-WAM 5 类 |
|---|---|---|
| 改语言指令 | ✅ 做了 | ❌ 不是 Uni-WAM 关心的 |
| 改初始 obs | ✅ 做了 | ❌ Uni-WAM 关心 off-policy obs，方向类似但目标不同 |
| **改 action**（直接扰动 policy 输出的 action）| ❌ **没做** | ✅ **5 类 off-expert 的核心** |

→ Ctrl-World 用 instruction 多样性和初始 obs 多样性来"诱导 policy 产生不同行为"，**但 policy 一旦决定要做什么，输出的 action 仍然是它的"自然 action"**，不会被强制扭曲。

→ **这意味着 Ctrl-World 实际跑的 WM 输入仍然在 expert-like action 分布内**。它的 WM 没机会在 off-expert action 上学也没机会在 off-expert action 上被评估。

→ **这正是 Uni-WAM 的关键 gap**：Uni-WAM 主张直接给 WM 喂 5 类 off-expert action，看它能不能 follow。Ctrl-World 跳过了这一步。

---

## ¶5 · Action perturbation function $\epsilon_a$ 是什么？

注意 Algorithm 1 第 5 行：`a_{t+1:t+H} = πθ(o_t, l, εa)`

这个 $\epsilon_a$ 是"action perturbation function"。**但 paper 在 §4.2 里没具体定义它**。结合下文（§5.4）和 ¶4 描述，可以推断 $\epsilon_a$ 实际是：
- "用 LLM rephrase instruction"（实际扰动语言）
- "reset 到随机初始位置"（实际扰动初始 obs）

→ **不是直接扰动 action**！

⚠️ **Ctrl-World 论文里这个符号有点 misleading**：`εa` 字面看起来是"扰动 action"，但实际它扰动的是 policy 的 input（instruction + initial obs），让 policy **自主输出更多样的 action**，不强行扭曲 action 本身。

🔥 **Uni-WAM 调研记录**：
- Ctrl-World 没有"action 扰动"机制
- 即使 algorithm 写了 $\epsilon_a$，实际是 instruction/obs 扰动
- → **Uni-WAM 的 A+B action 采样方案在 Ctrl-World 范式里没有对应物**

---

## 💡 §4.2 整段 takeaway

Ctrl-World 的两个用例都建立在一个**强假设**上：
> **"policy 自主输出的 action 已经覆盖了 WM 训练分布"**

具体做法：
- **Eval**：让 policy 自由 rollout（不强制 action 多样化）→ 看 task 成功率
- **Improve**：用 instruction + obs 扰动诱导 policy 产生不同**自然 action** → 留 success → SFT policy

**Uni-WAM 视角的核心质疑**：
- 如果 policy 是早期 RL checkpoint（sub-optimal），它输出的 action 离 expert 远 → WM 没见过 → 预测不可靠 → eval 不可信
- Ctrl-World 测的 3 个 policy（π0/π0-FAST/π0.5）都是 **expert-quality**，所以 WM 在 ID 上 work，evaluation 也 work
- **但如果换上离 expert 更远的 policy 呢？** Ctrl-World 没测，也没办法在自己的框架里测

→ 这就是为什么 Uni-WAM 必须直接给 WM 喂 off-expert action 做诊断，**不能依赖 policy 自主产出**。

---

## 🤔 §4.2 留下的问题

| Q | 在哪里查 |
|---|---|
| Ctrl-World 测的 policy 中，最差的那个离 expert 多远？ | §5.3 + §5.4 |
| "human-preference judgment" 怎么操作？多少 trial？ | §5.3 |
| ranking alignment 的相关系数具体多少？ | §5.3 Figure 7 |

---

[← §4.1 Learning World Model](04-method-learning.md) | [§5.1 Experiment Setup →](05-experiment-setup.md)
