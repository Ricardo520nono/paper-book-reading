# DiT: Scalable Diffusion Models with Transformers

**作者**：William Peebles (UC Berkeley) · **Saining Xie (NYU)**
**链接**：[arXiv 2212.09748](https://arxiv.org/abs/2212.09748) · [Project](https://www.wpeebles.com/DiT) · **CVPR 2023 (Oral)**
**定位**：谢赛宁代表作 · 现代视频生成 / 世界模型 backbone 的"母架构"

---

## 🌟 速读入口

👉 **[sections/summary.md](sections/summary.md)** —— 5 张图过全文（patchify / adaLN-Zero / scaling law 全讲清）

---

## 📌 一句话总结

**DiT = 把扩散模型默认的 U-Net backbone 换成纯 Transformer（ViT 风格）**，证明 **U-Net 的归纳偏置不是必需的**，且 DiT 继承 Transformer 的 scaling law（**Gflops ↔ FID 强相关，-0.93**）。最大的 DiT-XL/2 在 ImageNet 256 拿下 SOTA **FID 2.27**。

**三个要熟记的设计**：
1. **Latent diffusion**：在冻结 VAE 的 latent 空间做扩散（高效，可 scale）
2. **Patchify**：patch size `p` 控制 token 数 `T=(I/p)²` 和算力（小 p → 多 token → 高算力 → 低 FID，不增参数）
3. **adaLN-Zero**：把条件（timestep + class）注入 transformer 的最优方式 —— 从条件回归 γ/β/α，α 零初始化让 block 初始为恒等函数

---

## 📖 批读导航

| # | Section | 内容 | 状态 |
|---|---|---|---|
| - | [**summary.md**](sections/summary.md) | 🌟 5 张图串讲速读 | ✅ |
| 0 | [00-abstract.md](sections/00-abstract.md) | Abstract | ✅ |
| 1 | [01-introduction.md](sections/01-introduction.md) | §1 动机（U-Net 祛魅）+ §2 Related Work | ✅ |
| 3 | [03-method.md](sections/03-method.md) | §3 扩散基础 + DiT 设计空间（patchify + 4 种 block + adaLN-Zero）| ✅ |
| 4 | [04-experiments.md](sections/04-experiments.md) | §4 配置 + §5 scaling 实验 + SOTA + §6 结论 | ✅ |

---

## 🖼️ 关键 figure

| 图 | 内容 |
|---|---|
| figure-02-scaling | DiT vs U-Net 的 scaling 气泡图（又好又省）|
| **figure-03-architecture** | **DiT 架构全景 + 4 种 block 变体（最核心）** |
| figure-04-patchify | Patchify：patch size p → token 数 T=(I/p)² |
| figure-05-conditioning | 4 种条件注入对比（adaLN-Zero 最优）|
| figure-08-gflops-fid | Transformer Gflops vs FID 强相关（-0.93）|

---

## 🔗 为什么我（Ricardo）要熟读这篇

- **目标导师代表作**：要能随口讲清 adaLN-Zero / patchify / scaling 论点。
- **是 VLA-WM 调研的"地基"**：Wan2.2（Uni-WAM 计划的 backbone）、CogVideoX、Cosmos、EA-WM 等几乎全是 DiT 后代。理解 DiT 的条件注入谱系（in-context / cross-attn / adaLN）= 理解这些世界模型"action 怎么注入"的母版。
- **主线**：DiT → 视频 DiT → 世界模型。一作 Peebles 后来主导 OpenAI Sora（DiT 的视频版）。

---

由 Ricardo + Claude 协作整理 · Saining Xie 论文精读系列 · 第 1 篇
