[← 返回 EA-WM 主页](../README.md)

# §0 Abstract

## 原文 + 翻译

> "While recent world-action models jointly optimize future videos and actions, they predominantly treat video generation as an auxiliary representation for policy learning. Consequently, they insufficiently explore the inverse problem: leveraging action signals to guide video synthesis, thereby often failing to preserve precise robot spatial geometry and fine-grained robot-object interaction dynamics."

**翻译**：最近的 world-action model 联合优化 future video + action，但**主要把 video 生成当 policy learning 的辅助表示**。结果它们没充分探索**逆问题** —— 用 action 信号引导 video 合成 —— 经常**保不住机器人精确空间几何 + 细粒度 robot-object 交互动态**。

> "EA-WM projects actions and kinematic states directly into the target camera view as Structured Kinematic-to-Visual Action Fields (KVAFs)... To fully exploit this geometrically grounded representation, we introduce event-aware bidirectional fusion blocks that modulate cross-branch attention."

**翻译**：EA-WM 把 action 和 kinematic state **直接投影到目标相机视角**，变成 **Structured Kinematic-to-Visual Action Fields (KVAFs)**。用 **event-aware 双向 fusion blocks** 调制 cross-branch attention，捕捉物体状态变化和交互动态。

> "Evaluated on the comprehensive WorldArena benchmark, EA-WM achieves state-of-the-art performance, outperforming existing baselines by a significant margin."

**翻译**：在 WorldArena benchmark 上达到 SOTA，大幅超过现有 baseline。

---

## 💡 Abstract takeaway

| 维度 | 信息 |
|---|---|
| 核心创新 | KVAFs（把 action 投影成相机对齐的视觉场）+ event-aware 双向 fusion |
| 关键论点 | 现有 WAM 只做"video → 帮 action"，没做"action → 引导 video"的逆问题 |
| Backbone | Wan2.2-TI2V |
| Benchmark | WorldArena（翔哥 proposal 已收录）|

🔥 **Uni-WAM 视角**：这篇 paper 的切入角度 = "用 action 引导精确 video 生成" —— 和 ABot-PhysWorld（物理对齐）、Ctrl-World（policy eval）又不同。它关心的是**生成视频里机器人几何 + 交互动态的保真度**。

---

[§1 Introduction →](01-introduction.md)
