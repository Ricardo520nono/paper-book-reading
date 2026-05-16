# WorldArena: A Unified Benchmark for Evaluating Perception and Functional Utility of Embodied World Models

**作者**：Yu Shang, Zhuohang Li, Yiding Ma 等（清华 THU + SJTU + HKU + Princeton + CAS + USTC + PKU + NUS）
**链接**：[arXiv 2602.08971](https://arxiv.org/abs/2602.08971) · [GitHub](https://github.com/tsinghua-fib-lab/WorldArena) · [Leaderboard](https://huggingface.co/spaces/WorldArena/WorldArena) · [Project](https://world-arena.ai) · [CVPR 2026 Challenge](http://cvpr2026challenge.world-arena.ai/)
**Uni-WAM 调研分类**：**World Model Benchmark**（翔哥 proposal "相关工作" 表已收录）

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 16 个视觉质量指标深度拆解 + Visual Integrity Gate 复用建议

---

## 📌 一句话总结

WorldArena = 清华 FIB-Lab 牵头的统一具身 WM benchmark。**评估 3 维**：
1. **视频感知质量**（16 个指标 × 6 子维度）
2. **具身任务功能**（Data Engine / Policy Evaluator / Action Planner）
3. **人类评估**

提出综合指标 **EWMScore**（16 个归一化指标的算术平均）。在 14 个代表性模型上跑了，发现"感知-功能 gap" —— **视觉质量高 ≠ 具身任务能力强**。

---

## 🎯 Ricardo 当前任务（关键）

> "**读 WorldArena 第一大类指标（16 个视觉质量）AI 辅助，筛选出我们能复用的**"

### 16 指标 × 6 子维度速查

| 子维度 | 指标（共 16）| 工具 |
|---|---|---|
| ① **Visual Quality** | Image Quality / Aesthetic Quality / **JEPA Similarity** | MUSIQ / CLIP-L / V-JEPA-2 |
| ② **Motion Quality** | Dynamic Degree / Flow Score / Motion Smoothness | RAFT / RAFT / VFIMamba |
| ③ **Content Consistency** | **Subject Consistency** ⭐ / **Background Consistency** ⭐ / Photometric Consistency | DINO / CLIP-B / SEA-RAFT |
| ④ **Physics Adherence** | **Interaction Quality** ⭐ / Trajectory Accuracy (TA) | Qwen3-VL / SAM3+NDTW |
| ⑤ **3D Accuracy** | Depth Accuracy / Perspectivity | Depth-Anything / Qwen3-VL |
| ⑥ **Controllability** | Instruction Following / Semantic Alignment / Action Following | Qwen3-VL / Qwen2.5-VL+CLIP / CLIP |

⭐ = Uni-WAM proposal 的 GPR 已选项

### 给 Visual Integrity Gate 的复用建议

| 推荐度 | 指标 | 理由 |
|---|---|---|
| 🟢 **强烈推荐** | Image Quality (MUSIQ) | 直接测单帧技术质量（模糊/伪影），无歧义 |
| 🟢 **强烈推荐** | Motion Smoothness (VFIMamba) | 崩坏时帧间跳变 → SSIM 暴跌 |
| 🟢 **强烈推荐** | Photometric Consistency (SEA-RAFT) | 像素跳变 → warp 误差暴增 |
| 🟡 谨慎加入 | Aesthetic Quality / Depth Accuracy / Perspectivity / JEPA Similarity | 各有利弊（详见 summary）|
| 🔴 不建议 | Dynamic Degree / Flow Score / Controllability 3 项 | 测能力不是测崩坏 |

### 自研 Gate 建议（任务清单第 3 条）

| 自研项 | 方法 |
|---|---|
| 机械臂主体存在性 | SAM3 检测全臂 → 二值 |
| 夹爪可提取性 | SAM3 + prompt "gripper" |
| 机械臂完整性 | SAM3 mask 面积变化率 → 阈值 |
| 任务物体存在性 | SAM3 检测目标物体 |

→ 详细分析见 [`sections/summary.md`](sections/summary.md)

---

## 📖 笔记结构

```
[arXiv 2602.08971] WorldArena/
├── README.md                     ← 本文件
├── sections/
│   └── summary.md                ← 🌟 16 指标深度拆解 + 复用建议（核心交付物）
└── code-reference/               ← Clone 自 GitHub，所有指标的实现代码
    ├── WorldArena/               ← 16 个指标的 Python 文件
    │   ├── imaging_quality.py    ← Image Quality (MUSIQ)
    │   ├── subject_consistency.py
    │   ├── background_consistency.py
    │   ├── motion_smoothness_metrics.py
    │   ├── flow_aepe_metrics.py  ← Photometric Consistency (SEA-RAFT)
    │   ├── depth_accuracy.py
    │   ├── trajectory_accuracy.py
    │   ├── semantic_alignment.py
    │   ├── action_following.py
    │   ├── flow_score.py
    │   ├── dynamic_degree.py
    │   ├── aesthetic_quality.py
    │   └── ...
    ├── config.yaml               ← 各模型 ckpt 路径
    ├── evaluate.py               ← 主入口
    └── aggregate_results.py      ← 含完整 16 指标列名（CSV column order）
```

⚠️ **paper.pdf 未包含** —— bash sandbox 网络限制，无法从 arXiv 直接下载。可从 [arxiv.org/abs/2602.08971](https://arxiv.org/abs/2602.08971) 自行下载到本地后放入此目录。

---

由 Ricardo + Claude 协作整理 · Metric 阶段 1 验证 · 视觉完整性 Gated 指标
