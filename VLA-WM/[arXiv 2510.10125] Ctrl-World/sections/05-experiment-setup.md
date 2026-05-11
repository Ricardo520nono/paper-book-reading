[← 返回 Ctrl-World 主页](../README.md)

# §5.1 Experiment Setups

> **本节阅读重点**：DROID 数据是什么 + 训练细节。**DROID 数据分布是 Uni-WAM 视角下的关键变量** —— 看 Ctrl-World 训在什么 action 分布上。

---

## ¶1 · DROID Platform and Dataset

> "Our experiments use the **DROID platform**, which features a **Panda robot arm equipped with a Robotiq Gripper**. The platform includes **one wrist-view camera and two randomly positioned third-view cameras** that observe the workspace. The DROID dataset contains **95,599 diverse trajectories collected from 564 scenes**, providing dense coverage of the workspace. This includes about **76k successful** and about **19k failed trajectories**. The inclusion of diverse actions and failure data is crucial, as it allows us to train a controllable world model that can simulate a wide range of future scenarios."

**翻译**：

**硬件**：
- Panda 机械臂 + Robotiq Gripper
- 3 个相机：1 个 wrist-view + 2 个随机放置的 third-view

**数据集**：DROID
- **95,599 条 trajectories**（约 96k）
- **564 个 scene**（workspace 覆盖密集）
- **约 76k successful + 约 19k failed**
- Paper 自己强调：**"diverse actions and failure data is crucial"** —— 多样 action + 失败数据对训练 controllable WM 很重要

🔥🔥🔥 **Uni-WAM 视角 —— 关键发现！**

之前在 §0 Abstract 里我**预判错了**："DROID 数据是 expert demo，0 off-expert"。

**事实修正**：
- DROID 有 **19k failed trajectories**（约 20% 是失败的）
- Paper 自己也意识到 "diverse actions + failure data is crucial"

**这意味着什么？**

| 维度 | 我的预判 | 实际情况 |
|---|---|---|
| Ctrl-World 训练数据是否 expert-only | ❌ "0 off-expert" | ✅ **有 19k 失败** |
| 训练数据是否在 expert manifold 上 | ❌ "完全在" | ⚠️ **大部分在，但有少量 off-expert（失败案例）** |

→ **Ctrl-World 比我预判的更接近 Uni-WAM 的方向** —— 它至少部分意识到了"训练需要多样 action / failure"。

**但是**——

⚠️ **DROID 的"failure"≠ Uni-WAM 的"off-expert"**：
- DROID 的 failure 是**真人遥操作时不小心失败的 trajectory**
- 这些 failure 仍然是**人类执行的、试图完成任务的尝试**
- → 它们**仍然在"human-feasible action distribution"内**

**vs Uni-WAM 5 类 off-expert**：
- **Counterfactual**（朝任务反方向）：不是人类失败案例，是**人类不会做的反向 action**
- **Random-feasible**（关节空间均匀采样）：完全脱离任务语义的 random action
- **Adversarial**：主动找 WM 弱点的对抗 action

→ DROID 的 19k failure 是 "**human-attempted off-expert**"，**只覆盖 Perturbed expert 和部分 Exploratory 类别**，**完全不包含 Counterfactual / Random-feasible / Adversarial**。

🔥 **修正后的 Uni-WAM 评估**：
- Ctrl-World 训练数据**部分包含 off-expert action**（19k failure）
- 但**只覆盖 Uni-WAM 5 类中前 2 类的部分**
- **Counterfactual / Random-feasible 仍是空白**

---

## ¶2 · Training Details

> "During training, our model jointly predicts outputs from all three cameras, each with a resolution of **192x320**. The model is conditioned on a **history of 7 frames**, with an interval of **1-2 seconds between frames**. We condition the model on the **next 15 future actions**, which corresponds to a one second action chunk in DROID. During interaction, if a policy's output is less than 15 steps, we pad the action chunk with dummy actions and only use the predictions for valid actions. We train the model on **2×8 H100 GPUs**, with a total batch size of 64. Training takes approximately **2-3 days**."

**翻译**：

| 参数 | 值 |
|---|---|
| 视频分辨率 | 192×320，3 个相机联合预测 |
| 历史 context | **7 帧**，间隔 1-2 秒 |
| 预测窗口 | **未来 15 帧**（= 1 秒 action chunk in DROID）|
| Padding 处理 | 不足 15 步则 pad dummy action，只用有效预测 |
| 训练资源 | 2×8 H100 GPU |
| 总 batch size | 64 |
| 训练时间 | 2-3 天 |

💡 **批注**：
- **7 帧历史 → 预测 15 帧未来**：这意味着 WM 的"上下文窗口"相对短
- **1 秒 action chunk**：H = 15 步 / 秒 = ~15Hz action rate（DROID 标配）
- 训练成本相对低（2-3 天 16xH100，比 video gen foundation model 训练便宜很多）—— **是因为 backbone SVD 1.5B 不大**，且 fine-tune 而非 from scratch

🔥 **Uni-WAM 视角**：
- **Action chunk 长度 H = 15** —— 这是 Uni-WAM 设计 off-expert action chunk 时的天然单位
- A+B 采样方案如果要直接评估 Ctrl-World，应该采样长度也是 15 帧的 chunk

---

## 💡 §5.1 整段 takeaway

| 维度 | Ctrl-World 实际情况 |
|---|---|
| 训练数据来源 | DROID（真机遥操作 95k 轨迹）|
| 是否纯 expert | ❌ 不是 —— 含 19k failure |
| Failure 性质 | **human-attempted failure**，不是 systematic off-expert |
| 覆盖 Uni-WAM 5 类的程度 | **部分 Perturbed expert** + **部分 Exploratory**，**完全不覆盖** Counterfactual / Random-feasible / Adversarial |
| Action 时序 | H = 15 步（~1 秒）|

**结论修正**：Ctrl-World 比预想的"更进步"，但仍**远远不到 Uni-WAM 的标准**。它的失败数据是被动收集的人类失败，不是主动设计的 off-expert 分布。

---

## 🤔 §5.1 留下的问题

| Q | 在哪里查 |
|---|---|
| 19k failure 具体是什么类型？(grasp 失败？过冲？)| paper 或 DROID 原 paper |
| Ctrl-World 训练时 success vs failure 是否平衡采样？ | Appendix A |
| 实验中评估的 policy 是否在 DROID failure data 上训练过？ | §5.3 + π0 / π0.5 原 paper |

---

[← §4.2 Using Ctrl-World](04-method-policy-eval.md) | [§5.3 Policy Evaluation →](05-policy-evaluation.md)
