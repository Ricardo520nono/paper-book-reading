[← 返回概念词典](README.md)

# PSNR · Peak Signal-to-Noise Ratio（峰值信噪比）

## 🎯 一句话定义

**PSNR = 衡量"重建图/视频和原图有多像"的数值指标，越高越像，越高越好**。最常用于**有损压缩 / 重建任务**（VAE、JPEG、MP4 编码），单位 **dB（分贝）**。

---

## 📐 计算公式

$$
\text{PSNR} = 10 \cdot \log_{10}\left(\frac{\text{MAX}^2}{\text{MSE}}\right)
$$

- **MAX** = 像素最大可能值（8-bit 图像 = 255）
- **MSE** = 均方误差（mean squared error）

```
对每个像素位置 (i, j):
    diff(i, j) = original(i, j) - reconstructed(i, j)
    平方再求和再除以总像素数 = MSE
```

**直觉**：MSE 越小（重建越准），分母越小 → log 越大 → PSNR 越高。

---

## 📊 典型范围

| PSNR | 视觉效果 | 例子 |
|---|---|---|
| **> 40 dB** | 肉眼几乎无差别 ⭐⭐⭐ | 顶级 VAE / 高质量 JPEG |
| **30–40 dB** | 质量好，能看出微小差异 ⭐⭐ | 大多数 VAE / 普通 JPEG |
| **20–30 dB** | 可接受但有明显瑕疵 ⭐ | 低码率视频 / 激进压缩 |
| **< 20 dB** | 肉眼很明显的失真 | 严重压缩 |

---

## 🔍 在 Wan VAE 评测里（Figure 7）

```
Wan-VAE         ≈ 37 dB    ⭐ SOTA 第一梯队
HunyuanVideo    ≈ 37 dB    优秀（但慢）
CogVideoX       ≈ 36 dB    很好
CVVAE           ≈ 34 dB    好
SVD             ≈ 34 dB    好（压缩比不同）
Mochi           ≈ 31 dB    一般
Open Sora Plan  ≈ 31 dB    一般
Step Video      ≈ 30 dB    一般（参数最大反而垫底）
```

---

## 🤔 为什么是 log 尺度？

像素差异跨度大（1 到几千），log 让数字更可读：

> **PSNR 每 +10 dB ≈ MSE 降到 1/10**
>
> 30 → 40 dB 不是"提高 33%"，是**误差缩小 10 倍**，质的飞跃。

这也是为什么 SOTA 都在 35–40 dB 区间抢 1–2 dB —— 看着小，实际是数量级差异。

---

## ⚠️ PSNR 的 3 个坑

1. **平等对待所有像素差异** —— 但人眼对亮度变化比色调变化敏感得多
2. **抓不到结构性质量** —— 颜色对但人脸糊了，PSNR 可能仍高
3. **某些情况下高 PSNR 反而看起来糟** —— 过度模糊化也能拿高分

> **业界共识**：PSNR 是 **必看但不够看** 的指标，要搭配：
> - **LPIPS** —— 用预训练 CNN 测"感知相似度"，更接近人眼
> - **SSIM** —— 看结构信息保留度
> - **人类主观评分** —— 最贵但最真实

⚠️ **这就是 §4.1.4 里 Wan 不光报 PSNR**，还做了 **Figure 8 视觉对比**（texture/face/text/high-motion 4 类）—— 补 PSNR 测不到的感知质量。

---

## 🔗 在我读的论文里出现的位置

| Paper | Section | 用途 |
|---|---|---|
| Wan (2503.20314) | §4.1.4 Quantitative | Figure 7 比较 8 个 VAE 的 PSNR |
| Wan (2503.20314) | §4.1.4 Qualitative | Figure 8 视觉对比补 PSNR |

---

## 🔗 相关概念

- [VAE](vae.md) — PSNR 是评估 VAE 重建质量的主要指标
- _(待补) LPIPS — 感知相似度_
- _(待补) SSIM — 结构相似度_
