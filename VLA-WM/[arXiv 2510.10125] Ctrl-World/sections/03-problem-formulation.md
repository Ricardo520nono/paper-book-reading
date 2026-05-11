[← 返回 Ctrl-World 主页](../README.md)

# §3 Problem Formulation

> **本节阅读重点**：看 Ctrl-World 怎么数学化定义 policy + world model 的交互形式。**两个公式决定了它是不是 AC-WM**。

---

## 原文 + 翻译

> "We aim to develop a world model that can predict the future outcomes of actions proposed by a generalist robot policy. A modern generalist policy π typically maps multi-view observations and language instructions into a sequence of actions. Specifically, robot observation $o_t = [I_t^1, ..., I_t^n, q_t]$ includes n camera views $[I_t^1, ..., I_t^n]$ and robot pose $q_t$, the policy outputs an H-step action chunk given an instruction $l$:"

公式 (1)：
$$a_{t+1}, a_{t+2}, ..., a_{t+H} \sim \pi(\cdot | o_t, l)$$

**翻译**：
现代 generalist policy π **把多视角观测和语言指令映射成 action 序列**。
- 观测：$o_t = [I_t^1, ..., I_t^n, q_t]$ —— n 个相机视角 + 机器人 pose
- 输入：当前观测 $o_t$ + 指令 $l$
- 输出：未来 H 步的 action chunk $a_{t+1}, ..., a_{t+H}$

---

> "Our goal is to use a world model W to predict the outcomes of executing each step in $A_t = [a_{t+1}, ..., a_{t+H}]$. To enable multi-step interaction with the policy in imagination space, W must generate future multi-view observations:"

公式 (2)：
$$o_{t+1}, ..., o_{t+H} \sim W(\cdot | o_t, A_t)$$

**翻译**：
World model W 的角色 = **给定当前观测 + action chunk，预测未来 H 步观测**。

🔥🔥 **Uni-WAM 关联** —— 关键公式

注意这个公式：
$$o_{t+1}, ..., o_{t+H} \sim W(\cdot | o_t, A_t)$$

**$A_t$（action chunk）是必须输入**，不是 optional！

- 对比 EWMBench：$V = v_{\text{norm}}(g_{\text{world}}(f_{\text{proc}}(I, L, T)))$，**T 是 optional**
- Ctrl-World：**$A_t$ 强制存在** —— 没 action 输入根本不在它的 problem formulation 里

→ Ctrl-World **从数学定义上**就是 **真正的 AC-WM**（a 类）。

---

> "Then the final prediction $o_{t+H}$ can be sent back to policy π to produce the next action chunk $A_{t+H} \sim \pi(\cdot | o_{t+H}, l)$. In this way, the policy and world model interact auto-regressively, enabling long-horizon rollouts entirely within imagination space."

**翻译**：
最后一帧预测 $o_{t+H}$ 送回 policy → policy 生成下一个 action chunk $A_{t+H}$ → 再送给 W 预测下一段 → 如此循环。

→ **policy 和 WM 自回归交互**，构成 imagination-space rollout。

💡 **批注**：这是 AC-WM 作为 simulator 的标准 closed-loop 设计：
```
policy π:  o_t → a_{t+1:t+H}        (action 生成)
WM W:      (o_t, a_{t+1:t+H}) → o_{t+1:t+H}  (状态预测)
循环：     o_{t+H} → policy → a_{t+H+1:t+2H} → WM → o_{t+H+1:t+2H} → ...
```

---

## 💡 §3 整段 takeaway

Ctrl-World 的数学定义干净直接：
1. Policy π：$(o, l) \to a$
2. WM W：$(o, a) \to o'$
3. 两者交替运行 = imagination rollout

**Uni-WAM 视角的关键判断**：

| 维度 | Ctrl-World |
|---|---|
| **AC-WM 分类** | **a 类**（formulation 上 $A_t$ 强制输入）|
| **Action 表示** | action chunk $A_t = [a_{t+1}, ..., a_{t+H}]$（H 步）|
| **数学上是否能接 off-expert action？** | **可以** —— 公式 (2) 接受任意 $A_t$，不限定它来自 expert |
| **实操中是否测过 off-expert action？** | ❓ 待 §5 验证。**Formulation 允许 ≠ 实验做了** |

🔥 **关键 nuance**：
- 在 problem formulation 这一层，Ctrl-World 是干净的 AC-WM
- **但训练数据 + 评估数据 是否包含 off-expert action 是另一回事**
- 等 §4.1 看 training objective，§5.1 看 training data，§5.3 看 evaluation 才能下定论

---

## 🤔 §3 留下的问题

| Q | 在哪个 section 找答案 |
|---|---|
| action chunk 的具体数据类型？joint angle？6D pose？ | §4.1 Frame-level Action Conditioning |
| H（chunk 长度）多大？ | §5.1 Setups（应该会给）|
| 公式 (2) 里的 W 是否在训练时见过 off-expert $A_t$？ | §4.1 Training Objective + §5.1 DROID 数据分析 |

---

[← §2.2 AC-WM Related Work](02-related-work-acwm.md) | [§4.1 Learning World Model →](04-method-learning.md)
