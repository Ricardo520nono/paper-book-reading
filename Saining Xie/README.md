# Saining Xie 论文精读系列

> **谢赛宁（Saining Xie）是我（Ricardo）的目标导师** —— 我希望进入他在 NYU 的组里做 RA。
> 这个文件夹专门收录他的重要论文，**逐篇精读、熟读于心**，做到能随口讲清每篇的核心贡献、关键设计和影响。
> 与 `VLA-WM/`（研究方向调研）并列，是我"了解导师、贴近导师研究品味"的专属仓库。

---

## 🎯 为什么读他的论文

谢赛宁的工作有鲜明的研究品味：**做减法、立基准、用最朴素的设计验证最本质的问题**（"某个被认为必需的归纳偏置，其实并不必需"）。这条线从 ResNeXt、到 ConvNeXt、到 DiT 一以贯之。读他的论文不只是学知识，更是学**怎么提问、怎么做 clean 的科学验证**。

而且他的多篇工作正好是我研究方向（视频生成 / 世界模型）的**地基** —— 尤其 DiT，是几乎所有现代视频世界模型 backbone 的母架构。

---

## 📚 论文索引

| # | 论文 | 年份/会议 | 一句话 | 状态 |
|---|---|---|---|---|
| 1 | [**DiT** (Scalable Diffusion Models with Transformers)](./[arXiv%202212.09748]%20DiT/README.md) | CVPR 2023 (Oral) | 把扩散模型的 U-Net 换成纯 Transformer，证明 scaling law（Gflops↔FID），ImageNet SOTA | ✅ 已精读 |

> 后续会陆续补充他的其它代表作（如 ConvNeXt、SimCLR 相关、Cambrian、表征学习等方向）。

---

## 📌 阅读约定

- 每篇建独立子目录：`[arXiv XXXX.XXXXX] 论文名/`
- 内含：`README.md`（导航）+ `sections/`（含 `summary.md` 核心速读 + 完整批读）+ `figures/`（关键图，**绝不省略**）
- summary 是重点：要能脱稿讲清核心 idea、关键设计、实验论断、对领域的影响
- 中英混合，原文引用 + 完整翻译

---

由 Ricardo + Claude 协作整理 · 目标导师论文精读
