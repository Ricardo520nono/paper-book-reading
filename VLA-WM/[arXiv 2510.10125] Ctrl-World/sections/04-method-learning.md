[← 返回 Ctrl-World 主页](../README.md)

# §4.1 Learning World Model Ctrl-World 🔥🔥

> **本节阅读重点**：**Ctrl-World 的核心方法**。看它怎么把 SVD 改造成 AC-WM。三个组件 + 训练目标。**这是判 a/b 分类、看 action 注入方式的关键**。

---

## 🎯 Figure 2：Ctrl-World 架构图（🌟 全篇最关键的一张图）

![](../images/figure-02.png)

**怎么看这张图（从左到右）**：

**左半部分（输入构造）**：
- 上排时间轴：稀疏历史帧 $o_{t-km}, ..., o_{t-2m}, o_{t-m}, o_t$ + 未来要预测的 $o_{t+H}$
- **History Poses + Action Chunk Poses**（蓝绿色块）：历史 pose + 未来 action 转 pose
- **Spatial Tokens**：每个 frame 拆成 P = N×H×W 个 token（N 个相机视角）
- **CLIP Semantic Tokens**：用 CLIP 提语义特征作为条件

**中间（Spatial + Temporal Transformer 主干）**：
- Spatial Transformer：处理空间维度的 token
- Temporal Transformer：处理时间维度
- 这两个串起来 × N 次（N 是 transformer block 数）

**右半部分（Frame-Level Cross-Attention）**：
- 🌟 **这是 Ctrl-World 核心创新**：每一帧的 visual token 通过 cross-attention 关注**该帧对应的 pose embedding**
- 历史帧 attend 到真实 pose（$q_{t-km}, ..., q_t$）
- 未来帧 attend 到 action 转换出来的 pose（$a'_{t+1:t+H}$）

🔥🔥 **Uni-WAM 视角下，这张图的关键信息**：
- **Action 注入位置**：Spatial Transformer 内部的 Frame-Level Cross-Attention
- **Action 表示**：Cartesian 6D pose（不是 joint angle）
- **每帧严格对齐**：第 t 帧的 visual token 只 attend 到第 t 帧的 pose
- **新增模块**：Frame-Level Cross-Attention 是从 SVD backbone 上**新加的**，其他都 inherit

→ 想测 5 类 off-expert action 时，**直接换右边 pose embedding 就行**，架构层完全允许。但**训练数据没见过 → WM 不一定能消化**（这正是 Uni-WAM 要诊断的）。

---

## ¶1 · 整体目标：从 passive video gen 改造成 controllable AC-WM

> "Our goal is to learn a world model that can be used to evaluate and improve modern VLA policies. To achieve this, the model must first support **multiview observations** that are commonly used by such policies. It is also important for the model to be **controllable — reliably and closely follow the action inputs** — even when initialized from a pre-trained backbone that lacks such control. Finally, the model must **maintain temporal consistency over long horizons**, even in the presence of occlusions, to produce coherent rollouts. We initialize our world model from a pretrained video diffusion backbone with spatial-temporal transformers (Blattmann et al., 2023b) and introduce three key adaptations, illustrated in Figure 2."

**翻译**：

Ctrl-World 的目标：训练一个能评估和改进 VLA policy 的 WM。3 个要求：
1. 支持多视角观测
2. **可控** —— 能可靠跟随 action 输入（哪怕 pretrained backbone 本身不支持）
3. 长时一致（哪怕有遮挡）

**Initialize from**：pretrained video diffusion backbone (Blattmann 2023b = **Stable Video Diffusion, SVD**)，做 3 个 adaptation。

🔥 **Uni-WAM 关联**：
- "even when initialized from a pre-trained backbone that lacks such control" —— 这句话**暗示了 b 类的可能性**。原来的 SVD 是 passive video gen，Ctrl-World 通过 fine-tune **赋予 action control 能力**。这可以视为 **"在通用 video model 上做 AC-finetune"** —— 接近 Uni-WAM 调研的 **b 类** 定义（"WM 但开源提供 AC-WM finetune pipeline"）
- ⚠️ 但 Ctrl-World 没明确说"我提供 finetune pipeline" —— 它只是**自己做了 finetune**，结果是一个 a 类 model。**分类要看代码开源情况**

---

## ¶2 · 组件 1：Multi-View Joint Predictions

> "State-of-the-art VLA models often rely on multiple third-person cameras for global context and wrist-mounted cameras for precise interactions. To match this, the world model must generate spatially consistent predictions across all views at each step. Prior work has shown that feed-forward transformers can effectively capture spatial relationships between multi-view cameras in a scalable manner. Following prior work, we concatenate the N input images—each containing H × W tokens—along the token dimension and jointly predict all views $o_{t:t+H}$. In experiments, we find multi-view joint prediction also improves consistency and substantially reduces hallucinations."

**翻译**：

SOTA VLA 依赖多个 third-person 相机 + wrist-mounted 相机。WM 必须跨视角空间一致。

**做法**：把 N 张多视角图像（每张 H×W tokens）沿 token 维度 **concatenate**，然后**联合预测所有视角**。

→ 多视角联合预测同时改善一致性 + 减少幻觉。

💡 **批注**：这是工程细节，对 Uni-WAM 分类判断不重要。**跳过深入数学**。

---

## ¶3 · 组件 2：Pose-conditioned Memory Retrieval Mechanism

> "Prediction errors in world models tend to accumulate over long rollouts, leading to drift and incoherence. To mitigate this, we augment the model input with past frames. To prevent the context from becoming too long, we **sample k history frames with a stride m**, enabling the model to predict $o_{t+1:t+H} \sim W(\cdot|o_{t-km}, ..., o_t, l)$. Additionally, we embed the corresponding robot arm poses $[q_{t-km}, ..., q_t]$ into frames $[o_{t-km}, ..., o_t]$ via **frame-wise cross-attention** within spatial transformer. This allows the model to use the arm pose to identify relevant frames from the past, effectively anchoring future predictions to relevant history."

**翻译**：

WM 在长 rollout 时误差累积 → drift / 不连贯。**解法**：
1. 输入加 **k 个历史帧**，间隔为 stride m（防止 context 太长）
2. 把对应的 **robot arm pose $[q_{t-km}, ..., q_t]$** 嵌入每个历史帧 via **frame-wise cross-attention**

→ 用 pose 来"identify relevant past frames"，把预测 anchor 到相关历史。

💡 **批注**：
- 这是工程优化（长时一致性）
- 关键 idea：**pose 作为"内容索引"**。如果当前 pose 和某个历史帧的 pose 相似，模型应该注意那一帧的内容（比如"机械臂回到差不多的位置 → 物体应该还在那"）
- 对 Uni-WAM 分类不重要，但是个值得记的工程技巧

---

## ¶4 · 组件 3：Frame-level Action Conditioning 🔥🔥（决定一切的关键）

> 💡 **基础概念**：什么是 AC-WM、action 注入有哪几种方式、为什么"架构层能喂 ≠ 训练层能消化" —— 详见 [`_concepts/action-conditioned-wm.md`](../../../_concepts/action-conditioned-wm.md)



> "The pretrained video model conditions only on text and image, which limits its control precision. To enable full controllability, we additionally **condition the model on the action sequence $[a_{t+1:t+H}]$** output by the policy. We also **transform each action sequence into Cartesian-space robot arm poses $[a'_{t+1:t+H}]$** and concatenate with past poses $[q_{t-km}, ..., q_{t-m}, q_t]$. **Frame-wise cross-attention** (Zhu et al., 2024; He et al., 2025) is then applied within the spatial transformer, allowing the visual tokens of each frame to attend to its associated pose embedding. For history frames, this pose corresponds to $[q_{t-km}, ..., q_{t-m}, q_t]$, while for future frames, it corresponds to $[a'_{t+1:t+H}]$."

**翻译**：

预训练 video model 只接 text + image，控制精度不够。Ctrl-World **额外接 policy 输出的 action 序列**：
1. 取 policy 输出 $[a_{t+1:t+H}]$
2. **把 action sequence 转换成 Cartesian-space robot arm pose $[a'_{t+1:t+H}]$**（关键变换！）
3. 和历史 pose $[q_{t-km}, ..., q_t]$ concatenate
4. 用 **frame-wise cross-attention** 让每一帧的视觉 token attend 到对应的 pose embedding
   - 历史帧 → pose 是 $[q_{t-km}, ..., q_t]$（真实历史 pose）
   - 未来帧 → pose 是 $[a'_{t+1:t+H}]$（action 转换出来的目标 pose）

🔥🔥🔥 **Uni-WAM 关联 —— 核心信息！** 决定了 Ctrl-World 的分类和 action 注入方式

### Action 注入方式（回答主题问题 Q1）

**Ctrl-World 的 action 是：Cartesian-space robot arm pose**（即 6D pose / 末端位姿）

- ❌ 不是 joint angle（关节角度）
- ❌ 不是 latent action（隐式 action）
- ❌ 不是 language（语言指令）
- ✅ **是 6D pose**（位置 + 朝向）

**注入机制**：cross-attention，每帧视觉 token attend 到对应的 pose embedding。

⚠️ **重要 nuance**：
- Policy π 输出的"action"可能是 joint-space action（具体什么取决于 policy）
- Ctrl-World **强制将其转换到 Cartesian-space pose** $a' = \text{FK}(a)$（forward kinematics 或类似变换）
- 也就是说 **Ctrl-World 的 WM 实际接收的是"末端位姿轨迹"**，不是"关节动作序列"

### 对 Uni-WAM 5 类 off-expert 的可测性

| Off-expert 类别 | Ctrl-World 接得了吗？|
|---|---|
| **Perturbed expert**（pose 加噪）| ✅ 可以 —— 直接给扰动版 pose 序列 |
| **Counterfactual**（反方向 pose）| ✅ 可以 —— 给反向 pose 轨迹 |
| **Exploratory**（早期 policy sub-optimal pose）| ✅ 可以 —— 让 sub-optimal policy 输出 pose 喂进来 |
| **Random-feasible**（关节空间均匀采样）| ⚠️ **不直接** —— 需要先把 joint angle 转 pose，可能丢失关节空间信息 |
| **Adversarial**（对抗）| ✅ 可以 |

→ Ctrl-World 的 pose 输入接口理论上**支持 4/5 类 off-expert 测试**，但**它自己没测**。

### 关键贡献者：Zhu et al. 2024

> "Frame-wise cross-attention (Zhu et al., 2024; He et al., 2025)"

Zhu et al. 2024 是 **frame-level action conditioning** 的原始 paper。**这是 Uni-WAM 调研的高优先级候选**（已在 backlog 标注）。

---

## ¶5 · Training Objective

> "We initialize our model with the pretrained **1.5B Stable-Video-Diffusion (SVD) model**. To inherit the knowledge and structure in the pretrained video model, we **only newly initialize an action-projection MLP for the input actions** and keep other parameters unchanged at initialization. Then this action-conditioned world model is fine-tuned with diffusion loss. During training, the prediction target $x_0 = o_{t+1:t+H}$ is perturbed with Gaussian noise..."

公式 (3)：
$$L = E_{x_0, \epsilon, t'} \|\hat{x}_0(x_{t'}, t', c) - x_0\|^2$$

where $c = [q_{t-km}, ..., q_t, a'_{t+1:t+H}, o_{t-km}, ..., o_t]$

**翻译**：

- **Backbone**：**1.5B SVD**（Stable Video Diffusion, Blattmann 2023a）
- **新加的参数**：**只有一个 action-projection MLP**（把 raw action 投影成 embedding）
- **其他参数 inherit** from SVD
- **Loss**：diffusion loss（学预测 clean target $x_0$）

🔥🔥 **Uni-WAM 关联 —— 这一段决定 a/b 分类**

**关键事实**：
1. **Backbone = 通用 video model（SVD）**，本身**不是** AC-WM
2. **加 action 的代价 = 加一个 MLP + fine-tune**
3. **所有其他权重 inherit**

这描述了一个**完整的 "video model → AC-WM" 的 finetune 流程**：
- 拿 SVD
- 加 action projection MLP
- 加 frame-level cross-attention（在 spatial transformer 里）
- 用 diffusion loss fine-tune

**问题**：Ctrl-World 是否**开源**这个 finetune pipeline？
- 如果开源 → **b 类**（"WM 但开源提供 AC-WM finetune pipeline"）
- 如果不开源 → **a 类**（"原本就是 AC-WM"，end-to-end 训出来的）

→ **待确认**：查 GitHub repo 看是否提供 SVD → Ctrl-World 的 finetune 脚本。

---

## 💡 §4.1 整段 takeaway

| 维度 | Ctrl-World |
|---|---|
| Backbone | **Stable Video Diffusion 1.5B**（passive video gen）|
| Action 注入位置 | **Frame-level cross-attention** in spatial transformer |
| Action 表示 | **Cartesian-space 6D pose**（不是 joint angle）|
| 新增参数 | **只有一个 action-projection MLP** |
| 训练 loss | Diffusion loss |
| AC-WM 分类初判 | **a 类**（自己训出一个 AC-WM），**但实际架构非常接近 b 类**（SVD + finetune pipeline）|

**最关键的发现**：
- **Action 表示是 Cartesian pose** —— 比 joint angle 更 abstract，但**也意味着 Uni-WAM 想测"关节空间 random-feasible"时需要先转换**
- **架构上是 b 类的雏形** —— 从 SVD 这种通用 video model 改造成 AC-WM 的流程很清晰，**如果他们 release finetune 代码就是 b 类**

---

## 🤔 §4.1 留下的问题

| Q | 在哪里查 |
|---|---|
| Ctrl-World 是否开源 SVD finetune 脚本？ | GitHub repo |
| Cartesian pose 是 7D (x,y,z + quaternion) 还是 6D？ | Appendix A or repo |
| frame-level cross-attention 具体 mask 怎么设？ | Appendix A |
| 训练时 action chunk 内部 H 步之间的关系？ | Appendix A |

---

[← §3 Problem Formulation](03-problem-formulation.md) | [§4.2 Using Ctrl-World →](04-method-policy-eval.md)
