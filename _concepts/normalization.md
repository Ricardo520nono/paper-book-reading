[← 返回概念词典](README.md)

# Normalization 层（归一化）

## 🎯 一句话定义

**Normalization = 神经网络中"把激活拉回标准状态"的层**，避免训练时数值爆炸 / 消失。位置：通常在变换层（Conv / Linear / Attention）之后、非线性激活之前。

---

## 🤔 为什么需要？

深层网络训练时，**激活值会层层放大或缩小**，几十层下来变成 NaN/Inf 导致训练崩溃。Normalization 在每隔几层后**把激活拉回 mean ≈ 0、std ≈ 1**，让训练稳定。

---

## 📐 基本配方（4 步）

```
(1) 算激活的均值 μ 和标准差 σ
(2) 减均值: x - μ              ← 中心化
(3) 除以标准差: (x - μ) / σ     ← 标准化
(4) 加可学习的 γ, β: γ · y + β  ← 让模型决定要不要反归一化
```

---

## 📊 5 大类型 —— 区别就一句话："在哪个轴上算 mean/std"

对 4D tensor `(B, C, H, W)`：

| Norm 类型 | 沿哪些轴算 | 直觉 | 代表场景 |
|---|---|---|---|
| **BatchNorm** | B, H, W（每 channel 独立） | "这一批 N 个样本同一 channel 一起算" | 大 batch CNN（ResNet） |
| **LayerNorm** | C, H, W（每 sample 独立） | "一张图 / 一个 token 内部所有特征一起算" | Transformer / NLP |
| **InstanceNorm** | H, W（每 sample × 每 channel 独立） | "一张图的一个 channel 当一个整体算" | 风格迁移 |
| **GroupNorm** | C 分 G 组，组内算 H, W | "折中的 LayerNorm" | 小 batch CNN / 视频 VAE 老路线 |
| **RMSNorm** | LayerNorm 简化（只算 RMS，不减均值） | "只 scale 不 center，更轻" | Llama / Wan-VAE / 现代 LLM |

---

## 🧱 它们的差别图示

```
4D tensor: (B=4, C=6, H, W)

BatchNorm:                          LayerNorm:
┌─────┬─────┬─────┬─────┐           ┌─────┬─────┬─────┬─────┐
│  B0 │  B1 │  B2 │  B3 │           │  B0 │  B1 │  B2 │  B3 │
│ ████│ ████│ ████│ ████│ <─ 同     │ ▒▒▒▒│     │     │     │ <─ 一个
│  C0 │  C0 │  C0 │  C0 │  channel  │ All │     │     │     │  sample
│ 一起│ 一起│ 一起│ 一起│  一起算   │ chan│     │     │     │  内部
└─────┴─────┴─────┴─────┘           └─────┴─────┴─────┴─────┘
                                       一起算

GroupNorm (G=2):                    RMSNorm:
┌─────┬─────┬─────┬─────┐           像 LayerNorm，但：
│  B0 │     │     │     │           - 不算均值
│ ▒▒▒ │     │     │     │ <─ C 分组   - 不减均值  
│ C0,1│     │     │     │   一组算    - 只除以 √(mean(x²))
│ ▓▓▓ │     │     │     │             更快、参数更少
│ C2,3│     │     │     │
└─────┴─────┴─────┴─────┘
```

---

## 🎬 不同场景适合什么

| 场景 | 推荐 Norm | 为什么 |
|---|---|---|
| **大 batch 图像分类** | BatchNorm | batch 大统计准确 |
| **小 batch / 视频 VAE 老路线** | GroupNorm | 不依赖 batch 维 |
| **Transformer / LLM** | LayerNorm 或 **RMSNorm** | 序列长度变，沿 feature 维稳定 |
| **风格迁移** | InstanceNorm | 每张图独立处理 |
| **现代视频生成模型（Wan-VAE）** | **RMSNorm** | 简单 + 跟 feature cache 友好 |

⚠️ **关键约束**：在视频 / 视频 VAE 这种 **batch 小、有时序依赖** 的场景，**BatchNorm 基本不能用**（batch 维不稳 + 跨样本统计破坏因果）。所以视频 VAE 选项基本是 **GroupNorm vs LayerNorm vs RMSNorm**。

---

## 🧪 RMSNorm 比 LayerNorm 简化在哪？

```
LayerNorm:                          RMSNorm:
1. 算均值 μ                          1. 跳过
2. 减均值: x - μ                     2. 跳过
3. 算 std σ = √(mean((x-μ)²))        3. 算 RMS = √(mean(x²))
4. 除以 σ                            4. 除以 RMS
5. γ · y + β                         5. γ · y  (β 经常省略)
```

**核心简化**：去掉了 mean subtraction 那一步。研究（Zhang & Sennrich 2019）发现这一步**贡献很小**，但去掉后：
- 计算更快（省一次统计 + 减法）
- 实现更简单
- 跨 batch / 跨时间 / 跨 chunk 时**更容易保持因果**（因为没有"跨整体算均值"这件事）
- 性能几乎不掉

**Llama 系列、SD3、Wan-VAE 都已经切到 RMSNorm**。

---

## 🔗 在我读的论文里出现的位置

| Paper | Section | 用法 |
|---|---|---|
| Wan (2503.20314) | §4.1.1 Model Design | 把 GroupNorm 替换成 RMSNorm 来 preserve temporal causality + 配合 feature cache |

---

## 🔗 相关概念

- [VAE](vae.md) — Wan-VAE 用了 RMSNorm
- _(待补) Feature Cache — RMSNorm 让 cache 跨 chunk 更顺_
- _(待补) Transformer 架构 — LayerNorm / RMSNorm 是标配_
