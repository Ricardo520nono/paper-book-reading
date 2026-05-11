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
- [x] 核心故事 · Action 采样 + Obs 采样
- [x] 核心故事 · Gated Metrics（GPR + TA）
- [x] **Model architecture（MoT + Shared Attention + 一体化）**⬅ 当前
- [x] **Data Pipeline（仿真器 + Cosmos-Transfer）**⬅ 当前
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

---

## 核心故事 · Gated Metrics（GPR + TA）

### 1. 为什么需要 Gated 架构

朴素 metric（直接算视觉相似度，如 PSNR/LPIPS/FVD）对 AC-WM **不够用**：

- **Bug 1：视觉好看 ≠ action follow 对**
  - WM 可以画出"清晰、物理合理"的视频，但完全无视 action（伪造 expert 行为）
  - 朴素 metric 给高分 → 错误结论
- **Bug 2：视觉崩溃时算"轨迹精度"无意义**
  - 机械臂都消失/穿模了，再去算"末端轨迹对齐度"纯属噪声

**解法：Gated 打分**

```
video V
  ↓
Gate: 视觉合理吗？(GPR check)
  ├ Pass → 算 TA
  └ Fail → TA = 0
```

**核心思想**：**视觉合理性是"准入资格"，不是"评分维度"** —— 像考试有及格线，没过线直接 0 分。

### 2. 7-Step 评估 Pipeline

| Step | 做什么 |
|---|---|
| 1 | 选定 task 集合 T |
| 2 | 每个 task 采样 N_obs 个 obs（分 on-policy / off-policy）|
| 3 | 每个 obs 按 5 类 off-expert × 强度等级采样 N_act 个 action chunk |
| 4 | 每个 (o, a) 让 WM 生成视频 V，总样本数 = \|T\| × N_obs × N_act |
| 5 | 每个 V 过 Visual Integrity Gate → pass/fail |
| 6 | Pass 算 TA；Fail TA = 0 |
| 7 | 按多维度聚合，输出 scoreboard |

数字感：5 任务 × 100 obs × 50 action = **25,000 个 video**。

### 3. Visual Integrity Gate（GPR = Gate Pass Rate）

**借用 WorldArena 成熟指标 + 阈值化**（不重新发明轮子）。5 个 component：

| Component | 借用指标 | 测什么 | 用法 |
|---|---|---|---|
| 主体存在性 | Subject Consistency (DINO)| 机械臂在视频中是否始终存在 + 同一性 | 阈值化 |
| 末端可提取性 | SAM3 bbox 提取成功率 | 能不能找到 gripper（TA 的前置条件）| **二值**（100% 才 pass）|
| 物理交互合理性 | Interaction Quality (Qwen3-VL)| 接触/抓取/碰撞符不符合物理常识 | 阈值化 |
| 背景稳定性 | Background Consistency (CLIP)| 桌子/墙/灯光稳不稳 | 阈值化 |
| VLM binary check | "机械臂始终完整存在？" | catch-all 兜底 | Qwen3-VL yes/no |

**新引入的工具**：
- **DINO**：Meta 自监督 ViT，提 dense feature 用于"主体一致性"判断
- **SAM3**：Segment Anything v3，零样本分割 → bbox 提取
- **Qwen3-VL**：阿里通义 VLM，做物理合理性判分 + 兜底
- **CLIP**：已学，背景区域 feature 提取

**GPR 计算**：5 个 component **全部 pass** 才算 GPR pass；GPR = pass 样本数 / 总样本数。

### 4. TA（Trajectory Accuracy）—— WorldArena 数学

#### Pipeline

| Step | 做什么 |
|---|---|
| 1 | SAM3 每帧检测机械臂 bbox → NMS + 置信度过滤 → bbox 中心 = 末端位置 |
| 2 | 对 GT video 同样处理 |
| 3 | 漏检帧用线性插值补 |
| 4 | 用 NDTW 算两条轨迹的对齐度 |
| 5 | 取倒数 → 归一化到 [0, 1] |

#### 线性插值（公式 14）

$$p_i = (1-\alpha) p_{prev} + \alpha p_{next}, \quad \alpha = (i-prev)/(next-prev)$$

人话：漏检的第 i 帧，按时间比例在前后两个有效帧之间画直线插值。

#### NDTW（公式 15）—— 核心

$$\text{NDTW}(GT, P) = \min_\pi \frac{1}{|R|} \sqrt{\sum_{(i,j) \in \pi} \|r_i - p_j\|^2}$$

| 符号 | 含义 |
|---|---|
| π | 一个对齐路径（把 GT 第 i 帧和 P 第 j 帧配对）|
| (i,j) ∈ π | 路径上的配对点 |
| \|\|r_i - p_j\|\|² | 配对点的欧氏距离平方 |
| sum + sqrt | 累积距离取 L2 |
| 1/\|R\| | 按 GT 长度归一化（"N"的来源）|
| min_π | 在所有可能路径里挑距离最小的（DTW 的精髓）|

**DTW 的核心价值**：允许"时间错位"。
- GT 第 5 帧到点 A，P 第 7 帧才到点 A
- 朴素帧对齐 → 算出错误的大距离
- DTW 把它们对齐起来，承认"位置对，时间错"
- 动态规划求最优路径，复杂度 O(\|R\| × \|P\|)

#### 取倒数 + 归一化（公式 16）

$$S_{traj\_raw} = 1 / \text{NDTW}(GT, P)$$

NDTW 越小越好 → 取倒数变成"越大越好" → 再归一化到 [0,1] = 最终 **S_traj**。

#### TA 的物理意义

> TA ≈ 1：WM 预测的末端轨迹和 GT 几乎重合 → action follow 得很好
> TA ≈ 0：两条轨迹差很远 → WM 没 follow action

### 5. 翔哥 vs WorldArena 的差异

| 维度 | WorldArena | 翔哥 Uni-WAM |
|---|---|---|
| TA 公式 | 一样 | 直接复用 |
| 用途 | 单独打分 | **Gate pass 才算**（Fail → TA = 0）|
| 组合 | 和其他 19 个 metric 加权 | 只配 GPR 用 |

**翔哥的创新 ≠ 新公式，而是新用法**（Gated 架构）。

### 6. Scoreboard 结构

输出按 (off-expert 类别 × obs 类别) 交叉，每个 cell 报告 GPR 和 TA：

| | On-policy obs | Off-policy obs |
|---|---|---|
| Perturbed expert | GPR=X1, TA=Y1 | GPR=X2, TA=Y2 |
| Counterfactual | GPR=X3, TA=Y3 | GPR=X4, TA=Y4 |
| Exploratory | ... | ... |
| Random-feasible | ... | ... |

Reviewer 一眼能看出 WM 在哪个 cell 崩盘。

---

## 🔥 拷打记录（Q3 系列，格式 B：题+答一起给）

### Q3.1（格式 B）

**问**：Gated 架构的核心思想是"视觉合理性是准入资格，不是评分维度"。为什么不能直接把"视觉合理性"和"轨迹精度"加权求和成一个分数？

**答**：

> 加权求和会让"视觉好但 action 不 follow"的 WM（WM 类型 B）混进高分区。
>
> 例：加权 score = 0.5×视觉 + 0.5×轨迹。WM B 视觉满分 1.0，轨迹 0.0（完全没 follow，只是画 expert 行为）→ 加权 score = 0.5 → **看起来过关**。但 WM B 是对 AC-WM **最致命的失败模式**（伪造）。
>
> **Gated 架构**：视觉不合格直接 0 分，绝不让"视觉好但作弊"的 WM 蒙混过关。
>
> **设计原理**：当两个指标存在**因果依赖**（视觉合理是轨迹精度有意义的前提），它们**不应该加权**，而应该**串行**。加权假设独立，串行承认有先后。

---

### Q3.2（格式 B）

**问**：5 个 gate component 里，"末端可提取性"用二值判断（要么 100% pass 要么 fail），其他 4 个用阈值化。为什么这个要这么严格？

**答**：

> **末端可提取性是下游 TA 计算的前提条件**。
>
> TA 依赖 SAM3 提取末端轨迹。任意一帧提取失败 → 轨迹缺点 → 要么补漏（污染数据）要么截断（丢信息）→ NDTW 计算建立在不完整数据上 → TA 不可靠。
>
> 其他 4 个 gate 是**对视频本身质量的判断**，缺一点不影响 TA 计算 → 阈值化够用。
>
> **设计哲学**：Gate component 的严格程度取决于它对下游 metric 的依赖类型 —— **软依赖用阈值，硬依赖用二值**。

---

### Q3.3（格式 B）⭐

**问**：翔哥设计成"gate fail → TA = 0"，而不是"gate fail → 这个样本不计入平均"。差别在哪？为什么前者更安全？

**答**：

> | 设计 | "TA = 0" | "不计入" |
> |---|---|---|
> | 数学含义 | fail 拉低平均 | fail 消失 |
> | 行为激励 | 鼓励 WM 全样本合理 | 允许"战略性崩坏" |
> | 攻击者视角 | 无法逃避 | 可被 exploit |
>
> **"不计入"的攻击场景**：WM 发现自己 follow 不了 counterfactual action，**故意把这些样本视频生成崩**（gate 必 fail）→ counterfactual 被丢弃 → 平均 TA 只剩 WM 擅长的 perturbed 样本 → **平均分被拉高**。
>
> 这是 **metric gaming** 的经典 pattern：**给"失败"一个出口，它就会被滥用**。
>
> **"TA = 0" 的安全性**：视觉崩坏 = 失败，必须计入坏分。WM 无法通过"崩坏"逃避。唯一拿高分的路径 = **既视觉合理又 action follow**。
>
> **通用设计哲学**：当你设计 metric 时，**永远问"如果 WM 故意作弊，哪里有空子可钻"**。Metric 必须把所有"作弊路径"都堵死，让"做对事"成为唯一拿高分的路径。同样原则适用于 RL reward shaping、benchmark 设计、考试制度设计。

---

## 📌 本节小结

| 关键概念 | 一句话 |
|---|---|
| Gated 架构 | 视觉合理性是准入门，过了再算精度，没过零分 |
| GPR（5 component）| Subject(DINO) + 末端(SAM3) + 物理(Qwen3-VL) + 背景(CLIP) + VLM 兜底，全 pass 才算 |
| TA 数学 | SAM3 → 末端中心点 → 线性插值补漏 → NDTW（允许时间错位的距离）→ 取倒数归一化 |
| NDTW 精髓 | DTW 允许两条轨迹"时间错位对齐"，比朴素帧对齐更宽容 |
| 翔哥的创新 | **不是新公式，是新用法** —— Gated 架构 |
| Metric gaming 防御 | Fail → 0 分而不是不计入，堵死作弊路径 |

---

## Model Architecture（MoT 框架）

### 1. 一句话总览

> 把**视频生成**和**动作预测**统一到一个 **Mixture-of-Transformers (MoT)** 框架里

### 2. MoT vs MoE 区别

| | Mixture-of-Experts (MoE)| Mixture-of-Transformers (MoT)|
|---|---|---|
| 谁是 "expert"| 模型内部的 FFN 子模块 | **一整个 Transformer**（不同模态）|
| 怎么切换 | 输入路由（按 token 选 FFN）| 不同分支并行 + 部分参数共享 |
| 例子 | Wan2.2 高/低噪双专家 | 翔哥这里：视频生成分支 + 动作分支 |

### 3. 两个分支

| 分支 | 模型 | 角色 |
|---|---|---|
| **生成分支** | Wan2.2-TI2V-5B（DiT 30 层 + flow matching，5B Dense）| WM（world model）：(obs, action) → next_obs video |
| **动作分支** | Qwen3-VL-4B（VLM backbone + 连续 action chunk flow matching）| VLA policy / IDM |

**为什么动作分支用 Qwen3-VL 而不是纯 action transformer？**
- Action prediction 需要"理解 obs 语义"（看红方块在哪 → 才知道往哪伸）
- VLM 现成的 vision-language 能力可复用，省去重训 vision encoder

### 4. 深度耦合：Shared Attention（QKV 共享）

**做法**：前 N 层（如 N=10）的 attention，两个分支**共享 Q/K/V 权重**；后面各自独立。

**为什么共享 QKV ≈ "深度耦合"？**
- QKV 决定了"模型如何看输入"
- 共享 QKV → 两个分支用**相同视角**观察输入
- 强迫底层达成"对场景的共同理解"
- 上层再分化做不同任务

**优势**：
- ✅ 参数共享（前 N 层不复制两份）
- ✅ 信息流通（视频与动作表征互相校准）
- ✅ **底层共识 + 上层专精**（多任务架构经典套路）

### 5. 双独立 timestep 采样器

Flow matching 需要在 t=0→1 之间多步迭代。**双独立 timestep** = 两个分支各自有自己的 t，可以独立调度迭代次数。

**为什么这样设计？**
- 视频生成：高维 + 复杂结构 → 需要多步去噪
- 动作预测：低维 + 平滑 → 少量步骤就够

**类比**：两个并行渲染线程，各自调整渲染精度，不绑死在同一个时钟。

### 6. 一体化模型（一份权重 × 4 种用法）

| 用法 | 输入 | 输出 | 角色 |
|---|---|---|---|
| **VLA** | obs + language | action | 标准 VLA policy |
| **WM** | obs + action | next_obs (video)| world model（正向）|
| **IDM** | obs + next_obs | action | inverse dynamics model（逆向）|
| **VLA + WM joint** | obs + language | action AND next_obs | 同时输出动作 + 想象未来（planning）|

**关键概念：IDM（Inverse Dynamics Model）**

| 方向 | 输入→输出 |
|---|---|
| **WM**（正向）| (obs, action) → next_obs |
| **IDM**（逆向）| (obs, next_obs) → action |

**一套权重做多任务的机制**：
- 训练时联合训练（混合 batch）
- 推理时**改变采样策略**（哪些输入给定 / 哪些要生成）
- Flow matching 的灵活性允许"给定一些维度，生成另一些维度"

### 7. 与 Action Following 的连接（翔哥标 TODO）

> **TODO**：补强 argument，说明为什么该架构 specifically 增强 action following
> **候选方向**：动作分支让 IDM 能力反向正则化 WM，迫使其学习真正的 action→obs 映射，而非 (s, a, s') 配对记忆

**核心 hypothesis 链**：

```
WM 训练目标：(obs, action) → next_obs
  → 容易记 (s, a, s') pair（circular reasoning 第三种形式）

IDM 训练目标：(obs, next_obs) → action
  → 必须"理解"前后帧差异才能反推 action

联合训练 + 共享权重：
  IDM 必须从 next_obs 推出 action
  → 模型必须真把 action 信息编码进 next_obs
  → WM 部分不能"看 obs 就照搬 expert next_obs"

反向约束 = 正则化：
  WM 被迫学"真正的 action→obs 映射"，不能再记 pair
```

**类比**：加密器（WM）+ 解密器（IDM）配套训练 —— 加密器如果偷懒不编码 action，解密器就推不出来。

**翔哥标 TODO 的原因**：argument 还需补强（数学上正则化强度难量化，IDM 也可能用 obs-pair pattern 作弊推 action）。

---

## Data Pipeline

### 1. 四步骤

> 1. 仿真器中按采样 off-expert action chunks
> 2. Replay 得到 (sim_obs, action, sim_next_obs)
> 3. Cosmos-Transfer 迁移到真实视觉风格
> 4. 输出 (real_style_obs, action, real_style_next_obs) 用于 WM 训练

### 2. 为什么需要这条 pipeline

**核心问题**：训练 WM 需要 **(obs, action, next_obs) 三元组**，且必须覆盖 off-expert action。

真实机器人数据的问题：
- ❌ 只有 expert demo（没有 off-expert）
- ❌ 让真机做 off-expert action 太危险
- ❌ 标注成本高

**翔哥的解法**：仿真器造数据 + sim-to-real 视觉迁移。

### 3. 四步骤详解

| Step | 做什么 | 关键工具 |
|---|---|---|
| 1 | 用 A+B 采样方案生成 5 类 off-expert action chunk | Isaac Sim / MuJoCo / Habitat |
| 2 | 仿真器里 replay action → 得到 (sim_obs, action, sim_next_obs)| 物理仿真器 |
| 3 | 用 Cosmos-Transfer 把仿真图像迁移成真实视觉风格 | NVIDIA **Cosmos-Transfer** |
| 4 | 输出 (real_style_obs, action, real_style_next_obs) 训 WM | - |

### 4. Cosmos-Transfer 的角色

**问题**：仿真渲染图像"塑料感"明显（光照不真实/材质简单/缺阴影反射）→ WM 训在仿真图像上没法迁移到真机。

**Cosmos-Transfer 的本质**：**保结构换皮肤**
- 输入：仿真"塑料感"图像
- 输出：看起来像真实摄像头拍的图像
- 关键：**保留几何 + 动作**，**只改视觉风格**

**为什么这个分工可行？**
- 仿真器在**物理几何 + 动力学**上是对的（接触点、运动轨迹正确）
- 仿真器在**视觉真实感**上差
- Cosmos-Transfer 专补这个差距

### 5. 这条 pipeline 的设计哲学

**本质**：
> **用仿真器换"动作分布广度"，用 Cosmos-Transfer 换"视觉真实感"** —— 两者结合解锁 "off-expert + photo-real" 训练数据。

| 方案 | 动作分布广度 | 视觉真实感 |
|---|---|---|
| 真机 finetune | ❌ 只有 expert | ✅ 真实 |
| 仿真器 only | ✅ 任意 off-expert | ❌ 塑料感 |
| **仿真 + Cosmos-Transfer** ⭐ | ✅ 任意 off-expert | ✅ 真实 |

### 6. 风险点

- ⚠️ Cosmos-Transfer 本身有 distribution shift（迁移后仍有 sim-real gap）
- ⚠️ 仿真器物理保真度有限（流体/变形/复杂接触）
- ⚠️ Pipeline 不端到端，每步误差累积

---

## 🔥 拷打记录（Q4 系列，格式 B：题+答一起给）

### Q4.1（架构核心）

**问**：为什么是"共享 QKV"而不是"共享 FFN"或"直接拼成一个大 Transformer"？

**答**：

> **QKV 决定了"模型如何看输入"** —— 共享 QKV = 强迫两个分支用相同视角观察输入。
>
> | 方案 | 含义 | 问题 |
> |---|---|---|
> | 拼大 Transformer | 完全合并 | 任务冲突，互相干扰 |
> | 共享 FFN | 共享变换层，视角各自独立 | 底层就分叉 → 后期对齐困难 |
> | **共享 QKV** ⭐ | 视角统一 + 变换独立 | **底层共识 + 上层专精** |
>
> **底层共识 + 上层专精**是多模态多任务架构的经典设计原则。

---

### Q4.2（IDM 反向正则化）⭐

**问**：为什么 IDM 训练好了，反过来能"治"WM 记 pair 的毛病？

**答**：

> **机制**：IDM 必须从 (obs, next_obs) 反推 action → 这要求 next_obs **真的包含 action 信息**。如果 WM 只记 pair（看 obs 就照搬 expert next_obs，忽略 action）→ next_obs 里没有 action 信号 → IDM 推不出来 → 联合训练时 IDM loss 高 → 反向梯度强迫 WM 把 action 信息**真的编码进** next_obs。
>
> **类比**：
> - WM = 加密器：(obs, action) → next_obs（含密文）
> - IDM = 解密器：(obs, next_obs) → action
> - 加密器偷懒不编码 action → 解密器无法工作
> - 联合训练 = 加密+解密配套 → 加密器必须真编码
>
> **直击 circular reasoning 的根**：WM 单训时可"记 pair"，加入 IDM 后 **(s, s') → a 这条逆向通路必须存在** → 强迫学 action↔obs 真实因果关系。
>
> **翔哥标 TODO 因为**：正则化强度难量化；IDM 也可能用 obs-pair pattern 作弊推 action。

---

### Q4.3（Data pipeline 设计哲学）

**问**：为什么不直接训 video diffusion model 在真机数据上 finetune？翔哥这套 pipeline 的本质优势是什么？

**答**：

> **本质优势：解耦"动作分布广度"和"视觉真实感"两个独立目标**。
>
> | 方案 | 动作广度 | 视觉真实 |
> |---|---|---|
> | 真机 finetune | ❌ | ✅ |
> | 仿真 only | ✅ | ❌ |
> | **仿真 + Cosmos-Transfer** ⭐ | ✅ | ✅ |
>
> 两个目标在原始数据源上不可兼得：真机视觉真但动作受限，仿真动作任意但视觉差。Cosmos-Transfer 是**这个 trade-off 的 bypass**。
>
> **核心 insight**：
> > 当两个目标在原始数据源上不可兼得时，**解耦成两个独立步骤**，分别用最擅长的工具解决。
>
> **同类例子**：
> - GAN：generator + discriminator 分开训
> - 大模型：pretrain（通识）+ posttrain（品味）分开
> - Sim-to-real RL：仿真学 policy + 真机微调

---

## 📌 Model + Data 小结

| 关键概念 | 一句话 |
|---|---|
| MoT | Mixture-of-Transformers：不同模态各一个 Transformer + 部分共享，区别于 MoE 的 FFN 路由 |
| Shared Attention | 前 N 层 QKV 共享 → 底层共识 + 上层专精 |
| 双 timestep | 两分支独立调度 flow matching 迭代步数 |
| 一体化 4 用法 | VLA / WM / IDM / VLA+WM joint，一份权重切换采样策略 |
| IDM 反向正则化 | 加密+解密配套训，强迫 WM 真编码 action 而非记 pair（TODO 论证）|
| Data pipeline | 仿真器解锁动作广度 + Cosmos-Transfer 解锁视觉真实 = 既要又要 |
| 解耦目标设计哲学 | 不可兼得的两目标分开解决，trade-off bypass |

---

## 🎓 Proposal 整体闭环

到此为止，翔哥 proposal v2 的核心内容全部讲完。整体逻辑：

```
研究动机
  ↓
核心观察（WM 记 pair 而非学 dynamics）
  ↓
核心故事：Benchmark Action Following Fidelity
  ├─ Motivation（off-expert 术语 + 5 类分层）
  ├─ Action 采样（A+B 方案）
  ├─ Obs 采样（on/off-policy）
  └─ Gated Metrics（GPR + TA）
  ↓
Model architecture（MoT + Shared Attention + 一体化）
  ↓
Data Pipeline（仿真 + Cosmos-Transfer）
  ↓
[下一步：开搜 AC-WM survey]
```

**贯穿全文的核心论证**：
> WM 记 pair → 评估 policy 时循环论证 → 必须用 off-expert action 暴露 → 设计 Action Following Fidelity benchmark → 用 Gated Metrics 严格打分 → 用 MoT + IDM 反向正则化训出真 WM → 用仿真+Cosmos-Transfer 解锁训练数据
