[← 返回 Uni-WAM 调研库](../README.md)

# Uni-WAM Proposal · 小白向逐节理解

> **目的**：在正式启动 AC-WM 调研之前，把翔哥的 proposal 吃透。Ricardo 是具身智能新人，所以**每一节都从最基础概念讲起 + 拷打 Q&A 闭环**。
>
> **Proposal 版本**：v2（[proposal/Uni-WAM-proposal.pdf](../proposal/Uni-WAM-proposal.pdf)）

---

## 📑 章节进度

- [x] 研究动机（用途 1：推理时 prefilter / 用途 2：训练时 DAgger 伪 oracle）
- [x] 核心观察（4 部分：训练数据偏倚 / Cosmos-Predict2.5 初测 / 理论-实践矛盾 / Circular reasoning）
- [x] 核心故事 · Motivation 子节（术语切换 + 5 类 off-expert 分层）
- [x] **核心故事 · Action 采样 + Obs 采样**⬅ 当前
- [ ] 核心故事 · Gated Metrics（GPR + TA）
- [ ] Model architecture（MoT + Wan2.2-TI2V-5B + Qwen3-VL-4B + Shared Attention）
- [ ] Data Pipeline

---

## 核心故事 · Motivation 子节

### 1. 术语切换：OOD action → off-expert action

翔哥把"OOD action"换成了"off-expert action"，把整个 benchmark 命名为 **"Action Following Fidelity"** 而不是 "Action Following on OOD"。

**为什么换？两个理由：**

**理由 1：OOD 这个词有历史包袱**
- "OOD" 在 ML 圈是个老词，原本指**分类器鲁棒性**研究（训练见过猫狗，测试来个长颈鹿）
- Reviewer 一看到 OOD，脑子里冒出来的是 ODIN / Mahalanobis distance 那一套 framework
- 但我们要测的根本不是这个 —— 用 OOD 会让对比对象跑偏

**理由 2：OOD 隐式做了二分（ID vs OOD），但 action 分布不是二分的**
- 机器人 action 是**连续的实数向量**（如 7-DoF joint angle）
- Expert demo 占据 action 空间的一小片**连续 manifold**
- 脱离 expert 有很多种方式（小扰动 / 朝反方向 / 完全随机 / 对抗），难度差距巨大
- ID/OOD 二分把这些都塞进同一个桶，掩盖了差异

→ 翔哥用 "off-expert" 把它作为**连续、可分层**的概念来用。

---

### 2. Off-expert action 5 层分层表

以**机械臂抓红方块**为例（红方块在右，目标盒在左）：

| 层级 | 定义 | 举例 | 测什么 | 难度 |
|---|---|---|---|---|
| **Perturbed expert** | expert action 上加噪 / 缩放 / 时延 | expert 每帧加 ±2° 高斯噪声 | DAgger 噪声扰动（鲁棒性下限）| ⭐ |
| **Counterfactual** ⭐ | 朝任务反方向 / 嫁接他任务 expert | expert 左右镜像 → 应该往左走 | **dynamics vs task prior 解耦**| ⭐⭐⭐ |
| **Exploratory** | 早期 RL checkpoint 的 sub-optimal action | epoch 50 policy 输出的动作 | AC-WM 作为 RL simulator 的核心用例 | ⭐⭐ |
| **Random-feasible** | 关节空间均匀采样 + 运动学约束 | 7 joint angle 随机但不撞自己 | dynamics 极限（无任务语义）| ⭐⭐⭐⭐ |
| **Adversarial** | 主动搜索让 WM 出错的方向 | attacker 找穿模动作 | future work | ⭐⭐⭐⭐⭐ |

### 3. ⭐ Counterfactual 为什么是核心区分点

**核心问题**：WM 到底学到了 **dynamics（物理因果）**，还是只学到了 **task prior（任务模式）**？

```
理想 WM（真学了 dynamics）:
  喂"反方向 action" → 生成"机械臂往左走，远离红方块"的 video
  （物理上正确，任务上失败）

烂 WM（只记 expert pattern）:
  喂"反方向 action" → 还是生成"机械臂往右抓红方块"的 video
  （它根本没看 action，靠 task prior 在脑补 expert）
```

**Counterfactual 把 dynamics 和 task prior 这两个被 expert 数据耦合的能力解耦开**：
- Expert 数据里 dynamics 和 task pattern 总是绑定出现
- WM 只要记住 task pattern 就能在 expert 上表现好
- Counterfactual 强行喂"task pattern 不可能出现的 action" → dynamics 必须独立工作才能产生合理 video

→ 这就是为什么 counterfactual 是**最干净的暴露武器**。

### 4. 整体难度梯度

```
            难度 / 诊断深度
                ↑
   Adversarial   ⭐ future work, 主动找弱点
        ↑
   Random        测 dynamics 的极限
   -feasible
        ↑
   Exploratory   测 RL simulator 用例
        ↑
   Counterfactual ⭐ 核心：测 dynamics vs task prior
        ↑
   Perturbed     最容易，测最基础的鲁棒性
   expert
```

记忆口诀：**扰一下 / 反着来 / 抄旧的 / 乱采点 / 使坏**

---

## 🔥 拷打记录

### Q1.1（格式 A · 我先答）

**问**：为什么翔哥要把"OOD action"换成"off-expert action"？

**Ricardo 答**：因为 robo 里的 action 是连续的数值，而 OOD 的分法是二分的。

**Claude 评分**：✅ 答对了一半。抓住了"连续 vs 二分"的结构性矛盾，但漏掉了"OOD 在 ML 历史上指分类器鲁棒性，会让 reviewer 联想到错误的对比对象"这个 reviewer 视角的理由。

**满分答**：
> (1) "OOD" 在 ML 历史上指分类器鲁棒性那套研究，会让 reviewer 想到错误对比对象；(2) action 空间是连续的，ID/OOD 二分掩盖了不同偏离方式的难度差异。

**Takeaway**：写 proposal 时，**评审视角**和**技术正确性**是两层独立考虑，翔哥的术语选择体现了"政治嗅觉"。

---

### Q1.2（格式 A · 我先答）⭐ 核心题

**问**：为什么 Counterfactual 是 5 类里最深刻的诊断点？它揭露 WM 的什么本质问题？

**Ricardo 答**：因为它揭露了 WM 根本没学会 dynamics 这个点。如果换成朝反方向的 action，WM 在学会 dynamics 的情况下，应该输出的是反方向的动作，虽然任务会失败。

**Claude 评分**：✅ 满分。抓住了"物理正确 + 任务失败 = dynamics 真的学到"这个机制。

**更精确措辞（写 paper / 答辩用）**：
> Counterfactual 把 dynamics 和 task prior 这两个被 expert 数据耦合的能力解耦开。Expert 数据里 dynamics 和 task pattern 总是绑定出现，WM 只要记住 task pattern 就能在 expert 上表现好；而 counterfactual 强行喂一个 task pattern 不可能出现的 action，dynamics 必须独立工作才能产生合理 video。

---

### Q1.3（格式 A · 我先答）

**问**：Random-feasible 和 Counterfactual 都是"远离 expert"的 action，但翔哥把它们放在两个不同的层级。它俩的本质区别是什么？

**Ricardo 答**：本质区别在于任务语义上。Counterfactual 还是在朝某个有意义的方向偏，但 random-feasible 是完全脱离任务语义的，纯粹考验 dynamics。

**Claude 评分**：✅ 满分。

**对比表**：

| 维度 | Counterfactual | Random-feasible |
|---|---|---|
| 有任务语义? | ✅ 有（反任务 / 他任务）| ❌ 无 |
| 考验什么? | dynamics vs task prior 解耦 | dynamics 极限 |
| WM 还能"作弊"吗? | 还能用相反方向的 task prior 蒙混 | task prior 通道彻底关闭 |

→ Random-feasible 是 off-expert 的"鲁棒性上限"，因为它把 task prior 这个作弊通道彻底关掉了。

---

## 📌 本节小结

| 关键概念 | 一句话 |
|---|---|
| off-expert vs OOD | 连续分层 vs 二元分类，避开 reviewer 历史包袱 |
| Action Following Fidelity | benchmark 名称，避开"OOD"的负面联想 |
| 5 类分层 | Perturbed → Counterfactual → Exploratory → Random-feasible → Adversarial |
| Counterfactual 核心价值 | 解耦 dynamics 和 task prior，最干净的暴露武器 |
| Random-feasible 核心价值 | 关闭 task prior 通道，测 dynamics 极限 |
| 5 类的"难度" | 不只是数值偏差大小，而是**关闭 task prior 作弊通道的彻底程度** |

---

## 核心故事 · Action 采样 + Obs 采样

### 1. Action 采样

#### 核心一句话

> **采样：限定在 physically plausible chunk manifold**

三个关键词：
- **chunk**：机器人 action 是一段段给的（VLA 标准做法）—— 单帧太抖，chunk 保证轨迹平滑
- **physically plausible**：关节限位 / 速度限制 / 不穿模 / 连续可微
- **manifold**：高维空间里的低维曲面 —— 所有合法 action chunk 构成的连续曲面

#### 翔哥的关键观点

> **不追求"全分布覆盖"。降维参数化 + 分层采样。**

| 思路 1：全分布覆盖 | 思路 2：分层降维采样 ⭐ |
|---|---|
| 在 plausible manifold 上密集采样 | 每类 off-expert 用少数参数描述 |
| 完整无偏，但**组合爆炸**，大部分没诊断价值 | 高效、可控、每个采样都有明确目标 |

#### 两个采样方案（A + B 互补）

| 方案 | 怎么做 | 适合 |
|---|---|---|
| **方案 A**（以 expert 为锚 + 低维噪声）| 拿 expert chunk → 投影到低维子空间（PCA / spline）→ 加可控噪声 → 反投影 | perturbed / exploratory（小偏离，仍带 expert 语义）|
| **方案 B**（motion primitive 库）| 预定义 primitive 库（直线/弧/抓取）+ 低维参数控制 | counterfactual / random-feasible（大偏离，需独立语义）|

**关键：为什么 counterfactual 必须用 B？**
- 方案 A 是"以 expert 锚扰动"—— 无论噪声多大，都还是 expert 附近，**走不到反方向**
- 方案 B 是"primitive 重新组装"—— 可以选反方向 primitive 或他任务 primitive

#### 5 类 × 2 方案 映射

| Off-expert 类别 | 采样方案 |
|---|---|
| Perturbed expert | 方案 A（小噪声）|
| Exploratory | 方案 A（大噪声）|
| Counterfactual | 方案 B（反方向 primitive）|
| Random-feasible | 方案 B（参数空间均匀采样）|
| Adversarial | 未来工作 |

### 2. Obs 采样

| 类型 | 来源 | 测的问题 |
|---|---|---|
| **On-policy obs** | expert trajectory 中暂停 | 真实 obs 下，WM 能否跟随任意 off-expert action（隔离变量：只测 action 偏离）|
| **Off-policy obs** | off-expert policy rollout | obs + action **联合**都偏离时能否 following（更难，接近真实部署）|

**为什么需要两类？**
- Expert 数据里 obs-action 高度耦合（expert obs 下 → expert action）
- WM 训练时只见过这种耦合 → 可能学到"obs 是 expert 的话，action 也该是 expert 的"
- **On-policy obs + off-expert action**：打破这个耦合，看 WM 还能不能 follow action
- **Off-policy obs + off-expert action**：双重偏离，**真实 policy rollout 场景**

**和 circular reasoning 挂钩**：
- Circular reasoning 第二种形式：WM 训练数据来自 expert → 只在 expert 分布内 work，但 policy 评估恰恰需要 expert 分布外的能力
- Off-policy obs 这一项就是**直接测**这种循环论证的暴露程度

---

## 🔥 拷打记录（Q2 系列）

### Q2.1（格式 B · 我直接给答）

**问**：翔哥不追求"全分布覆盖"。这个选择背后的 trade-off 是什么？

**Ricardo 初答**：更容易实现？

**Claude 判断**：方向沾边但没切中。这题难度偏高，切换格式 B 直接讲。

**完整 trade-off 分析**：

| 全分布覆盖（放弃了） | 分层降维采样（换来了）|
|---|---|
| ✅ 理论完整、无偏 | ✅ 诊断目标明确 |
| ✅ 未来兼容性强 | ✅ 算力可控（几百样本就够）|
| ❌ 组合爆炸 | ✅ 可解释（直接定位崩盘层级）|
| ❌ 大部分采样没诊断价值 | ❌ 可能漏掉某些 failure mode |
| ❌ 算力不可承受 | ❌ 依赖参数化的合理性 |
| ❌ 关键 corner case 信号被稀释 | |

**一句话总结**：
> **完整性 vs 高效诊断力 = trade-off**。
> 翔哥放弃完整性换诊断力 —— benchmark 目的不是"穷举 action"，而是"高效定位 WM 弱点"。

**研究品味 takeaway**：
> 当 sample 空间太大时，"分层 + 降维 + 每层有诊断目标"是几乎所有好 benchmark 的共同做法。同类例子：ImageNet vs CIFAR-10、综合 metric vs 单项 metric、完整 ablation vs 关键因素 ablation。

---

### Q2.2（格式 B · 我直接给答）

**问**：方案 A 和方案 B 分工的依据是什么？为什么 counterfactual 必须用 B？

**Ricardo**：请直接给答案。

**标准答**：

**分工依据**：离 expert 的距离 + 是否需要"反任务"

| 维度 | 方案 A | 方案 B |
|---|---|---|
| 采样起点 | 真实 expert chunk | primitive 模板 |
| 构造方式 | 在 expert **附近**扰动 | **重新组装**整条轨迹 |
| 能离 expert 多远 | 受噪声强度限制 | 几乎不受限 |
| 能不能朝反方向 | ❌ 不能 | ✅ 可以 |

**Counterfactual 必须用 B 的原因**：
> Counterfactual 本质 = "朝任务反方向 / 嫁接他任务"。
> - 方案 A 是"以 expert 为锚 + 扰动"，无论噪声多大都还是 expert 大致方向 + 偏差 —— **无法走反方向**
> - 方案 B 是 primitive 重新组装，可以选反方向 primitive 或他任务 primitive

**深层 insight**：采样方案的选择反映了对失败模式的假设：
- A 假设："WM 在 expert 附近够用就行"
- B 假设："WM 必须能处理 expert 以外的整个 plausible 空间"
- 翔哥同时用 A+B = 两种失败模式都要测

---

### Q2.3（格式 A · 我先答）⭐

**问**：如果一个 WM 在 on-policy obs 上 following 良好，但在 off-policy obs 上崩盘，这说明 WM 缺了什么？为什么不能用来评估 policy？

**Ricardo 答**：在 on-policy obs 上表现好说明 task-prior 做得好？但在 off-policy obs 上崩盘说明没学会 dynamics？

**Claude 评分**：⚠️ 方向对但措辞不精确。
- "task-prior 做得好"不够准 —— action 已经是 off-expert，能 follow 说明 WM 在 expert obs 下能处理非 expert action，**比 task-prior 更强**
- "没学会 dynamics"太极端 —— 应该是 **WM 把 dynamics 和 expert obs 分布绑定了**

**满分答**：

> 这种 WM 的失败模式是：**它学到的 dynamics 是"条件在 expert obs 分布上的"**，不是"通用 dynamics"。
>
> - On-policy obs 下：WM 能跟随 off-expert action（obs 在训练分布内）
> - Off-policy obs 下：obs 已经超出训练分布 → dynamics 失效
>
> **不能评估 policy 的原因**：
> Policy 评估的本质是"让 policy 自由 rollout"。Policy 会犯错累积，几步之后 obs 就偏离 expert —— **off-policy obs 是 policy 评估的常态**。
> WM 在 off-policy obs 崩盘 → 评估出来的 policy 都是错的（policy 一旦偏离 expert 就被 WM 判错）。
>
> 这正是 **circular reasoning 第二种形式**：WM 训练数据来自 expert → 只能在 expert 分布内 work → 但 policy 评估需要 expert 分布外能力。

**3 个 takeaway**：
1. "dynamics" 不是孤立能力，它有 conditioning 分布。WM 学的是"在某个 obs 分布上的 dynamics"
2. On-policy / off-policy obs 区分的是 **conditioning 分布**，不是 dynamics 本身
3. Policy 评估必须用 off-policy obs，因为 policy rollout 的常态就是偏离 expert
