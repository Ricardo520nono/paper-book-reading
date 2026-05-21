# WorldArena 2.0: Extending Embodied World Model Benchmarking on Modality, Functionality and Platform

**作者**：Yu Shang, Yinzhou Tang, Yiding Ma, Zhuohang Li, Lei Jin 等（清华牵头，Yong Li 通讯）+ SJTU + 浙大 + Stanford + HKU + Princeton + CAS + USTC + PKU + NUS
**链接**：[arXiv 2605.21971](https://arxiv.org/abs/2605.21971) · [Project](https://world-arena.ai)
**Uni-WAM 调研分类**：**World Model Benchmark**（WorldArena 1.0 的升级版，proposal 已收录上游）

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 4 张图过全文 + 三轴扩张拆解 + `Action Response Sensitivity` 改名深挖（核心交付物）

---

## 📌 一句话总结

WorldArena 2.0 = **把 WorldArena（1.0）沿三个轴往「更接近真实部署」扩了一圈**：

1. **模态（Modality）**：vision-only → **visuotactile**（加触觉，UniVTAC 仿真器）
2. **功能（Functionality）**：离线评估 → **在线 RL 环境**（WM 当 POMDP proxy，GRPO 闭环训 policy）
3. **平台（Platform）**：simulator-only → **仿真 + 真机**（RoboTwin 2.0 + LIBERO + AgileX ALOHA）

16 个视觉质量指标**基本沿用 1.0**，唯一实质改动：`Action Following` → **`Action Response Sensitivity`**。
在 12 个具身 WM 上跑，核心结论是 **巨大的 sim-to-real usability gap** —— 仿真表现不是真机部署的可靠代理。

---

## 🎯 对我们 proposal 的关键价值

| 点 | 价值 |
|---|---|
| **proposal 已收录 WorldArena** | 2.0 是直接上游 + baseline 来源，必须跟踪 |
| ⭐ **`Action Response Sensitivity` 改名** | 最关键 —— 可能和我们「Action Following Fidelity」撞概念，**必须去 GitHub 查清定义**（对手 or baseline 组件？）|
| **sim-to-real gap 官方背书** | 给「分布内表现好 ≠ 真部署可靠」提供官方实验支撑 |
| **WM 当 RL 环境（闭环）** | 闭环/累积误差/action-consistency 是同源痛点的另一切面 |
| **明确区分 WM vs WAM** | 权威背书我们的分类轴 |
| **baseline 名单** | 12 模型 + 商用 Veo3.1/Wan2.6，评估设置可照搬 |

⚠️ **依旧没碰的轴**：和我们读过的所有 paper 一样，2.0 的所有评估 action 仍来自 expert/GT trajectory，**「off-expert action 下的可靠性」依然空着** —— 正是我们 proposal 的立足点。详见 [05-conclusion.md](sections/05-conclusion.md) 的 Q-WA2.1 / Q-WA2.2 拷打。

---

## 📖 批读导航

| # | Section | 内容 | 状态 |
|---|---|---|---|
| - | [**summary.md**](sections/summary.md) | 🌟 4 张图串讲速读 + 三轴拆解 + 16 指标对照 | ✅ |
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | ✅ |
| 1 | [01-introduction.md](sections/01-introduction.md) | 三大局限 → 三个扩张轴 + 三个贡献 | ✅ |
| 2 | [02-related-work.md](sections/02-related-work.md) | EWM 三类 + benchmark 三型 | ✅ |
| 3 | [03-method.md](sections/03-method.md) | §3.1 1.0→2.0 + §3.2 触觉 + §3.3 RL 环境 + §3.4 sim-to-real | ✅ |
| 4 | [04-experiments.md](sections/04-experiments.md) | 触觉/RL 环境/跨平台 + 16 指标全表 | ✅ |
| 5 | [05-conclusion.md](sections/05-conclusion.md) | 结论 + Q-WA2.1/Q-WA2.2 拷打 | ✅ |

---

## 🖼️ 关键 figure

| 图 | 内容 |
|---|---|
| figure-01 | 三轴扩张总览（同心圆：1.0 内圈 → 2.0 外圈）—— 全文地图 |
| figure-02 | 视触觉 WM 标准化架构 (a) + UniVTAC 评估管线 (b) |
| figure-03 | WM 当 RL 环境的三段式管线（训练 / RL 优化 / 评估）|
| figure-04 | 三个测试平台（RoboTwin 2.0 / LIBERO / AgileX ALOHA）|

---

由 Ricardo + Claude 协作整理 · WorldArena 2.0 完整批读 · proposal benchmark 上游跟踪
