[← 返回概念词典](README.md)

# Diffusion 基础（Timestep / MLP / Modulation）

> 本文聚焦"读 DiT 章节必须先懂的 3 个基础"。完整 Diffusion 数学（forward/reverse process / score function / DDPM 等）以后单独写。

## 🎯 一句话定义（三连）

| 概念 | 一句话 |
|---|---|
| **Diffusion 推理** | "从纯噪声开始，迭代地一步步去噪，最终得到清晰视频" |
| **Timestep `t`** | 告诉模型"你现在在去噪过程的哪一步"的标量数字 |
| **MLP for time** | 把标量 `t` 翻译成"控制 block 行为的 modulation 参数" |
| **Modulation** | 用 timestep 的 modulation 参数对 LayerNorm 后的激活做 scale + shift |

---

## Layer 0 · Diffusion 推理流程

```
   step T (开始)       step T-1           step T-2    ...    step 0 (结束)
   ┌────────┐          ┌────────┐         ┌────────┐         ┌────────┐
   │ 纯噪声 │   ──>    │ 略好点 │   ──>   │ 隐约像 │   ...   │ 清晰   │
   │  x_0   │          │        │         │        │         │  x_1   │
   └────────┘          └────────┘         └────────┘         └────────┘
        │                  │                  │                  │
        ↑                  ↑                  ↑                  ↑
    模型预测            模型预测           模型预测            完工
    "怎么去噪"          "怎么去噪"         "怎么去噪"
```

**关键点**：模型在每一步都**做同样的"如何去噪"任务**，但**任务难度和侧重点随 t 变化**。

---

## Layer 1 · Timestep —— "现在是哪一步去噪"

```
t = 0  ───────────────────────────────────────  t = 1
└─ 噪声端                                            └─ 清晰端

   ▲              ▲              ▲
   │              │              │
 早期步         中期步          晚期步
 任务：建立大       任务：调整结构    任务：精修细节
 块布局（"画面有     ("熊猫的姿势    （"熊猫脚趾的
  熊猫和纸板"）     ")              几根毛"）
```

**为什么模型必须知道 t**：
- **早期 t**（高噪声）：只能粗略判断 → 任务是搭骨架
- **中期 t**：画面有结构了 → 任务是调整布局
- **晚期 t**（低噪声）：只剩细节 → 任务是精修

**没有 t**，模型分不清"该搭骨架还是该精修"。

**类比**：装修房子，第 1 天该拆墙，第 30 天该刷油漆。Timestep 就是 diffusion 模型的"天数计数器"。

⚠️ **t 的方向约定不同论文不一样**：
- **Flow Matching / Wan**：t ∈ [0, 1]，**t=0 = 噪声**，**t=1 = 清晰**（往大 = 走向清晰）
- **DDPM 经典**：t ∈ {0, ..., 1000}，**t=0 = 清晰**，**t=T = 噪声**（往大 = 走向噪声）

读 paper 先确认方向，否则意思搞反。

---

## Layer 2 · MLP 是什么

**MLP = Multi-Layer Perceptron（多层感知机）= 最基础前馈神经网络**：

```
输入 ──> Linear ──> Activation ──> Linear ──> Activation ──> 输出
       (矩阵乘法)  (非线性，如    (矩阵乘法)
                   ReLU/SiLU/GELU)
```

**两个核心元件**：
- **Linear（线性层 / 全连接层 / Dense）**：`y = Wx + b`
- **Activation（激活函数）**：非线性化（否则两个 Linear 等于一个 Linear）

**类比**：通用格式翻译器。给它一种输入格式 + 期望输出，它学一组矩阵能完成这个映射。

---

## Layer 3 · MLP 在 DiT 里的具体作用 —— Time Conditioning

```
       Timestep (一个数字 t = 0.5)
              ↓
          ┌───────┐
          │  MLP  │   ← 学习 t -> modulation params 的映射
          └───┬───┘
              │
              ▼
     [α1, β1, α2, β2, α3, β3]   ← 6 个 modulation params
              │
              ▼
       注入到 block 内 3 个 LayerNorm 各 2 个
```

每个 LayerNorm 后做调制：

```
普通 LayerNorm:                AdaLN (Wan 用的):
y = LN(x)                       y = α · LN(x) + β
                                     ↑      ↑
                              scale     shift  (来自 timestep)
```

**为什么这么干**：
- 不同 t 时 block 应有不同行为（早期 vs 晚期任务不同）
- 不能给每个 t 训一个独立模型（太贵）
- 折中：**block 参数固定 + timestep 控制 modulation**
- 同一组 block 参数被 timestep 调制成"相对 t 的不同行为"

**类比**：钢琴（block）键位永远 88 个，但**踏板**（modulation）改变发声效果。Timestep 是 DiT 的"踏板"。

⚠️ **6 = 3 × 2**：block 内 3 个 LayerNorm × 每个 2 个 (scale + shift) = 6。

⚠️ **完整版 AdaLN-zero (Peebles & Xie 2023)**：除了 scale + shift 还有 gate (γ)，是 9 个参数（3 × 3）。Wan 简化为 6 (3 × 2)，省一组参数。

---

## Wan 的进一步优化：Shared MLP

> "This MLP is shared across all transformer blocks, with each block learning a distinct set of biases."

```
方案 A（朴素）           方案 B（Wan 选）
──────────────────────────────────────────────
N 个 block，N 个 MLP    N 个 block 共享 1 个 MLP
                        每个 block 学独立 bias
──────────────────────────────────────────────
N × MLP 参数            1 × MLP + N × bias
                        ≈ 25% 参数减少 + 性能提升
```

**为什么共享 MLP 反而提升**：
1. **强制 timestep 表征一致**：所有 block 看到同样的"时间编码"，协同更顺
2. **隐式正则化**：参数被多 block 复用，必须学"通用"特征，不容易过拟合
3. **减少冗余学习**：N 个 MLP 各自学一遍同一件事是浪费

---

## 🔗 在我读的论文里出现的位置

| Paper | Section | 用法 |
|---|---|---|
| Wan (2503.20314) | §4.2.1 | DiT block + 6 modulation params + shared MLP for time |

---

## 🔗 相关概念

- [VAE](vae.md) — pixel ↔ latent 的桥梁
- [Encoder-Decoder](encoder-decoder.md) — 通用模式
- [Normalization](normalization.md) — LayerNorm 是 modulation 的作用对象
- _(待补) Flow Matching — diffusion 训练目标_
- _(待补) Cross-Attention vs Self-Attention_
