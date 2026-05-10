[← 返回论文 README](../README.md)

# 99 · Wan 2.2 增量补丁（vs Wan 2.1）

> **为什么这一节独立成文**：Wan 2.2 没有独立 arXiv 论文，它在 [Wan-Video/Wan2.2 GitHub README](https://github.com/Wan-Video/Wan2.2) 的 Citation 块里引用的就是这篇 2503.20314（即 Wan 论文）。所以"Wan 2.2 的新东西"全部记录在它的 GitHub README 的 *Introduction of Wan2.2* 那一节，本文档对这一节做批读。

---

## 📌 一图速看：Wan 2.1 → Wan 2.2 改了什么

```
Wan 2.1                              Wan 2.2
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
单一 dense backbone                   ✅ MoE：双专家（high-noise + low-noise）
                                       ↳ 27B 总参，每步只激活 14B
                                       ↳ 按 SNR 阈值切换专家
─────────────────────────────────────────────────────────────────
1.3B / 14B 两档                       ✅ A14B 系列(MoE) + TI2V-5B(Dense, 新)
                                       + S2V-14B (音频驱动, 新模型)
                                       + Animate-14B (角色动画, 新模型)
─────────────────────────────────────────────────────────────────
Wan-VAE: 4×8×8                       ✅ Wan2.2-VAE: 4×16×16 (压缩率 64)
                                       ↳ 配合 patchify 总压缩 4×32×32
─────────────────────────────────────────────────────────────────
通用视觉数据                         ✅ 数据 +65.6% 图像 / +83.2% 视频
                                       + 美学标注（光线/构图/对比/色调）
─────────────────────────────────────────────────────────────────
8 个下游任务                          ✅ 同上，但 5B 模型一个就支持 T2V+I2V
─────────────────────────────────────────────────────────────────
```

---

## 📄 原文 + 批注

> 来源：[Wan-Video/Wan2.2 GitHub README — *Introduction of Wan2.2* 节](https://github.com/Wan-Video/Wan2.2#introduction-of-wan22)

### 总纲

> "Wan2.2 builds on the foundation of Wan2.1 with notable improvements in generation quality and model capability. This upgrade is driven by a series of key technical innovations, mainly including the Mixture-of-Experts (MoE) architecture, upgraded training data, and high-compression video generation."

💡 **3 个改进的相对重要性**：
- **MoE 架构** ⭐⭐⭐：架构层面的根本性改动，是 Wan 2.2 之所以叫 2.2 而不是 2.1.x 的核心理由
- **数据规模 + 美学标注** ⭐⭐：训练侧的工程改动，决定了能力上限
- **高压缩 5B 模型** ⭐⭐：让 Wan 真正进入消费级 GPU 时代（4090 也能跑），扩大了使用人群

⚠️ **不要混淆"模型架构"和"模型能力"**：MoE 是结构层面的（怎么组织参数），数据/美学是训练层面的（喂什么），高压缩是 efficiency 层面的（推理多快）。组会上如果被问"2.2 主要改了什么"，按这三个层面回答最稳。

---

### (1) Mixture-of-Experts (MoE) Architecture

![Wan 2.2 MoE Architecture (来自 Wan-Video/Wan2.2 GitHub)](https://github.com/Wan-Video/Wan2.2/raw/main/assets/moe_arch.png)

💡 **图怎么读** —— 这张图分左右两个 panel，展示同一个 MoE 在去噪过程不同阶段的专家激活：

**(a) Early Denoising Stage（高噪声阶段，紫色路径）**
- x_T（纯噪声起点）→ **High-Noise Expert**（实色高亮，正在工作）→ x_t →
- Low-Noise Expert（淡色，待命）→ x_0
- **此时模型由 high-noise expert 负责**，建立画面整体布局

**(b) Later Denoising Stage（低噪声阶段，绿色路径）**
- x_T → High-Noise Expert（淡色，已交班）→ x_t →
- **Low-Noise Expert**（实色高亮，正在工作）→ x_0
- **此时模型由 low-noise expert 负责**，精修视频细节

⚠️ **注意**：两个 panel 表示的是**同一个 MoE 在不同 timestep 上的不同行为**（用谁工作变了，整套 MoE 没变）。**专家切换由 SNR 阈值 t_moe 决定**（详见下文批注）。

---

#### 三个最容易困惑的点（每个都要搞清楚）

##### 困惑 1：两个专家的参数是不是不一样的？

**完全不一样，是两套独立的 14B 参数**。

```
Wan 2.2 MoE 拆解:
─────────────────────────────────────────────────
High-Noise Expert: 一个独立的 ~14B 参数 DiT
                   拥有自己的所有权重 (W^H_attn, W^H_FFN, ...)

Low-Noise Expert:  另一个独立的 ~14B 参数 DiT
                   拥有自己的所有权重 (W^L_attn, W^L_FFN, ...)
                   ↑ 和 W^H 完全不同的两套数字

总参数: ~27B (不是 28B 因为共享了部分 embedding/conditioning)
每步激活: 14B (只调用其中一个专家)
─────────────────────────────────────────────────
```

⚠️ **Wan 2.2 MoE ≠ LLM MoE**（如 Mixtral / DeepSeek-V3）：
- **LLM MoE**：8 个 expert 是 FFN 子模块，按 token 路由，每个 token 选 top-2
- **Wan 2.2 MoE**：2 个 expert 是"两个完整 14B DiT"，按 timestep 路由

类比：
- LLM MoE = 一个公司里 8 个会计随机分配处理客户
- **Wan 2.2 MoE = 一个项目分两阶段，前期请建筑师，后期请室内设计师，两人完全不同专业**

##### 困惑 2：High-noise / Low-noise 到底指什么？

指的是 **diffusion 中输入数据的噪声水平**：

```
Wan / Flow Matching 约定：
─────────────────────────────────────────
t = 0          t = 0.5           t = 1
└─ 起点         └─ 中段           └─ 终点
 纯噪声        半噪+半清晰        清晰视频
 x_0           x_t                x_1

噪声水平: HIGH ─────────────────> LOW
```

- **High-noise 阶段** = 早期去噪 = 输入还是一团噪声
- **Low-noise 阶段** = 晚期去噪 = 输入接近清晰视频

⚠️ **顺序提醒**：
- "**Early** Denoising Stage" = 早期 = **HIGH** 噪声 ← high-noise expert 上场
- "**Later** Denoising Stage" = 后期 = **LOW** 噪声 ← low-noise expert 上场

**先 high 后 low** 是因为 diffusion 推理顺序本来就是从噪声走到清晰（[详见 training-vs-inference.md](../../../_concepts/training-vs-inference.md)）。

##### 困惑 3：为什么 high-noise → 粗，low-noise → 精细？

**这是 diffusion 任务本质决定的，不是 Wan 强加的设定**。

```
高噪声阶段（看不清）：              低噪声阶段（看得清）：
┌──────────────────┐                ┌──────────────────┐
│ ░▓▒░▓▒█▒░▓▒█▒░  │                │   👤      🌳      │
│ ▒█░▓▒░█▓▒░█▒░▓  │                │                   │
│ ░▓▒█░▒▓░▒█▓░▒█  │                │                   │
└──────────────────┘                └──────────────────┘
能做的判断：                         能做的判断：
✓ "左边好像有人物"  ← 大局        ✓ 已经看到合理人物 + 树
✓ "右边好像有树"    ← 大局        → 任务: 精修
✗ 眉毛形状？        看不清          ✓ 加眉毛细节   ← 精细
✗ 树叶纹理？        看不清          ✓ 加树叶纹理   ← 精细
```

**一句话**：**去噪 = 信息逐步揭露**。早期信息少 → 只能做粗粒度决策（layout）；晚期信息多 → 可以做细粒度决策（details）。

##### 为什么不让单个模型干这两件事？

```
单模型既要会"画草图"又要会"画工笔"：
   ↓ 有限参数必须在两种能力之间取舍
   ↓ 两边都只是 OK，没有专精

两个专家各管一段：
   ↓ High-Noise Expert: 14B 全力学"从噪声建结构"
   ↓ Low-Noise Expert:  14B 全力学"从轮廓打磨细节"
   ↓ 两边都做到极致 → 整体质量提升
```

⚠️ **专精优势**：相同 14B 参数预算，**专注一种任务**比**两种任务均摊**学得更精。这就是 MoE 比 dense 模型在同等激活参数下质量更高的根本原因。

##### 3 个类比帮助记忆

| 类比 | High-Noise Expert | Low-Noise Expert |
|---|---|---|
| **画油画** | 起底色、铺大块构图 | 精修细节、亮点高光 |
| **写小说** | 列大纲、定章节结构 | 润色措辞、推敲对白 |
| **盖房子** | 建筑师：定结构 / 空间布局 | 室内设计师：选家具 / 调色彩 |

每一阶段对**专业能力的要求完全不同** —— 用不同的人做最对路。

---

> "Wan2.2 introduces Mixture-of-Experts (MoE) architecture into the video generation diffusion model. MoE has been widely validated in large language models as an efficient approach to increase total model parameters while keeping inference cost nearly unchanged."

💡 **类比理解**：
- 在 LLM 里，MoE（如 Mixtral、DeepSeek-V3）是"很多个小专家共享一套门控网络，每次前向只激活几个"
- 在 Wan 2.2 里，MoE 用得**完全不一样** —— 不是"按 token 路由"，而是"**按扩散时间步路由**"
- 核心想法：**扩散过程的不同阶段需要不同的能力**，所以让不同阶段用不同的专家，每个专家专门解决自己阶段的事

> "In Wan2.2, the A14B model series adopts a two-expert design tailored to the denoising process of diffusion models: a high-noise expert for the early stages, focusing on overall layout; and a low-noise expert for the later stages, refining video details."

💡 **为什么要分两个专家？**
- 扩散模型的 denoise 过程是从纯噪声 → 干净视频
- 早期（高噪声）：模型只能"看到"模糊轮廓，主要任务是确定**整体布局**（场景、运动趋势、人物位置）
- 晚期（低噪声）：模型能"看到"清晰细节，主要任务是**精修细节**（纹理、面部、小物体）
- 这两个任务的难度和所需能力不一样 —— 用同一套参数硬干，相当于让一个人既画草稿又画工笔，势必有取舍。**Wan 2.2 给草稿和工笔各配一个专精的人。**

> "Each expert model has about 14B parameters, resulting in a total of 27B parameters but only 14B active parameters per step, keeping inference computation and GPU memory nearly unchanged."

💡 **为什么是 27B 不是 28B？** 因为两个专家**共享一部分参数**（embedding、conditioning 等），只有 DiT 主体是各自一份，所以是 ~14B + ~13B = 27B 而不是 28B。

⚠️ **关键 quiz 点**：MoE 让总参数从 14B → 27B（容量翻倍），但**每步只激活 14B**（计算量不变）。这就是 MoE 的精髓 —— 用稀疏激活换容量提升。

> "The transition point between the two experts is determined by the signal-to-noise ratio (SNR), a metric that decreases monotonically as the denoising step *t* increases. At the beginning of the denoising process, *t* is large and the noise level is high, so the SNR is at its minimum, denoted as SNR_min. In this stage, the high-noise expert is activated. We define a threshold step t_moe corresponding to half of the SNR_min, and switch to the low-noise expert when t < t_moe."

💡 **专家切换规则用一句话讲**：
- 切换点 = SNR 降到 `SNR_min / 2` 的那个步数
- 步数大、SNR 小 → high-noise expert 上场
- 步数小、SNR 大 → low-noise expert 上场

```
denoise step t:    [大 → 小]
SNR:                [小 → 大]
                    ─────────●──────────
                              ↑
                          t_moe (SNR = SNR_min/2)
                              │
                  high-noise  │  low-noise
                  expert      │  expert
                  (布局)      │  (细节)
```

> "To validate the effectiveness of the MoE architecture, four settings are compared based on their validation loss curves... The Wan2.2 (MoE) (our final version) achieves the lowest validation loss, indicating that its generated video distribution is closest to ground-truth and exhibits superior convergence."

💡 **消融实验设计很妙**：
- 完整 Wan 2.1 (无 MoE) — baseline
- Wan2.1 backbone + Wan2.2 high-noise expert — 只换前半段
- Wan2.1 backbone + Wan2.2 low-noise expert — 只换后半段
- 完整 Wan 2.2 MoE — 两个专家都用 2.2

通过这个对比能拆出："是不是只换一个专家就够好？" 答案：不行，**两个专家协同才是最优的**。这论证了"早晚阶段用不同的专精模型"这个设计本身的合理性。

---

### (2) Cinematic-level Aesthetics（电影级美学）

> "Wan2.2 incorporates meticulously curated aesthetic data, complete with detailed labels for lighting, composition, contrast, color tone, and more. This allows for more precise and controllable cinematic style generation, facilitating the creation of videos with customizable aesthetic preferences."

💡 **这是数据侧的改进，不是架构改进**：Wan 2.2 在训练数据里多了一类"美学标注数据" —— 每段视频额外标注了打光（硬光/柔光/逆光）、构图（中心/对角/对称）、对比度、色调（冷/暖/复古）等维度。

💡 **效果**：你给 prompt 时可以直接说"温暖电影色调，逆光，中心构图"，模型能听懂并照做。Wan 2.1 没有这种细粒度美学控制，**这是 Wan 2.2 视觉效果"更电影感"的直接来源**。

---

### (3) 数据规模升级

> "Compared to Wan2.1, Wan2.2 is trained on a significantly larger data, with +65.6% more images and +83.2% more videos."

💡 这是工程数字，记住就行：
- 图像 +65.6%
- 视频 +83.2%

💡 **Why this matters**：视频比图像增长更猛（+83.2% vs +65.6%），符合"加大视频专用数据投入"的方向。直接收益是动作多样性、复杂运动场景的处理能力。

---

### (3.5) Prompt Extension 升级（隐藏的工程进步）

> **来源**：Wan 2.2 GitHub README 的 "Using Prompt Extension" 章节
>
> **背景**：Wan 2.1（[§4.5 Prompt Alignment](04-prompt-eval.md#-45-prompt-alignment--用户-prompt-怎么对齐训练分布)）已经引入用 LLM rewrite 用户 prompt 的机制。Wan 2.2 把这个机制升级了。

#### 不同任务用不同 rewriter

```
任务         默认 (Dashscope API)         本地替代 (HuggingFace)
─────────────────────────────────────────────────────────────
T2V          qwen-plus（纯文本 LLM）       Qwen2.5-14B/7B/3B-Instruct
I2V          qwen-vl-max（VL 多模态 LLM）   Qwen2.5-VL-7B/3B-Instruct
TI2V-5B      自动分流：                     同上
              - 纯文本 → qwen-plus
              - 有图   → qwen-vl-max
```

⚠️ **关键升级**：**I2V 任务用 VL 模型 rewrite**，不是纯文本 LLM。

#### 为什么 I2V 必须用 VL 模型？

```
纯文本 LLM 看不到图，rewrite 用户的"让它动起来"还是"让它动起来"
VL 模型 (qwen-vl-max) 能：
  1. 看图 → 识别图像内容（"白色波斯猫坐在窗台"）
  2. 结合用户意图 → 改写为详细 prompt
                  ("柔光摄影风格，白色波斯猫从窗台站起，缓缓走向花园...")
  3. 改写后的 prompt 既贴合图像，又有充分细节
```

🎯 **核心洞察**：**在 I2V 任务里，rewriter 自身就是个多模态模型**。这呼应了"prompt 适配层是部署工程的隐藏环节"的洞察 —— Wan 2.2 把这个适配层做得更智能。

#### 实战使用注意事项

⚠️ 推理时的命令行参数：
- `--use_prompt_extend`：启用 prompt extension
- `--prompt_extend_method 'dashscope'` 或 `'local_qwen'`
- `--prompt_extend_model`：指定具体模型路径

**默认是关闭的**！如果不开，模型直接吃用户原 prompt → **生成质量打折扣**。这是个**部署上的常见坑**。

---

### (4) 高效高清 Hybrid TI2V（5B 模型 + 新 VAE）

> "To enable more efficient deployment, Wan2.2 also explores a high-compression design. In addition to the 27B MoE models, a 5B dense model, i.e., TI2V-5B, is released. It is supported by a high-compression Wan2.2-VAE, which achieves a T×H×W compression ratio of 4×16×16, increasing the overall compression rate to 64 while maintaining high-quality video reconstruction."

💡 **TI2V-5B 是 Wan 2.2 的"消费级旗舰"**：
- 名字解读：**T**ext + **I**mage **to** **V**ideo —— 一个模型同时支持 T2V 和 I2V，不用换 checkpoint
- 5B Dense（不是 MoE！）—— 设计哲学和 A14B 系列完全不同
- 配套的 Wan2.2-VAE 把压缩比从 Wan 2.1 的 4×8×8（=256）拉到 4×16×16（=1024，整体 ×64）

⚠️ **常见误解**：很多人以为 TI2V-5B 是"小一点的 MoE 模型" —— **不是！** 它是 Dense 模型，纯粹靠 VAE 压缩比换效率。

> "With an additional patchification layer, the total compression ratio of TI2V-5B reaches 4×32×32. Without specific optimization, TI2V-5B can generate a 5-second 720P video in under 9 minutes on a single consumer-grade GPU, ranking among the fastest 720P@24fps video generation models. This model also natively supports both text-to-video and image-to-video tasks within a single unified framework, covering both academic research and practical applications."

💡 **Patchify 后总压缩**：4×32×32（含 patch 后）—— 意思是模型实际处理的 latent token 数量比 Wan 2.1 少了 ×16 倍（空间维度）。这就是它为什么能在 4090 上跑动的根本原因。

💡 **5 秒 720P < 9 分钟 on 单卡消费级 GPU** —— 这个数字翔哥可能会问，记一下。

---

## 💡 核心信息速查

### Wan 2.1 vs Wan 2.2 对照表（answer to Q1）

| 维度 | Wan 2.1 | Wan 2.2 |
|---|---|---|
| **核心架构** | Dense DiT | **MoE 双专家**（high-noise + low-noise）按 SNR 切换 |
| **参数规模** | 1.3B / 14B | A14B (27B 总, 14B 激活) + TI2V-5B Dense |
| **VAE** | Wan-VAE 4×8×8 | **Wan2.2-VAE 4×16×16**（压缩率 ×4） |
| **训练数据** | baseline | +65.6% 图像 / +83.2% 视频 |
| **美学控制** | 无细粒度 | ✅ 光线/构图/对比/色调标注 |
| **消费级支持** | 1.3B 需 8.19GB VRAM | ✅ TI2V-5B 在 4090 上跑 720P@24fps |
| **新模型谱系** | T2V / I2V | + **S2V**（音频驱动）+ **Animate**（角色动画） |

### Wan 2.2 模型谱系全览

| 模型 | 类型 | 参数 | 任务 | 分辨率 | VRAM |
|---|---|---|---|---|---|
| **T2V-A14B** | MoE | 27B/14B 激活 | Text → Video | 480P + 720P | ≥80GB |
| **I2V-A14B** | MoE | 27B/14B 激活 | Image → Video | 480P + 720P | ≥80GB |
| **TI2V-5B** ⭐ | **Dense** | **5B**（全激活） | Text/Image → Video（统一） | 720P@24fps | ≥24GB（4090） |
| **S2V-14B** | Dense | 14B | Speech + Image → Video | 480P + 720P | ≥80GB |
| **Animate-14B** | Dense | 14B | 角色动画 / 替换 | 1280×720 | ≥80GB |

#### ⚠️ 命名陷阱：A14B 里的 "14B" 是**激活参数**，不是总参数

```
"A14B" 拆解：
   A    = Active（激活）
   14B  = 每步激活 14B 参数

但总参数 = 27B (两个 14B expert，部分共享 embedding/conditioning)

为什么强调 14B 而不是 27B：
✓ VRAM 估算按 14B 算（每步只有 14B 在 GPU）
✓ 算量按 14B 算（FLOPS 按激活参数）
✓ 官方命名习惯（参考 Mixtral 8×7B）

隐藏意义：用 I2V-A14B 时，显存/速度感觉和 14B Dense 一样
        但质量比 14B Dense 高（因为容量是 27B）
        这就是 MoE 的精髓 —— 同等推理成本，更高质量
```

#### ⚠️ 容易混淆：TI2V-5B 是**真 5B**，不是 MoE

```
TI2V-5B 全名拆解：
   "TI2V" = Text+Image to Video（统一模型，同一 ckpt 处理两类任务）
   "5B"   = 真的 5B 参数 (Dense，没有 MoE)

它和 I2V-A14B 是完全不同的两个模型：
                I2V-A14B (MoE)        vs    TI2V-5B (Dense)
─────────────────────────────────────────────────────────
总参数        27B                          5B
激活参数      14B / 步                     5B / 步
架构          MoE 双专家                   Dense 单一模型
VRAM          ~80GB                        ~24GB（4090 可跑）
质量天花板    更高（27B 容量）              中等（5B 上限）
典型场景      工业部署                      个人/学术/快速迭代
```

#### 判断你之前用的是哪个？

```bash
# I2V-A14B (27B/14B 激活):
python generate.py --task i2v-A14B --ckpt_dir ./Wan2.2-I2V-A14B ...

# TI2V-5B (5B Dense):
python generate.py --task ti2v-5B --ckpt_dir ./Wan2.2-TI2V-5B ...
```

或看 checkpoint 文件夹名：
- `Wan2.2-I2V-A14B/` → 27B/14B 激活的 MoE
- `Wan2.2-TI2V-5B/` → 5B Dense

### TI2V-5B vs I2V-A14B 关键区别（answer to Q2）

| 对比维度 | TI2V-5B | I2V-A14B |
|---|---|---|
| **架构类型** | Dense | **MoE**（双专家） |
| **总参数 / 激活参数** | 5B / 5B（全激活） | 27B / **14B 激活** |
| **VAE** | **Wan2.2-VAE 4×16×16**（高压缩） | Wan-VAE 4×8×8（标准） |
| **支持任务** | **T2V + I2V 统一**（一个 ckpt 都能干） | I2V only（需要图像输入） |
| **支持分辨率** | 720P@24fps | 480P + 720P |
| **VRAM 门槛** | **24GB**（4090 可跑） | **80GB**（A100/H100 级别） |
| **生成 5s 720P 视频耗时** | < 9 分钟（单卡 4090） | 更慢（需多卡 FSDP+Ulysses 才划算） |
| **质量天花板** | 中等（5B 上限） | **更高**（27B 容量） |
| **典型场景** | 个人 / 学术研究 / 快速迭代 | 工业级 / 商业部署 / 最高质量 |

⚠️ **关键 takeaway**：**TI2V-5B 用"高压缩 VAE + 小模型 + 统一框架"换可用性；I2V-A14B 用"MoE 架构 + 大模型"换质量上限**。两者不是替代关系，是不同 segment。

---

## 💡 核心洞察

1. **MoE 在 diffusion 里的用法和 LLM 不一样**：LLM 的 MoE 按 token 路由（同一时刻不同 token 走不同专家），diffusion 的 MoE 按时间步路由（同一 token 在不同 step 走不同专家）。这是个**很巧妙的迁移**。

2. **Wan 2.2 走"双产品线"策略**：A14B 追求质量天花板，5B 追求可用性下限。这意味着 Alibaba 既要做工业级供给（云端推理），又要让开源社区/学术界有得玩。这是开源策略，不是单纯的技术选择。

3. **VAE 是 Wan 2.2 隐藏的英雄**：从 4×8×8 升到 4×16×16，让 5B 模型能在 4090 上跑 720P。如果只换 VAE 不动 backbone，已经是巨大的工程胜利。

4. **没有独立 arXiv 论文是个有意思的信号**：说明 Wan 2.2 的 Alibaba 团队选择了"工程化 release"而非"学术化 publish"——直接发模型权重 + GitHub 文档，这是大厂技术报告的趋势。

---

## 🤔 我的 Follow-up 问题

1. SNR_min/2 这个切换点是怎么调的？是消融实验扫出来的，还是有理论依据？
2. high-noise expert 和 low-noise expert 是从同一个 backbone 微调而来，还是各自从头训？
3. TI2V-5B 的 Wan2.2-VAE 压缩比那么高，长视频会不会因为 VAE 重建误差累积失真？
4. 在 RoboTwin 仿真上做 I2V 推理时，我用的是 I2V-A14B 还是 TI2V-5B？为什么我们组选这个？（**这个明天得搞清楚**）

---

## 🔥 拷打记录

_(待填充 — Claude 会在这一节问我 3~4 个问题，我用自己的话回答，然后把 Q&A 整理回来)_

---

[← 返回论文 README](../README.md)
