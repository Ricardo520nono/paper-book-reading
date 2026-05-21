[← 返回主页](../README.md)

# §4 Experiments

## §4.1 Visuotactile World Model Evaluation（触觉模态评估）

基于 UniVTAC 仿真器，两个接触丰富任务：**Insert HDMI** 和 **Lift Bottle**。把现成 WM（Vidar / Genie Envisioner / Wan2.2）用 §3.2 管线升级成视触觉 WM，并和触觉 VLA baseline **ACT** 对比。触觉预测质量用 **PSNR / SSIM**，下游用任务成功率。

### Table 1：视触觉 WM 任务成功率（UniVTAC）

| Model | PSNR↑ | SSIM↑ | Insert HDMI | Lift Bottle | Avg. |
|---|---|---|---|---|---|
| ACT (Baseline) | – | – | 20 | **80** | 50 |
| Vidar | 13.97 | 0.278 | 70 | 0 | 35 |
| Genie Envisioner | 13.36 | 0.456 | 0 | 0 | 0 |
| **Wan2.2** | **21.26** | **0.746** | **100** | 0 | 50 |

> "Wan2.2 achieves higher tactile prediction quality than specialized embodied models, as general-purpose world models retain richer cross-modal knowledge priors that align more effectively with tactile modalities. Consequently, Wan2.2 attains 100% success on Insert HDMI."

🔥 **翻译（反直觉发现 1）**：通用大模型 **Wan2.2 触觉预测质量反超专用具身模型** —— 因为通用 WM 保留了更丰富的跨模态知识先验，更易对齐触觉模态。结果 Insert HDMI 直接 100%，证明「触觉预测能力强 → 细粒度接触操作表现好」。

> "Conversely, Lift Bottle yields a counter-intuitive outcome where the ACT baseline achieves 80% while all world models fall to 0%, because this long-horizon task demands sustained force control... yet the long-horizon planning capability of current world models remains limited."

⚠️ **翻译（反直觉发现 2）**：**Lift Bottle 上 ACT 80%，所有 WM 全 0%**。因为这是 long-horizon 任务，需要持续力控（触觉只起高频反馈作用），而当前 WM 的**长程规划能力仍很弱**。

**§4.1 结论**：标准化触觉注入管线让 WM 获得了触觉理解能力，在接触密集任务上接近现有触觉 VLA。但长程力控仍是 WM 短板。

---

## §4.2 World Model Evaluation as RL Environments（当 RL 环境评估）

实现细节见 Appendix B.1：基于 **RLinf** 框架，π₀.₅ 当基础 policy，RoboTwin 2.0 的 **adjust bottle / click bell** 两任务，采 3000 条轨迹训 WM + reward model，1000 条专家轨迹 SFT 出初始 policy，**GRPO** 优化。

### Table 2：不同 reward model 训出的 policy 成功率（RoboTwin 2.0）

| Method | Proxy Click | Proxy Adjust | VLM Click | VLM Adjust | Sim Click | Sim Adjust |
|---|---|---|---|---|---|---|
| SFT | 43.75 | 55.08 | 43.75 | 55.08 | 43.75 | 55.08 |
| **Simulator-based RL（天花板）** | **87.30** | **78.90** | 87.45 | 78.90 | 87.45 | 78.90 |
| OpenSora | 56.25 | 60.16 | 55.27 | 57.03 | 53.13 | 58.00 |
| IRASim | 53.13 | 61.33 | 53.52 | 58.98 | 50.78 | 59.38 |
| iVideoGPT | 52.53 | 56.25 | 48.44 | 58.59 | 52.15 | 60.93 |
| Cosmos-Predict-2.5(action) | 67.38 | 63.48 | 54.10 | 58.40 | 63.09 | 61.13 |
| RoboScape | 68.75 | 60.74 | 55.46 | 59.38 | 63.48 | 59.18 |
| **Ctrl-World** | 69.53 | **70.70** | 66.80 | 65.04 | 69.92 | 66.02 |
| **WoVR** | **75.00** | 67.19 | 69.38 | 64.45 | 72.07 | 61.35 |

🔥 **关键发现**：

> "although policies trained with existing world models cannot yet outperform those trained on the simulator, the top-performing strategy has achieved comparable performance."

- 用 WM 当 RL 环境训的 policy **还干不过用真 simulator 训的**（87.30/78.90 天花板），但最强的已接近。
- **WoVR** 短程任务（Click Bell）最强；**Ctrl-World** 长程任务（Adjust Bottle）SOTA。
- OpenSora / IRASim / iVideoGPT 受限于视频生成质量，提升微弱；Cosmos-Predict-2.5(action) 和 RoboScape 表现不错。

**三种 reward model 对比**（proxy-based / VLM-based / similarity-based）：

> "the proxy-based reward achieves the robustest performance. This is owing to the fact that the VLM-based reward is not finetuned on this task and the similarity-based reward highly depends on the observation prediction performance."

**翻译**：**proxy-based reward 最稳**（ResNet backbone，吃视觉观测+文本指令端到端出即时奖励）。VLM-based（Qwen-3.5 当评分器）没在任务上 finetune；similarity-based（预测观测 vs 目标态特征相似度）太依赖观测预测质量。

> Appendix B.2 + Figure 5：随 policy-环境交互步数增加，**几乎所有 WM 都能在不同程度上引导 policy 更新**（成功率曲线上升）。

---

## §4.3 Cross-platform Evaluation and Analysis（跨平台评估）

### 跨平台视频质量（沿用 1.0 的 6 维 16 指标，Table 4-9）

跨 RoboTwin / LIBERO / 真机 AgileX 三平台，14 个模型（含商用 Veo3.1 / Wan2.6）。

> "Commercial models such as Veo3.1 and Wan2.6 lead in visual quality, yet their advantage narrows on physics adherence, where embodied models such as CtrlWorld and IRASim achieve comparable or better trajectory accuracy. This indicates that visual fidelity alone is an insufficient proxy for dynamics modeling. Among embodied models, WoW and CtrlWorld consistently rank highly on physics adherence and content consistency."

🔥 **翻译**：商用模型 Veo3.1 / Wan2.6 视觉质量领先，但**优势在 physics adherence 上收窄** —— Ctrl-World / IRASim 的 trajectory accuracy 反而相当或更好。**说明视觉保真不是动力学建模的充分代理**（延续 1.0 的「感知-功能 gap」）。具身模型里 **WoW 和 CtrlWorld** 在 physics adherence + content consistency 上稳居前列。

#### Table 4（RoboTwin）— Visual / Motion / Content（节选关键值）

| Model | Image Q | JEPA Sim | Motion Smooth | Subject Cons | Photom Cons |
|---|---|---|---|---|---|
| Cosmos-Predict2.5(text) | **0.6668** | 0.3126 | 0.7882 | 0.7488 | 0.1383 |
| Cosmos-Predict2.5(action) | 0.4489 | **0.9296** | 0.7100 | 0.8197 | 0.3528 |
| CtrlWorld | 0.3522 | 0.9185 | 0.7377 | **0.8411** | 0.1729 |
| Wan2.2 | 0.3884 | 0.7575 | 0.7019 | 0.8388 | **0.4776** |
| Wan2.6 | **0.6824** | 0.7229 | **0.8539** | 0.7517 | 0.1904 |
| Veo3.1 | 0.6605 | 0.5694 | 0.6989 | 0.7878 | 0.3247 |

#### Table 5（RoboTwin）— Physics / 3D / Controllability（节选关键值）

| Model | Interaction Q | **Traj Acc** | Depth Acc | Instr Follow | Sem Align | **Action Resp Sens** |
|---|---|---|---|---|---|---|
| CtrlWorld | 0.6212 | **0.4766** | **0.9300** | 0.7960 | 0.7272 | 0.0210 |
| IRASim | 0.5656 | 0.3639 | 0.9312 | 0.7788 | 0.6604 | 0.0526 |
| CogVideoX | 0.5940 | 0.3526 | 0.9097 | 0.7828 | 0.7268 | 0.0076 |
| Veo3.1 | **0.7872** | 0.1231 | 0.7421 | 0.8276 | **0.9328** | 0.0852 |
| Wan2.6 | 0.7280 | 0.1182 | 0.7144 | 0.8032 | 0.8536 | **0.0992** |

> ⚠️ 注意最后一列 **`Action Response Sensitivity`** —— 这是 1.0 里 `Action Following` 改的名。数值都很小（0.006~0.10），**Veo3.1/Wan2.6 这种商用强生成模型反而最高**。这个量到底测什么、对我们 proposal 意味什么，见 [summary.md](summary.md) 的深挖标记 + 待办（去 GitHub 查实现）。

#### Table 6-9：LIBERO + 真机
同样 16 指标。真机上（Table 9）**Ctrl-World 的 Trajectory Accuracy 0.6865 一骑绝尘**（其它多在 0.0x~0.36），Depth Accuracy 0.9888 也最高。商用 Veo3.1 真机 Interaction Quality 0.86 / Semantic Alignment 0.93 领先，但 Traj Acc 仅 0.0445。

### 跨平台任务成功率（Table 3）

> "As a data engine, no world model matches real demonstration data, and the performance gap widens from simulation to real-world tasks. Real-world evaluation remains the most challenging setting, with only a few models achieving non-zero success rates, all far below practical deployment requirements."

🔥🔥 **翻译（核心结论）**：当 data engine 用，**没有一个 WM 能比真实示教数据强**，且 gap 从仿真到真机越拉越大。真机评估最难，只有少数模型非零成功率，全部远低于实用门槛。

### 跨平台 ranking 相关性（Figure 6-9）

> "For perceptual quality, visual quality, motion quality, physics adherence, and 3D accuracy show strong correlations across platforms... By contrast, content consistency and controllability exhibit weaker correlations... task success correlates positively between the two simulators but drops greatly when compared with real-world performance."

**翻译**：
- **感知质量**：visual / motion / physics / 3D 跨平台**强相关**（低层保真 + 几何推理迁移得还行）。
- **content consistency + controllability** 跨平台**弱相关**（语义/指令对齐对 domain 更敏感）。
- **task success**：两个仿真器间正相关（RoboTwin↔LIBERO Spearman 0.771），但**和真机比就崩**（RoboTwin↔Real 仅 0.348, p=0.499）。

> "These results reveal a clear sim-to-real gap, showing that simulation performance—whether perceptual or functional—is not a reliable proxy for real-world deployment and that physical evaluation remains indispensable."

**翻译**：清晰的 sim-to-real gap —— **仿真表现（无论感知还是功能）都不是真机部署的可靠代理，物理评估不可省。**

---

## 💡 §4 整段 takeaway

| 实验 | 结论 |
|---|---|
| Table 1 触觉 | 通用 Wan2.2 触觉预测反超专用模型；但长程力控 WM 全崩（Lift Bottle 0%）|
| Table 2 RL 环境 | WM 当 RL 环境训 policy 还达不到真 simulator，但 WoVR/Ctrl-World 接近；proxy reward 最稳 |
| Table 3 跨平台成功率 | data engine 没一个能超真实示教数据；sim→real gap 越拉越大 |
| Table 4-9 16 指标 | 视觉保真 ≠ 动力学；WoW/CtrlWorld 物理维度稳；⚠️`Action Following`→`Action Response Sensitivity` 改名 |
| Fig 6-9 相关性 | 感知质量跨平台强相关，task success 到真机就崩（Spearman 0.348）|

🔥 **Uni-WAM 视角**：
- 所有评估的 action 仍来自 **RoboTwin/LIBERO/真机示教的 expert/GT trajectory**，**没有 off-expert action 测试**（§4.2 RL 环境里 policy 探索会产生非最优 action，但没单独量化 WM 在这些 action 下的保真度）。
- baseline 名单（12 模型 + 2 商用）和评估设置可直接照搬。
- `Action Response Sensitivity` 是和我们 proposal 最可能撞概念的指标，**优先级最高**去查它的精确定义。

---

[← §3 Method](03-method.md) | [§5 Conclusion →](05-conclusion.md)
