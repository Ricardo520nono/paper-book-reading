[← 返回概念词典](README.md)

# CLIP · Contrastive Language-Image Pre-training

## 🎯 一句话定义

**CLIP = 把图像和文字嵌入到同一个语义空间的双编码器系统**（OpenAI 2021）。包含 Image Encoder 和 Text Encoder，输出的向量可以**直接比较图文相似度**。是当代视觉 AI 的通用理解基座。

---

## 📐 整体结构

```
                     CLIP
        ┌─────────────────────────┐
        │                         │
图像 ──>│   Image Encoder         │── image embedding (~768 维向量)
        │   (ViT 或 ResNet)        │
        │                         │
文本 ──>│   Text Encoder          │── text embedding (~768 维向量)
        │   (Transformer)         │
        │                         │
        └─────────────────────────┘

       两个 encoder 输出在同一个向量空间
       → 可以直接算 "图像 ↔ 文字" 的余弦相似度
```

---

## 🤔 CLIP 为什么是里程碑

### 老范式：监督式分类
- 标签是固定类别（"猫"/"狗"/...）
- 模型学：图像 → 类别概率
- 缺点：**只认训练见过的类别**，新类别要重新标 + 训

### CLIP 的对比学习范式
- 数据：**4 亿对 (图像, 文本)** 从互联网爬来（不是标签，是自然语言描述）
- 训练目标：拉近"匹配图文"的向量距离，推开"不匹配"的
- 结果：图像和文字嵌入到**同一个向量空间**

### 关键能力：零样本分类

```
给 CLIP 没见过的"独角鲸"照片
+ 候选文字: ["独角鲸", "海豚", "鲸鱼", "鲨鱼"]

CLIP 算每张图-文字相似度
→ 选最高的 → "独角鲸"
（不需要单独标过独角鲸数据）
```

---

## 🧠 CLIP Image Encoder 具体输出什么

```
输入: 图像 (H × W × 3)
       ↓
   Image Encoder
       ↓
输出: 一个向量 (~768 维)  ← 图像的"语义指纹"
```

**这个向量编码了**：
- 图里有什么（物体、场景）
- 整体氛围（明亮 / 阴沉 / 复古...）
- 构图风格（中心 / 全景...）

**它不**编码：
- 像素细节（纹理、颜色微调...）
- 精确空间位置（这一点和 VAE latent 不同！）

⚠️ **关键差别**：CLIP feature 是 **global/holistic**（全局浓缩），VAE latent 是 **spatial**（保留空间结构）。

---

## 📊 常见版本

| 版本 | 参数 | 输出维 | 速度 | 用法 |
|---|---|---|---|---|
| ViT-B/32 | ~150M | 512 | 快 | 入门 / 玩具 |
| ViT-B/16 | ~150M | 512 | 中 | 中等 |
| **ViT-L/14** ⭐ | **~430M** | **768** | 中慢 | **生成模型最常用**（SD / Wan / Hunyuan） |
| ViT-L/14@336px | ~430M | 768 | 慢 | 高分辨率版 |
| ViT-H/14 | ~1B | 1024 | 慢 | 高质量 |
| ViT-G/14 | ~2B | 1280 | 很慢 | SOTA |

⚠️ Wan paper 没明说，**大概率 ViT-L/14**（业界生成模型默认）。

---

## 🎬 在生成模型里的 3 个常见用法

### ① Text Encoder（文生图 / 文生视频）

```
prompt: "一只蓝色的鲸鱼" → CLIP Text Encoder → 文本向量 → 注入扩散模型
```

代表：Stable Diffusion 1.x（用 CLIP text encoder），SD3（换成 T5）

### ② Image Encoder（I2V / 个性化）

```
输入图 → CLIP Image Encoder → 图像向量 → 作为 global context 注入 DiT
```

代表：**Wan-I2V** ✓，I2VGen-XL，IP-Adapter

### ③ 两端齐用（图文检索）

算图和文字相似度做跨模态搜索 / 评测（CLIP Score 指标）

---

## 🔗 在 Wan-I2V 里 CLIP 的角色

```
                       输入图（first frame）
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
        Wan-Encoder (VAE)               CLIP Image Encoder
        ─────────────────               ────────────────────
        4D 张量 (16 × t × h × w)         1D 向量 (~768)
        保留空间结构                      高级语义浓缩
        Channel-concat 进 DiT             Cross-attention 注入 DiT
        "图怎么长的"（构图细节）          "图是什么"（主题/场景）
```

**两路互补，缺一不可**：
- 缺 VAE：DiT 不知道首帧像素长啥样 → 视觉细节崩
- 缺 CLIP：DiT 不知道首帧"主题"是啥 → 生成跑偏

⚠️ **同一张图过两个 encoder ≠ 冗余**，是**两种不同信号**。

---

## 🔗 在我读的论文里出现的位置

| Paper | Section | 用法 |
|---|---|---|
| Wan (2503.20314) | §5.1 I2V | Image encoder 抽全局语义 → decoupled cross-attn → DiT |

---

## 🔗 相关概念

- [Encoder-Decoder](encoder-decoder.md) — CLIP 是个特殊的 encoder（不是经典的 enc-dec）
- [VAE](vae.md) — VAE encoder 和 CLIP image encoder 在 Wan-I2V 里互补
- _(待补) ViT (Vision Transformer)_
- _(待补) 对比学习 (Contrastive Learning)_
