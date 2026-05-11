[← 返回 Uni-WAM 调研库](../../README.md)

# EWMBench (2025/05) · focused 阅读

> **arXiv**：2505.09694
> **作者**：AgiBot + SJTU + MMLab-CUHK + HIT
> **PDF**：[../../papers/EWMBench.pdf](../../papers/EWMBench.pdf)
> **官方**：https://github.com/AgibotTech/EWMBench

---

## 📐 笔记结构说明

- **Part 1 · 串讲（带读）**：把 paper 里和 Uni-WAM 相关的 section 逐段过一遍，中英混合，跳的明跳。**先读 Part 1 再读 Part 2**。
- **Part 2 · 总结**：读完之后的浓缩 + 和 Uni-WAM 的对照 + 拷打。

### 这次跳过的 section（不读）

- §2 Related Work（和我们工作无关）
- §3.4 MLLM Prompt Suite Design（prompt 工程细节）
- §4.1 Main Results（哪个 model 拿第一不重要）
- §4.4 Further Analysis（具体 model 失败案例细节）
- Appendix 大部分（除了 metric 数学定义）

---

# Part 1 · 串讲

## §1 Introduction · 这篇 paper 要回答什么问题

### 第一段：背景铺垫

> "Creative AI has advanced rapidly... Building on the momentum of text-to-video diffusion models, recent efforts have expanded their scope from generating high-fidelity videos to serving as **embodied world models (EWMs)** capable of synthesizing physically actionable scenes from **language instructions** (e.g., 'move the robot arm approaching the cup') **or physical action instructions, i.e., an action policy sequence**."

**翻译**：创意 AI 发展很快。T2V diffusion model 演化出了 EWM（embodied world model）—— 能根据 **language instructions** 或 **action policy sequence** 生成"物理可执行的场景"。

### 💡 批注：第一个关键判断

注意这里说 EWM **既能接 language，也能接 action**。**但这只是定义层面**，下面我们看 EWMBench 实际测什么。

⚠️ **预告**：等会读到 §3.1，你会看到一个公式 `V = vnorm(gworld(fproc(I, L, T)))` —— **T（trajectory）是 optional**。这意味着 paper 实际评估时**不强制 model 接受 action 输入**。这正是 Uni-WAM 和 EWMBench 的分歧点。

### 第二段：他们提出的核心问题

> "Despite advancements in EWMs, a fundamental question remains unresolved: **'How can we determine whether a video generation model qualifies as a good embodied world model, beyond merely serving as a general-purpose video generator?'**"

**翻译**：现有 video generation benchmark（如 VBench）只看视觉保真度、language alignment、人类喜好。**这些对 EWM 不够用** —— EWM 还需要 embodiment 动作合理性、物理一致性。

> "Embodied generation tasks have unique requirements, such as coherence in embodiment motion and plausibility in action execution... For instance, in robotic manipulation scenarios, **the background, object configuration, and embodiment structure (e.g., robot morphology) are expected to remain static, while only the robot's pose and interactions evolve according to instructions**."

**翻译**：embodied 场景下，**背景/物体配置/机器人形态应该不变**，**只有姿态和交互在变**。这种结构化要求是 EWM 不同于通用 T2V 的关键。

### 💡 批注

这一段定义了 EWMBench 的核心目标：**评估"模型是否懂 embodied 场景的结构化约束"**。这个思路在 Uni-WAM 的 **GPR Visual Integrity Gate** 里被几乎完整继承（主体一致性 / 背景稳定性 / 物理交互合理性）。

### 第三段：三维评估的正式提出

> "We design an evaluation protocol based on three key aspects:
> (1) **Visual Scene Consistency**, ensuring static elements like the background, objects, and embodiment structure remain unchanged during motion;
> (2) **Motion Correctness**, requiring the generated embodiment trajectory to be coherent and aligned with the task objective;
> (3) **Semantic Alignment and Diversity**, assessing the model's alignment with linguistic instructions and its ability to generalize across diverse tasks."

**翻译**：三维评估正式登场 —— **Scene / Motion / Semantic**。

### 💡 批注

这就是翔哥表里写的"三维评估：场景一致性、运动正确性、语义对齐"。**EWMBench 是第一个把"embodied 视频"和"通用 T2V 视频"在评估上区分开的 paper**。Uni-WAM 在概念上继承了这套划分，但把 Motion 这个维度做得更深（5 类 off-expert + Gated 架构）。

---

## §3.1 Overview · 关键公式

> "Evaluation Task Formulation: An embodied world model generates a video as expressed in Equation 1, where I, L, and T represent the input context image, language, and trajectory, respectively. We provide this unified information, including up to four initial images. **The action trajectory, formatted as a sequence of 6D poses, is optional for generation model inference.**"

公式 (1)：
$$V = v_{\text{norm}}(g_{\text{world}}(f_{\text{proc}}(I, L, T)))$$

**翻译**：
- **输入**：image I + language L + trajectory T（**T 是 optional**）
- **g_world**：被测的 generation model
- **f_proc**：preprocessing
- **v_norm**：把生成的 video frame 归一化

### 💡 批注 ⭐ 这里是核心判断点

**T 是 optional 这一句非常重要**。它意味着：
- EWMBench 接受 **language-only 输入**的 T2V model（OpenSora / LTX / Kling 等都属于这种）
- 也接受 **action-conditioned 输入**的 model（如果你愿意的话）
- **但 benchmark 的设计基线 = T2V**

**Uni-WAM 不一样**：T **不是 optional，是必须**。Uni-WAM 评估的核心问题是"WM 在喂 action 时能不能 follow"，没 action 输入就根本谈不上"action following"。

**所以**：
- EWMBench 是 **"input-permissive"** benchmark（包容多种输入模型）
- Uni-WAM 是 **"input-prescriptive"** benchmark（强制要求接 action）

这是两个 benchmark 在哲学上的根本差异。

---

## §3.2 Dataset Construction · 数据集

> "We developed our evaluation dataset using the open-source **Agibot-World** dataset. **Ten tasks** were carefully selected based on their clear operational goals and sequential dependencies..."

> "...action trajectories were encoded into voxel grids, and a **greedy algorithm** was employed to **select the most diverse trajectories** for each task."

> "For fine-grained evaluation, we adopted a task-oriented decomposition strategy. Each high-level task was broken down into a sequence of **4 to 10 atomic sub-actions**, with each sub-action paired with a step-level caption."

**翻译**：
- 数据集来源：**Agibot-World**（公开的最大真机操作数据集）
- 选 **10 个 task**（有清晰的 action-ordering 约束）
- 用 **voxel grid + greedy 算法** 在每个 task 里挑最多样化的轨迹
- 每个 task 分解成 **4-10 个原子 sub-action**，每个 sub-action 配一句 caption
- 总共 **30 个 sample**（来自 abstract / §1）

### 💡 批注

**两个细节值得记**：

1. **voxel grid + greedy 选轨迹** 这个方法本身可以借鉴 —— 当 Uni-WAM 要从仿真器里挑 action chunk 来做 benchmark 时，这是个现成的"多样性最大化"工具。

2. **Sub-action 分解**：把任务切成 4-10 个原子动作，每个配文本 caption。这种细粒度标注是为了支持后面的 **semantic alignment** 评估（用 BLEU 把 caption 和生成 video 的描述对齐）。

3. ⚠️ **但有个隐含问题**：所有 30 个 sample 的轨迹都来自 **Agibot-World 真机执行** —— 也就是说**全是 expert demo**。这一点 paper 自己没强调，但它后面没法测 off-expert action 就是因为这个。

---

## §3.3 Evaluation Metrics · 三维评估的具体实现

这是 paper 最干货的一节。翻译 + 拆解：

### A. Scene Consistency（场景一致性）

> "We introduce the **Scene Consistency** metric, which examines visual layouts, object permanence, and viewpoint coherence. **DINOv2, fine-tuned on an embodied dataset**, extracts patch-level frame representations. Cosine similarity between patch embeddings of consecutive and initial frames quantifies frame-to-frame consistency."

**翻译**：
- 工具：**DINOv2，在 embodied 数据上 fine-tune 过**
- 做法：每帧提 patch-level feature → 算"当前帧 vs 初始帧"+ "当前帧 vs 上一帧"的 cosine similarity
- 高分 = 场景稳定

### 💡 批注

**DINOv2 fine-tuned on embodied data 是关键工程细节**。原版 DINOv2（在 ImageNet 上 pretrain）对机械臂场景不灵敏 —— paper 在 §A.3.1 里专门做了"DINOv2 vs VBench"的对比验证。

→ Uni-WAM 的 **GPR Component 1（主体存在性 / Subject Consistency, DINO）几乎是直接复用这个**。借鉴时**必须也用 embodied 数据 fine-tune** 一下，不能直接拿原版 DINOv2。

### B. Action Motion · 三个轨迹指标

> "The quality of generated motions is evaluated through a **Trajectory-Based Evaluation**, which compares generated trajectories with ground truth trajectories... we use the **end-effector's (EEF) trajectory** as the evaluation target and EEF is detected with our **finetuned detector**."

**翻译**：
- 评估对象：**end-effector（末端执行器）轨迹**
- 用一个 fine-tuned detector 提取每帧 EEF 位置 → 串成轨迹
- 拿生成的 EEF 轨迹 和 GT EEF 轨迹 比对

#### 三个指标各自的角色

> "**Symmetric Hausdorff Distance (HSD)** measures spatial alignment by calculating the **maximum deviation** between points on generated and GT trajectories."

| 指标 | 测什么 | 直觉 |
|---|---|---|
| **HSD** | 空间最大偏离 | 两条轨迹"最远的地方差多少" |
| **NDTW** | 空间-时间对齐 | DTW 允许时序错位，看顺序对不对 |
| **DYN** | 运动动力学 | 速度/加速度的分布相似度（用 Wasserstein 距离 + motion normalization） |

> "To ensure fairness, generative models are required to produce **three candidate trajectories** for each task. The best trajectory is selected based on Hausdorff distance."

**翻译**：每个 task 让 model 生成 3 条候选 → 用 HSD 挑最好的一条参与评估。

### 💡 批注

**3 件事值得记**：

1. **NDTW 在 EWMBench 已经登场了**（前面学 Uni-WAM 时讲过 NDTW 是 WorldArena 来的，**其实更早 EWMBench 就用了**）。Uni-WAM 的 TA 工具栈起源在 EWMBench。

2. **"生成 3 条挑最好"** 这个设计有趣：承认 video generation model 是随机的（diffusion 采样有方差），所以给 model 3 次机会。**Uni-WAM 可不可以借鉴？** 可以，但要小心 —— 如果允许"挑最好"，metric gaming 风险上升（前面 Q3.3 讲过）。Uni-WAM 的 Gated 架构对此更严格。

3. **EEF 是评估对象**：只看末端，不看整条手臂。这是 EWMBench 自己承认的 limitation。

### C. Semantic Alignment + Diversity

> "For semantic alignment, we use the generated video's language caption as an intermediate representation, comparing them to ground truth annotations to compute an alignment score... For semantic diversity, we use **CLIP model**, global video features are extracted, and the diversity score is computed as **1 − similarity**."

**翻译**：
- **Alignment**：用 video MLLM 给生成的 video 写 caption → 和 GT caption 算 BLEU
- **Diversity**：CLIP 提全局 feature → 1 - cosine similarity = 多样性分数

### 💡 批注

- Semantic 这部分**对 Uni-WAM 影响较小** —— Uni-WAM 的核心是 GPR + TA，不涉及 BLEU。
- 但 CLIP 用来做 diversity 是个干净的工程做法，可以记一下。

---

## §4.2 Human Evaluation · 人类对齐验证

> "To evaluate the alignment between automated metrics and human judgment, we conducted a human evaluation on videos generated by four representative models: **LTX_FT, Kling-1.6, Hailuo I2V-01-live, and OpenSora-2.0**. Annotators ranked the predictions based on overall quality, assigning **3 points to the best, 2 to the second-best, and 0 to the worst**."

> "...EWMBench's rankings align more closely with human judgments than VBench rankings, indicating stronger consistency with human perception."

**翻译**：4 个 model 让 annotator 排序（3/2/0 计分）→ 聚合人类排名 → EWMBench 排名 vs VBench 排名 vs 人类排名 → **EWMBench 比 VBench 更接近人类**。

### 💡 批注

**这是 paper 给自己刷"可信度"的关键章节**。
- 没有这一章，EWMBench 只是个工程指标
- 有了这一章，EWMBench 才能说"我比 VBench 更适合 embodied 场景"

**Uni-WAM 后面也需要类似的 human eval validation** —— 不然 reviewer 会问"凭什么 GPR + TA 是合理 metric？"。

---

## §4.3 Complementarity of Trajectory Metrics · 三指标互补性 ablation

这是**全 paper 我觉得最值得借鉴的实验设计**。

> "To validate the necessity of employing all three trajectory consistency metrics—HSD, nDTW, and DYN—we conducted controlled experiments involving **sequence reversal, outlier insertion, and frame repetition**."

**翻译**：做 3 个"对抗性测试"，证明 3 个指标各管一段，不可互相替代。

### 三个测试的结果

| 对抗测试 | 做了什么 | 哪个指标降了 | 说明什么 |
|---|---|---|---|
| **Sequence reversal**（序列反转）| 把轨迹倒过来播 | 只 **NDTW** 大降 | NDTW 管时序 |
| **Outlier insertion**（异常点插入）| 在轨迹里塞几个错误的点 | **HSD + DYN** 大降 | 这两个管空间精度和运动完整性 |
| **Frame repetition**（帧重复）| 把某些帧重复 | **NDTW 升、DYN 降** | DYN 管运动光滑性 |

> "These findings **confirm the complementary roles** of the three metrics in providing a comprehensive evaluation of trajectory quality."

### 💡 批注 ⭐ 这是最值得 Uni-WAM 学的范式

EWMBench 这种"对抗性 ablation"的做法是个**好范式**：
> 你说我有 N 个 metric？我证明给你看 N 个 metric 每个都不可少 —— **构造 N 个不同的 corruption，每个只能被一个 metric 捕捉**。

→ **Uni-WAM 也应该做一个类似的 ablation**：
- 翔哥的 GPR 有 5 个 component（DINO / SAM3 / Qwen3-VL / CLIP / VLM 兜底）
- 5 个真的都不可替代吗？能不能构造 5 种 corruption 让每个 component 各自 catch？
- 这种 ablation 是 reviewer 必问的，提前准备。

---

## §5 Conclusions and Limitations · 它自己承认了啥

> "**Limitations and Future Work.**
> First, our method currently focuses on the trajectory of the **robotic arm's end-effector**, but future work will incorporate the **state and configuration of the entire arm**.
> Second, the current evaluation is conducted in **fixed-viewpoint scenes**; future research will explore **flexible viewpoints**, such as dynamic camera setups.
> Lastly, we aim to **extend the scope** of embodied tasks—from the current manipulation tasks to more diverse domains, including **navigation and mobile manipulation**."

**翻译**：EWMBench 自己承认 3 个 limitation：
1. 只跟末端，未来要追整条手臂
2. 固定视角，未来要 dynamic camera
3. 只 manipulation，未来要 navigation / mobile

### 💡 批注 ⭐⭐ 这一段是最关键的对照

**注意 paper 自己承认的 limitation 里完全没有：**
- ❌ "测试数据仅来自 expert 训练分布"
- ❌ "从未测 off-expert action"
- ❌ "OOD action 评估"
- ❌ "action 分布覆盖率"
- ❌ "失败轨迹的预测能力"

**翔哥在表格里写的"没做到 ❌" 一条都没出现在 paper 自己的 limitation 里**。

→ 这就是 Uni-WAM 真正的 paper opportunity：**翔哥挖到的 gap 是 EWMBench 作者自己都没意识到的盲点**（或者意识到了但不想说）。

**对照视角**：
- Paper 自己看：limitation 是"scope 不够大"
- 翔哥的诊断：limitation 是"评估对象本身有偏倚（只看 expert）"

后者比前者深一个量级。

---

# Part 2 · 总结（读完之后的浓缩）

## 一句话总结

EWMBench = **T2V embodied video 评估的开山祖**。从场景/运动/语义三维评测 T2V WM，在 Agibot-World 上跑 7 个模型，提出 DINOv2-FT 场景指标 + HSD/NDTW/DYN 轨迹指标，并用人类评估证明优于 VBench。

⚠️ 它的设定本质是 **T2V**（输入语言+初始帧，**action trajectory T 是 optional**），不是 **AC-WM**。

---

## 翔哥的"✅ 做了什么"逐条验证

| 翔哥的描述 | paper 出处 | 验证结果 |
|---|---|---|
| 三维评估：场景一致性、运动正确性、语义对齐 | §1 第三段 + §3.3 | ✅ 准确 |
| HSD / NDTW / DYN 指标 | §3.3 B + Appendix A.3.2 | ✅ 准确 |
| 在 Agibot-World 上评估 7 个模型 | §3.2 + §4.1 | ✅ 准确（7 个：EnerVerse_FT / LTX_FT / Kling / COSMOS / Hailuo / LTX / OpenSora） |
| 评估指标与人类判断高度相关 | §4.2 | ✅ 准确（vs VBench 对比） |

---

## 翔哥的"❌ 没做到什么"逐条验证 + Uni-WAM 怎么补

| 翔哥的诊断 | paper 证据 | Uni-WAM 怎么补 |
|---|---|---|
| 仅来自训练分布（成功轨迹）| §3.2 数据全来自 Agibot-World expert demo | 5 类 off-expert + 仿真器造数据 |
| 从未测 random / sub-optimal action | §3.1 公式 T optional，benchmark 不强制接 action | AC-WM 强制 action 输入 + A+B 采样 |
| 没有 OOD action 评估 | T2V 设定下没 action 输入 → 无 OOD action 概念 | 5 类 off-expert 分层 |
| 没有量化 action 分布覆盖率 | paper 完全没分析 action 分布 | A+B 采样的低维参数化天然提供覆盖率 |
| 没测失败轨迹的预测能力 | GT 永远 expert | Counterfactual 类别 = 必然失败的 action，看 WM 能否物理正确生成 |

---

## 🔥 我的额外发现

### 1. NDTW 起源在这里，不是 WorldArena
之前学 Uni-WAM 时讲过 NDTW 来自 WorldArena。**其实更早 EWMBench 就用了**。Uni-WAM 的 TA 工具栈起源在 EWMBench。

### 2. DINOv2-FT 是 GPR Component 1 的祖宗
Uni-WAM 的"主体存在性"几乎直接复用 EWMBench 的做法。借鉴时**必须也用 embodied 数据 fine-tune** 一下。

### 3. 三指标互补性 ablation 是好范式
EWMBench §4.3 的"对抗性 corruption + 看哪个指标降" 是个值得 Uni-WAM 借鉴的 ablation 范式。Uni-WAM 5 个 GPR component 也应该做一个类似的"5 corruption × 5 component"互补性验证。

### 4. Paper 自己的 limitation 完全没碰翔哥的诊断
EWMBench 自己说 limitation 是 scope 不够（只末端、固定视角、只 manipulation），**完全没意识到"评估对象本身偏倚 expert"**。翔哥挖到的是更深一层的盲点。

---

## 和 Uni-WAM 的关系总结

| 方面 | EWMBench | Uni-WAM |
|---|---|---|
| 评估对象 | **T2V WM**（T optional）| **AC-WM**（T 强制）|
| Scene Consistency 工具 | DINOv2-FT | **直接复用** → GPR Component 1 |
| Trajectory 指标 | HSD + NDTW + DYN | **只复用 NDTW** → TA |
| Action 分布 | 只测 expert | **5 类 off-expert 分层** |
| 评估架构 | 多指标平铺 | **Gated 架构**（GPR → TA 串行）|
| Human alignment | ✅ 做了 | 待做 |
| 互补性 ablation | ✅ 做了 | 待做（值得借鉴范式） |

**总评**：EWMBench 不是 Uni-WAM 的对手，是它的**工具供应商**和**研究路径起点**。Uni-WAM 在它没碰的方向上（off-expert action）开新战场。

---

## 🔥 拷打（减量 · 1 题 · 格式 B）

### Q-EWM.1

**问**：翔哥为什么把 EWMBench 列在 Bench 参考类（表里写 "参考用"），而不是 AC-WM 调研候选？用一句话说清楚它们的"根本分歧"。

**答**：

> **根本分歧在 §3.1 那个公式**：`V = vnorm(gworld(fproc(I, L, T)))`，**T（trajectory）是 optional**。
>
> EWMBench 是 **input-permissive** —— 接受 T2V 模型（语言 only 输入）也接受 AC 模型，benchmark 默认基线是 T2V。
> Uni-WAM 是 **input-prescriptive** —— **T 必须强制存在**，因为评估的核心问题"WM 能不能 follow off-expert action"没 action 输入根本谈不上。
>
> 翔哥列 EWMBench 为"参考用" = **方法论可借鉴**（DINOv2-FT / NDTW / 互补性 ablation 范式），**评估对象不可对照**（T2V vs AC-WM 是两条不同赛道）。
>
> 这就是为什么 GPR Component 1 抄了 DINOv2-FT、TA 抄了 NDTW —— **工具直接复用**，**但把工具用在"必接 action"的设定下**，挖出 EWMBench 因 T-optional 而无法触及的"off-expert action"这个盲点。

---

## 📌 一句话存档

EWMBench = **T2V embodied video 评估鼻祖** + **NDTW/DINOv2-FT 工具栈的源头** + **互补性 ablation 范式的好教材**。Uni-WAM 拿走它的工具，跳出它的 T-optional 设定，在它的盲点（off-expert action）上开新战场。
