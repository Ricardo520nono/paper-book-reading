[← 返回 Action Images 主页](../README.md)

# §1 Introduction

## ¶1 · 核心 gap：video 泛化 ≠ policy 泛化

> "World action models have made rapid progress in predicting future observations, but turning this predictive ability into policy generalization remains an open challenge. In particular, strong video generation does not automatically produce a strong policy: a model may successfully synthesize plausible future frames, yet still fail to decide how to act in unseen environments."

**翻译**：WAM 在预测未来观测上进步快，但把这种预测能力转成 **policy 泛化** 仍是 open challenge。**强 video generation 不会自动产生强 policy** —— 模型能合成合理的未来帧，但在未见环境里仍可能不知道怎么动。

🔥 **Uni-WAM 关联**：这个"video 泛化 ≠ policy 泛化"的 gap，和翔哥 proposal 的"WM 记 pair 而非学 dynamics"是相邻问题 —— 都在质疑"生成质量高 ≠ 真的理解 action"。

## ¶2 · 根因：action 表示方式不对

> "A key reason is that action is still not represented in a form that world models can naturally generalize. Existing approaches typically follow one of two paths. Some attach a separate policy head or action module on top of a world model... Others adapt video models to action generation using representations that are not spatially grounded in image space. In both cases, the model's predictive knowledge of the world is only indirectly connected to acting."

**翻译**：根因 = **action 没用 WM 能自然泛化的形式来表示**。现有两条路：
1. **挂单独 policy head / action module** —— 让额外网络从 video feature 解码控制
2. **用非空间 grounded 的表示** 把 video 模型改成 action 生成

两条路里，模型的"世界预测知识"和"行动"只是**间接连接** → 泛化的负担转嫁给专门控制模块，**而那里恰恰是 transfer 断裂的地方**。

## ¶3 · 解法：multi-view action video

> "We formulate policy learning as video generation and address policy generalization at the representation level. We propose multi-view action videos... we convert each action into a pixel-grounded action representation that explicitly tracks robot-arm motion in image space across multiple views. This design makes action native to the video model itself: the same video backbone can observe, predict, condition on, and generate action, enabling a zero-shot policy."

**翻译**：把 policy learning 表述成 video generation，**在表示层面**解决 policy 泛化。提出 **multi-view action video** —— 把 action 转成 pixel-grounded 表示，在多视角图像空间里显式追踪机械臂运动。

→ **让 action 成为 video model 的 native 表示**：同一个 video backbone 能 observe / predict / condition on / generate action → zero-shot policy。

## ¶4 · 为什么要 multi-view（不是为了多观测）

> "The motivation is not merely to add more visual observations, but to bridge the gap between 2D image and the 7-DoF robot action in the 3D space. A single view often provides only a ambiguous projection of motion."

**翻译**：multi-view 的动机**不是为了多观测**，而是**桥接 2D 图像和 3D 空间里的 7-DoF action 之间的鸿沟**。单视角对 motion 只是一个有歧义的投影，难以从像素一致地推断完整 action。多视角让 pixel-grounded action image 更**可重建**，遮挡时更鲁棒。

💡 **批注**：这是一个很关键的设计动机 —— action 本质是 3D 的，单视角投影丢深度信息。多视角不是锦上添花，是**让 action-as-image 这个表示能 work 的必要条件**。

## ¶5 · 三个贡献

| 贡献 | 内容 |
|---|---|
| **识别 gap** | video 泛化 ≠ policy 泛化，且这个 gap 可以在 **action 表示层面**解决 |
| **multi-view action representation** | 把 robot 控制翻译成 action image（pixel-grounded），用它构建 **zero-shot policy，不需 policy head** |
| **统一 video-space 表示** | 同一个 WM 支持 video-action 联合生成 / AC video gen / action labeling |

---

## 💡 §1 整段 takeaway

Action Images 的故事链：
1. WAM 预测能力强，但 policy 泛化弱（video 泛化 ≠ policy 泛化）
2. 根因：action 表示不对（policy head 外挂 / 非空间 grounded）
3. 解法：把 action 也变成 pixel-grounded 的 multi-view video
4. multi-view 是必要的（桥接 2D↔3D）
5. 结果：video backbone 自己当 zero-shot policy + 统一多任务

**Uni-WAM 视角**：这篇 paper 的核心论点"**在表示层面解决问题**"和 Uni-WAM 的"benchmark + IDM 反向正则化"是不同的攻击点，但都在回答同一个深层问题 —— **怎么让 WM 真的"理解" action 而不只是生成好看的 video**。

---

[← §0 Abstract](00-abstract.md) | [§2 Related Work →](02-related-work.md)
