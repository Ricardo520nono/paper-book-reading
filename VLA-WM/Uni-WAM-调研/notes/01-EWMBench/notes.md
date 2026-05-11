[← 返回 Uni-WAM 调研库](../../README.md)

# EWMBench (2025/05) · focused 阅读

> **arXiv**：2505.09694
> **作者**：AgiBot + SJTU + MMLab-CUHK + HIT
> **PDF**：[../../papers/EWMBench.pdf](../../papers/EWMBench.pdf)
> **官方**：https://github.com/AgibotTech/EWMBench

---

## 一句话总结

**针对 T2V（text-to-video）embodied world model 的评估 benchmark**，从场景/运动/语义三维评估，在 Agibot-World 上测了 7 个模型。

⚠️ **关键**：这是 **T2V WM** 的 benchmark，**不接收 action 输入** —— 这正是它"没做到 OOD action"的根本原因。

---

## ✅ 翔哥点出"做了什么"逐条对照

### 1. 三维评估：场景一致性 / 运动正确性 / 语义对齐

paper §3.3 明确这 3 维：

| 维度 | 测什么 | 工具 |
|---|---|---|
| **A. Scene Consistency** | 静态元素（背景/物体/embodiment 结构）保持一致 | **DINOv2 fine-tuned on Agibot-World** + 帧间 cosine similarity |
| **B. Action Motion** | 生成轨迹的物理一致性、任务逻辑、交互约束 | 三个指标互补（见下）|
| **C. Semantic Alignment + Diversity** | 视频和 instruction 是否对齐 + 任务空间多样性 | BLEU + CLIP |

### 2. 设计轨迹评估指标：HSD / NDTW / DYN

| 指标 | 测什么 | 用什么算 |
|---|---|---|
| **HSD**（Symmetric Hausdorff Distance）| 空间对齐 —— 生成 vs GT 轨迹最大偏离 | 几何距离 |
| **NDTW**（Normalized Dynamic Time Warping）| **空间-时间对齐** —— 顺序和时机 | DTW（你已学过）|
| **DYN**（Dynamic Consistency）| 运动动力学 —— 速度/加速度 | Wasserstein 距离 + motion normalization |

**互补性验证**（§4.3，重要！）：他们做了 3 个对抗测试证明 3 个指标各管一段：

| 对抗测试 | 哪个指标降 | 含义 |
|---|---|---|
| 序列反转（sequence reversal）| 只 NDTW 大降 | NDTW 管时序 |
| 异常点插入（outlier） | HSD + DYN 大降 | 它俩管空间精度 + 运动完整性 |
| 帧重复（frame repetition）| NDTW 升、DYN 降 | DYN 管运动光滑性 |

→ **3 个指标确实在测不同的东西**，不是冗余。

### 3. 在 Agibot-World 上评估 7 个模型

数据集：**30 samples × 10 tasks**（来自 Agibot-World，最大真实机械臂操作数据集）。

任务列表（部分）：Take Toast / Brush Bottle / Restock Freezer / Hang Shower Head / Heat Food……

7 个被测模型：
- EnerVerse_FT（fine-tune on Agibot，排名第 1）
- LTX_FT（fine-tune，排名第 2）
- Kling
- COSMOS
- Hailuo
- LTX（未微调）
- OpenSora

**关键发现**：**域内微调 (FT) 显著提升排名** —— EnerVerse_FT 和 LTX_FT 都进前二。但 LTX_FT 仍有"empty grasping"（空抓）的问题。

### 4. 评估指标与人类判断高度相关

§4.2 Human Evaluation：
- 4 个代表性模型让 annotator 打分（3/2/0 分）
- EWMBench 排名 vs VBench 排名 vs 人类排名
- **EWMBench 比 VBench 更接近人类判断**（Figure 6 (B)）

→ 这是 paper 给自己刷的"信任度"证据 —— 不只是工程指标，是真的能反映人类感知。

---

## ❌ 翔哥点出"没做到什么"逐条对照 + Uni-WAM 怎么补

### 1. 测试数据仅来自训练分布（成功轨迹）

**EWMBench 的设计**：
- GT 轨迹来自 Agibot-World 的真实机械臂执行
- 生成轨迹 vs GT 轨迹比对（HSD/NDTW/DYN）
- → **所有 GT 都是成功 expert 轨迹**

**Uni-WAM 怎么补**：
- 5 类 off-expert action 分层（Perturbed/Counterfactual/Exploratory/Random-feasible/Adversarial）
- 用仿真器 + Cosmos-Transfer 造 off-expert 数据
- → 跳出 expert 分布

### 2. 从未测 random / sub-optimal action

**根本原因**：EWMBench 是 **T2V**（text-to-video）→ 输入只有 language instruction + 初始帧，**没有数值 action 输入**。无法控制"feed 一个 random action"。

**Uni-WAM 怎么补**：
- 是 **AC-WM**（action-conditioned WM）
- A+B 采样方案在 plausible chunk manifold 上分层采样
- 可以喂任意 off-expert action

### 3. 没有 OOD action 的评估

**承接上一点**：T2V 设定下根本没有 action 输入，谈不上 OOD action。

**Uni-WAM 的核心创新**：直接面对 off-expert action（不叫 OOD），提出 Action Following Fidelity benchmark。

### 4. 没有量化 action 分布覆盖率

EWMBench 只看"生成 video 和 GT video 像不像"，没有任何对 action 分布的统计。

**Uni-WAM 怎么补**：A+B 采样方案的低维参数化本身就是 action 分布的可控覆盖（PCA / spline / motion primitive 参数空间）。

### 5. 没测失败轨迹的预测能力

EWMBench GT 永远是 expert 成功轨迹。WM 在"失败场景"下的预测能力没法测。

**Uni-WAM 怎么补**：Counterfactual action 类别天然就是"任务必然失败的 action"。WM 应该生成"机械臂往错误方向走"的物理合理 video，而不是"硬掰回 expert"。

---

## 我的额外发现 🔥（翔哥没点但读完发现重要）

### 🔥 1. EWMBench 已经埋了 NDTW + DTW 这个种子

翔哥的 TA（Trajectory Accuracy）= **NDTW**。EWMBench **已经把这个指标用在 embodied 场景里**了。
→ Uni-WAM 用 NDTW 不算原创，是站在 EWMBench / WorldArena 肩膀上。
→ Uni-WAM 的原创性**不在新指标**，而在"5 类 off-expert × Gated 架构"的**用法**。

### 🔥 2. DINOv2 fine-tuned 是 GPR 主体一致性的祖宗

EWMBench 的 Scene Consistency 用 **DINOv2 fine-tuned on Agibot-World**。
→ Uni-WAM 的 GPR component 1（Subject Consistency, DINO）几乎是直接拿来用。
→ 这个 fine-tune 细节很关键 —— **DINOv2 必须 fine-tune 在 embodied 数据上才好用**，原版 DINO 不行。

### 🔥 3. EWMBench 的"3 指标互补性验证"是 Uni-WAM 可借鉴的范式

§4.3 用 sequence reversal / outlier / repetition 三个对抗测试，证明 3 个指标"各管一段"。
→ Uni-WAM 5 个 gate component 之间是否也能做类似的"互补性验证"？这会让 benchmark 更可信。
→ **写论文时这是一个可借鉴的 ablation 模板**。

### 🔥 4. Paper 自己的 Limitations 比翔哥的诊断"温和"

EWMBench paper §5 自己说的 limitations：
- 只跟踪 end-effector，未来要跟踪整条手臂
- 固定视角，未来要 dynamic camera
- 只 manipulation，未来要 navigation

**翔哥的诊断**（"训练分布偏倚 / 没测 OOD action"）**完全没出现在 paper 自己的 limitation 里** —— 这才是 Uni-WAM 真正切入的空白。

→ **结论**：EWMBench 作者自己没意识到这是问题（或者意识到了但不想点破）。Uni-WAM 把这个"沉默的 gap"挖出来 = 巨大的 paper opportunity。

---

## 和 Uni-WAM 的关系

| 方面 | EWMBench 已有 | Uni-WAM 复用 / 改造 / 新增 |
|---|---|---|
| 三维评估 | Scene / Motion / Semantic | **改造**：换成 GPR Gate + TA 串行 |
| HSD/NDTW/DYN | 全套都有 | **复用 NDTW**，HSD/DYN 暂不用 |
| Scene Consistency (DINOv2-FT) | 有 | **直接复用** 作为 GPR component 1 |
| Agibot-World 数据 | 用了 | Uni-WAM 可参考用 |
| T2V 设定 | T2V only | **核心 gap**：Uni-WAM 必须 AC-WM |
| Off-expert action | 无 | **核心创新**：5 类分层 |
| Gated 架构 | 无 | **核心创新** |

**总评**：EWMBench 是 Uni-WAM 的**最近一棵参考树** —— 不是直接对手，是 T2V 评估那边的成熟工作。Uni-WAM 走的是**正交方向**（AC-WM + off-expert）。

---

## 🔥 拷打（减量：1 题，格式 B）

### Q-EWM.1

**问**：翔哥为什么把 EWMBench 列在 **Bench 参考类**（表格里写 "参考用"），而不是 AC-WM 调研候选？用一句话说清楚 EWMBench 和 Uni-WAM 在 benchmark 设计上的"根本分歧"。

**答**：

> **根本分歧**：EWMBench 是 **T2V WM** 的 benchmark（输入 = 语言+初始帧，无 action），Uni-WAM 是 **AC-WM** 的 benchmark（输入必含数值 action）。**输入信号种类不同 → 评估对象不同 → 不是同一条赛道**。
>
> EWMBench 评估的是"模型能否从语言指令生成符合 task 的 video"；Uni-WAM 评估的是"模型能否在任意 off-expert action 下生成物理合理的 video"。
>
> 翔哥把它列入"Bench 参考用"，意思是：**评估方法论**（DINOv2-FT、NDTW、三维评估）值得借鉴，但**评估对象**（T2V vs AC-WM）不同 —— 不能拿 EWMBench 直接当 Uni-WAM 的对照基线，但可以**继承它的指标技术栈**。
>
> 这就是为什么 GPR 直接抄了它的 DINOv2-FT，TA 直接抄了它的 NDTW —— 工具一样，**只是把工具用在不同输入设定下**。

---

## 📌 一句话存档

> EWMBench = **T2V embodied video 的 benchmark 之祖**。Uni-WAM 拿走它的"工具"（DINO-FT, NDTW, 互补性验证范式），跳过它的"设定"（T2V → AC-WM），把它没碰的 off-expert action 这个 gap 当卖点。
