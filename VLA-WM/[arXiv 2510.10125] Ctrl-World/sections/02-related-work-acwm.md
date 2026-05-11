[← 返回 Ctrl-World 主页](../README.md)

# §2.2 Action-Conditioned World Models（Related Work）

> **本节阅读重点**：看 Ctrl-World 怎么定位自己 vs 其他 AC-WM。**§2.1 Video Generation Models for Robotics 跳过**（和我们工作无关），只读 AC-WM 这一段。

---

## 原文 + 翻译

> "Although pretrained video models are powerful, they are often **only conditioned on high-level language instructions**. Nonetheless, some prior works have explored using action-conditioned predictive models, both in low-dimensional state spaces (Nagabandi et al., 2020) and with image observations (Hafner et al., 2019; 2020; Hansen et al., 2022; Wu et al., 2023; Oh et al., 2015; Huang et al., 2025). Many of these approaches learn **task-specific models** (Hafner et al., 2019), while we focus on training **generalist, multi-task world models**. Building on early works (Finn & Levine, 2017; Ebert et al., 2018; Xie et al., 2019; Dasari et al., 2019; Yang et al., 2023; Wu et al., 2024) as well as more recent approaches that leverage **diffusion** (Quevedo et al., 2025; Chen et al., 2024; Ball et al., 2025 [Genie 3]; Gao et al., 2025; Ren et al., 2025; Hafner et al., 2025) and **frame-level action conditioning** (Zhu et al., 2024), we propose a model that incorporates multi-view prediction, long-horizon temporal coherence, and fine-grained controllability."

**翻译**：

虽然 pretrained video model 很强，但它们通常**只 conditioned on 高级语言指令**。不过有些 prior work 探索了 action-conditioned predictive model：
- **低维 state space**：Nagabandi 2020
- **image observation** + **任务专属 model**：Hafner DreamerV1-V3、Hansen TD-MPC、Wu、Oh ATARI、Huang

这些大多是**task-specific 模型**。Ctrl-World 不同 —— 它训练 **generalist, multi-task world model**。

Ctrl-World 站在两条研究线的肩膀上：

1. **早期 AC-WM**：Finn & Levine 2017、Ebert 2018、Xie 2019、Dasari 2019、Yang 2023、Wu 2024
2. **近期 diffusion 类 AC-WM**：Quevedo 2025、Chen 2024、**Ball 2025（Genie 3）**、Gao 2025、Ren 2025、Hafner 2025
3. **Frame-level action conditioning 起源**：**Zhu et al., 2024**

Ctrl-World 的贡献 = 把上面这些方向**整合 + 加入多视角 / 长时一致 / 细粒度控制**。

---

## 💡 这一节对 Uni-WAM 调研的价值

### 🔥 它给了我们一份现成的 AC-WM 候选清单

paper 引用的这一票工作里，**很多都是我们要广撒网调研的对象**。从 backlog 角度梳理：

| 工作 | Ctrl-World 引用号 | Uni-WAM 调研归类（初步）|
|---|---|---|
| **DreamerV1-V3** | Hafner 2019/2020/2025 | a 类候选（经典 AC-WM 系列）|
| **TD-MPC** | Hansen 2022 | a 类候选 |
| **Daydreamer** | Wu 2023 | 待确认 |
| **Genie 3** | Ball 2025 | a 类候选（已在 backlog）|
| **Zhu 2024**（frame-level action conditioning 起源） | Zhu 2024 | a 类候选（关键 method 起源）|
| Yang 2023 | Yang 2023 | a 类候选 |

→ **Action**：把这些条目加进 Uni-WAM-调研/README.md 的 backlog。等 Ctrl-World 读完后做。

### 🔥 task-specific vs generalist 的区分

Ctrl-World 强调自己是 **generalist multi-task** AC-WM，区别于 Dreamer 这类 task-specific 模型。

**Uni-WAM 视角**：这个区分本身**没碰 off-expert action 这个轴**。
- Task-specific AC-WM：每个 task 训一个 WM
- Generalist AC-WM：一个 WM 跨多个 task
- **但两者都可能"只在 expert action 上训"**

→ Generalist 不代表 action 分布广 —— 这个区分容易被混淆。**Uni-WAM 要明确区分"task 维度泛化"和"action 分布维度泛化"是两个独立的轴**。

### 🔥 diffusion 类 AC-WM 是当前主流方向

Ctrl-World 列的"近期 approach"里 6 个 diffusion-based AC-WM 提到 5 个。说明：
- **Diffusion 是当前 AC-WM 的主流架构**
- Uni-WAM 设计用 Wan2.2-TI2V-5B（flow matching diffusion）正好踩在这条主流线上

---

## 🤔 §2.2 留下的问题

| Q | 备注 |
|---|---|
| Zhu et al. 2024 的"frame-level action conditioning"具体是什么？ | Ctrl-World 直接引用作为自己的技术起源 |
| Genie 3（Ball 2025）作为 AC-WM 是 a 类还是 c 类？ | DeepMind 出品，可能没开源 → d 类 |
| DreamerV1-V3 的训练数据范围如何？ | 调研时需查 |

---

[← §1 Introduction](01-introduction.md) | [§3 Problem Formulation →](03-problem-formulation.md)
