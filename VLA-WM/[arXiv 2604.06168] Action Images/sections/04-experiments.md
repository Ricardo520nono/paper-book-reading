[← 返回 Action Images 主页](../README.md)

# §4 Experiments

---

## §4.1 Text-Controlled Action & Video Joint Generation（primary 评估）

> "Given a language instruction and the initial multi-view observations, the model jointly generates future robot videos and corresponding multi-view action videos, from which executable controls are obtained by decoding the predicted action images. ... one-trial open-loop evaluation."

**评估设定**：给语言指令 + 初始多视角观测 → 模型联合生成未来 robot video + action video → 解码出可执行控制。**一次开环预测，不在线 replanning** —— 直接反映学到的 pixel-grounded action 表示的质量和泛化能力。

### Zero-shot policy 结果（Table 2）

baseline：MV-Policy（Diffusion Policy 多视角扩展）/ π0.5 / MolmoAct（VLA 系）/ TesserAct / Cosmos-Policy（WM 系）。

| Methods | RLBench 平均 | Real 平均 | 总评 |
|---|---|---|---|
| MV-Policy | ~0 | 0 | 几乎全 0 |
| π0.5 | 低 | 0 | RLBench 个别项有分 |
| MolmoAct | 低 | 低 | 个别项 |
| TesserAct | 0 | 0 | 全 0 |
| Cosmos-Policy | 低 | 0 | 个别项 |
| **Ours** | **30/60/50/15** | **40/20/15/45/10** | **全面领先** |

> "our method delivers the best overall zero-shot performance across simulation and real-world tasks. The improvement is most evident under strong distribution shift."

🔥 **关键发现**：**分布漂移越强，Action Images 优势越明显** —— 支持"pixel-grounded action 表示 → 更可泛化 zero-shot policy"的核心论点。

⚠️ **zero-shot 设定的细节**：
- RLBench：评估的 task 从训练 split 完全移除，但机械臂和环境是见过的
- 真机：物体、环境、机械臂（xArm）**全未见**
- 所有设定下，语言指令的形式和训练时相似

### RLBench in-domain 结果（Table 3）

加一个**可选的 learned action head**（轻量 MLP，从 video latent + camera params + decoded action/obs 回归连续 7-DoF action）：
- 不加 head：Avg 20.6（和 TesserAct / Cosmos-Policy 持平）
- **加 action head：Avg 36.7**（大幅提升，尤其精度敏感任务）

💡 **批注**：action head 不是主张 zero-shot policy 必需的，只是测"学到的表示能不能支持更强解码"。

### Joint Generation Quality（Table 4）

对比 Cosmos-Predict / Cosmos-Policy / TesserAct：

| Models | PSNR↑ | SSIM↑ | FVD↓ | LPIPS↓ | 2DErr↓ | 3DErr↓ |
|---|---|---|---|---|---|---|
| Cosmos-Policy | 18.29 | 53.41 | 192.58 | 0.418 | 2.11 | 19.4 |
| TesserAct-RGB | 20.31 | 60.19 | 147.83 | 0.372 | 1.55 | 14.2 |
| **Ours** | **23.48** | **78.62** | **143.74** | **0.209** | 1.61 | **12.2** |

→ 所有 video 指标领先，同时保持 action 精度。

---

## §4.2 Additional Unified-Model Capabilities

### Action-conditioned video generation（Table 5）
对比 Tora（2D 轨迹条件 video gen baseline）：

| Models | PSNR↑ | SSIM↑ | LVD↓ | LPIPS↓ |
|---|---|---|---|---|
| Tora | 19.76 | 52.43 | 187.41 | 39.62 |
| **Ours** | **31.35** | **67.16** | **115.02** | **21.78** |

→ 全指标大幅领先 —— 统一 video-space 表示能更有效地用 action 输入做未来 video 预测。

### Video-to-action labeling（Table 6）
对比 TAPIR / CoTracker3（point-tracking baseline）：

| Models | Traj Err↓ | Jaccard@4↑ | Avg Jaccard↑ |
|---|---|---|---|
| CoTracker3 | 12.91 | 46.15 | 31.20 |
| **Ours** | **5.785** | **64.92** | **46.71** |

→ 大幅领先 —— pixel-grounded action 表示不仅能控制和生成，**也能从 video 反推 action**（≈ IDM 能力）。

---

## §4.3 Qualitative Results

> "We first evaluate zero-shot rollouts on an xArm platform, where the objects and environment are unseen."

- **xArm 平台 zero-shot**（Figure 5）：未见物体/环境。模型生成未来观测 + 多视角 action image → 解码成 3D 轨迹 → **真机 replay 验证可执行性**
- **FR3M 房间数据集**（Figure 6）：未见物体/任务/环境。和 LTX-2-Fast 对比 —— Action Images 生成的 video 目标定位更准
- 🔥 **关键**：尽管训练时 BridgeV2 **没有 action 监督**，模型仍能生成连贯的 action image —— 证明**学到的 action 生成能力能跨数据集/域迁移**

---

## 💡 §4 整段 takeaway

| 实验 | 结论 |
|---|---|
| Zero-shot policy（Table 2）| 全面领先 robot policy baseline，分布漂移越强优势越明显 |
| In-domain（Table 3）| 加可选 action head 后 Avg 36.7 大幅领先 |
| Joint Gen Quality（Table 4）| video 指标全领先 + action 精度保持 |
| Action-cond video gen（Table 5）| PSNR 31.35 vs Tora 19.76 |
| Video-to-action labeling（Table 6）| Traj Err 5.785 vs CoTracker3 12.91 |

🔥 **Uni-WAM 视角**：
- Action Images 在**一个统一模型**上打过 5 类专门 baseline —— 证明"统一 video-space 表示"是 win-win
- 但注意：评估的 action 全来自 **policy 自身产出或 ground-truth trajectory**，**没有主动喂 off-expert action**
- §4.2 的 action-conditioned video gen 评估 ≈ Ctrl-World / EnerVerse-AC 的同款评估 —— 测"给 GT action 生成的 video 准不准"，不测"给 off-expert action 还能不能保持物理合理"

---

[← §3 Method](03-method.md) | [§5 Conclusion →](05-conclusion.md)
