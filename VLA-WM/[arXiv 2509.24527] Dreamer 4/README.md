# Dreamer 4: Training Agents Inside of Scalable World Models

**作者**：Danijar Hafner, Wilson Yan, Timothy Lillicrap
**机构**：Google DeepMind
**链接**：[arXiv 2509.24527](https://arxiv.org/abs/2509.24527) · 发表 2025/09 · [Project Page](https://danijar.com/dreamer4)
**Uni-WAM 调研分类**：**a 类 AC-WM**（场景是 Minecraft game，不是 robot manipulation）

> ⚠️ **重要更名**：之前 backlog 误写 "DreamerV3 latest"。**实际是 Dreamer 4**（不是 V3 续作；是架构 + 训练范式重启）。

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 5 张图过完整篇 paper（5 分钟入门）

> 其他 section 暂未展开。如果调研需要深挖某节，再单独写。

---

## 📌 一句话总结

Dreamer 4 = **第一个在 Minecraft 上纯 offline 拿到 diamond 的 agent**。靠 3-phase pipeline（unlabeled video pretrain → labeled finetune → imagination RL）+ 高效 transformer 架构，real-time inference on 1 GPU。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM，自己设计架构）|
| **场景** | Minecraft game（**不是 robot manipulation**）⚠️ |
| **Action 注入** | Action 作为 token 拼接（不是 cross-attention）|
| **Action 表示** | 键盘 + 鼠标动作（121 类离散）|
| **训练数据** | VPT 2541 小时 contractor gameplay + 大量 unlabeled YouTube video |
| **AC finetune pipeline** | 提供（3-phase）|
| **OOD action 测试** | 无（关注 imagination RL，不关注 OOD action）|

**对 Uni-WAM 的价值**：**方法论借鉴**（3-phase 训练 / unlabeled video pretrain），**不是直接对照**。

---

## 📖 笔记结构

```
[arXiv 2509.24527] Dreamer 4/
├── README.md          ← 本文件
├── paper.pdf          ← 原始 PDF
├── images/            ← 关键 figure
│   ├── figure-01.png  ← imagination rollout overview
│   ├── figure-02.png  ← 架构图（tokenizer + dynamics）
│   ├── figure-03.png  ← 主结果 bar chart
│   ├── figure-04.png  ← ablation
│   └── figure-05.png  ← human interaction
└── sections/
    └── summary.md     ← 🌟 5 张图过完全文（**先读这个**）
```

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第二篇
