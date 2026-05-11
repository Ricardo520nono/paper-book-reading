[← 返回 Ctrl-World 主页](../README.md)

# §5.3 World Model for Policy Evaluation 🔥🔥

> **本节阅读重点**：Ctrl-World 验证"imagination 排名 ≈ 真机排名"的实验。**这是 Uni-WAM 关心的 circular reasoning 问题最直接的实验现场** —— 看他们怎么验证，承认了什么 gap。

---

## ¶1 · 实验目标

> "In this section, we evaluate whether Ctrl-World can be used to evaluate the instruction-following ability of generalist robot policies and **accurately reflect their performance rankings in the real world**. We set up our own DROID platform and randomly place two third-person cameras around the workspace. Similar to how prior works have seen DROID policies generalize to new setups, we find that Ctrl-World, pretrained solely on the open-sourced DROID dataset, can **make accurate future predictions zero-shot in our newly configured scene with novel camera placements**."

**翻译**：

实验目标：验证 Ctrl-World 能否准确反映 policy 在真实世界的性能排名。
- 自己搭一个 DROID 平台，随机放 2 个 third-person 相机
- Ctrl-World（只在开源 DROID 上 pretrain）**零样本泛化**到新场景 + 新相机位姿

🔥 **Uni-WAM 关联**：
- "**accurately reflect rankings**" —— 这是 Uni-WAM 关心的核心场景：能否用 WM 替代真机做 policy evaluation
- "zero-shot in our newly configured scene with novel camera placements" —— Ctrl-World 在**视觉 OOD**（新场景 / 新相机）下泛化好。但**注意**：相机角度变了，**机械臂动作分布没变** —— policy 仍然按 DROID 学的方式动作

---

## ¶2 · Policies and Tasks

> "We evaluate **three publicly released policies**, $\pi_0$, $\pi_0$-FAST, and $\pi_{0.5}$, across diverse tasks including Pick-and-Place, Towel-Folding, Drawer, Wipe-Table, Close-Laptop, Pull-tissue and Stack tasks on our DROID platform. We initialize real-world and world model rollouts with the **same initial observations** and execute each policy, following Algorithm 1."

**翻译**：

| 维度 | 内容 |
|---|---|
| 测的 policy | **π₀ / π₀-FAST / π₀.₅**（3 个公开 VLA policy）|
| 任务 | Pick-and-Place / Towel-Folding / Drawer / Wipe-Table / Close-Laptop / Pull-tissue / Stack（7 个 task）|
| 真机 vs WM 对照 | **相同初始 obs**，分别在真机和 WM 里跑 |
| 指标 | instruction-following rate + success rate |

🔥🔥 **Uni-WAM 视角 —— 关键判断**：

**这 3 个 policy 都是 expert-quality**：
- π₀ / π₀-FAST / π₀.₅ 都是 Physical Intelligence 发布的 SOTA generalist policy
- 它们都**在大量真机数据上训过**，**已经是 expert-level**

**这意味着实验本质上是"expert-quality policy × ID setting"**：
- 视觉上有 OOD（新相机位姿）✅
- **行为上完全 ID**（policy 输出的 action 都在它训练分布内）⚠️

→ Ctrl-World 验证的是：**当 policy 行为本身在 expert 分布内时，WM 评估准**。

→ **没有验证**：当 policy 行为偏离 expert 时（比如早期 RL checkpoint），WM 还准吗？

→ 这就是 Uni-WAM 的核心 gap：**Ctrl-World 的 ranking alignment 只在 expert-quality policy 之间成立**。

---

## ¶3 · Quantitative Results (Figure 7)

paper 给出回归方程：

| 指标 | 真机 vs WM 回归方程 | 含义 |
|---|---|---|
| Instruction Following | $y = 0.87x - 0.04$ | WM 上的指令跟随率 ≈ 0.87 × 真机率 - 0.04 |
| Success Rate | $y = 0.81x - 0.11$ | WM 上的成功率 ≈ 0.81 × 真机率 - 0.11 |

**翻译**：

斜率 0.87 / 0.81 → WM 排名和真机**正相关但偏低估**。截距 -0.04 / -0.11 → WM 系统性低估了 policy 表现。

💡 **批注**：
- **正相关 + 单调** = ranking 保留（一个 policy 真机好，WM 上也好）
- **斜率 < 1** = WM 上的差异被压缩
- **截距 < 0** = WM 整体悲观

这意味着 **WM 不是完美 oracle，但作为 ranker 是 usable**。

🔥 **Uni-WAM 视角**：
- 这个相关系数是**在 expert-quality policy 集合上算的**（π₀ / π₀-FAST / π₀.₅）
- 这些 policy 之间的差异本身就不大 → ranking 容易对
- **关键问题**：如果加入一个"故意不太行的 policy"（比如 random 或早期 checkpoint），WM 还能 rank 对吗？Ctrl-World **没测**

---

## ¶4 · Paper 自己承认的 gap（关键！）

> "Our results show that policy's **high-level instruction-following behavior** in the world model is closely correlated with that observed in the real world. However, we notice some gaps in evaluating **low-level execution**, specifically in precise modeling of complex physics dynamics such as collisions, objects sliding away, rotations, etc. (e.g., interaction with laptop is imprecise in Figure 6). We also observe that generalist policies tend to keep retrying in the real world after failed attempts, **which the world model sometimes does not capture**. Although **some failure trajectories are included in the DROID dataset**, there are still **many failure modes outside the data distribution**. We expect that **collecting additional in-domain policy rollout data would improve the fidelity of the learned dynamics and narrow this gap**."

**翻译**：

Ctrl-World 自己承认 3 个 gap：
1. **Low-level execution 不准**：复杂物理（碰撞 / 物体滑动 / 旋转）建模不精
2. **Retry 行为捕捉不到**：真机里 policy 失败后会重试，WM 里有时捕捉不到
3. ⭐ **"some failure trajectories are included in the DROID dataset, there are still many failure modes outside the data distribution"** —— **承认 DROID 的 failure 分布有限**，还有很多 failure mode 在数据分布外

**Paper 的解法**：收集更多 in-domain policy rollout 数据，缩小 gap。

🔥🔥🔥 **Uni-WAM 视角 —— 这一段是最关键的对照**

Paper 自己点出："**still many failure modes outside the data distribution**"！

这就是翔哥 proposal 里讲的"循环论证 + 数据分布偏倚"的同一件事，但 **Ctrl-World 的处理方式完全不同**：

| 维度 | Ctrl-World 的做法 | Uni-WAM 的做法 |
|---|---|---|
| 诊断 | "需要更多 in-domain rollout 数据" | "WM 在 OOD action 上不可靠 → benchmark 暴露这点" |
| 解法 | **收集更多真机数据填补** | **设计 5 类 off-expert action + 用仿真器 + IDM 反向正则化** |
| 哲学 | 假设 "数据足够 → WM 可靠" | 假设 "WM 必须能在 OOD 上 work，否则 evaluator 不可信" |
| 范围 | 数据工程问题 | **方法论问题** |

→ **Ctrl-World 把这个问题当成"data engineering"，Uni-WAM 把它当成"benchmark + 训练方法"问题**。这是两条完全不同的研究路径。

→ **Uni-WAM 的卖点**：把"failure modes outside data distribution"作为**一类必须诊断的目标**，而不是当成"数据收集 bug"。

⚠️ **这段话是 Uni-WAM 必须引用的 motivation**：Ctrl-World 自己承认"数据外有很多 failure mode"，但没系统化解决。Uni-WAM 接住这一棒。

---

## 💡 §5.3 整段 takeaway

| 维度 | Ctrl-World 实验情况 |
|---|---|
| 测的 policy 数 | 3 个（π₀ / π₀-FAST / π₀.₅）|
| Policy 质量 | 都是 expert-level VLA |
| OOD 维度 | **视觉 OOD**（新相机位姿、新场景）✅，**行为 ID**（policy 自然 action）⚠️ |
| Ranking alignment | y=0.87x-0.04 (instruction) / y=0.81x-0.11 (success) |
| Paper 自己承认的 gap | Low-level physics 不准 + retry 行为 + **"failure modes outside DROID distribution"** |

**最关键的 takeaway**：

> Ctrl-World **自己承认**了 "data distribution 外有 many failure modes"，但**把它当成 data engineering 问题**（"收集更多数据"），**而不是 method 问题**。
>
> Uni-WAM 接住这个 explicit gap，主张 **"必须有 benchmark 暴露这个问题 + 必须有方法（IDM 反向正则化）解决它"**。

---

## 🤔 §5.3 留下的问题

| Q | 答 |
|---|---|
| Ctrl-World 测过 random policy / 弱 policy 吗？ | ❌ 没测 |
| ranking alignment 是否在跨 policy quality 上保持？ | ❌ 没验证 |
| 失败模式包括哪些？ | paper 提了 collision / sliding / rotation，但**没量化**|

---

[← §5.1 Experiment Setup](05-experiment-setup.md) | [§6 Conclusion →](06-conclusion.md)
