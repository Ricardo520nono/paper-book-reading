# Genie Envisioner: A Unified World Foundation Platform for Robotic Manipulation

**作者**：Yue Liao, Pengfei Zhou, Siyuan Huang, Donglin Yang, Shengcong Chen, Yuxin Jiang, Yue Hu, Jingbin Cai, Si Liu, Jianlan Luo, Liliang Chen, Shuicheng Yan, Maoqing Yao, Guanghui Ren
**机构**：AgiBot Genie Team + LV-NUS Lab + BUAA
**链接**：[arXiv 2508.05635](https://arxiv.org/abs/2508.05635) (v3 2025/11) · [Project](https://genie-envisioner.github.io) · [GitHub](https://github.com/AgibotTech/Genie-Envisioner)
**Uni-WAM 调研分类**：**a 类 AC-WM 平台 + b 类候选（pipeline 开源）** ⭐⭐

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 3 张图过完平台（5 分钟入门）

---

## 📌 一句话总结

Genie Envisioner = AgiBot 出品的**统一世界基础模型平台**, 4 件套（**GE-Base** 大 video WM + **GE-Act** policy + **GE-Sim** action-conditioned simulator + **EWMBench** benchmark）共享 backbone, 全部 closed-loop 跑在 AgiBot World 1M+ trajectories 上。**EnerVerse-AC 的升级 + 扩展**。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（GE-Sim 是 AC-WM）+ b 类（GE-Base backbone 完整开源 finetune pipeline）|
| **场景** | 真机 robot manipulation (AgiBot World)|
| **Action 注入** | **不在 GE-Base 里**（GE-Base 是 instruction-conditioned）<br>Action condition 在 **GE-Sim** 衍生（与 EnerVerse-AC 思路一脉相承）|
| **Action 表示** | 6D end-effector pose + gripper (双臂时 14D) |
| **训练数据** | AgiBot-World-Beta（**1M+ trajectories, 3000+ hours video**）|
| **AC finetune pipeline** | ✅ 完整开源（GE-Base / GE-Act / GE-Sim）|
| **OOD action 测试** | ❌ 无（平台聚焦"基础设施"，不做 OOD action 评估）|

**对 Uni-WAM 的价值**：
- ✅ **统一平台范式启发**：Uni-WAM proposal 的"MoT 一体化"在 GE 上有现实版本
- ✅ **数据规模 reference**：1M+ trajectories 是真机数据上限
- ✅ GE-Sim 思路是 EnerVerse-AC 的演进, 同团队
- ⚠️ Paper 是"基础设施"定位, **没系统对照 off-expert action 评估**

---

## 📖 笔记结构

```
[arXiv 2508.05635] Genie Envisioner/
├── README.md          ← 本文件
├── paper.pdf          ← 原始 PDF
├── images/            ← 关键 figure
│   ├── figure-01.png  ← 全平台 overview (GE-Base + Act + Sim + Bench)
│   ├── figure-03.png  ← GE-Base 架构 (autoregressive + causal block)
│   ├── figure-07.png  ← GE-Act 3-Stage 训练 pipeline
│   └── figure-14.png  ← GE-Sim action 注入机制 (Pose2Image + Motion Vector)
└── sections/
    └── summary.md     ← 🌟 3 张图过完全文（**先读这个**）
```

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第六篇（真具身 manipulation 平台）
