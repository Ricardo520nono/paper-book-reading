[← 返回 Action Images 主页](../README.md)

# §5 Conclusion

## 原文 + 翻译

> "We presented a world action model that formulates policy learning as video generation through a unified video-space representation of observation and action. Our key idea is to translate 7-DoF robot control into interpretable action images, yielding a pixel-grounded action in the form of multi-view videos. This design allows the video backbone itself to serve as a zero-shot policy model, without requiring a separate policy head or action module."

**翻译**：提出一个 WAM，通过**统一 video-space 表示**把 policy learning 表述成 video generation。核心 idea = 把 7-DoF 控制翻译成可解释的 action image（pixel-grounded 的多视角视频）。这让 **video backbone 自己就是 zero-shot policy**，不需要单独 policy head / action module。同一个模型支持 video-action 联合生成 / AC video gen / action labeling。

## ¶ Limitations（paper 自己承认的）

> "Our current system demonstrates strong open-loop results, but has not yet been fully developed into a closed-loop policy. ... In future work, we plan to distill our model for faster inference and integrate it into a closed-loop control pipeline."

**翻译**：paper 自己承认的 limitation：
1. **只有 open-loop 结果，还没发展成 closed-loop policy**
2. 未来计划：蒸馏加速 + 集成进 closed-loop 控制 pipeline

🔥🔥 **Uni-WAM 视角 —— 关键对照**

Action Images 自己承认的 limitation：**只 open-loop，缺 closed-loop**。

**完全没提**：
- ❌ off-expert action 评估
- ❌ 训练/评估数据的 action 分布偏倚（DROID/RLBench/BridgeV2 的 action 都来自 expert demo 或 GT trajectory）
- ❌ counterfactual / random-feasible / adversarial action 测试

→ 和之前读的**所有 paper 一样** —— 没有一篇碰 off-expert action 这个轴。

---

## 📌 整篇 paper 读完的高层 takeaway

### 是不是 AC-WM？
✅ 是真正的 AC-WM —— mask 策略 2（Action-Conditioned Video Generation）就是标准 AC-WM 用法。但 Action Images **更进一步**：它把 AC-WM 当成统一模型的一种 mode，不是全部。

### 分类（a/b/c/d/e）？
**a 类 AC-WM**（真具身 robot manipulation），基于 Wan 2.2 fine-tune。

### Action 注入方式？
**独特路线 —— action 不是"注入"，是"变成 video"**：7-DoF action → 3 个语义 3D 点 → RGB Gaussian heatmap → action video，和 robot video 在同一 video space。

### 训练数据？
DROID（80k 真机）+ RLBench（180k 仿真）+ BridgeV2（30k video-only）。Action 都来自 expert demo / GT trajectory。

### 核心创新角度？
**统一 video-space 表示** —— 让 video backbone 自己当 zero-shot policy，不需要单独 action module。一个 backbone + 4 mask 策略 cover 所有任务。

### 测过 off-expert / OOD action 吗？
**❌ 没有**。zero-shot 测的是"policy 在未见物体/环境下成功率"，action 是 policy 自己产生的；AC video gen 评估用的 action 是 GT trajectory。

### 这篇对 Uni-WAM 的价值？
🔥 **三重价值**：
1. **同 backbone（Wan 2.2）** —— 和 Uni-WAM proposal 计划用的 Wan2.2-TI2V-5B 完全同系列，**训练范式、数据处理、3D-VAE 用法可直接参考**
2. **"action 即视频"是 Uni-WAM 没考虑过的 action 表示路线** —— 可以作为对照（Uni-WAM proposal 用的是 frame-level action injection，更接近 Ctrl-World 那派）
3. **4 mask 策略 = Uni-WAM "一体化 MoT" 的现实参照** —— 特别是 mask 策略 3（video-to-action labeling）就是 Uni-WAM 想要的 IDM

---

## 🔥 拷打（1 题，格式 B）

### Q-AImg.1

**问**：Action Images 的"4 种 mask 策略"（joint gen / AC video gen / video-to-action labeling / video-only）和 Uni-WAM proposal 的"一体化 MoT（VLA / WM / IDM / VLA+WM joint）"看起来是同一个 idea。**它俩到底是不是一回事？Uni-WAM 的一体化设计还有独特性吗？**

**答**：

> **核心 idea 确实高度相似 —— 都是"一个 backbone + 切换 input/output 组合 = 多任务统一"。但有三个关键区别，Uni-WAM 的设计仍有独特性。**
>
> **1. 统一的"机制"不同：mask vs 采样策略**
> - Action Images：用 **latent token 上的 mask** 切换任务（mask 掉哪些 token = 哪个任务）
> - Uni-WAM：用 **flow matching 的采样策略** 切换（"给定哪些维度、生成哪些维度"）
> - → 两者都能实现"一个 backbone 多任务"，但 Uni-WAM 的采样策略路线和它的 MoT 双分支架构耦合，不是单纯 mask
>
> **2. 架构不同：单 backbone vs MoT 双分支**
> - Action Images：**单个 Wan 2.2 backbone**，action 和 video 都是同一个 backbone 处理的 token
> - Uni-WAM：**MoT（Mixture-of-Transformers）** —— 生成分支（Wan2.2-TI2V-5B）+ 动作分支（Qwen3-VL-4B）+ Shared Attention 耦合
> - → Action Images 是"action 也是 video，所以一个 backbone 够了"；Uni-WAM 是"action 和 video 是不同模态，需要两个分支 + 共享底层"
>
> **3. 关键缺失：IDM 反向正则化**
> - Action Images 有 video-to-action labeling（mask 策略 3）≈ IDM 的功能，**但它只是"多一个能用的任务"**，不是用来"正则化 WM"
> - Uni-WAM 的 IDM 是**有明确目的的**：用 IDM 能力**反向约束 WM**，强迫 WM 学真实 action→obs 映射而非记 pair（proposal Model 章节的 candidate argument）
> - → Action Images 把 IDM 当"附加能力"，Uni-WAM 把 IDM 当"训练 WM 的正则化武器" —— **这是 Uni-WAM 最独特的设计**
>
> **结论**：Action Images 验证了"统一多任务 backbone"这个 idea 在 robot 领域**可行且有效**（实验上一个模型打过 5 类专门 baseline）—— 这对 Uni-WAM 是**正面信号**。但 Uni-WAM 的独特性在于：(1) MoT 双分支处理"action 是独立模态"的假设；(2) **IDM 不只是附加能力，而是反向正则化 WM 的核心机制**。Action Images 没做第 2 点 —— 它的 video-to-action labeling 只是被动学一个 task，没有"用 IDM 去治 WM 记 pair 的毛病"这层设计。

---

[← §4 Experiments](04-experiments.md) | [返回主页 →](../README.md)
