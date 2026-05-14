[← 返回 ABot-PhysWorld 主页](../README.md)

# §5 Experiments

---

## §5.1 Implementation Details

> "We conduct all experiments on a cluster of 128 Nvidia H20 GPUs. The training pipeline consists of three stages: TI2V foundational training, DPO, and A2V training."

**三阶段训练**：

| 阶段 | 配置 |
|---|---|
| **TI2V 基础训练** | Wan2.1-I2V-14B-480P，输入 crop 到 480×832，81 帧；6000 步，global batch 128，lr 1e-5 |
| **DPO** | LoRA rank-64（scaling 64）注入 self-attn + FFN（q/k/v/o, ffn.0, ffn.2）；AdamW lr 1e-6，**β=5000**；500 步/epoch × 100 epoch |
| **A2V** | VACE 框架；复制 DiT 层 **(0,5,10,15,20,25,30,35)** 做可训练 context 分支，backbone 冻结；batch 16，lr 5e-5，20000 步 |

💡 **批注**：`β=5000` 非常大（一般 DPO β 是 0.1-1 量级）—— paper 说"为了在 diffusion model 里强化偏好信号"。这是个值得记的工程细节。

---

## §5.2 Evaluation Setup

### Text-Conditioned Generation
- **PAI-Bench** 的 **PBench** 数据集，robot 子集 174 个复杂 manipulation 视频（来自 BridgeData V2 / AgiBot / Open X-Embodiment）
- MLLM-as-Judge（Qwen2.5-VL-72B-Instruct）做二值 VQA
- **Domain Score**：886 个问题，三维度 —— 空间 36.3% / 时间 28.6% / 物理 34.1%
- 还有多维质量指标：subject/background consistency, overall consistency, aesthetic, imaging quality, motion smoothness, i2v consistency
- 零样本评估用 EZSbench

### Action-Conditioned Generation
- 从 A2V 数据集均匀采 200 个实例（每个含初始帧 + 结构化 action 序列）
- **视觉对齐**：PSNR（像素精度）+ SSIM（局部纹理保真）
- **轨迹精度**：nDTW —— 用 fine-tuned YOLO detector 定位每帧 gripper，提取轨迹和 GT 比 nDTW

### Baselines
- 文本条件：Cosmos-Predict 2.5-2B, GigaWorld-0, UnifoLM-WMA-0, WoW-wan 14B, Veo 3.1, Sora v2 Pro, Wan 2.5
- **动作条件：EnerVerse-AC, Gen-Sim** ⭐

🔥 **Uni-WAM 关联**：动作条件的 baseline 直接是 **EnerVerse-AC** —— 我们已经读过的 paper。所以 ABot-PhysWorld 和 EnerVerse-AC 是**正面对比关系**。

---

## §5.3 Evaluation Results

### PBench（Table 1）

| 模型 | Avg | Domain Score |
|---|---|---|
| Veo 3.1 | 0.8045 | 0.8350 |
| Sora v2 Pro | 0.7652 | 0.7626 |
| Our Model | 0.8232 | 0.8785 |
| **Our Model + DPO** | **0.8491** | **0.9306** |

> "our DPO-augmented model achieves the highest average score (0.8491) and sets a new state-of-the-art Domain Score (0.9306)"

🔥 **关键发现**：
- **DPO 把 Domain Score 从 0.8785 → 0.9306**（物理保真度大涨）
- **Quality Score 几乎不变**（0.7678 → 0.7676）
- → 证明"物理对齐**不牺牲**视觉质量"
- Veo 3.1 / Sora v2 Pro：Quality Score 高但 Domain Score 低（**偏感知不偏物理**）

### EZSbench（Table 2）

| 模型 | Avg | Quality | Domain |
|---|---|---|---|
| **Our Model** | **0.8030** | **0.7694** | **0.8366** |
| WoW-wan 14B | 0.7780 | 0.7609 | 0.7951 |
| Cosmos-Predict 2.5 | 0.7394 | 0.7089 | 0.7698 |

> "our model achieves the highest overall average score (0.8030)... This confirms that the physical fidelity improvements generalize beyond the training distribution."

→ 物理保真度的提升能**泛化到分布外**。

### Action-Conditioned Generation（Table 3）⭐

| 模型 | PSNR | SSIM | Traj. Consis. |
|---|---|---|---|
| **EnerVerse-AC** | 20.42 | 0.7542 | 0.8157 |
| Gen-Sim | 18.05 | 0.7413 | 0.6195 |
| **Ours** | **21.09** | **0.8126** | **0.8522** |

🔥 **关键发现**：**ABot-PhysWorld 在动作条件生成上全面超过 EnerVerse-AC** —— PSNR / SSIM / 轨迹一致性三项都领先。

### 定性分析（Figure 5）

![](../images/figure-05.png)

各 baseline 的物理违例被逐一标出：
- Sora v2 / Veo 3.1：dense contact 时 gripper/object 形变
- GigaWorld-0 / Cosmos：抓取穿模
- WoW：无接触抓取 + 几何形变
- UnifoLM / Wan 2.5：认错目标
- **ABot-PhysWorld**：正确识别目标 + 时空连贯 + 无形变穿模

---

## 💡 §5 整段 takeaway

| 实验 | 结论 |
|---|---|
| PBench | DPO 版 SOTA（Avg 0.8491），Domain Score 0.9306 超 Veo 3.1 / Sora v2 Pro |
| EZSbench | OOD 上仍 SOTA（0.8030），物理保真度能泛化 |
| Action-Conditioned | **全面超 EnerVerse-AC**（PSNR 21.09 / SSIM 0.8126 / Traj 0.8522）|

🔥 **Uni-WAM 视角**：Table 3 是和 EnerVerse-AC 的**直接对决** —— ABot-PhysWorld 赢了。但注意：**这个对决测的是"视觉对齐 + 轨迹一致性"，不是"off-expert action 下的 dynamics 保真度"**。Uni-WAM 关心的 off-expert action 评估，ABot-PhysWorld 同样没碰。

---

[← §4 EZSbench](04-benchmark.md) | [§6 Conclusion →](06-conclusion.md)
