[← 返回 Genie Envisioner 主页](../README.md)

# 串讲 · 几张图过完 Genie Envisioner

> **arXiv 2508.05635 (v3 2025/11)** · 2025/08 首次发布 · 作者 Liao / Zhou / Huang 等 14 人 (AgiBot Genie Team + LV-NUS + BUAA)
>
> **Uni-WAM 视角的前置判断**：**a 类 AC-WM 平台** ⭐⭐ — 真具身 manipulation；**EnerVerse-AC 的同团队后续 + 统一框架**。

---

## 1. 这篇 paper 解决的是什么问题？

| 痛点 | 现状 |
|---|---|
| **现有 robot manipulation 系统是"补丁堆叠"** | data collection / training / evaluation 各自独立, 用不同 representation |
| **Manual reprogramming 难规模化** | 新任务、新模态、新场景都需要手工调整, hindering scalability |
| **现有 video gen WM 缺 closed-loop** | T2V model 能生成视频但**不能 act**, 形成不了 policy 闭环 |

**Genie Envisioner 一句话定位**：
> AgiBot 团队把**数据 / 训练 / 评估 / 模拟**全部塞进**一个 closed-loop video generative world model framework**。模式：先训一个超大 video diffusion (GE-Base)，再衍生出 policy (GE-Act) + simulator (GE-Sim) + benchmark (EWMBench)。

---

## 2. 方案全景 · Figure 1 看懂（详细带读版）

![](../images/figure-01.png)

Figure 1 是这篇论文的"地图"——读懂它,全文一大半就通了。我们一块一块拆。

### 2.1 第一眼:这张图的"机翼"布局

注意正中间那架紫色的飞机——这不是装饰,是作者特意安排的视觉隐喻。**机身 = GE-Base**(整个平台的基座),两翼挂着两个功能模块,最下面是一套评估套件。所以全图实际上是**四个区**:

```
                ┌───────────────────────────────────────────────┐
                │  上方:GE-Base World Foundation Model         │  ← 机身
                │      (基座:视频生成基础模型)                 │
                └───────────────────────────────────────────────┘
                ┌─────────────────────┐  ┌────────────────────┐
                │  左翼:GE-Act        │  │  右翼:GE-Sim       │
                │  (World Action      │  │  (World Simulator)  │  ← 两翼
                │   Model,出动作)    │  │  (闭环仿真)         │
                └─────────────────────┘  └────────────────────┘
                ┌───────────────────────────────────────────────┐
                │  底部:EWMBench                                │
                │       (评估套件)                              │
                └───────────────────────────────────────────────┘
```

🔑 **看图最先要抓的一点**:这张图想强调的核心是**"unified(统一)"** ——**四个模块共享同一个 GE-Base 基座**,不是四个独立系统拼一起。机翼可以拆下,机身始终在;基座撑起整个平台。

---

### 2.2 上方机身:GE-Base(视频基础模型)

**一句话钉死它是什么**:GE-Base 是一个**大规模 instruction-conditioned 视频扩散模型**,在 **AgiBot-World-Beta 100 万条真机操作 episode** 上预训练(图里那个红火苗 + AgiBot 星标就是在强调这一点)。它和 DiT 是同一家族(diffusion transformer),只是任务从"生成单张图"换成了"生成机器人操作的视频片段"。

#### 三路输入(Condition 区)

GE-Base 不凭空生成视频,它要看三样东西:

**① Observation(多视角观测)** —— 三张图:`Left View + Head View + Right View`。AgiBot 这套机器人头上和左右各装摄像头,**三个角度同时看**,这是模型当前的"眼睛"。为什么要多视角?单视角看不到自己手臂背面、看不到桌面遮挡区域,多视角让模型对 3D 场景有更完整的理解(Figure 3 的 cross-view causal block 专门处理这种跨视角一致性)。

**② Instruction(指令)** —— 自然语言任务描述,比如 `Pick up`、`Place`……告诉模型"接下来要干什么",通过文本编码器变成语义向量。类似 DiT 里的 class label `y` 起的作用 —— 当条件指导生成。

**③ Memory(记忆)** —— 这是 GE-Base 区别于单帧扩散模型的特色,有三个关键子部件:

- **History Frames**:不只看"当前这一帧",还看**过去发生过的若干帧**。机器人操作是连贯过程,没历史就会失忆(比如"刚才已经把瓶子拿起来了"的信息就丢了)。
- **Sampler**:历史帧可能成百上千,不能全塞,所以用一个 Sampler **稀疏采样几个关键帧**。
- **Long-horizon Context**:采出来的关键帧拼成"长程上下文",和当前观测一起喂给模型 ——让 GE-Base **既能看清"当下"又能记得"之前"**,生成长视频时保持时间连贯。

> ⚠️ **澄清一个易混点**:Memory 进 GE-Base 时**不是文本形式**,而是**视频 token 形式**。具体路径:历史帧 → Sampler 稀疏采样 → 每帧过 Video VAE encoder 压成 latent → patchify 切成 token → 拼进主序列。和 Observation 走的是同一条预处理管线,**只有 Instruction 那一路是文本 token**。

#### 中间:GE-Base 本体

机身的飞机就是 GE-Base。它做的事可以简化成:

```
三视角观测 + 指令 + Memory  ──→  GE-Base  ──→  下一段多视角视频(video chunk)
```

几个关键点:

- **一次只生成一小段视频**(叫 video chunk,比如几秒/几十帧),不是一次几分钟。这是因为视频生成开销巨大,而且要给"看清当下→生成下一段→再回头看→再生成"的循环留余地。
- 内部架构(Figure 3 详讲)是 **autoregressive 多视角视频扩散 Transformer**:多个视角并行生成 + 用 causal block 让三视角之间**互相通信保持一致**(不能左视角生成的是杯子、右视角却是碗)。
- 飞机贯穿上下两半——强调 **GE-Base 是平台所有其他模块的共享基座**。

#### 输出 + 自回归回路(关键)

输出是**多视角生成视频**(右边那个网格,每行对应一个视角,展示了不同操作场景的连续帧)。

仔细看:右边 Generated Video 那块有一条**虚线箭头反向指回 Memory**——这正是 **autoregressive(自回归)** 的本质:

```
[当前obs + 指令 + 历史memory]
         ↓
       GE-Base
         ↓
   生成 video chunk_1
         ↓
   chunk_1 加入历史 memory
         ↓
       GE-Base
         ↓
   生成 video chunk_2
         ↓
       (循环……)
```

就像"接龙写故事"——写一段(chunk),把它加进上下文,再续写下一段,每一段都依赖前面所有段,保证整个长视频是连贯的一条故事线。

---

### 2.3 左翼:GE-Act(World Action Model)

**一句话钉死**:GE-Act = 挂在 GE-Base 上的"动作分支",把 GE-Base 学到的"对世界的视觉理解",**翻译成真机能执行的一串动作**。

类比一下:**GE-Base 是"看懂世界"的视觉皮层,GE-Act 是"指挥肌肉"的运动皮层**。后者依赖前者的感知,负责把感知转成动作输出。

#### 数据流(从右往左追)

```
                   GE-Base
                     │  Latent Features
                     ▼
              ┌─────────────┐
              │ Action Policy│
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │ Action Chunk │  ← 一串动作,不是单个动作
              └──────┬──────┘
                     │ Execute
                     ▼
        ┌──────────────────────────┐
        │  各种机器人(多臂/形态)   │
        └──────────────────────────┘
        例:Pour water / Assemble box / Fold clothes
```

#### 关键细节

**① 输入是 Latent Features,不是 Generated Video**

划重点:GE-Act 吃的是 GE-Base **中间层的视觉 latent 特征**(还没被解码回像素的"内部状态"),不是 GE-Base 输出的像素视频。这么设计的两个原因:

- **省算力**:不需要先解码成视频、再让 GE-Act 重新编码——直接共享中间表示,跳过来回转换的浪费。
- **信息更丰富**:latent 比解码后的像素视频保留了更多结构化信息(像素化会损失细节)。

> 类比 DiT 你学过的概念:latent 是"VAE 压缩后的中间表示,信息密度高且省算力"。GE-Act 直接拿这层 latent 用,等于"我不需要看你画的图,我看你脑子里那张图的草稿就够了"。

**② Action Policy:并行 Transformer 分支**

它是一条**和 GE-Base 并行的 Transformer 分支**(类似挂了一个"动作头"),通过 cross-attention 从 GE-Base 那边吸取视觉信息,用 flow-matching(扩散家族的一种,跟 DiT 的去噪是亲戚)做动作生成 —— 从噪声出发,一步步去噪成"清晰的动作序列"。架构细节在 Figure 6。

**③ Action Chunk:一次出一串动作**

输出是 **Action Chunk** —— 注意这个词:**一次性输出"一串"动作,而不是一次只出一个动作**。图里画 4 个小方块是示意,实际是 **54 步一 chunk**。

每个"单步动作"是一个向量,装着这一瞬间机器人每个关节该转到哪、夹爪该张到多少;chunk 就是把这种"一帧动作"按时间排起来,N 个一组,合成一段未来的"动作脚本"。

为什么要 chunk 而不是单步?这是现代 VLA(π₀、ACT、Diffusion Policy)的共识做法:

- **一次预测多个未来步骤** → 减少推理频率 → 实时性好。
- **整段动作更平滑连贯**,不像单步预测容易抖。
- **拉开"推理时间 vs 执行时间"的余量**(后面那条 200ms / 54 步就是这个意思)。

**④ 实时性:200ms 生成 54 步**

论文里强调 GE-Act 能在 **200ms 内生成 54 步动作**(用一块 RTX 4090)。这个数字翻译一下:

- 动作模型设定为 30Hz → `54 步 ÷ 30步/秒 = 1.8 秒`(机器人执行这一 chunk 要 1.8 秒)。
- 推理这一 chunk 只用 0.2 秒。
- **机器人执行当前 chunk 的 1.8 秒里,下一 chunk 早在 0.2 秒就算好了**。等当前 chunk 跑完,下一 chunk 已经在等着——动作流连续,机器人永远不需要"停下来等模型想"。
- **9 倍余量 + 消费级 GPU + 能装机器人身上做 on-board inference**——这是它能在真机上跑实时操作的工程基础。

**⑤ Execute → 多种真机形态**

下面那一排不同的机械臂(双臂、人形、单臂……)是想强调 **cross-embodiment(跨形态)** —— GE-Act 不绑定一种机器人,**可以迁移到 AgiBot G1、AgileX Cobot Magic、Dual Franka 等多个平台**(只需少量该平台的微调数据,比如"一小时遥操作")。下方实拍图(Pour water、Assemble box、Fold clothes)是真机执行成功样例,都是接触丰富的硬任务。

#### GE-Act vs 传统 VLA(理解"统一"的关键)

| | 传统 VLA(π₀、OpenVLA 等)| GE-Act |
|---|---|---|
| 中介表示 | 把视觉→**语言空间**(VLM 把图变成"描述") | 把视觉→**视觉 latent 空间**(GE-Base 的中间表示)|
| 优势 | 语义抽象 | 保留空间/时间细节,更适合物理交互 |
| 平台耦合 | 一般需单独训 VLM + policy | **共享 GE-Base,policy 是"挂件"** |

GE 团队的论点:**机器人操作是物理任务,过语言空间会丢空间细节;直接在视觉空间做更合适**。这就是 "GE 是 vision-centric platform" 的来源。

---

### 2.4 右翼:GE-Sim(World Simulator)

**一句话钉死**:GE-Sim = **把 GE-Base 改造成一个"神经渲染器/神经仿真器"** —— 给它喂一段动作,它就生成"如果机器人按这段动作执行,接下来会看到什么视频"。

**和 GE-Act 反向看就明白了**——同一个 GE-Base 被两种不同接口复用:

| | GE-Act(左翼) | GE-Sim(右翼) |
|---|---|---|
| Action 的角色 | **输出**(模型生成动作) | **输入条件**(外部喂动作进来) |
| 输出 | 动作 chunk | 渲染出来的未来视频 |
| 一句话 | "看世界 → 出动作" | "给动作 → 生未来" |
| 类比 | 机器人的运动皮层 | 机器人的"想象力" |

> **两翼互为反向**:GE-Act 是 obs → action,GE-Sim 是 action → obs。这是 GE-Base 被两种接口复用的精妙之处。

#### 数据流(从右往左追)

```
Instruction(Wipe / Iron / Pack / Insert...)
       │
       ▼
Action Models(ACT / GR1 / Octo / π₀ / OpenVLA / ...)  ← 各种外部 policy
       │
       ▼ Action Condition
   ┌───────────┐
   │  GE-Base  │  (改造成 action-conditioned 版)
   └─────┬─────┘
         │ Rendered Action Execution
         ▼
   生成的视频(下面那条胶片)
         │
         └────── 🔄 Close-loop simulation ──── 回到 Action Models
```

#### 关键零件

**① Action Models(右上)**:ACT / GR1 / Octo / π₀ / OpenVLA…… 都是**别人家训好的 policy**。GE-Sim 不挑模型,**谁都能接进来测**——它是一个**通用的 policy 测试平台**,不绑定特定策略。

**② Instruction**:和 GE-Base 那边一样,告诉系统"这一轮的任务"。这指令同时给 Action Models(让它出对应任务的动作)和 GE-Base(让它生成对应任务上下文的视频)。

**③ Action Condition(动作 → 条件)**:外部 policy 算出来的动作序列,**作为"条件"喂给 GE-Base**。这一步是 GE-Sim 的灵魂:**action 在这里从"输出"变成了"输入条件"**。

但注意:action 是一串数值(关节角/末端位姿/夹爪),GE-Base 是个视频扩散模型,**它不会直接读这种数字**。所以中间必须有一套"把动作翻译成 GE-Base 能理解的条件"的机制 —— 论文用的是 **Pose2Image + Motion Vector** 两套组合(详见 Figure 14 / §5 GE-Sim)。

> 💡 **这正好是 Uni-WAM proposal 的核心关切**:**"action 怎么注入到 WM 里去"**。GE-Sim 的方案(Pose2Image + Motion Vector)是众多 action 注入方案中的一种,和 EA-WM 的 KVAFs、Ctrl-World 的 frame-level cross-attention、Action Images 的 mask 方案是**同一谱系的兄弟**。

**④ GE-Base(action-conditioned 版)** —— 中间共享的基座

注意飞机机身那架 GE-Base **同时出现在左翼和右翼**——但右翼用的是它的"**action-conditioned 变体**":

- 左翼用 GE-Base 是直接拿它的视觉 latent 喂给 GE-Act;
- 右翼用 GE-Base 是给它**加一个动作条件接口**,让它从"指令条件视频生成"升级成"指令+动作 双条件视频生成"。
- 但**底层权重是同一个基座 GE-Base 的微调版本**,不是从零另训的网络。这就是"统一平台"的真正体现。

**⑤ 输出:Generation for Action Execution(渲染视频)**:GE-Base 吐出一段视频,展示"**如果机器人按你给的这段动作走,接下来 N 帧会是什么样**"。下面那条胶片就是示意——你能看到机械臂一帧帧动起来执行任务的画面。

**⑥ 🔄 Close-loop simulation(闭环箭头,关键)**:Action Models 和生成的视频之间那个**双向循环箭头**,是 GE-Sim 真正威力的体现:

```
Policy 给出动作 a₁
     ↓
GE-Sim 生成"按 a₁ 走"的下一段视频 v₁
     ↓
Policy 看到 v₁,基于 v₁ 给出 a₂
     ↓
GE-Sim 生成 v₂
     ↓
... (循环)
```

把 GE-Sim **当成"虚拟真机"用** —— policy 不用真机就能跑完一整个任务,GE-Sim 全程当环境陪它演。

> 💡 **澄清开环 vs 闭环**:判断标准不是"有没有反馈",而是**"动作是不是由观测驱动的"**:
> - 预录动作 → WM 按动作画视频 = **开环**(就算把上一 chunk 最后一帧反馈给下一 chunk 当上下文,只要动作是事先定好的,就还是开环)。
> - Policy 看 obs → 临场决策 action → 喂回 WM → WM 生成新 obs → Policy 再看……= **闭环**(obs 真的影响了下一步 action)。
>
> GE-Sim 的 🔄 箭头就是闭环的标志。

#### GE-Sim 的两大用途

1. **Closed-loop Policy Evaluation(闭环 policy 评估)**:不用买十台真机、不用招遥操作员,把新 policy 接进 GE-Sim 跑几百次任务、看成功率 —— 省钱、安全、可批量、可复现。
   - ⚠️ **但这正是 Uni-WAM proposal 要拷打的点**:GE-Sim 的可信度依赖"WM 在 policy 出的各种动作下能否生成真实视频"。policy 是 expert 时 GE-Sim 可能没事;但 policy 是早期菜鸟、出 off-expert action 时,GE-Sim 还可信吗?**这正是 Action Following Fidelity benchmark 要量化的事**。
2. **Controllable Data Generation(可控数据生成)**:给一个指令 + 一段动作,GE-Sim 生成对应视频。把这些 (action, video) 对**当合成数据**喂给下游 policy 训练 → **数据增强**。论文叫 **Data Engine**。

---

### 2.5 底部:EWMBench(评估套件)

**一句话钉死**:EWMBench = **专门给具身视频世界模型打分的评估套件**。GE-Base / GE-Act / GE-Sim 都能生成视频,但生成得**好不好、能不能信任**需要有人评 —— EWMBench 干的就是这件事。

#### 为什么需要它

机器人世界模型生成视频有个天然评估难题:**生成出来的画面没有 ground truth**(模型脑补的,谁来打分?)。视频生成圈早就有的 FID(回忆 DiT)只测**单帧视觉真实度**,但机器人场景的好坏远不只"画得像"——还要看动作连不连贯、物理违不违和、操作对不对。所以**专门为具身视频造一套评估**很有必要。

#### 三个底座

Figure 1 底部把 EWMBench 拆成三块:

**① Dataset:Comprehensive Evaluation Set(评估数据集)** —— **提供"标准考卷"**。一组精心策划的评估用机器人操作场景,涵盖家庭和工业。所有 WM 都在同一份考卷上跑,才好跨模型比较。(类比:DiT 用的 ImageNet 让大家都在同一个 benchmark 上算 FID。)

**② Tools:Multi-dimensional Toolkit(多维评估工具箱)** —— **提供"测量仪器"**。一组可复用的评测脚本/工具,覆盖 perception / prediction / control。比如:SAM 提取末端轨迹、CLIP/DINO 算视觉一致性、RAFT 算光流评运动质量、VLM 评指令-动作对齐……(类比 WorldArena 的 16 指标速查表——工具盒子里的"测量笔"。)

**③ Metric:Evaluation Framework(评估框架)** —— 把工具组织成**三大维度**:

| 维度 | 代表什么 | 关注的"动态" |
|---|---|---|
| 🎬 **Scene** | 场景质量 | **空间(spatial)** —— 画面里的物体、背景、布局对不对 |
| 🐍 **Motion** | 运动质量 | **时间(temporal)** —— 帧之间运动连贯吗?物理合理吗?轨迹准吗? |
| 🧠 **Semantics** | 语义对齐 | **语义(semantic)** —— 视频内容跟指令/任务要求一致吗? |

这三维基本覆盖了"一段机器人操作视频好不好"的所有关键侧面。

#### EWMBench 在平台里的角色

布局上 EWMBench 放在**最底部、横跨整图宽度**,不连任何箭头 —— 这是有讲究的:

- 它**为 GE-Base / GE-Act / GE-Sim 三个模块"统一服务"** —— 上面任何一个模块产出的视频都能扔到 EWMBench 打分。
- 它**不参与生成流程**,只在"想评估某个模块好不好"时被拿出来用。

> 类比:GE-Base / GE-Act / GE-Sim 是**运动员**,EWMBench 是**裁判+评分表+成绩册**。裁判不下场,但每个运动员的表现都靠它衡量。

> 📌 EWMBench 后来被作者**独立写成一篇论文**(`[arXiv 2505.09694] EWMBench`),专门讲它的设计细节、指标公式、跨模型评估结果。Figure 1 里的描述只是简略版,真正细节都在那篇里。

---

### 2.6 Figure 1 一句话收束

> **Genie Envisioner = 一个共享的视频扩散基座(GE-Base) + 两个挂在上面的功能机翼(GE-Act 控真机、GE-Sim 做闭环仿真) + 一个评估套件(EWMBench)**,组成"训练 / 部署 / 评估"一体化的具身操作平台。

🔥 **关键 insight**:
- **GE-Base 是 backbone**,所有其他模块都从它衍生(机身,撑起整张图)
- **policy 和 simulator 共享 backbone** —— Uni-WAM proposal 里"MoT 一体化"思路的现实版本
- **EVAC 演化为 GE-Sim** —— 同团队同思路,但更大规模、更系统化
- **两翼互为反向**:GE-Act 是 obs → action,GE-Sim 是 action → obs

---

## 🔄 前置认知：GE 是 EnerVerse-AC 的"宏大版"

| 维度 | EnerVerse-AC (2025/05) | **Genie Envisioner (2025/08+)** |
|---|---|---|
| 范围 | 单个 AC-WM (评估器 + 数据引擎) | **平台**: WM + Policy + Simulator + Benchmark |
| Backbone | EnerVerse VDM | **LTX-Video 2B** + 自家改造 |
| 训练数据 | AgiBot World 一部分 | **AgiBot-World-Beta 全集, 1M+ trajectories, 3000+ hours** |
| 训练计算 | 32 A100 × 8 days | **GE-Base: 16×8 = 128 A100 × 数天; GE-Act: 8 A100 × 36 hours** |
| Policy output | ❌ 不直接出 action（只是 simulator）| ✅ **GE-Act 直接出 action chunk** |
| Benchmark | 无 | **EWMBench 配套** |
| 开源 | ✅ | ✅ (https://genie-envisioner.github.io) |

→ **Genie Envisioner = EnerVerse-AC 升级 + Policy 模块 + Benchmark**。同团队（Yuxin Jiang 也在作者列表里）。

---

## 3. 核心方法 · Figure 3 + Figure 7

### Figure 3：GE-Base 架构 = autoregressive video chunk generation

![](../images/figure-03.png)

**(a) 左图**：autoregressive 工作流
- 历史 frames + noise → GE-Base → 下一段 video chunk
- 一个 chunk 一个 chunk 地预测，每个 chunk 内多视角联合

**(b) 右图**：Causal Block 内部
- **Self-Spatial Attention**（左视角 / 前视角 / 右视角各做空间 attention）
- **Cross-Attention**（跨视角信息交换）
- 总共 N+M blocks

🔥 **Action 注入方式 ≠ 在 GE-Base 里**：
- **GE-Base 是 instruction-conditioned**（接语言, **不接 action**）
- 它本质是个 **language-conditioned video gen model**
- Action conditioning **发生在 GE-Sim** 中（GE-Sim 是 GE-Base 的 action-conditioned 衍生）

→ 严格说 GE-Base 自己**不是 AC-WM**，但**整个平台是**。GE-Base + action condition adapter = GE-Sim = AC-WM。

### Figure 14：GE-Sim 的 action 注入机制（这才是真正的 AC-WM 部分）

![](../images/figure-14.png)

**§5 GE-Sim 是 paper 真正的 AC-WM 部分**，独立成章。核心创新 = **Hierarchical Action-Conditioning Mechanism**（图 a 左侧）。

#### Action 输入（7D × K 步）

每一步 action = **7D vector**：`[x, y, z, roll, pitch, yaw, gripper_openness]`
- 位置（xyz）+ 朝向（rpy）+ 夹爪状态 = 7D
- 双臂时拼成 14D（左 7 + 右 7）
- K 步合在一起 = `A ∈ R^{K × 14}`

#### 两路注入（Pose2Image + Motion Vector）

**第 1 路: Pose2Image Conditioning**（视觉 token 层注入）

每个 timestep i 的 pose `a_i` → 画成 pose image `P_i`:
1. **位置** (x_i, y_i, z_i) → 用相机内外参 project 到 2D 像素坐标
2. **朝向** (r_i, p_i, y_i) → 转 rotation matrix, 把三个正交轴 project 到 image plane（指示方向）
3. **gripper** o_i → 画在 unit circle 上，**颜色深浅代表开合**（淡色 = 开，深色 = 闭）
4. **左右臂** 用不同色区分

→ pose image `P_i` 和历史帧 `I_i` 都用**同一个 video encoder ε 编码**, 然后**element-wise add**：

$$v_i = \varepsilon(I_i) + \varepsilon(P_i)$$

合成 token `v_i` **作为 visual token 注入 generation stream**。

🔥 **这就是 EnerVerse-AC 的 "Spatial-Aware Pose RGB" 思路** —— **把 6D pose 画成 RGB 图，然后和 obs 图一起编码**。

**第 2 路: Motion Vector Conditioning**（cross-attention 注入）

计算连续 pose 的 delta：

$$\Delta a_i = a_i - a_{i-1} = [\Delta p_i, \Delta r_i]$$

→ 经过 learnable encoder → **和 reference image style token concatenate** → **通过 cross-attention 注入到每个 DiT block**

🔥 **这就是 EnerVerse-AC 的 "Delta Action Cross-Attention" 思路** —— **temporal 动作变化通过 cross-attention 注入**。

#### 训练（§5.2 简略）

- 从 **GE-Base-MR**（high-temporal-resolution variant）初始化
- 在 **full AgiBot-World-Beta** 上训
- 用 **ground-truth action trajectories** 做 conditioning input
- 训练 corpus 加入 **failure cases**（incomplete behaviors, suboptimal control）— 和 EVAC 一脉相承

#### GE-Sim vs EVAC 的关系

| | EnerVerse-AC | **GE-Sim** |
|---|---|---|
| Spatial-Aware Pose RGB | ✅ | ✅ Pose2Image Conditioning |
| Delta Action Cross-Attention | ✅ | ✅ Motion Vector Conditioning |
| Gripper magnitude RGB | ✅ | ✅（合并在 Pose2Image 里）|
| Failure data | ✅ 人工 augmented | ✅ AgiBot-World-Beta 含 failure |
| Backbone | EnerVerse VDM | GE-Base (LTX-Video 2B 或 COSMOS2 2B)|
| 规模 | 中 | 大（用 GE-Base-MR）|

→ **GE-Sim ≈ EnerVerse-AC 的升级实现** —— 同思路、同两路注入、更大 backbone、更大数据。

### Figure 7：GE-Act 3-Stage 训练

![](../images/figure-07.png)

**3 阶段 pipeline**（不同 GPU 时长）：
- **Stage 1**: Action Pre-training（54 step 序列 @ 30Hz, **AgiBot-World-Beta full data**, 3 days × 16×8 GPU）
- **Stage 2**: Task-Specific Video Adaptation（freeze GE-Base, 微调到 task data, 12 hours × 8 GPU）
- **Stage 3**: Task-Specific Action Specialization（24 hours × 8 GPU）

🔥 **GE-Act 是 VLA 范式**: vision-language-action, 但**搭载在 video world model 上**作为 backbone。

---

## 4. 实验（简略）

Paper 的实验主要 demo GE-Base 在 multi-view robot manipulation 视频生成上质量高（Figure 5 等定性结果）+ GE-Act 在 AgiBot 上能完成多种 robot task。

具体的 ranking alignment / OOD action 测试**不是 Genie Envisioner 的重点** —— 它聚焦"作为基础设施"，不是 evaluation framework。

→ **量化对比 baseline** 在 EWMBench 上（不在这篇 paper 里）。

---

## 5. Summary · 整篇 paper 一段话

> **Genie Envisioner (GE)** 是 AgiBot Genie Team 提出的 robot manipulation **统一世界基础模型平台**。它把**数据 / WM 训练 / Policy 推理 / Simulator / Benchmark** 全部塞进**一个 closed-loop video-generative framework**:
> - **GE-Base** = 大规模 video diffusion (LTX-Video 2B based)，instruction-conditioned 多视角 video gen
> - **GE-Act** = lightweight flow-matching decoder, 把 latent → action chunk（policy）
> - **GE-Sim** = action-conditioned 衍生, 当 simulator 用（**EnerVerse-AC 的演进**）
> - **EWMBench** = 配套 benchmark
>
> 训练数据：AgiBot-World-Beta **1M+ trajectories, 3000+ hours video**。模型、checkpoints、code 全开源。

### 三个最该记住的点

| # | 点 | 一句话 |
|---|---|---|
| 1 | **平台范式** | 4 个模块共享 backbone, 一体化 closed-loop（数据 / WM / Policy / Sim / Bench）|
| 2 | **GE-Base 是 instruction-conditioned, 不直接接 action** | Action 在 GE-Sim 衍生 |
| 3 | **EnerVerse-AC 的演化** | EVAC = GE-Sim 的前作, 同团队同思路, 但更小规模 |

### 一句话标签

> **Genie Envisioner = AgiBot 出品的"robot 版 Cosmos 平台"** — 不只是一个 model, 是一个 4-in-1 unified framework。

---

### 🔥 Uni-WAM 视角的简评

**对 Uni-WAM 的高价值**:
- ✅ **平台范式启发**：Uni-WAM proposal 里"MoT 一体化"思路在 GE 上有现实版本（不同模块共享 backbone）
- ✅ **数据规模参考**：AgiBot 1M+ trajectories 是真机数据上限的 reference
- ✅ **GE-Sim 作为 AC-WM 的实现**：和 EnerVerse-AC 思路一脉相承, 但更大规模, **是 Uni-WAM 的直接竞品**
- ⚠️ Paper 主要做"基础设施", **没系统对照 off-expert action 评估**

**Uni-WAM 仍能切入的空白**:
- GE 平台**不做** off-expert action 系统化诊断
- EWMBench 是从 GE 系列的"自评"角度做的, 不像 Uni-WAM 的 Action Following Fidelity 那样**主动暴露 WM 弱点**

**分类**：**a 类 AC-WM 平台**（GE-Sim 部分）+ **b 类候选**（GE-Base 是 backbone, 提供完整 finetune pipeline）。
