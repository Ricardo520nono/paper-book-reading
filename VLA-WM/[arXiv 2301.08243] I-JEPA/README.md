# I-JEPA: Self-Supervised Learning from Images with a Joint-Embedding Predictive Architecture

**作者**：Mahmoud Assran, Quentin Duval, Ishan Misra, Piotr Bojanowski, Pascal Vincent, Michael Rabbat, Yann LeCun, Nicolas Ballas
**机构**：Meta AI (FAIR) / McGill / Mila / NYU
**链接**：[arXiv 2301.08243](https://arxiv.org/abs/2301.08243) · CVPR 2023
**定位**：**JEPA 入门母论文**（图像版 JEPA，不是视频 world model）

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 3 张图过完整篇 paper（5 分钟入门）

> 这篇先不做逐 section 批读，先把 **JEPA 到底是什么** 讲透。后面如果继续读 V-JEPA / V-JEPA 2，再回来看这篇会很顺。

---

## 📌 一句话总结

I-JEPA = **看图的一部分（context），去预测另一部分（target）在表征空间里的表示**，而不是补目标区域的像素。

---

## 🎯 这篇 paper 最该学会什么

| 点 | 一句话 |
|---|---|
| **JEPA 和 MAE 的根本差别** | **JEPA 预测表征，MAE 重建像素** |
| **JEPA 和 DINO 类方法的差别** | **JEPA 是表示预测，不是纯表示对齐** |
| **为什么能更偏语义** | target block 故意做得比较大，context 也足够大，但不能直接露答案 |

**这篇 paper 的价值**：
- ✅ **是读懂 V-JEPA 家族前最好的入门文**
- ✅ 把"为什么预测表征而不是像素"讲得最清楚
- ✅ 把"任务怎么出题，决定模型学什么"这件事讲得很明白
- ⚠️ **它不是世界模型 paper**：对象是单张 image representation learning，不是交互式 video prediction

---

## 📖 笔记结构

```text
[arXiv 2301.08243] I-JEPA/
├── README.md                      ← 本文件
├── paper.pdf                      ← 原始 PDF
├── images/                        ← 关键 figure
│   ├── figure-02-architectures.png
│   ├── figure-03-ijepa-overview.png
│   └── figure-04-masking-strategy.png
└── sections/
    └── summary.md                 ← 🌟 3 张图过完整篇（先读这个）
```

---

由 Ricardo + Codex 协作整理 · JEPA 入门篇
