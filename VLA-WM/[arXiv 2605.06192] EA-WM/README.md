# EA-WM: Event-Aware Generative World Model with Structured Kinematic-to-Visual Action Fields

**作者**：Zhaoyang Yang, Yurun Jin, Lizhe Qi, Cong Huang, Kai Chen
**机构**：Fudan University / Zhongguancun Academy / Zhongguancun Institute of AI / USTC / DeepCybo
**链接**：[arXiv 2605.06192](https://arxiv.org/abs/2605.06192) (2026/05)
**Uni-WAM 调研分类**：**a 类 AC-WM（真具身 robot manipulation）** ⭐⭐ Uni-WAM 最强架构参照

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 2 张图过完整篇 paper（5 分钟入门）

---

## 📌 一句话总结

EA-WM = **"把整条机械臂都画进 action field"的 event-aware 双分支 AC-WM**。把 action + kinematic state 渲染成 KVAFs（相机对齐的视觉场：手臂骨架/joint/gripper/末端 heatmap/pose 轴），用双分支（video + KVAF）+ EDLS 驱动的 event-aware 双向 fusion。Wan2.2-TI2V backbone，WorldArena P3CScore SOTA 76.60，直接打败 Ctrl-World / IRASim / Genie Envisioner。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM，真具身 manipulation）|
| **场景** | 仿真 robot manipulation（WorldArena / RoboTwin）|
| **Backbone** | **Wan2.2-TI2V**（和 Uni-WAM proposal 计划用的 Wan2.2-TI2V-5B 完全一致）|
| **Action 注入** | **KVAFs** —— forward kinematics + camera projection 把 action 渲染成"整条手臂"的视觉场 + 独立 KVAF branch + event-aware 双向 fusion |
| **Action 表示** | 渲染成视觉场（手臂骨架 / joint landmark / gripper 几何 / 末端 heatmap / pose 轴）|
| **训练数据** | WorldArena RoboTwin 数据（仿真），action 全是 expert / GT trajectory |
| **核心创新角度** | 用 action 引导精确 video 生成（"逆问题"），KVAFs + EDLS |
| **OOD action 测试** | ❌ 无（WorldArena 的 action 全是 expert / GT trajectory）|

**对 Uni-WAM 的价值（⭐⭐ 最强架构参照）**：
- ✅ **同 backbone（Wan2.2-TI2V）**
- ✅ **双分支 + 双向 fusion ≈ Uni-WAM 的 MoT + Shared Attention** —— EA-WM 在 WorldArena SOTA，**证明这个架构哲学 work**
- ✅ **直接 baseline 对比**（Table 4：Ctrl-World 74.03 / IRASim 69.75，EA-WM 78.13）—— Uni-WAM 可照搬评估设置
- ✅ **WorldArena 是 proposal 已收录的 benchmark**
- ✅ **EDLS（帧差 latent 监督）思路** 对 Uni-WAM 有启发
- ⚠️ 但和 Uni-WAM 区别本质（详见 [05-conclusion.md](sections/05-conclusion.md) 的 Q-EAWM.1）：动作分支模态不同 / 缺 IDM 反向正则化 / 没碰 off-expert action

---

## 📖 批读导航

| # | Section | 内容 | 状态 |
|---|---|---|---|
| - | [**summary.md**](sections/summary.md) | 🌟 串讲速读（2 张图过全文）| ✅ |
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | ✅ |
| 1 | [01-introduction.md](sections/01-introduction.md) | 正问题 vs 逆问题 + domain misalignment + 两个贡献 | ✅ |
| 2 | [02-related-work.md](sections/02-related-work.md) | robotic video WM + WAM + action 表示谱系 | ✅ |
| 3 | [03-method.md](sections/03-method.md) | KVAFs 构造 + 双分支架构 + event-aware fusion + EDLS | ✅ |
| 4 | [04-experiments.md](sections/04-experiments.md) | WorldArena 主结果 + ablation + baseline 对比 | ✅ |
| 5 | [05-conclusion.md](sections/05-conclusion.md) | Limitations + 结论 + Q-EAWM.1 拷打 | ✅ |

---

## 🖼️ 关键 figure

| 图 | 内容 |
|---|---|
| figure-01 | Raw Action Conditioning vs KVAFs Conditioning 对比（核心动机图）|
| figure-02 | EA-WM 架构总览（KVAFs construction + Latent encoding + Z-WM architecture）|

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第九篇（完整批读）
