[← 返回概念词典](README.md)

# 训练 vs 推理 ——（特别是 Diffusion 模型）

## 🎯 核心答案

| 维度 | 训练 (training) | 推理 (inference) |
|---|---|---|
| **架构** | 一样的（Figure 10 那个 block） | 一样的 |
| **参数** | **正在被更新** | **冻结固定** |
| **DiT forward 次数** | **1 次 / 样本** | **~50 次 / 视频**（迭代） |
| **起点** | 真实视频 → 加噪声 | 纯噪声 |
| **输出怎么用** | 算 loss → backprop | 更新 x 走向清晰 |
| **梯度** | 算 + 反传 | 不算 |

✅ "训练学参数，推理用参数" 这个直觉是**完全对的**。
⚠️ 但**数据流向**不一样 —— diffusion 推理是个**循环 50 次的过程**，这是它和 LLM 推理的关键区别。

---

## Side-by-side：训练 vs 推理 一步走流程（Wan / Flow Matching 为例）

### 训练 (one batch step)

```
1. 数据集抽一对 (真实视频, prompt)
2. VAE Encoder: 真实视频 → x_1 (clean latent)
3. 随机采样:
     x_0 ~ N(0, I) (纯噪声)
     t ~ logit-normal
4. 算 x_t = t·x_1 + (1-t)·x_0  (半成品)
   算 v_t = x_1 - x_0  (GT 速度)
5. DiT forward: pred_v = DiT(x_t, prompt, t)
6. Loss = MSE(pred_v, v_t)
7. Backprop → 更新 DiT 参数
   (重复 10⁹ 次)
```

### 推理 (生成 1 个视频)

```
1. 用户给 prompt (+ 图 for I2V)
2. T2V: x_init = N(0, I)
   I2V: VAE encode 输入图作为锚点
3. for step in range(50):
     pred_v = DiT(current_x, prompt, current_t)
     current_x += step_size · pred_v
     current_t += step_size
4. VAE Decoder: final_x → 视频
```

---

## 关键洞察：DiT forward 次数差 50 倍

| 阶段 | 单位 | DiT forward 次数 |
|---|---|---|
| 训练（按样本算） | 1 次梯度更新 | **1 次** |
| 推理（按视频算） | 1 个完整视频 | **~50 次** |

**这是 diffusion 推理慢的根本原因**：不是模型大，而是循环多。

```
LLM 生成 1 段回答:                    Diffusion 生成 1 个视频:
─────────────────────                 ─────────────────────
N 次 forward (生成 N tokens)          50 次 forward (迭代去噪)
但每次只生成 1 个 token               每次处理整个视频 latent (~6 万 token)
```

---

## 训练时 / 推理时 各组件的角色

| 组件 | 训练时 | 推理时 |
|---|---|---|
| **VAE Encoder** | 跑（GT 视频 → x_1） | T2V: 不跑；I2V: 跑 1 次（输入图） |
| **VAE Decoder** | **不跑**（DiT 在 latent 学就够了） | **必跑**（latent → 最终视频） |
| **umT5 (文本编码)** | 跑（prompt → token），冻结 | 跑（prompt → token），冻结 |
| **DiT** | 1 forward / 样本，**更新参数** | 50 forward / 视频，**冻结** |

⚠️ 新手坑：**训练时 VAE Decoder 不跑**。模型只在 latent 空间学。

---

## 推理慢的工程后果

50 次 DiT forward 串行 → **必须做工程优化才能商业化**：
- **Distillation**（蒸馏）：把 50 步压到 4 步（牺牲质量换速度）
- **Quantization**（量化）：FP16 / INT8 → 算得快
- **Parallelism**（并行）：多 GPU 拆分 attention
- **Cache**（缓存）：相邻 step 的中间结果复用

这就是 Wan paper §4.3 §4.4 整两章存在的意义。

---

## 🔗 在我读的论文里

| Paper | Section | 用法 |
|---|---|---|
| Wan (2503.20314) | §4.2 训练，§4.4 推理 | 训练用 Flow Matching，推理 50 步 + 大量工程优化 |

---

## 🔗 相关概念

- [Diffusion 基础](diffusion-basics.md) — Timestep 在训练和推理时怎么用的
- [VAE](vae.md) — 训练 / 推理时 VAE 的角色不同
- [Encoder-Decoder](encoder-decoder.md)
- _(待补) Flow Matching 数学完整版_
- _(待补) Sampling Schedule（采样步数怎么排）_
