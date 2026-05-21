[← 返回主页](../README.md)

# §1 Introduction

## 动机：从「视觉预测器」到「交互式环境」

> "In embodied applications, however, the practical value of a world model depends on more than visual realism. Beyond generating visually accurate future observations, these models need to capture physically grounded dynamics to support downstream reasoning and decision-making, such as action planning and policy learning."

**翻译**：在具身应用里，WM 的实用价值**不止于视觉逼真**。除了生成视觉精确的未来观测，模型还得捕捉**物理上扎实的动力学**，以支撑下游推理与决策（动作规划、策略学习）。

> "This transition from visual predictors to interactive environments naturally requires a corresponding shift in evaluation methods."

**翻译**：从「视觉预测器」到「交互式环境」的转变，自然要求评估方法也跟着变 —— 现代 benchmark 正逐渐从「视觉质量」扩到「功能效用（functional utility）」。

## 现有 benchmark 的两大类 + WorldArena 的定位

> "Existing benchmarks for world models generally fall into two categories. The first one primarily focuses on the quality of video generation... The second category advances toward embodied functionality, assessing whether world models can capture action-conditioned dynamics to facilitate specific downstream tasks."

**翻译**：现有 WM benchmark 大致两类：
1. **只看视频生成质量**（时空一致性、视觉保真）
2. **进到具身功能**：测 WM 能否捕捉 action-conditioned 动力学来支撑下游任务 —— 代表作 WorldSimBench / WorldEval / World-in-World / WoW-World-Eval / **WorldArena**。

> "Among these, WorldArena represents an important step toward unified embodied world model evaluation. It provides a systematic benchmark that jointly evaluates perceptual quality and downstream functional utility... including data engine, action planning and policy evaluation."

**翻译**：其中 **WorldArena（1.0）** 是统一具身 WM 评估的重要一步 —— 既看生成视频的保真度，也看它对具身任务（**data engine / action planning / policy evaluation**）的有用性，把「预测精度」和「任务层有效性」连了起来。

## 🔥 三大局限（2.0 要补的洞）

> "Specifically, there remain three major limitations."

**翻译**：但这些 benchmark 仍有三大局限：

**① 局限一：几乎只做 vision-only**

> "First, prevailing benchmarks are predominantly confined to vision-only settings. This overlooks the multimodal nature of embodied interactions, where tactile feedback is essential for resolving contact-rich dynamics and physical friction."

忽略了具身交互的多模态本质 —— **触觉反馈对解析「接触丰富的动力学」和物理摩擦至关重要**，这些视觉看不全。

**② 局限二：下游评估只到开环 planning / 静态 policy 评估**

> "Second, downstream evaluation is largely restricted to open-loop planning or static policy evaluation, rarely investigating the capacity of a world model to serve as an interactive reinforcement learning environment that supports iterative policy improvement through imagined rollouts."

很少研究 WM 能否当**交互式 RL 环境**，通过「想象 rollout」支撑迭代式的 policy 改进。

**③ 局限三：结果几乎全在仿真**

> "Finally, benchmark results are currently obtained almost exclusively in simulation, leaving it entirely unclear whether strong performance in controlled virtual environments translates to real-world deployment."

仿真里强，到底能不能迁到真机部署？**完全不清楚。**

## 三个扩张轴 + 三个贡献

> "To address these limitations, we introduce WorldArena 2.0, which extends WorldArena along three coordinated dimensions: modality, functionality, and platform."

- **模态轴**：基于 **UniVTAC 仿真器**搭标准框架，让视觉 WM 感知/预测更丰富的多模态感官流（接触、力、打滑、材料交互）。
- **功能轴**：把 WM 当**在线交互 RL 环境**训具身 agent —— 模型必须在迭代交互里保持稳定 + action-consistent + 产 reward-relevant 状态转移 + 不严重累积误差。
- **平台轴**：除了两个仿真环境（**RoboTwin + LIBERO**），引入 **AgileX Split-Type ALOHA 真机**，测两个真机任务：**pour water（倒水）+ wipe table（擦桌）**。

> 主要贡献三条：
> - 提出 WorldArena 2.0，沿模态/功能/平台三轴全面升级具身 WM 评估。
> - 首创纳入 visuotactile 模态、并把 WM 当在线 RL 环境评估的标准化协议。
> - 在 **12 个具身 WM** 上跨仿真+真机做了大量实验，揭示了仿真与现实的一致趋势，同时凸显出**显著的 sim-to-real 可用性 gap**。

---

## 💡 §1 批注（Uni-WAM 视角）

- 三个局限是「评估覆盖面」的局限（模态/闭环/真机），**不是「action 分布」的局限**。我们 proposal 攻的是第四个维度：评估用的 action 永远是 expert/GT，没人喂 off-expert。
- 「imagined rollouts 支撑 policy 改进」+「不严重累积误差」+「action-consistent」这几个词，和我们关心的「WM 在 off-expert action 下会不会崩」是高度相关的痛点描述 —— 可以在 proposal 里引用它作为「闭环可靠性很重要」的论据。

---

[← §0 Abstract](00-abstract.md) | [§2 Related Work →](02-related-work.md)
