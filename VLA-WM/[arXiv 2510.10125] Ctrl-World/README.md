# Ctrl-World: A Controllable Generative World Model for Robot Manipulation

**作者**：Yanjiang Guo, Lucy Xiaoyang Shi, Jianyu Chen, Chelsea Finn
**机构**：Tsinghua + Stanford
**链接**：[arXiv 2510.10125](https://arxiv.org/abs/2510.10125) · 发表 2025/10（v3 2026/03）
**Uni-WAM 调研分类**：**a 类（原本就是 AC-WM）**

> **本笔记从 Uni-WAM 调研视角阅读**。读完后**提炼到 Uni-WAM 视角**单独写在 [`../Uni-WAM-调研/notes/Ctrl-World.md`](../Uni-WAM-调研/notes/Ctrl-World.md)（待写），并补充到 [`../Uni-WAM-调研/README.md`](../Uni-WAM-调研/README.md) 的调研主表。

---

## 📌 一句话总结

> Ctrl-World 是一个**可控、多视角**的 generative world model，把 generalist robot policy 的 action chunk 当条件输入，用来在 closed-loop 里**评估 / 改进 policy** —— 在 DROID 数据上训，**作为 policy evaluator** 时 ranking 与真机一致，**作为 data synthesizer** 时把 π0.5 在新任务的成功率从 38.7% → 83.4%。

---

## 🎯 Uni-WAM 视角的学习目标

读完这篇我必须能回答：

### Q1：Ctrl-World 的 action 是怎么注入的？（关键，决定它是不是真正的 AC-WM）
> 答案在 §4.1 "Frame-level Action Conditioning"。

### Q2：Ctrl-World 训练数据来自哪？只有 expert 吗？
> 答案在 §5.1 Experiment Setups（DROID dataset）。

### Q3：Ctrl-World 测过 off-expert / OOD action 吗？
> 答案在 §5.3 + §6。**这是 Uni-WAM 最关心的 gap**。

### Q4：Ctrl-World 怎么处理"循环论证"问题？（policy 训练数据 → 评估的 WM 也是这个数据）
> 关键看 §4.2 + §5.3 如何论证 "WM evaluation ↔ real evaluation" 的一致性。

---

## 📖 批读导航

| # | Section | 内容 | Uni-WAM 相关性 | 状态 |
|---|---|---|---|---|
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | 🔥 必读 | ⏳ 写中 |
| 1 | [01-introduction.md](sections/01-introduction.md) | Introduction：要解决的问题 + Ctrl-World 自己的 pitch | 🔥 必读 | ⏳ 待写 |
| 2.1 | ~~02-related-work-videogen.md~~ | Video Generation Models | 选读 | ⏭️ **跳过** |
| 2.2 | [02-related-work-acwm.md](sections/02-related-work-acwm.md) | Action-Conditioned World Models（看它怎么定位自己 vs 其他 AC-WM）| 🔥 必读 | ⏳ 待写 |
| 3 | [03-problem-formulation.md](sections/03-problem-formulation.md) | Problem Formulation（W 的输入输出形式定义）| 🔥 必读 | ⏳ 待写 |
| 4.1 | [04-method-learning.md](sections/04-method-learning.md) | Multi-View + Pose Memory + **Frame-level Action Conditioning** | 🔥🔥 **核心** | ⏳ 待写 |
| 4.2 | [04-method-policy-eval.md](sections/04-method-policy-eval.md) | Using Ctrl-World for Policy Evaluation and Improvement | 🔥🔥 **核心** | ⏳ 待写 |
| 5.1 | [05-experiment-setup.md](sections/05-experiment-setup.md) | Setups（DROID 数据 / 训练细节）| 🔥 必读 | ⏳ 待写 |
| 5.2 | ~~05-quality-analysis.md~~ | 视觉质量评估 | 选读 | ⏭️ **可跳**（基础指标）|
| 5.3 | [05-policy-evaluation.md](sections/05-policy-evaluation.md) | 实测：WM 评估 vs 真机评估 排名一致性 | 🔥🔥 **核心** | ⏳ 待写 |
| 5.4 | ~~05-policy-improvement.md~~ | 用 WM 造合成数据训 policy（在 π0.5 上做了实验）| 选读 | ⏭️ **可跳**（不影响 a 类判断）|
| 6 | [06-conclusion.md](sections/06-conclusion.md) | Conclusion + Limitations | 🔥 必读 | ⏳ 待写 |

**Push 策略**：只 push 上面 🔥 标记的 section，跳过的不写。

---

## 🚨 标记说明（看笔记时关注）

| 标记 | 含义 |
|---|---|
| 🔥 | Uni-WAM 强相关的段落 / 句子（直接影响 a/b/c/d 分类、或揭示 off-expert gap）|
| 💡 | Claude 批注（背景补充 / 对比 / 解释）|
| 🎯 | 必须能回答的问题 |
| 🤔 | 留下的疑问 |
| ⚠️ | 容易踩的坑 |

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第一篇
