[← 返回 Action Images 主页](../README.md)

# §0 Abstract

## 原文 + 翻译

> "World action models (WAMs) have emerged as a promising direction for robot policy learning, as they can leverage powerful video backbones to model the future states. However, existing approaches often rely on separate action modules, or use action representations that are not pixel-grounded, making it difficult to fully exploit the pretrained knowledge of video models and limiting transfer across viewpoints and environments."

**翻译**：World action model (WAM) 是 robot policy learning 的有前景方向 —— 能借强大的 video backbone 建模未来状态。但现有方法常依赖**单独的 action module**，或用**非 pixel-grounded** 的 action 表示 —— 难以充分利用 video 模型的预训练知识，限制了跨视角/跨环境的迁移。

> "In this work, we present Action Images, a unified world action model that formulates policy learning as multiview video generation. Instead of encoding control as low-dimensional tokens, we translate 7-DoF robot actions into interpretable action images: multi-view action videos that are grounded in 2D pixels and explicitly track robot-arm motion."

**翻译**：本文提出 **Action Images** —— 一个统一的 WAM，把 policy learning 表述成 **multiview video generation**。不把控制编码成低维 token，而是把 **7-DoF action 翻译成可解释的 action image**：pixel-grounded、显式追踪机械臂运动的多视角 action video。

> "This pixel-grounded action representation allows the video backbone itself to act as a zero-shot policy, without a separate policy head or action module. Beyond control, the same unified model supports video-action joint generation, action-conditioned video generation, and action labeling under a shared representation."

**翻译**：这种 pixel-grounded 表示让 **video backbone 自己就能当 zero-shot policy**，不需要单独的 policy head / action module。除了控制，同一个统一模型还支持 video-action 联合生成、action-conditioned video 生成、action labeling。

> "On RLBench and real-world evaluations, our model achieves the strongest zero-shot success rates and improves video–action joint generation quality over prior video-space world models."

**翻译**：RLBench + 真机评估上，达到最强 zero-shot 成功率，video-action 联合生成质量超过之前的 video-space WM。

---

## 💡 Abstract takeaway

| 维度 | 信息 |
|---|---|
| 核心创新 | 把 7-DoF action 翻译成 **action image**（pixel-grounded）|
| 关键结果 | video backbone 本身 = zero-shot policy，**不需要 policy head** |
| 统一能力 | 1 个模型 = 联合生成 + AC video gen + action labeling |
| Backbone | Wan 2.2 |

🔥 **Uni-WAM 视角**：注意它和之前所有 paper 的根本区别 —— 别人把 action 当"外挂条件"，它把 action 当"另一段视频"。

---

[§1 Introduction →](01-introduction.md)
