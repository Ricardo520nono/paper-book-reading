[← 返回 WorldArena 主页](../README.md)

# 串讲 · WorldArena 16 个视觉质量指标深度拆解

> **arXiv 2602.08971 (2026/02)** · 清华 THU + SJTU + HKU + Princeton + CAS + USTC + PKU + NUS · CVPR 2026 Challenge
> [Paper](https://arxiv.org/abs/2602.08971) · [GitHub](https://github.com/tsinghua-fib-lab/WorldArena) · [Leaderboard (HuggingFace)](https://huggingface.co/spaces/WorldArena/WorldArena) · [Project Page](https://world-arena.ai)
>
> **本笔记重点**：Ricardo 的当前任务是"读 WorldArena 16 个视觉质量指标，筛选可复用的"。本 summary **逐指标深度拆解 + 复用建议**。

---

## 1. WorldArena 是什么

> 翔哥 proposal 已收录的 benchmark（"World Model Benchmark" 类，2026/02）

WorldArena 是清华 FIB-Lab 牵头的**统一具身 WM benchmark**，三大评估维度：
1. **视频感知质量**（16 个指标 × 6 子维度）⭐ 本笔记重点
2. **具身任务功能** （Data Engine / Policy Evaluator / Action Planner）
3. **人类评估**

外加 **EWMScore** —— 把 16 个 normalized 指标取平均 ([0,100]) 作为综合分。

---

## 2. 16 个指标 × 6 子维度 全景

| 子维度 | 指标（共 16）| 用到的核心模型 |
|---|---|---|
| **① Visual Quality**（视觉质量）| Image Quality / Aesthetic Quality / JEPA Similarity | MUSIQ / CLIP-L+aesthetic head / V-JEPA-2 |
| **② Motion Quality**（运动质量）| Dynamic Degree / Flow Score / Motion Smoothness | RAFT / RAFT / VFIMamba |
| **③ Content Consistency**（内容一致性）| Subject Consistency / Background Consistency / Photometric Consistency | DINO+RAFT / CLIP-B+RAFT / SEA-RAFT |
| **④ Physics Adherence**（物理保真）| Interaction Quality / Trajectory Accuracy | Qwen3-VL（VLM）/ SAM3+NDTW |
| **⑤ 3D Accuracy**（3D 精度）| Depth Accuracy / Perspectivity | Depth-Anything / Qwen3-VL（VLM）|
| **⑥ Controllability**（可控性）| Instruction Following / Semantic Alignment / Action Following | Qwen3-VL / Qwen2.5-VL+CLIP+BLEU / CLIP |

**合计：3 + 3 + 3 + 2 + 2 + 3 = 16 ✓**

---

## 3. 逐指标深度拆解（按子维度）

### ① Visual Quality（视觉质量）

#### 1. Image Quality（图像质量）
- **模型**：[MUSIQ](https://github.com/google-research/google-research/tree/master/musiq)（Multi-scale Image Quality Transformer，CLIP-friendly variant from pyiqa）
- **怎么算**：对视频每一帧跑 MUSIQ 打分 → 平均
- **测什么**：单帧的**技术质量** —— 模糊、噪声、伪影、压缩 artifacts
- **代码**：`imaging_quality.py`
- **范围**：0-100，越高越好

#### 2. Aesthetic Quality（美学质量）
- **模型**：CLIP-L/14 视觉编码器 + LAION 美学打分头
- **怎么算**：每帧抽 CLIP feature → 过 linear 美学头 → 平均
- **测什么**：图像的**主观美学评分**（构图、光线、颜色、风格）
- **代码**：`aesthetic_quality.py`

#### 3. JEPA Similarity（JEPA 语义相似度）
- **模型**：V-JEPA-2 (Meta)
- **怎么算**：用 JEPA 提取生成视频和 GT 视频的 latent → 算 distance
- **测什么**：生成 video 和 GT video 的**语义级相似度**
- **代码**：`JEDi/` 目录（独立环境）

### ② Motion Quality（运动质量）

#### 4. Dynamic Degree（动态程度）
- **模型**：RAFT（光流估计）
- **怎么算**：算相邻帧的稠密光流，取 magnitude 前 5% 像素的平均
- **测什么**：视频里**有多少运动** —— 过静态 = 太死，过动态 = 抖动
- **代码**：`dynamic_degree.py`

#### 5. Flow Score（光流分数）
- **模型**：RAFT
- **怎么算**：算稠密光流后取 mean magnitude
- **测什么**：**整体运动强度**（类似 Dynamic Degree 但取均值不取 top-5%）
- **代码**：`flow_score.py`

#### 6. Motion Smoothness（运动平滑度）
- **模型**：[VFIMamba](https://github.com/MCG-NJU/VFIMamba)（Video Frame Interpolation）
- **怎么算**：用 VFIMamba 在已有相邻帧之间**插帧**，比较插帧和原帧的 SSIM
- **测什么**：帧间运动是否平滑（**崩坏 / 跳变会暴露**）
- **代码**：`motion_smoothness_metrics.py`

### ③ Content Consistency（内容一致性）

#### 7. Subject Consistency（主体一致性）⭐ Uni-WAM proposal 已选
- **模型**：DINOv1 (vit-base/16) + RAFT
- **怎么算**：用 RAFT 找运动区域，DINO 算运动主体的帧间 cosine 相似度
- **测什么**：**机械臂（主体）跨帧身份是否稳定**（消失 / 变形 = 崩）
- **代码**：`subject_consistency.py`

#### 8. Background Consistency（背景一致性）⭐ Uni-WAM proposal 已选
- **模型**：CLIP-B/32 + RAFT
- **怎么算**：用 RAFT 找**非运动区域**，CLIP 算这些区域的帧间相似度
- **测什么**：**背景**（桌子、墙）是否跨帧稳定
- **代码**：`background_consistency.py`

#### 9. Photometric Consistency（光度一致性）
- **模型**：[SEA-RAFT](https://github.com/princeton-vl/SEA-RAFT)（更精确的光流）
- **怎么算**：算相邻帧的光流 → 用光流 warp 上一帧到下一帧 → 算 warp 后和真实下一帧的 photometric 误差
- **测什么**：**像素亮度跨帧是否连续**（崩坏会跳变）
- **代码**：`flow_aepe_metrics.py`

### ④ Physics Adherence（物理保真）

#### 10. Interaction Quality（交互质量）⭐ Uni-WAM proposal 已选
- **模型**：Qwen3-VL-8B（VLM）
- **怎么算**：VLM 看 video + prompt，回答"接触/抓取/碰撞是否物理合理"
- **测什么**：robot-object 交互的物理合理性（穿模 / 漂浮 = 崩）
- **代码**：`VLM_judge.py`

#### 11. Trajectory Accuracy（轨迹精度）⭐ Uni-WAM proposal 用作 TA
- **模型**：SAM3 + NDTW
- **怎么算**：SAM3 检测末端 → 提取轨迹 → 和 GT action 期望轨迹算 NDTW
- **测什么**：生成视频的末端运动**和指令 action 的对齐度**
- **代码**：`trajectory_accuracy.py`

### ⑤ 3D Accuracy（3D 精度）

#### 12. Depth Accuracy（深度精度）
- **模型**：Depth-Anything V2
- **怎么算**：估计深度图，对比 GT 深度，算 Abs Rel
- **测什么**：3D 深度结构合理性
- **代码**：`depth_accuracy.py`

#### 13. Perspectivity（透视一致性）
- **模型**：Qwen3-VL（VLM）
- **怎么算**：VLM 判断"视角和透视是否物理合理"
- **测什么**：相机视角 / 透视关系一致性
- **代码**：`VLM_judge.py`

### ⑥ Controllability（可控性）

#### 14. Instruction Following（指令跟随）
- **模型**：Qwen3-VL（VLM）
- **怎么算**：VLM 看 video + 指令，判断"是否按指令执行"
- **代码**：`VLM_judge.py`

#### 15. Semantic Alignment（语义对齐）
- **模型**：Qwen2.5-VL（caption）+ CLIP-B（视觉对齐）+ BLEU
- **怎么算**：用 VLM 给视频生成 caption → 算和 GT 指令的 BLEU + CLIP 相似度
- **代码**：`semantic_alignment.py`

#### 16. Action Following（动作跟随）
- **模型**：CLIP（ViT-B/32）
- **怎么算**：把 action 表述成文本，对每帧算 CLIP image-text 相似度
- **代码**：`action_following.py`

---

## 4. ⭐ Visual Integrity Gate 复用建议（核心）

> Uni-WAM proposal 的 GPR（5 检测项）已经从 WorldArena 借了 3 个：
> Subject Consistency / Background Consistency / Interaction Quality
>
> 剩下 13 个里，哪些值得加入 / 哪些不适合 gate？

### 🟢 强烈建议加入 Visual Integrity Gate

| 指标 | 为什么强烈推荐 | 用法（阈值化 / 二值）|
|---|---|---|
| **Image Quality (MUSIQ)** | 直接测**单帧技术质量**（模糊/伪影），无歧义，崩坏必降 | 阈值化（< 阈值 = fail）|
| **Motion Smoothness (VFIMamba)** | 崩坏时帧间会跳变 → VFIMamba 插帧 SSIM 会暴跌 | 阈值化 |
| **Photometric Consistency (SEA-RAFT)** | 崩坏导致像素亮度跳变 → warp 误差暴增 | 阈值化 |

→ 这 3 个**专门测"技术崩坏"**，是 visual integrity 的天然候选。

### 🟡 建议谨慎加入（看 Uni-WAM 的设定）

| 指标 | 利 | 弊 |
|---|---|---|
| **Aesthetic Quality** | 美学暴跌可能反映崩坏 | 偏主观，**off-expert action 下美学可能就是低**，不一定崩 |
| **Depth Accuracy** | 3D 错乱时会暴露 | 需要 GT depth，我们的 setting 可能没有 |
| **Perspectivity (VLM)** | 视角崩坏会暴露 | VLM 判断有噪声 |
| **JEPA Similarity** | 语义相似度反映崩坏 | **需要 GT video**，off-expert setting 下可能没有 |

### 🔴 不适合加入 Visual Integrity Gate（属于"测能力"而非"测崩坏"）

| 指标 | 为什么不适合 |
|---|---|
| **Dynamic Degree** | 测运动量，**off-expert action 下运动量天然不同**，不是"崩坏"信号 |
| **Flow Score** | 同上 |
| **Trajectory Accuracy** | 这是**Uni-WAM 的 TA**，是 gate 后的核心指标，不是 gate 本身 |
| **Instruction Following** | 测"指令是否完成"，是 task-level 指标，不是 visual 崩坏 |
| **Semantic Alignment** | 同上，task-level |
| **Action Following** | 同上 |

---

## 5. ⭐ 我们自研 Gate 的建议（结合任务清单第 3 条）

> Ricardo 任务清单第 3 条："有些我们自己的 Gated（是否有夹爪、是否机械臂完整、可以用检测模型）写代码"

这些**WorldArena 没直接覆盖**，需要我们自研。建议方案：

| 自研 Gate 项 | 实现方式 | 复用 WorldArena 的工具 |
|---|---|---|
| **机械臂主体存在性** | SAM3 检测全臂 → 二值（每帧都有 = pass）| 可以用 WorldArena 的 SAM3 调用代码 |
| **夹爪可提取性** | SAM3 + prompt "gripper" → 二值 | 同上 |
| **机械臂完整性** | SAM3 mask 面积变化率 → 阈值化（mask 突然消失/变 0 = fail）| 自写检测逻辑 |
| **物体是否消失** | 任务相关物体（碗、积木）的 SAM3 检测 → 二值 | 自写 |

→ **WorldArena 给了我们 SAM3 调用的现成代码框架**（在 `trajectory_accuracy.py` 里），可以直接拿来扩展。

---

## 6. 验证有效性的实验方案建议

> Ricardo 任务清单第 2 条："用极限/亦辰的 WM 推理视频复现这些指标，验证它们的有效性"

**实验设计**：
1. 拿极限 / 亦辰的 WM 生成几组视频（一定要有**视觉崩坏的样本**作为 negative case）
2. 对每个候选指标（建议先做 🟢 推荐的 3 个 + 🟡 的 4 个），计算分数
3. 用 **AUC / Recall** 判断指标能否区分崩坏 vs 不崩坏

**关键复现路径**（用 WorldArena 的代码）：
```bash
cd code-reference/WorldArena
# 单独跑某个指标
python evaluate.py --dimension image_quality motion_smoothness photometric_consistency \
  --config config.yaml
```

**最快验证方案**：
1. 先用 `image_quality` (MUSIQ) 试 —— 最简单，pyiqa 一行调用
2. 再加 `motion_smoothness` —— VFIMamba 也现成
3. 最后试 `photometric_consistency` —— SEA-RAFT 设置稍复杂

---

## 7. 给翔哥汇报时的关键 takeaway

| 项 | 现状 |
|---|---|
| **proposal 现有的 5 个 gate 项** | 主体存在性 / 末端可提取性 / 物理交互合理性 / 背景稳定性 / VLM 兜底 |
| **proposal 已经从 WorldArena 借了** | Subject Consistency / Background Consistency / Interaction Quality（3/16）|
| **建议补加** | Image Quality / Motion Smoothness / Photometric Consistency（**3 个"测崩坏"的强候选**）|
| **建议自研** | 机械臂完整性 / 夹爪可提取性 / 任务物体存在性（用 SAM3）|
| **建议不加** | Dynamic Degree / Flow Score / 所有 Controllability 类指标（不是测崩坏）|

→ **GPR 最终可能 = 3 已选 + 3 强候选 + 2-3 自研 ≈ 8-9 个 component**

---

## 8. 我已经做了什么 / 你接下来做什么

**我已经做的**（在 `code-reference/` 里）：
- ✅ Clone WorldArena GitHub repo
- ✅ 拿到 16 个指标的全部 Python 实现代码
- ✅ 整理出 6 子维度 → 16 指标的完整映射
- ✅ 列出哪些可复用 / 哪些不适合的清单

**你接下来要做的**：
1. **筛选**：从 🟢🟡 候选里圈定要复用的（建议先 3 个 🟢）
2. **复现**：用极限/亦辰的 WM 视频跑这些指标
3. **验证**：算 AUC / Recall 看指标能否区分崩坏 vs 不崩坏
4. **自研**：用 SAM3 写"机械臂完整性 / 夹爪可提取性"的代码

---

## 🔗 参考

- 完整代码：[`code-reference/WorldArena/`](../code-reference/WorldArena/) （16 个指标的 Python 实现全在里面）
- 配置文件：[`code-reference/config.yaml`](../code-reference/config.yaml)
- 主入口：[`code-reference/evaluate.py`](../code-reference/evaluate.py)
- 聚合脚本：[`code-reference/aggregate_results.py`](../code-reference/aggregate_results.py)（含完整 16 指标列名）
