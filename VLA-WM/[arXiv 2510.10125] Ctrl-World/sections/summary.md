[← 返回 Ctrl-World 主页](../README.md)

# 串讲 · 一张图 + 几句话读懂 Ctrl-World

> **目的**：用最少的字数 + 几张关键 figure 把这篇 paper 串起来。不重复 sections/ 里的逐段细节，**只抓骨架**。

---

## 1. 这篇 paper 解决的是什么问题？

**两个痛点**（都属于 generalist robot policy 的研发循环）：

| 痛点 | 现状 | 为什么麻烦 |
|---|---|---|
| **Policy Evaluation** | 评估 VLA policy 要在**真机上反复跑 rollout** | 慢、贵、难规模化，阻碍迭代 |
| **Policy Improvement** | policy 出问题后**只能收集更多 expert 数据**再训 | 同样慢、贵；且很多 failure 模式无法靠加数据自然出现 |

**一句话**：研究员手里没有一个**快 + 便宜 + 反馈驱动**的工具来评估和改进 VLA policy。

**Ctrl-World 的解法定位**：
> 把"真机 rollout"换成"在 imagination 里 rollout" —— **用 world model 当 simulator**。

→ 后面我们会看到，它怎么搭这个 simulator（§4.1）、怎么用它评估（§4.2 + §5.3）、怎么用它改进（§5.4）。

*暂无对应 figure —— problem 用文字讲就够。Figure 1 是 solution overview，等到第 2 点"它的方案是什么"时再上。*

---

> 后续点位等 Ricardo 指定再补 ⏳
