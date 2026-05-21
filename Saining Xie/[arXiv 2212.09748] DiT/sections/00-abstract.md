[← 返回主页](../README.md)

# §0 Abstract

> "We explore a new class of diffusion models based on the transformer architecture. We train latent diffusion models of images, replacing the commonly-used U-Net backbone with a transformer that operates on latent patches."

**翻译**：我们探索一类**基于 Transformer 架构的新扩散模型**。训练图像的 latent diffusion model，把常用的 **U-Net backbone 换成一个在 latent patch 上操作的 Transformer**。

> "We analyze the scalability of our Diffusion Transformers (DiTs) through the lens of forward pass complexity as measured by Gflops. We find that DiTs with higher Gflops—through increased transformer depth/width or increased number of input tokens—consistently have lower FID."

**翻译**：用「前向复杂度（Gflops）」这个视角分析 **Diffusion Transformer (DiT)** 的可扩展性。发现：Gflops 更高的 DiT（通过增加 transformer 深度/宽度，或增加输入 token 数）**一致地有更低的 FID**。

> "In addition to possessing good scalability properties, our largest DiT-XL/2 models outperform all prior diffusion models on the class-conditional ImageNet 512×512 and 256×256 benchmarks, achieving a state-of-the-art FID of 2.27 on the latter."

**翻译**：除了优秀的可扩展性，最大的 **DiT-XL/2** 在 class-conditional ImageNet 512×512 和 256×256 上**超过所有此前的扩散模型**，在后者上拿到 **SOTA FID 2.27**。

---

## 💡 摘要批注

- 三个关键词：**Transformer backbone（换骨架）、latent patches（在 latent 上切块）、scalability（可扩展性，用 Gflops 衡量）**。
- "consistently have lower FID" 这句是全文骨架 —— 它要建立的是一条**经验 scaling law**（算力↑→质量↑），而不只是"刷了个 SOTA"。
- FID 2.27 是当时 ImageNet 256 生成的最强成绩，超过 StyleGAN-XL（2.30）和所有 U-Net 扩散模型。

---

[§1 Introduction →](01-introduction.md)
