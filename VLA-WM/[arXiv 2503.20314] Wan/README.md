# Wan: Open and Advanced Large-Scale Video Generative Models

**作者**: Wan Team, Alibaba Group
**链接**: [arXiv 2503.20314](https://arxiv.org/abs/2503.20314) · [GitHub Wan2.1](https://github.com/Wan-Video/Wan2.1) · [GitHub Wan2.2](https://github.com/Wan-Video/Wan2.2)
**注意**: Wan 2.2 没有独立 arXiv 论文 —— 它的 GitHub README 在 Citation 里引用的就是这篇 2503.20314。Wan 2.2 的增量（MoE + TI2V-5B）我们整理在 [`sections/99-wan22-delta.md`](sections/99-wan22-delta.md)。

---

## 📌 一句话总结

Wan 是阿里通义实验室的全开源视频生成基座，提供 1.3B（消费级 GPU 友好）和 14B（性能旗舰）两个尺寸，基于 DiT + 自研时空 VAE，覆盖 T2V / I2V / 视频编辑 / T2I / 个性化 / 相机控制 / 实时生成 / 音频生成 八种下游任务。

---

## 🎯 学习目标（明天组会拷打 ready）

读完这篇 + Wan 2.2 增量后，我必须能流利回答这两题：

### Q1: Wan 2.1 和 Wan 2.2 的区别是什么？
> 我应该能讲清楚 4 个层面的差别：架构、模型规模、数据、能力扩展。
> 答案在 [`sections/99-wan22-delta.md`](sections/99-wan22-delta.md) —— 读完后我应该能不看文档讲出来。

### Q2: TI2V-5B 和 I2V-14B 的区别是什么？
> 我应该能讲清楚：参数规模、是否 MoE、VAE 压缩比、支持任务、运行条件、应用场景。
> 答案分布在论文 Section 5.1（I2V）+ Wan 2.2 README delta（TI2V-5B）。

---

## 📖 批读导航（按论文章节顺序）

按部就班，慢慢来比较快。每节 4 步循环：批读 → 我读 → 拷打 → Q&A 回写。

| # | Section | 内容 | 重要性 | 状态 |
|---|---|---|---|---|
| 0 | [00-abstract.md](sections/00-abstract.md) | 摘要：Wan 是什么，4 个 key features | 🔥 必读 | ✅ 已完成（Round 1 拷打 done） |
| 1 | [01-introduction.md](sections/01-introduction.md) | 三大 Gap + Wan 解法 + 双尺寸 + Openness | 🔥 必读 | ✅ 已完成（Round 2 拷打 done） |
| 2 | [02-related-work.md](sections/02-related-work.md) | T2V/I2V 相关工作 | 选读 | ⏳ 待写 |
| 3 | [03-data-pipeline.md](sections/03-data-pipeline.md) | 数据策划、密集 caption | ⭐ 重点 | ⏳ 待写 |
| 4.1 | [04-method-vae.md](sections/04-method-vae.md) | 时空 VAE 架构 | 🔥 必读 | ⏳ 待写 |
| 4.2 | [04-method-dit.md](sections/04-method-dit.md) | DiT 视频扩散模型 + 训练 | 🔥 必读 | ⏳ 待写 |
| 4.3-4.4 | [04-scaling-inference.md](sections/04-scaling-inference.md) | 训练效率 / 推理优化 | ⭐ 重点 | ⏳ 待写 |
| 4.5-4.7 | [04-prompt-eval.md](sections/04-prompt-eval.md) | Prompt 对齐 / 评测 | 选读 | ⏳ 待写 |
| 5.1 | [05-i2v.md](sections/05-i2v.md) | Image-to-Video（贴近你的使用场景） | 🔥 必读 | ⏳ 待写 |
| 5.2-5.7 | [05-other-applications.md](sections/05-other-applications.md) | 视频编辑 / T2I / 个性化 / 相机 / 实时 / 音频 | 选读 | ⏳ 待写 |
| 6 | [06-conclusion.md](sections/06-conclusion.md) | 局限与结论 | ⭐ 重点 | ⏳ 待写 |
| **99** | [99-wan22-delta.md](sections/99-wan22-delta.md) | **Wan 2.2 vs 2.1 增量**（提前写好的延伸） | 🔥 必读 | 🟡 已批读，等读到时再拷打 |

---

## 🔑 核心贡献（先记这个）

1. **新型时空 VAE**：4×8×8 压缩比，替代 SD3 的 VAE，专为视频设计
2. **可扩展预训练策略**：从 14B 模型展示视频生成的 scaling laws
3. **大规模数据策划**：包含密集视频 caption pipeline
4. **自动化评测指标**：自建 Wan-Bench
5. **8 个下游任务**：T2V / I2V / 视频编辑 / T2I / 视频个性化 / 相机控制 / 实时生成 / 音频生成
6. **首个支持中英文视觉文本**的视频生成模型
7. **1.3B 模型仅需 8.19 GB VRAM**：消费级 GPU 友好

---

## 📊 关键数字

| 指标 | 值 |
|---|---|
| Wan 2.1 模型规模 | 1.3B / 14B |
| Wan 2.1 VAE 压缩比 | 4×8×8（时间×高×宽） |
| 1.3B 模型 VRAM | 8.19 GB |
| 14B 模型 Wan-Bench score | 0.72（同期最高） |
| 训练数据 | 数十亿图像 + 视频 |

（Wan 2.2 的关键数字见 99-wan22-delta.md）

---

## 🔥 拷打记录

每读完一个 section 都会在这里追加 Q&A 摘要。

_(待开始)_
