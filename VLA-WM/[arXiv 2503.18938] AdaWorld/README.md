# AdaWorld: Learning Adaptable World Models with Latent Actions

**作者**：Shenyuan Gao, Siyuan Zhou, Yilun Du, Jun Zhang, Chuang Gan
**机构**：HKUST / Harvard / UMass Amherst / MIT-IBM Watson AI Lab
**链接**：[arXiv 2503.18938](https://arxiv.org/abs/2503.18938) · ICML 2025 · [Project Page](https://adaptable-world-model.github.io)
**Uni-WAM 调研分类**：**a 类 AC-WM**（但 action 是 latent action，不是数值）

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 4 张图过完整篇 paper（5 分钟入门）

> 其他 section 暂未展开。如需深挖某节，再单独写。

---

## 📌 一句话总结

AdaWorld = **从 unlabeled video 里 unsupervised 抽出 latent action 作为"通用接口"**，让 WM 跨环境通用 + 新环境少量 finetune（30 秒）就能 adapt。最大卖点：**完全不需要 action label**。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM）|
| **场景** | 跨环境通用（Habitat / Minecraft / DMLab / nuScenes）|
| **Action 注入** | Latent action 作为条件输入到 autoregressive WM |
| **Action 表示** | **Latent action（连续向量，learned）—— 不是数值，不是离散**|
| **训练数据** | **Unlabeled video**（无需 action label）⭐ |
| **AC finetune pipeline** | 提供（few-shot adaptation）|
| **OOD action 测试** | 无（关注跨环境泛化，不关注 OOD action）|

**对 Uni-WAM 的价值**：
- ✅ **latent action 思路启发**：可以考虑从公开 video（如 YouTube）学 action 表示，绕开仿真器+Cosmos-Transfer 的数据 pipeline
- ✅ **information bottleneck 设计**：可能启发 IDM 反向正则化
- ⚠️ **5 类 off-expert 测试不直接套用**：Uni-WAM 用数值 action，AdaWorld 用 latent

---

## 📖 笔记结构

```
[arXiv 2503.18938] AdaWorld/
├── README.md          ← 本文件
├── paper.pdf          ← 原始 PDF
├── images/            ← 关键 figure
│   ├── figure-01.png  ← 三种 paradigm 对比（overview）
│   ├── figure-02.png  ← Latent Action Autoencoder
│   ├── figure-03.png  ← Action-Aware Pretraining
│   └── figure-06.png  ← PSNR few-shot 实验结果
└── sections/
    └── summary.md     ← 🌟 4 张图过完全文（**先读这个**）
```

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第三篇
