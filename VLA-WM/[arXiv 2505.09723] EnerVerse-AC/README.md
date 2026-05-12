# EnerVerse-AC: Envisioning Embodied Environments with Action Condition

**作者**：Yuxin Jiang, Shengcong Chen, Siyuan Huang, Liliang Chen, Pengfei Zhou, Yue Liao, Xindong He, Chiming Liu, Hongsheng Li, Maoqing Yao, Guanghui Ren
**机构**：AgiBot + SJTU + MMLab-CUHK
**链接**：[arXiv 2505.09723](https://arxiv.org/abs/2505.09723) · 2025/05 · [GitHub](https://annaj2178.github.io/EnerverseAC.github.io)
**Uni-WAM 调研分类**：**a 类 AC-WM (真具身 robot manipulation)** ⭐

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 3 张图过完整篇 paper（5 分钟入门）

---

## 📌 一句话总结

EnerVerse-AC = AgiBot 出品的 **"多视角 + 多级 action 注入 + 人工 failure 数据"** robot AC-WM。在 EnerVerse 前作上加 action condition, 5 相机多视角联合预测, 逐 task + 逐 training step 都和真机评估对齐, 可作为 policy evaluator + data engine。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM, 真具身）|
| **场景** | 真机 robot manipulation (AgiBot 数据集)|
| **Action 注入** | **Multi-Level**: Spatial-Aware Pose RGB + Delta Action Cross-Attention + Gripper Magnitude RGB |
| **Action 表示** | 6D end-effector pose + gripper(开/关) |
| **训练数据** | AgiBot World（私有, 1M+ trajectories）+ **人工 augmented failure trajectories** ⭐ |
| **AC finetune pipeline** | GitHub 已开源（待确认 finetune script）|
| **OOD action 测试** | ❌ 无（仍是 expert + human failure, 不是 systematic off-expert）|

**对 Uni-WAM 的价值**：
- ✅ **多视角 + 多级 action 注入**工程做法可借鉴
- ✅ **人工 augmented failure**思路和 Uni-WAM 的"仿真器造 off-expert"异曲同工
- ✅ **逐 task / 逐 step 对齐验证**比 Ctrl-World 更严格
- ⚠️ Failure 仍是 human-attempted, 未覆盖 counterfactual / random-feasible

---

## 📖 笔记结构

```
[arXiv 2505.09723] EnerVerse-AC/
├── README.md          ← 本文件
├── paper.pdf          ← 原始 PDF
├── images/            ← 关键 figure
│   ├── figure-01.png  ← Overview (hardware + multi-view + action)
│   ├── figure-02.png  ← 架构 (Action Condition Input + Diffusion Model)
│   └── figure-07.png  ← 真机 vs EVAC 逐 task / 逐 step 对齐
└── sections/
    └── summary.md     ← 🌟 3 张图过完全文（**先读这个**）
```

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第五篇（真具身 manipulation）
