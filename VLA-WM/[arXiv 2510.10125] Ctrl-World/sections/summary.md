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

*暂无对应 figure —— problem 用文字讲就够。Figure 1 是 solution overview，下面就上。*

---

## 2. 这篇 paper 的解决方法全景 · 一张图看懂

![](../images/figure-01.png)

**按 Figure 1 从左到右读**：

### 输入（左侧）
- **Instru**（紫块）：language instruction，比如 "place the blue block on the plate"
- **Cam 1 / 2 / 3**（绿块）：3 个相机视角的当前 obs（2 个 third-person + 1 个 wrist-view）
- **N step action chunk**（灰块）：policy 输出的 N 步数值 action（具体到 6D pose）

### 核心循环（中间，重点！）

```
Generalist Policy → 给我未来 N 步该怎么动 (action chunk)
        ↓
World Model    → 给我执行完这 N 步后画面变啥样 (Pred 1/2/3 三个视角)
        ↓
Pred 喂回 Policy → 下一个 N 步 action chunk
        ↓
循环 ×N 次
```

**关键名字 = "policy-in-the-loop world model rollout"**。两个 agent **互相喂数据**：
- Policy 决定动作 → WM 预测后果 → 后果喂回 policy → 再决定动作

### 输出（右侧）

整个 rollout 跑完 → 拿到一条 **synthetic trajectory**（合成轨迹）。这条轨迹有两种用法：

| 用例 | Figure 1 右侧展示 | 后面在哪节验证 |
|---|---|---|
| **Policy Evaluation** | 散点图（y=0.87x-0.04）—— WM 里的 policy 排名 ≈ 真机 ranking | §5.3 |
| **Policy Improvement** | 柱状图（+44.7%）—— 拿合成的 successful trajectory 做 SFT 提升 policy | §5.4 |

### 还有一个小细节：Memory

WM 旁边的 **Memory**（绿块）是 Ctrl-World 的关键工程组件 —— **稀疏历史帧 + pose 检索**，防止长 rollout 时漂移。详见 §4.1。

---

**一句话总结这张图**：
> Ctrl-World = **policy 和 WM 自回归交替运行的 simulator**，跑出来的合成 trajectory 既能用来给 policy 排名（evaluation），也能挑成功的来 SFT（improvement）。

🔥 **Uni-WAM 视角的小预警**：
- 这张图的 loop 流程**完全假设 policy 输出的 action 在 WM 训练分布内**
- 如果换一个早期 sub-optimal policy（输出 off-expert action）→ WM 还能正确预测吗？Figure 1 没回答
- 这是 Uni-WAM 关心的核心 gap

---

> 后续点位等 Ricardo 指定再补 ⏳
