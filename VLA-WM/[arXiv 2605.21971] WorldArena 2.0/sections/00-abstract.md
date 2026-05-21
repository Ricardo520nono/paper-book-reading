[← 返回主页](../README.md)

# §0 Abstract

> "World models have emerged as a central paradigm for embodied intelligence, enabling agents to predict action-conditioned future and reason about environmental dynamics. However, existing embodied world model benchmarks are still largely confined to vision-only prediction, offline embodied applications, and simulator-based evaluation, making them insufficient for assessing increasingly comprehensive world models."

**翻译**：世界模型已成为具身智能的核心范式，让 agent 能预测「动作条件下的未来」并对环境动力学进行推理。但现有的具身世界模型 benchmark 仍大多局限在**纯视觉预测、离线具身应用、基于仿真器的评估**三点上，不足以评估越来越全面的世界模型。

> "In this work, we introduce WorldArena 2.0, an expanded benchmark that systematically broadens embodied world model evaluation along three dimensions: modality, functionality, and platform."

**翻译**：本文提出 **WorldArena 2.0**，一个扩展的 benchmark，沿三个维度系统性拓宽具身 WM 评估：**模态、功能、平台**。

三个维度具体是：

- **模态（Modality）**：从 vision-only 扩到 **visuotactile（视觉+触觉）**，能评估多模态感知与预测。
- **功能（Functionality）**：从 policy evaluation / planning，扩到把 WM 当**交互式 RL 环境**做 policy 优化。
- **平台（Platform）**：从 simulator-only 扩到**仿真 + 真机**的多形态机器人测试套件。

> "Under a standardized protocol, WorldArena 2.0 comprehensively evaluates perceptual quality, interactive utility, and cross-platform performance, providing a comprehensive testbed for tracking progress toward embodied world models."

**翻译**：在标准化协议下，2.0 全面评估**感知质量、交互效用、跨平台表现**三方面，为追踪具身 WM 的进展提供一个综合 testbed。

**项目地址**：https://world-arena.ai

---

## 💡 摘要批注（Uni-WAM 视角）

- 「action-conditioned future」开篇就点了 AC-WM 的核心定义，和我们 proposal 完全一致。
- 三个「现有局限」（vision-only / offline / simulator-only）正是 2.0 三个扩张轴的镜像 —— **但注意：「off-expert action」不在它列举的三个局限里**。2.0 仍默认 action 来自正常分布。我们 proposal 补的恰好是它没列的第四条。

---

[§1 Introduction →](01-introduction.md)
