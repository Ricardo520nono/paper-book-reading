[← 返回 ABot-PhysWorld 主页](../README.md)

# §3 Method

> Method 三块：backbone（§3.1）→ 物理偏好对齐（§3.2）→ 动作条件生成（§3.3）。

---

## §3.1 Embodied Video Generation Backbone

> "we build upon Wan2.1-I2V-14B, and fully fine-tune it on our curated embodied dataset."

**翻译**：基于 **Wan2.1-I2V-14B**，在策划的具身数据集上**完整 fine-tune**。

🔥 **Uni-WAM 关联**：backbone 是 **Wan2.1-I2V-14B** —— 和 Uni-WAM proposal 计划用的 **Wan2.2-TI2V-5B 同系列**。所以 ABot-PhysWorld 的整个训练范式对 Uni-WAM 有直接参考价值。

---

## §3.2 Physical Preference Alignment ⭐

![](../images/figure-02.png)

> "While SFT teaches the model to reproduce training distributions, it treats all samples equivalently and cannot distinguish physically correct predictions from those containing violations such as object penetration or anti-gravity motion."

**翻译**：SFT 只教模型**复现训练分布**，对所有样本一视同仁，**不能区分物理正确 vs 物理违例**。所以要加一个**后训练偏好对齐 pipeline**（Figure 2 的 Stage 2）。

### §3.2.1 Decoupled VLM Discriminator（解耦判别器）

> "Evaluating physical plausibility with a single VLM risks self-evaluation hallucinations, where the same model that generates questions also judges answers. To prevent this, we decouple the evaluation into two roles."

**翻译**：用单个 VLM 评估物理合理性会有**自评估幻觉**（出题的和判题的是同一个模型）。所以解耦成两个角色：

| 角色 | 模型 | 做什么 |
|---|---|---|
| **Proposer**（出题）| Qwen3-VL 32B Thinking | 看首帧 + 指令 → 动态生成**任务专属物理 checklist**（Tier 1 致命违例单票否决 / Tier 2 微观保真区分）；显式构造正负向问题混合，防止打分模型谄媚 |
| **Scorer**（打分）| Gemini 3 Pro | 用 Chain-of-Thought（全局扫描 → 标可疑帧 → 回溯确认）对 N 个候选打分 |

**选样本**：用 **knockout tournament**（淘汰赛选最优 yw + loser-bracket 选最差 yl），O(N) 复杂度，避免全排列比较 → 得到 DPO triplet (x, yw, yl)。

💡 **批注**：这个"出题模型 ≠ 判题模型"的解耦设计，和翔哥 proposal Gated Metrics 的 GPR 思路一致 —— 都是怕"自己出题自己判"导致评估失真。

### §3.2.2 Diffusion-DPO Training

DPO loss（公式 1）：

$$L_{DPO} = -\mathbb{E}\left[\log\sigma\left(-\frac{\beta}{2}\left[\underbrace{(L_\theta(z_w) - L_\theta(z_l))}_{\text{Policy Diff.}} - \underbrace{(L_{ref}(z_w) - L_{ref}(z_l))}_{\text{Ref. Diff.}}\right]\right)\right]$$

**人话翻译**：
- $z_w$ = physics-compliant（合规）视频的 latent，$z_l$ = physics-violating（违例）视频的 latent
- DPO 目标 = **主动降低 $z_w$ 的预测误差，同时抬高 $z_l$ 的预测误差**
- 即"让模型更愿意生成物理合规的、更不愿意生成违例的"

**关键工程问题 + 解法**：
> "Standard DPO requires maintaining two complete computation graphs (πθ and πref), causing out-of-memory errors for a 14B DiT."

标准 DPO 要同时维护 policy model 和 reference model 两个完整计算图 → 14B DiT 直接 OOM。

**解法**：冻结 DiT backbone，注入 **LoRA（rank 64）** 到 self-attention（q/k/v/o）和 FFN 层。**禁用 LoRA 权重就得到 reference model** → 零额外内存算 $L_{ref}$。

---

## §3.3 Action-Conditioned Video Generation ⭐

![](../images/figure-04.png)

> "given the current observation and a future action sequence, it should produce physically plausible videos that faithfully follow the commanded trajectory. Directly injecting low-dimensional robotic commands into high-dimensional visual pipelines creates a semantic gap."

**翻译**：动作条件生成的目标 = 给当前观测 + 未来动作序列 → 生成"忠实跟随指令轨迹"的物理合理视频。难点：低维 robot 命令直接注入高维视觉 pipeline 有**语义鸿沟**。

### §3.3.1 Action Map Construction

> 💡 基础概念：action 注入的多种方式见 [`_concepts/action-conditioned-wm.md`](../../../_concepts/action-conditioned-wm.md)

**输入**：7D action `a ∈ R^7`（3D 位置 + 3D 朝向 + gripper openness），双臂扩展到 14D。

**怎么画成 Action Map**：
| 分量 | 怎么编码 |
|---|---|
| **3D 位置 (x,y,z)** | 用相机内外参 project 到 2D 中心 (u,v) |
| **3D 朝向** | rotation matrix 的三个主轴 → 投影到 image plane → 渲染成**彩色箭头**（长度编码深度）|
| **gripper openness** | (u,v) 处的**圆形 mask**，不透明度线性表示开合度 |
| **双臂区分** | 左/右臂用红/蓝通道 → 多通道 action map |

🔥 **和 EnerVerse-AC / GE-Sim 对比**：这个 Action Map 构造**和 EnerVerse-AC 的 Pose2Image、GE-Sim 的 Pose2Image Conditioning 思路几乎一样** —— 都是"把 pose 画成 RGB 图"。三篇 paper 在 action 表示上**收敛到了同一个做法**。

### §3.3.2 Action Injection

> "Existing action injection methods either use AdaLN for MLP-encoded actions, which hinders cross-embodiment generalization, or concatenate action maps directly with noisy latents for full fine-tuning, causing catastrophic forgetting of pre-trained physical priors."

**翻译**：现有 action 注入方法的两个问题：
- **AdaLN + MLP-encoded action**：阻碍跨 embodiment 泛化
- **action map 直接和 noisy latent 拼接 + full fine-tune**：灾难性遗忘预训练物理先验

**ABot-PhysWorld 的解法**（Figure 4 的并行结构）：
> "we clone selective blocks from the main DiT to form a parallel set of context blocks that process the action maps. The output of each context block is projected via zero-initialized convolution layers and added residually to the corresponding main DiT block"

公式 2：
$$x_i = \text{DiT}_i(x_{i-1}) + \alpha \cdot W_{zero} h_i^{(i)}$$

- clone 主 DiT 的**部分 block**（每隔 5 个）→ parallel context blocks 处理 action maps
- context block 输出经过 **zero-initialized conv** → 残差加到主 DiT 对应 block
- $\alpha$ = 控制 scale

> "Because the zero initialization ensures that the context branch contributes no signal at the start of training, the backbone weights remain undisturbed, preserving pre-trained physical priors while gradually learning action controllability."

🔥 **关键设计 = zero-init**：训练开始时 context 分支贡献为 0 → backbone 权重不被打扰 → **保留预训练物理先验**，同时逐渐学动作可控性。这是 VACE 框架的思路。

---

## 💡 §3 整段 takeaway

| 子节 | 一句话 |
|---|---|
| §3.1 Backbone | Wan2.1-I2V-14B full fine-tune（和 Uni-WAM 同系列）|
| §3.2 物理 DPO | decoupled discriminator（Qwen3-VL 出题 + Gemini 3 Pro 打分）+ LoRA Diffusion-DPO，主动教物理对错 |
| §3.3 动作注入 | Action Map（pose 画成 RGB）+ 并行 context block + zero-conv 残差融合，保留预训练物理先验 |

**最关键的两个设计**：
1. **DPO 用"禁用 LoRA = reference model"** 解决 14B 双计算图 OOM
2. **并行 context block + zero-init** 解决"动作注入会灾难性遗忘物理先验"

---

[← §2 Data Curation](02-data-curation.md) | [§4 EZSbench →](04-benchmark.md)
