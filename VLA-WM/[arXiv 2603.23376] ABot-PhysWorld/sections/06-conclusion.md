[← 返回 ABot-PhysWorld 主页](../README.md)

# §6 Conclusion

## 原文 + 翻译

> "We introduce ABot-PhysWorld, a physically grounded and action-controllable world model for embodied manipulation based on a 14B Diffusion Transformer. It integrates curated data, physical alignment through Diffusion-DPO, and spatial action injection to reduce physical violations while maintaining control across different embodiments."

**翻译**：ABot-PhysWorld 是基于 14B DiT 的 physically-grounded + action-controllable 具身 manipulation world model。整合了**策划数据 + Diffusion-DPO 物理对齐 + 空间动作注入**，减少物理违例同时保持跨 embodiment 控制。

> "We also propose EZSbench, a zero-shot benchmark featuring out-of-distribution scenarios and a decoupled evaluation protocol. Experimental results show state-of-the-art physical fidelity and improved trajectory consistency compared to Veo 3.1 and Sora v2 Pro."

**翻译**：还提出 EZSbench 零样本 benchmark。实验显示 SOTA 物理保真度 + 比 Veo 3.1 / Sora v2 Pro 更好的轨迹一致性。

## ¶ Limitations（paper 自己承认的）

> "The model currently relies on fixed-viewpoint data and lacks closed-loop evaluation. Future work will explore multi-view generation and real-world deployment."

**翻译**：paper 自己承认的 2 个 limitation：
1. **依赖固定视角数据** —— 未来要做 multi-view generation
2. **缺 closed-loop evaluation** —— 未来要做 real-world deployment

🔥🔥 **Uni-WAM 视角 —— 关键对照**

ABot-PhysWorld 自己承认的 limitation：**固定视角 + 缺 closed-loop**。

**完全没提**：
- ❌ off-expert action 评估
- ❌ 训练数据 action 分布偏倚（它训练数据全来自 expert demo，A2V 的 action 都是 ground-truth trajectory）
- ❌ counterfactual / random-feasible / adversarial action 测试

→ 这和之前读的所有 paper（Ctrl-World / GE / EnerVerse-AC）一样 —— **没有一篇碰 off-expert action 这个轴**。Uni-WAM 的 Action Following Fidelity 仍然是空白。

---

## 📌 整篇 paper 读完的高层 takeaway

经过完整批读，对 ABot-PhysWorld 的判断：

### 是不是 AC-WM？
✅ 是真正的 AC-WM（§3.3 Action-Conditioned Video Generation，7D action 注入）

### 分类（a/b/c/d/e）？
**a 类 AC-WM**（真具身 robot manipulation），自训。基于 Wan2.1-I2V-14B 改造 + 训练。

### Action 注入方式？
**Action Map（7D pose 画成 RGB）+ 并行 context block（VACE 式）+ zero-conv 残差融合**

### 训练数据？
3M clips（5 个公开数据集），过滤 + 平衡 + 物理感知标注。A2V 的 action 来自 ground-truth trajectory（**全 expert**）。

### 核心创新角度？
**物理合理性（physics plausibility）** —— 用 DPO 后训练消除穿模/反重力，这是和 Ctrl-World / GE / EnerVerse-AC 不同的切入角度。

### 测过 off-expert / OOD action 吗？
**❌ 没有**。EZSbench 的 OOD 是 robot/task/scene 组合，不是 action。

### 这篇对 Uni-WAM 的价值？
🔥 **三重价值**：
1. **同 backbone 系列**（Wan2.1 → Uni-WAM 用 Wan2.2）—— 数据 pipeline / 训练范式可直接参考
2. **DPO 物理对齐**和 Uni-WAM 的 GPR（Gated 物理门）是同一目标的两种实现 —— 一个训练时压制，一个评估时筛选
3. **直接 baseline** —— Table 3 已经把 EnerVerse-AC 比下去了，Uni-WAM 可以拿 ABot-PhysWorld 当对手 / baseline

---

## 🔥 拷打（1 题，格式 B）

### Q-ABot.1

**问**：ABot-PhysWorld 用 **DPO 物理对齐**消除穿模/反重力，Uni-WAM 用 **Gated Metrics (GPR)** 在评估时筛掉物理违例的视频。**这两个东西是不是在做同一件事？Uni-WAM 还有必要单独做 GPR 吗？**

**答**：

> **不是同一件事，Uni-WAM 的 GPR 仍有必要。** 三个层面的区别：
>
> **1. 作用阶段不同：训练时 vs 评估时**
> - ABot 的 DPO 是**训练时**机制 —— 让模型**学会**不生成物理违例
> - Uni-WAM 的 GPR 是**评估时**机制 —— 给任意一个 WM **打分**，判断它生成的视频合不合格
> - → 一个是"教模型"，一个是"考模型"。即使 ABot 用 DPO 训好了，你还是需要 GPR 这种**第三方 benchmark** 来客观衡量它
>
> **2. 服务的目标不同：生成质量 vs action following 诊断**
> - ABot 的 DPO 目标是"生成的视频物理合理" —— 它不关心"这个视频有没有 follow 喂进去的 off-expert action"
> - Uni-WAM 的 GPR 是 **Gated 架构的第一道门** —— 它的作用是"先确认视觉合理（准入资格），再用 TA 测 action following"。GPR 本身不是终点，是为 TA 服务的前置
> - → ABot 的 DPO 即使训得再好，**也回答不了"WM 在 counterfactual action 下能不能 follow"** 这个 Uni-WAM 的核心问题
>
> **3. 数据分布不同：expert 分布内 vs off-expert**
> - ABot 的 DPO 训练数据全来自 expert demo（A2V 用 ground-truth trajectory）—— 它学会的"物理合理"是**在 expert 分布内的物理合理**
> - Uni-WAM 的 GPR 要在 **5 类 off-expert action** 喂进去后判断 —— 测的是"WM 在它没见过的 action 下还能不能保持物理合理"
> - → 即使 ABot 的 DPO 在 expert 分布内消除了穿模，**喂一个 counterfactual action 时它会不会又穿模？没人知道** —— 这正是 Uni-WAM GPR 要回答的
>
> **结论**：ABot 的 DPO 和 Uni-WAM 的 GPR **目标相邻但不重叠**。ABot 是"训练时让生成质量变好"，Uni-WAM 是"评估时诊断 WM 在 off-expert action 下的可靠性"。**Uni-WAM 完全有必要单独做 GPR** —— 而且 ABot-PhysWorld 这种"DPO 训得很好的 model"恰恰应该被 Uni-WAM 的 benchmark 拿去测：**它在 off-expert action 下还守不守物理？**

---

[← §5 Experiments](05-experiments.md) | [返回主页 →](../README.md)
