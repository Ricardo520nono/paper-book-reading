# ABot-PhysWorld: Interactive World Foundation Model for Robotic Manipulation with Physics Alignment

**作者**：AMAP CV Lab, Alibaba Group（Yuzhi Chen, Ronghan Chen 等，详见 §7 Contributions）
**机构**：AMAP CV Lab, Alibaba Group
**链接**：[arXiv 2603.23376](https://arxiv.org/abs/2603.23376) (v2 2026/03) · [Project Page](https://github.com/amap-cvlab/ABot-PhysWorld)
**Uni-WAM 调研分类**：**a 类 AC-WM（真具身 robot manipulation）**

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 5 张图过完整篇 paper（5 分钟入门）

---

## 📌 一句话总结

ABot-PhysWorld = **"用 DPO 教 Wan2.1 守物理规矩"的 14B 具身 AC-WM**。三个核心创新：数据策划 pipeline + 物理感知 DPO 后训练 + 并行 context block 动作注入。配套 EZSbench 零样本 benchmark。PBench / EZSbench 双 SOTA，动作条件生成直接超 EnerVerse-AC。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM，真具身 manipulation）|
| **场景** | 真机 robot manipulation（BridgeData V2 / AgiBot / OXE 等）|
| **Backbone** | **Wan2.1-I2V-14B**（和 Uni-WAM proposal 的 Wan2.2 同系列）|
| **Action 注入** | Action Map（7D pose 画成 RGB）+ 并行 context block（VACE 式）+ zero-conv 残差融合 |
| **Action 表示** | 7D `[3D 位置, 3D 朝向, gripper]`（双臂 14D）|
| **训练数据** | 3M clips（5 公开数据集）；A2V 的 action 来自 ground-truth trajectory（全 expert）|
| **核心创新角度** | **物理合理性（physics plausibility）** —— DPO 消除穿模/反重力 |
| **OOD action 测试** | ❌ 无（EZSbench 的 OOD 是 robot/task/scene 组合，不是 action）|

**对 Uni-WAM 的价值**：
- ✅ **同 backbone 系列**（Wan2.1 → Wan2.2）—— 数据 pipeline / 训练范式直接可参考
- ✅ **DPO 物理对齐**和 Uni-WAM 的 GPR 是同一目标的两种实现（训练时压制 vs 评估时筛选）
- ✅ **Action Map 构造细节**可直接复用
- ✅ **直接 baseline** —— Table 3 已把 EnerVerse-AC 比下去，Uni-WAM 可拿 ABot 当对手
- ⚠️ 同样没碰 off-expert action 这个轴 —— Uni-WAM 切入空间仍在

---

## 📖 批读导航

| # | Section | 内容 | 状态 |
|---|---|---|---|
| - | [**summary.md**](sections/summary.md) | 🌟 串讲速读（5 张图过全文）| ✅ |
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | ✅ |
| 1 | [01-introduction.md](sections/01-introduction.md) | 问题背景 + 两个根因 + 三个贡献 | ✅ |
| 2 | [02-data-curation.md](sections/02-data-curation.md) | 数据策划：过滤 / 平衡 / 物理感知标注 | ✅ |
| 3 | [03-method.md](sections/03-method.md) | 方法：backbone + 物理 DPO + 动作注入 | ✅ |
| 4 | [04-benchmark.md](sections/04-benchmark.md) | EZSbench：双源构造 + decoupled 评估协议 | ✅ |
| 5 | [05-experiments.md](sections/05-experiments.md) | 实验：PBench / EZSbench / 动作条件 三组结果 | ✅ |
| 6 | [06-conclusion.md](sections/06-conclusion.md) | 结论 + Limitations + Q-ABot.1 拷打 | ✅ |

---

## 🖼️ 关键 figure

| 图 | 内容 |
|---|---|
| figure-01 | 数据策划 pipeline overview（过滤/平衡/标注）|
| figure-02 | 两阶段训练 pipeline（SFT + DPO）|
| figure-03 | EZSbench 构造 pipeline |
| figure-04 | 动作条件生成架构（并行 context block + zero-conv）|
| figure-05 | PAI-Bench 定性对比（各 baseline 物理违例标注）|

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第七篇（完整批读）
