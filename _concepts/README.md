# 📚 概念词典 _concepts/

读 paper 时遇到的基础概念，沉淀在这里。**和某篇具体 paper 无关 / 跨篇通用**的知识都进这个目录。

## 索引

| 概念 | 一句话 | 文件 |
|---|---|---|
| **VAE** (Variational Autoencoder) | AI 学出来的压缩器；pixel ↔ latent 的桥梁 | [vae.md](vae.md) |
| **Encoder-Decoder 架构** | "先压再解"的设计模式；信息瓶颈 + 模块化 | [encoder-decoder.md](encoder-decoder.md) |
| **PSNR** (Peak Signal-to-Noise Ratio) | 重建质量打分，越高越好，单位 dB | [psnr.md](psnr.md) |
| **Normalization 层** | 把激活拉回标准状态防止数值爆炸；BN / LN / GN / RMSNorm 区别 | [normalization.md](normalization.md) |

## 待写

- [ ] DiT (Diffusion Transformer)
- [ ] Flow Matching
- [ ] Attention 的 Q / K / V
- [ ] Diffusion 基础（DDPM / 扩散过程）
- [ ] Cross-attention vs Self-attention
- [ ] Spatio-temporal Attention（Full vs Separated）
- [ ] LPIPS / SSIM / KL 散度（评估指标 / loss）
- [ ] Inflation（2D → 3D 权重扩展）

## 使用约定

- 每个概念**一个 .md 文件**，自包含解释
- 文件结构：一句话定义 → 详细展开 → 在我读的论文里出现的位置 → 相关概念链接
- 在 paper 的 section 批读里**第一次**遇到某概念时，**链接到这里**而不是重复解释
