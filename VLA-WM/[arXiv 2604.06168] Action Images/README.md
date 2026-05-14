# Action Images: End-to-End Policy Learning via Multiview Video Generation

**作者**：Haoyu Zhen, Zixian Gao, Qiao Sun, Yilin Zhao, Yuncong Yang, Yilun Du, Tsun-Hsuan Wang, Yi-Ling Qiao, Chuang Gan
**机构**：UMass Amherst / UTokyo / NVIDIA / Harvard / Genesis AI
**链接**：[arXiv 2604.06168](https://arxiv.org/abs/2604.06168) (2026/04) · [Project Page](https://ActionImages.github.io)
**Uni-WAM 调研分类**：**a 类 AC-WM（真具身 robot manipulation）**

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 用 4 张图过完整篇 paper（5 分钟入门）

---

## 📌 一句话总结

Action Images = **"把 action 也画成视频"的统一 world-action model**。把 7-DoF action 翻译成 pixel-grounded 的 action image（RGB Gaussian heatmap），让 Wan 2.2 backbone 自己当 zero-shot policy，不需要单独 policy head。一个 backbone + 4 mask 策略 cover 联合生成 / AC video gen / video-to-action labeling / video-only gen。

---

## 🎯 Uni-WAM 视角的初步判断

| 维度 | 信息 |
|---|---|
| **分类** | a 类（自训 AC-WM，真具身 manipulation）|
| **场景** | 真机 + 仿真 robot manipulation（DROID / RLBench / BridgeV2）|
| **Backbone** | **Wan 2.2**（和 Uni-WAM proposal 计划用的 Wan2.2-TI2V-5B 完全同系列）|
| **Action 注入** | **独特路线** —— action 不是注入，是变成 video（action image = RGB Gaussian heatmap）|
| **Action 表示** | 7D `[位置, 朝向, gripper]` → 3 个语义 3D 点 → RGB Gaussian heatmap |
| **训练数据** | DROID 80k + RLBench 180k + BridgeV2 30k；action 全来自 expert demo / GT trajectory |
| **核心创新角度** | 统一 video-space 表示 —— video backbone 自己当 policy |
| **OOD action 测试** | ❌ 无（zero-shot 测的是 policy 泛化，不是 OOD action）|

**对 Uni-WAM 的价值**：
- ✅ **同 backbone（Wan 2.2）** —— 训练范式 / 3D-VAE / 数据处理直接可参考
- ✅ **"action 即视频"是 Uni-WAM 没考虑过的 action 表示路线** —— 可作对照
- ✅ **4 mask 策略 = Uni-WAM "一体化 MoT" 的现实参照**（mask 策略 3 ≈ IDM）
- ⚠️ 同样没碰 off-expert action —— Uni-WAM 的 IDM 反向正则化仍是独特设计

---

## 📖 批读导航

| # | Section | 内容 | 状态 |
|---|---|---|---|
| - | [**summary.md**](sections/summary.md) | 🌟 串讲速读（4 张图过全文）| ✅ |
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | ✅ |
| 1 | [01-introduction.md](sections/01-introduction.md) | video 泛化 ≠ policy 泛化 + 三个贡献 | ✅ |
| 2 | [02-related-work.md](sections/02-related-work.md) | robotics WM / generalist policy / 4D generation | ✅ |
| 3 | [03-method.md](sections/03-method.md) | Action as Images + Decoding + 统一训练 | ✅ |
| 4 | [04-experiments.md](sections/04-experiments.md) | zero-shot policy + 联合生成质量 + 附加能力 | ✅ |
| 5 | [05-conclusion.md](sections/05-conclusion.md) | 结论 + Limitations + Q-AImg.1 拷打 | ✅ |

---

## 🖼️ 关键 figure

| 图 | 内容 |
|---|---|
| figure-01 | Action Images overview（4 块：多视角观测 / pixel-grounded action image / 联合生成 / zero-shot 3D policy）|
| figure-02 | Action as Image（7-DoF → 3 个语义 3D 点 → RGB Gaussian heatmap）|
| figure-03 | Action Images Decoding（ray casting + side-view matching）|
| figure-04 | Unified world-action model training（4 mask 策略）|

---

由 Ricardo + Claude 协作整理 · Uni-WAM 调研第八篇（完整批读）
