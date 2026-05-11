[← 返回概念词典](README.md)

# Cross-Attention（交叉注意力）

> **一句话定义**：
> Cross-attention = **一个序列（Query）按需查询另一个序列（Key, Value）的信息，用相关性权重做加权融合**。
> Self-attention 是"自己查自己"，cross-attention 是"自己查别人"。

## 📌 速查

| 关键词 | 一句话 |
|---|---|
| **Attention** | "按需融合"的查表机制：query 看所有 key 算相关性 → 加权求和 value |
| **Q (Query)** | "我想问什么？" |
| **K (Key)** | "我有什么标题可以匹配？" |
| **V (Value)** | "标题对应的实际内容" |
| **Self-Attention** | Q/K/V 来自同一个 sequence —— 自己看自己 |
| **Cross-Attention** | Q 来自一个 sequence，K/V 来自另一个 sequence —— 一边问一边答 |

---

## 1. 直觉：进图书馆找书

想象你想写关于 "climate change" 的论文，进了图书馆：

| 角色 | 现实里是什么 |
|---|---|
| **Query (Q)** | 你脑子里的问题："climate change 是怎么影响海平面的？" |
| **Key (K)** | 每本书**书脊上的标题** |
| **Value (V)** | 每本书**里面的内容** |

**你做了什么？**
1. 拿着 Query 走过书架
2. 扫每本书的 Key（标题），打个**相关性分数**（"这本 0.9 相关 / 那本 0.05"）
3. 把所有分数归一化（softmax）→ "注意力权重"，加起来 = 1
4. 按权重对所有书的 Value（内容）做**加权求和** → 融合成你需要的答案

**这就是 attention 的全部直觉**：
> 给定一个 Query，看所有 Key 算相关性 → 用相关性权重对 Value 做加权求和 → 得到一个"按需融合"的输出。

---

## 2. 数学：就 3 行

设：
- $Q \in \mathbb{R}^{d}$：1 个 query
- $K \in \mathbb{R}^{n \times d}$：n 个 key（每个 d 维）
- $V \in \mathbb{R}^{n \times d}$：n 个 value（每个 d 维）

**第 1 行**：算相关性分数
$$\text{scores} = Q K^T \quad \text{(尺寸 1×n)}$$

**第 2 行**：归一化
$$\text{weights} = \text{softmax}(\text{scores}/\sqrt{d}) \quad \text{(尺寸 1×n，加起来=1)}$$

（除以 $\sqrt{d}$ 是为了数值稳定，叫 "scaled dot-product attention"。可以暂时忽略这个细节。）

**第 3 行**：加权求和
$$\text{output} = \text{weights} \cdot V \quad \text{(尺寸 1×d)}$$

**就这样**。没有别的。

如果有 m 个 query（不是 1 个），Q 变成 $\mathbb{R}^{m \times d}$，输出也变成 $\mathbb{R}^{m \times d}$，每个 query 独立算。

---

## 3. Self-Attention vs Cross-Attention 区别

**唯一区别：Q、K、V 来自哪里**。

### Self-Attention（自注意力）

**Q、K、V 来自同一个 input**。

例子：你读句子 "I love cats"
- 这 3 个词同时是 query 和 key/value
- "I" 这个词的 Q 去看 "I/love/cats" 的所有 K，融合出新的 "I" 表示
- 每个词同时看其他所有词，把上下文融合进来

**用处**：BERT / GPT / LLaMA 等所有 LLM 的基础。Transformer 主干就是 N 层 self-attention 堆起来。

### Cross-Attention（交叉注意力）⭐

**Q 来自一个 input，K、V 来自另一个 input**。

经典例子：**机器翻译**
- 把 "I love cats" 翻译成 "J'aime les chats"
- Decoder（法语侧）生成 "chats" 这个词时：
  - **Q** = decoder 当前要生成的位置（French side）
  - **K, V** = encoder 的英文 token 表示（English side）
- "chats" 的 Q 去看英文那一侧所有 K，发现 "cats" 最相关
- 把 "cats" 的 V 拿过来融合

**直觉**：**一边在"问"，另一边在"答"**。两边可以是完全不同的模态。

---

## 4. Cross-Attention 的常见用法（多模态融合的标准件）

| 任务 | Q 来自 | K, V 来自 |
|---|---|---|
| 机器翻译 | Decoder（目标语言）| Encoder（源语言）|
| Text-to-Image（如 Stable Diffusion）| Image latent token | Text token（CLIP 出来的）|
| Image Captioning | Decoder（caption 文字）| Encoder（图像 feature）|
| Video Generation | Visual token | Text / Action / Camera pose embedding |
| **Ctrl-World** | Visual token of frame t | Pose token of frame t |

→ **要做多模态条件控制，cross-attention 几乎是标配**。

---

## 5. 回到 Ctrl-World 的 Frame-Level Cross-Attention

现在再看 Ctrl-World 那张架构图就清楚了：

| 角色 | Ctrl-World 里是什么 |
|---|---|
| **Q**（"问"端）| 上排 visual tokens —— "我这一帧画面要长成什么样？" |
| **K, V**（"答"端）| 下排 pose tokens —— "这一帧的 action / pose 是 [x, y, z, θ, ...]" |

每个 visual token 拿着自己的 Q 去问 pose token："你那一帧打算让机械臂去哪？" → 拿到 pose 信息后融合进自己 → 生成对应的画面。

### "Frame-Level" 加的特殊约束

普通 cross-attention：上排**每个 Q** 看下排**所有 K**。

Ctrl-World 的 frame-level cross-attention：上排第 t 帧的 Q **只能看**下排第 t 帧的 K。**其他帧的 K 被 mask 掉**。

→ 强制"问"和"答"一一对应到**同一时刻**，保证 visual 和 action 在时间上严格对齐。

**这种"只允许某些位置相互看"的限制叫 attention mask**。Frame-level 实际上是一种 block-diagonal 的 attention mask（每个时间块内允许 attention，跨时间块的位置被屏蔽）。

---

## 6. 常见追问

### Q：Self-attention 和 cross-attention 哪个先有的？
A：Self-attention 出现在 2017 "Attention is All You Need" 论文，cross-attention 同时出现（encoder-decoder 架构里 decoder 通过 cross-attention 看 encoder）。两者一直配套使用。

### Q：为什么 cross-attention 用来做多模态融合很合适？
A：因为 Q 和 K/V 来自不同 sequence，**意味着它们维度、含义、长度都可以不同**。文本 token、图像 token、pose token 都可以做 K/V，只要 project 到同一个 hidden dim。

### Q：Cross-attention 计算量怎么样？
A：标准复杂度 $O(m \times n \times d)$，其中 m = query 数，n = key 数，d = 隐藏维度。**当 K/V 序列很长时（比如长视频）会变贵**。Frame-level 把 n 限制到每个 frame 内（1 个 pose token），计算量大大降低。

### Q：怎么知道一个 paper 用的是 self- 还是 cross-attention？
A：看架构图里 attention 模块的输入箭头。
- 一个箭头进 = self-attention（同一来源拆 Q/K/V）
- 两个箭头进 = cross-attention（一个来源做 Q，另一个做 K/V）

---

## 7. 在 Ricardo 的论文里出现的位置

| 论文 | 哪一节 | 用法 |
|---|---|---|
| **Wan** | DiT block 内部 | Self-attention + Cross-attention 配套：self 处理 video latent 自己之间，cross 让 latent attend 到 text |
| **Ctrl-World** | §4.1 Frame-level Action Conditioning + Figure 2 右侧 | Frame-level cross-attention：visual Q ↔ pose K/V，带 mask |
| **Uni-WAM proposal** | Model architecture · Shared Attention | 提到"前 N 层 Shared Attention（QKV 共享）"—— 两个分支共享 self-attention 的 QKV 投影矩阵 |

---

## 8. 相关概念链接

- [AC-WM & Action Injection](action-conditioned-wm.md) —— Cross-attention 是 action injection 的常见方式（方式 2）
- [Transformer Block 堆叠](transformer-block-stacking.md) —— Attention 是 transformer block 的核心子模块
- [Encoder-Decoder 架构](encoder-decoder.md) —— Cross-attention 是 encoder-decoder 之间的桥梁

---

## 9. 一句话存档（送给读者）

> **Attention = 按需融合的查表机制**。Self-attention 自己查自己，cross-attention 一个 sequence 查另一个 sequence。**几乎所有多模态条件控制（文本→图像、action→视频、语言→机械臂）都用 cross-attention 实现**。
