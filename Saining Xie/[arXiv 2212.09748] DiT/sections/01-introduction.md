[← 返回主页](../README.md)

# §1 Introduction + §2 Related Work

## §1 Introduction

### 动机：生成模型是 Transformer 浪潮的"钉子户"

> "Machine learning is experiencing a renaissance powered by transformers. Over the past five years, neural architectures for natural language processing, vision and several other domains have largely been subsumed by transformers. Many classes of image-level generative models remain holdouts to the trend, though."

**翻译**：机器学习正经历一场由 Transformer 驱动的复兴。过去五年，NLP、视觉等领域的神经架构大多被 Transformer 吞并。**但很多图像级生成模型仍是这股趋势的"钉子户"。**

> "diffusion models have been at the forefront of recent advances in image-level generative models; yet, they all adopt a convolutional U-Net architecture as the de-facto choice of backbone."

**翻译**：扩散模型是近期图像生成的前沿，**但它们都默认用卷积 U-Net 当 backbone**。

### U-Net 是怎么来的、为什么该被质疑

> "The seminal work of Ho et al. first introduced the U-Net backbone for diffusion models... The model is convolutional, comprised primarily of ResNet blocks... the high-level design of the U-Net from Ho et al. has largely remained intact."

**翻译**：Ho 等人（DDPM）首次把 U-Net 引入扩散模型，它继承自 PixelCNN++，主体是卷积 ResNet block，低分辨率处穿插 self-attention。后续 Dhariwal & Nichol 改了一些（如用 adaptive norm 注入条件），**但 U-Net 的高层设计基本没动过。**

### 本文的目标与论断

> "With this work, we aim to demystify the significance of architectural choices in diffusion models and offer empirical baselines for future generative modeling research. We show that the U-Net inductive bias is not crucial to the performance of diffusion models, and they can be readily replaced with standard designs such as transformers."

🔥 **翻译（核心论断）**：本文旨在**祛魅"架构选择在扩散模型里的重要性"**。我们证明：**U-Net 的归纳偏置对扩散模型的性能不是关键，可以直接换成 Transformer 这种标准设计。**

> "As a result, diffusion models are well-poised to benefit from the recent trend of architecture unification—e.g., by inheriting best practices and training recipes from other domains, as well as retaining favorable properties like scalability, robustness and efficiency."

**翻译**：这样扩散模型就能享受"架构统一"的红利 —— 继承其它领域的最佳实践和训练配方，并保留**可扩展性、鲁棒性、效率**等好性质。

> "We call them Diffusion Transformers, or DiTs for short. DiTs adhere to the best practices of Vision Transformers (ViTs), which have been shown to scale more effectively for visual recognition than traditional convolutional networks."

**翻译**：我们称之为 **Diffusion Transformer (DiT)**，遵循 **ViT 的最佳实践**（ViT 在视觉识别上已被证明比卷积网络更易 scale）。

### 怎么验证 scaling

> "we study the scaling behavior of transformers with respect to network complexity vs. sample quality... there is a strong correlation between the network complexity (measured by Gflops) vs. sample quality (measured by FID). By simply scaling-up DiT and training an LDM with a high-capacity backbone (118.6 Gflops), we are able to achieve a state-of-the-art result of 2.27 FID."

**翻译**：在 **LDM 框架**下（扩散在 VAE latent 空间训练）构建并 benchmark DiT 设计空间，成功用 Transformer 替换 U-Net。证明 DiT 是可扩展架构：**网络复杂度（Gflops）和样本质量（FID）强相关**。简单把 DiT 放大到 118.6 Gflops，就拿到 SOTA FID 2.27。

---

## §2 Related Work（精简）

**Transformers**：已在语言、视觉（ViT）、RL、meta-learning 中替换领域专用架构，并展现出随模型/算力/数据增长的卓越 scaling 性质。在生成上，Transformer 被用于自回归预测像素、离散 codebook（自回归 / masked 生成）、以及 DDPM 里合成非空间数据（如 DALL·E 2 生成 CLIP embedding）。**本文研究的是：把 Transformer 当图像扩散模型的 backbone 时的 scaling 性质。**

**DDPM（去噪扩散概率模型）**：扩散 / score-based 生成模型在图像上很成功，常超过 GAN。近两年的提升主要来自更好的采样技术（如 **classifier-free guidance**）、把模型重参数化成预测噪声而非像素、以及级联 DDPM。**所有这些模型都默认用卷积 U-Net。** 有并行工作（Jabri 等）提出基于 attention 的高效架构，**而本文探索的是纯 Transformer。**

**架构复杂度的度量**：图像生成文献常用参数量衡量复杂度，**但参数量是糟糕的代理**（没算进分辨率等影响性能的因素）。本文用**理论 Gflops** 来分析复杂度，和架构设计文献对齐。

---

## 💡 §1-2 批注

- 这篇的写作姿态是"**祛魅 + 立基准**"：不是发明花哨结构，而是证明"最朴素的 Transformer 就够了"，并把它做成可复现的 scaling 基准。这种"做减法、立 baseline"的品味，是谢赛宁系工作（也包括后来的 ConvNeXt、其它代表作）的典型风格 —— 值得学习。
- "U-Net inductive bias is not crucial" 这句是论文的灵魂。它和 ViT 的"卷积归纳偏置非必需"是同一种精神，只是搬到了生成领域。
- 用 **Gflops 而非参数量**衡量复杂度，是后面所有 scaling 结论成立的前提（因为 patch size 改 token 数 → 改 Gflops 但不改参数）。

---

[← §0 Abstract](00-abstract.md) | [§3 Diffusion Transformers →](03-method.md)
