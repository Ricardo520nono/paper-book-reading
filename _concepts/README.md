# 📚 概念词典 _concepts/

读 paper 时遇到的基础概念，沉淀在这里。**和某篇具体 paper 无关 / 跨篇通用**的知识都进这个目录。

## 索引

| 概念 | 一句话 | 文件 |
|---|---|---|
| **VAE** (Variational Autoencoder) | AI 学出来的压缩器；pixel ↔ latent 的桥梁 | [vae.md](vae.md) |
| **Encoder-Decoder 架构** | "先压再解"的设计模式；信息瓶颈 + 模块化 | [encoder-decoder.md](encoder-decoder.md) |
| **PSNR** (Peak Signal-to-Noise Ratio) | 重建质量打分，越高越好，单位 dB | [psnr.md](psnr.md) |
| **Normalization 层** | 把激活拉回标准状态防止数值爆炸；BN / LN / GN / RMSNorm 区别 | [normalization.md](normalization.md) |
| **Diffusion 基础** | Timestep / MLP / Modulation —— 读 DiT 章节必备 3 个基础 | [diffusion-basics.md](diffusion-basics.md) |
| **Transformer Block 堆叠** | "N×" 是什么意思；block 之间的参数关系 | [transformer-block-stacking.md](transformer-block-stacking.md) |
| **训练 vs 推理** | 架构相同；推理 = DiT 循环 50 次（这是 diffusion 慢的根因） | [training-vs-inference.md](training-vs-inference.md) |
| **Pre-training vs Post-training** | 学通识 vs 学品味；架构不变只换数据；起源 LLM 蔓延到所有 foundation model | [pretraining-vs-posttraining.md](pretraining-vs-posttraining.md) |
| **CLIP** | 把图像和文字嵌入同一向量空间的双编码器；视觉 AI 的通用理解器 | [clip.md](clip.md) |
| **AC-WM & Action Injection** | action 作为输入的 WM；架构层"能喂"≠ 训练层"能消化"；5 种主流注入方式 | [action-conditioned-wm.md](action-conditioned-wm.md) |
| **Cross-Attention** | "按需融合的查表机制"；Q 来自一边，K/V 来自另一边；多模态条件控制的标准件 | [cross-attention.md](cross-attention.md) |

## 待写

- [ ] DiT (Diffusion Transformer)
- [ ] Flow Matching
- [ ] Attention 的 Q / K / V
- [ ] Diffusion 完整数学（DDPM / forward-reverse / score function）
- [ ] Cross-attention vs Self-attention
- [ ] Spatio-temporal Attention（Full vs Separated）
- [ ] LPIPS / SSIM / KL 散度（评估指标 / loss）
- [ ] Inflation（2D → 3D 权重扩展）

## 使用约定

- 每个概念**一个 .md 文件**，自包含解释
- 文件结构：一句话定义 → 详细展开 → 在我读的论文里出现的位置 → 相关概念链接
- 在 paper 的 section 批读里**第一次**遇到某概念时，**链接到这里**而不是重复解释
