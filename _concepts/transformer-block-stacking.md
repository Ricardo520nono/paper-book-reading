[← 返回概念词典](README.md)

# Transformer Block 堆叠（"N×" 是什么意思）

## 🎯 一句话定义

**Transformer 类模型 = N 个长得一样但参数不同的 block 串成的"流水线"**。N 是网络深度（depth）。每个 block 有自己独立的参数，输入数据顺序流过 N 个 block，每过一个被 transform 一次。

---

## 🧠 核心澄清：N× 是空间堆叠，不是时间循环

```
朴素理解（错的）：               真实情况（对的）：
─────────────────────              ──────────────────────────
1 个 block 反复跑 N 次             N 个独立 block 串行连接
像 for loop                        每个 block 有自己的参数
共享同一组参数                     数据流过它们逐层被 transform
```

⚠️ **N× 在论文图里通常画在 block 的角落**，意思是 "**这个结构复制 N 份，串成流水线**"。

---

## 📐 完整 Transformer 架构 = Stacked Blocks

```
       输入
        ↓
   ┌─────────┐
   │ Patchify │   ← 入口预处理，1 次
   │ / Embed  │
   └────┬─────┘
        ↓
   ╔═════════╗   ← Block 1，自己的参数 W1
   ║         ║
   ╚════╤════╝
        ↓
   ╔═════════╗   ← Block 2，参数 W2（不同于 W1）
   ║         ║
   ╚════╤════╝
        ↓
       ...
        ↓
   ╔═════════╗   ← Block N，参数 WN
   ║         ║
   ╚════╤════╝
        ↓
   ┌─────────┐
   │Unpatchify│   ← 出口后处理
   │ / Project│
   └────┬─────┘
        ↓
       输出
```

**每个 block 有相同 shape，但完全独立的参数**。Figure 里画一个 block 加 N× 只是节省篇幅。

---

## 🔢 不同模型的 N（深度）

| 模型 | N (block 数) | D (hidden dim) | 总参数 |
|---|---|---|---|
| BERT-base | 12 | 768 | 110M |
| GPT-2 (small) | 12 | 768 | 117M |
| GPT-3 (175B) | 96 | 12288 | 175B |
| Llama 2 7B | 32 | 4096 | 7B |
| Llama 2 70B | 80 | 8192 | 70B |
| Stable Diffusion 1.5 (U-Net) | ~12 | 320 | 860M |
| **Wan 1.3B (估计)** | **~24** | **~2048** | **1.3B** |
| **Wan 14B (估计)** | **~32-40** | **~4096-5120** | **14B** |

**经验规律**：参数量 ≈ N × D² × 系数。

---

## 🤔 为什么要堆叠这么多 block？

### 单个 block 能做的事很有限
- 1 次 self-attention：每 token 看一眼其他 token
- 1 次 FFN：每 token 自己想一下
- 输出：input 的"略微优化版"

### N 个 block 串起来才有"深度"
- 每个 block 在前一个 block 的输出上**进一步精炼**
- token 经过 30 次"互相看 + 自己想" → 学到复杂规律
- 越深 → 越能学到复杂模式

**类比 1：编辑文章**
```
Block 1: 草稿 → 修一遍（大问题）
Block 2: → 再修（结构）
Block 3: → 再修（措辞）
...
Block N: → 最终稿
```

**类比 2：流水线工厂**
```
原材料 → [打磨] → [组装] → [喷漆] → [包装] → 成品
        Block1  Block2   Block3  Block4
```

---

## 🎛️ Block 内部 vs Block 之间的参数

以 Wan DiT block 为例：

| 模块 | 是否每个 block 独立 |
|---|---|
| **Self-Attention 的 Q/K/V/Output 矩阵** | ✅ 每 block 独立 |
| **Cross-Attention 的 Q/K/V/Output 矩阵** | ✅ 每 block 独立 |
| **FFN 的两层 Linear 权重** | ✅ 每 block 独立 |
| **LayerNorm 的 γ, β 学习参数** | ✅ 每 block 独立 |
| **Time MLP 的权重** | ❌ **N 个 block 共享 1 份**（Wan 的 25% 参数节省 trick） |
| **每个 block 注入 modulation 的 bias** | ✅ 每 block 独立 |

⚠️ 大多 Transformer 模型**所有参数都是 per-block 独立**。Wan 的 "shared time MLP" 是少见的优化技巧。

---

## 🔁 数据怎么流过这 N 个 block

```
经过 Block 1：
   token_0 = embed(input)        ← 入口
   token_1 = Block1(token_0)     ← Block 1 transform 一次

经过 Block 2：
   token_2 = Block2(token_1)     ← 在 token_1 基础上再 transform

经过 Block N：
   token_N = BlockN(token_{N-1}) ← 最终输出
```

每一步：
- **token_i 的 shape 不变**（保持 (B, L, D)）
- **token_i 的"语义信息"被精炼一次**

最后从 token_N 解码回最终输出。

---

## 🔑 在 DiT 里 N 个 block 的特殊之处

DiT 在每个 block 多接收两路条件：
- **文本条件**（通过 cross-attention，T-Tokens 注入到每个 block）
- **时间条件**（通过 time MLP 输出的 modulation params 注入到每个 block）

```
                    ┌─→ Block 1 ─┐
   T-Tokens   ──────┼─→ Block 2 ─┤
   (umT5 输出)      ├─→ ...     ─┤
                    └─→ Block N ─┘

                    ┌─→ Block 1 ─┐
   Time MLP   ──────┼─→ Block 2 ─┤  (但每 block 的 bias 不同)
   (1 个，共享)     ├─→ ...     ─┤
                    └─→ Block N ─┘
```

**所以 N 个 block 不是"机械重复"，而是"在外部条件指导下逐层精炼视频 latent"**。

---

## 🔗 在我读的论文里出现的位置

| Paper | Section | 用法 |
|---|---|---|
| Wan (2503.20314) | §4.2.1 | DiT block × N，Time MLP 共享 + bias 独立 |

---

## 🔗 相关概念

- [Encoder-Decoder](encoder-decoder.md) — block 堆叠是 Transformer 类架构的基础
- [Diffusion 基础](diffusion-basics.md) — Time MLP / Modulation 是 DiT 特有的"per-block 条件注入"
- [Normalization](normalization.md) — LayerNorm 是每个 block 内部反复用的稳定化层
