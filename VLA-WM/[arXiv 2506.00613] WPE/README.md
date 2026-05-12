# WorldGym (WPE): World Model as An Environment for Policy Evaluation

**作者**：Julian Quevedo, Ansh Kumar Sharma, Yixiang Sun, Varad Suryavanshi, Percy Liang, Sherry Yang
**机构**：Stanford / NYU / Google DeepMind
**链接**：[arXiv 2506.00613](https://arxiv.org/abs/2506.00613) (v3 2025/09)
**Uni-WAM 调研分类**：**a 类 AC-WM (Robot)**

> ⚠️ **命名变更**：paper v1 叫 WPE (World-model-based Policy Evaluation)，**v3 改名 WorldGym**。文件名沿用 WPE。
>
> ⚠️ **可能与翔哥已填的 "WorldGym (2025/05)" 是同一篇** —— 内容完全对得上（OOD image + language 测试 / 不考虑 OOD action）。

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 3 张图过完整篇 paper（5 分钟入门）

---

## 📌 一句话总结

WorldGym = **"AC video gen WM + VLM as reward"** 的 policy evaluation 框架。RT-1-X / Octo / OpenVLA 三个 VLA 在 WorldGym 里排名和真机相关性 **r=0.78**。能造 OOD image / OOD language 场景测 generalization。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM）|
| **场景** | 真机 robot manipulation (Bridge / Open X-Embodiment)|
| **Action 注入** | Chunk-wise diffusion forcing + bidirectional attention (16 帧 chunk) |
| **Action 表示** | end-effector control 数值 action |
| **训练数据** | Bridge V2 等公开 robot demo (无 off-expert)|
| **AC finetune pipeline** | 提供（fine-tune from video diffusion checkpoint）|
| **OOD action 测试** | ❌ 无 (OOD 只在 image + language 轴) |

**对 Uni-WAM 的价值**：
- ✅ **VLM as reward** 思路启发（Uni-WAM GPR Component 3 同思路）
- ✅ OOD image / language 测试范式可借鉴
- ⚠️ Method 贡献偏弱，主要看 **evaluation framework**
- ❌ 完全不测 OOD action（正是 Uni-WAM 切入空白）

---

## 📖 笔记结构

```
[arXiv 2506.00613] WPE/
├── README.md          ← 本文件
├── paper.pdf          ← 原始 PDF
├── images/            ← 关键 figure
│   ├── figure-01.png  ← Overview (3 路输入 + WM + VLM)
│   ├── figure-04.png  ← real world vs WM correlation (r=0.78)
│   └── figure-08.png  ← OOD Color Classification 测试示例
└── sections/
    └── summary.md     ← 🌟 3 张图过完全文（**先读这个**）
```

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第四篇
