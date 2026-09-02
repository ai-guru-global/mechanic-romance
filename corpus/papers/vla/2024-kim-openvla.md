# OpenVLA：开源 7B 视觉-语言-动作基座模型（OpenVLA: An Open-Source Vision-Language-Action Model）

> 把 RT-2 "闭源 55B"的范式**开源化、平民化、可微调化**：基于 Prismatic VLM（Llama 2-7B + DINOv2 + SigLIP），在 Open-X-Embodiment 上训练，发布 7B 全量权重与 LoRA/全量微调工具链，首次让任何一个实验室都能在自己的机械臂上跑一个"小号 RT-2"。

## 元信息

- **标题**：OpenVLA: An Open-Source Vision-Language-Action Model
- **作者**：Moo Jin Kim, Karl Pertsch, Siddharth Karamcheti, Ted Xiao, Ashwin Balakrishna, Suraj Nair, Rafael Rafailov, Ethan Foster, Grace Lam, Pannag Sanketi, Quan Vuong, Thomas Kollar, Benjamin Burchfiel, Russ Tedrake, Dorsa Sadigh, Sergey Levine, Percy Liang, Chelsea Finn
- **机构**：Stanford（TRI / SUAL）、UC Berkeley、TRI（Toyota Research Institute）、Google DeepMind、MIT
- **会议 / 年份**：arXiv 预印本 2024-06；后续入选 ECCV 2024 / CoRL 2024 系列讨论
- **论文链接**：https://arxiv.org/abs/2406.09246
- **代码 / 权重**：https://openvla.github.io/ 、https://github.com/openvla/openvla
- **关键词**：开源 VLA、Prismatic、OXE、LoRA 微调、DINOv2+SigLIP

## BibTeX

```bibtex
@article{kim2024openvla,
  title   = {{OpenVLA}: An Open-Source Vision-Language-Action Model},
  author  = {Kim, Moo Jin and Pertsch, Karl and Karamcheti, Siddharth and Xiao, Ted and
             Balakrishna, Ashwin and Nair, Suraj and Rafailov, Rafael and Foster, Ethan and
             Lam, Grace and Sanketi, Pannag and Vuong, Quan and Kollar, Thomas and
             Burchfiel, Benjamin and Tedrake, Russ and Sadigh, Dorsa and Levine, Sergey and
             Liang, Percy and Finn, Chelsea},
  journal = {arXiv preprint arXiv:2406.09246},
  year    = {2024}
}
```

## 一句话概括

OpenVLA 是 RT-2 范式的**第一个真正可用开源基座**：7B 参数、双视觉编码器融合、动作分桶 token 化、在 970K 条 OXE 真机 episode 上训练，附带完整的 LoRA/全量微调脚本与多本体（WidowX、Franka、UR5）真机部署示例——把"VLA 不再是 Google 专利"变成现实。

---

## 核心问题

### 1. RT-2 之后，VLA 成了"奢侈品"

RT-2 证明了 VLA 的威力，但**只有 Google 能玩**：55B 模型、私有数据、内部 TPU。学术界面临三个空白：

- **权重不公开**：无法复现，无法做安全/可解释/鲁棒性研究。
- **不可微调**：没有梯度通路，下游实验室没法往自己机器人上迁。
- **训练栈不透明**：co-finetune 的工程细节（数据配比、token 化、调度）全是黑箱。

### 2. 数据规模 vs 模型规模的权衡

55B 太大，但 RT-2 5B 版本泛化差很多。**多大的开源模型能在多少 OXE 数据上达到 RT-2 同档泛化？** 这是 OpenVLA 要回答的工程经验问题。

### 3. 微调可行性

真机下游任务数据通常只有几百条。**怎么让 7B VLA 在小数据上高效微调且不灾难性遗忘？**——LoRA 是答案，但需要专门验证。

---

## 方法

### 1. 基座：Prismatic VLM

OpenVLA 不从头造 VLM，而是站在 **Prismatic VLMs**（Karamcheti et al., 2024）的肩膀上：

| 组件 | 选择 | 说明 |
| --- | --- | --- |
| 语言模型 | **Llama 2-7B** | 开源、社区微调生态成熟 |
| 视觉编码器 1 | **SigLIP** (ViT-L/14, 224²) | 语言对齐的 CLIP 变体，强语义 |
| 视觉编码器 2 | **DINOv2** (ViT-L/14) | 自监督，强空间/物体结构 |
| 融合 | **多模态投影器**：concat 两路 patch token → MLP → 喂入 Llama | 双编码器互补（语义 + 几何） |

```
   图像 I ──► SigLIP ──┐
                       ├── concat ──► projector MLP ──► 36 个视觉 token
              DINOv2 ──┘                                     │
                                                            ▼
   指令 ℓ ─────────────────────────────► [ Llama 2-7B (decoder) ] ──► 动作 token 序列
```

> 设计哲学：**用 SigLIP "看懂语义"，用 DINOv2 "看清结构"**。单一 CLIP 不足以支撑精细 manipulation 的几何需求，这是相比 RT-2（纯 PaLI 单编码器）的工程改进。

### 2. 动作表示：分桶 token 化（沿用 RT-2）

每一维动作独立分 256 bin，复用 Llama 词表里低频 token。一条动作 → $d$ 个 token（$d$ 随本体变化，单臂 7 维 → 7 token）。

$$
\text{tok}(a_i) = \text{round}\!\left(255 \cdot \frac{a_i - a_i^{\min}}{a_i^{\max} - a_i^{\min}}\right), \quad a_i \in \mathbb{R}
$$

**关键差异于 RT-2**：OpenVLA 不限定本体——OXE 数据里不同机器人动作维度不同，OpenVLA 用一个**统一的归一化 + 分桶**接口，每个本体单独定义 min/max，模型只看到 token 序列长度可变。

### 3. 训练数据：Open-X-Embodiment

- **数据集**：OXE 中**真机**子集（去掉纯仿真），共约 **970K 条 episodes**，22 个本体混合。
- **指令处理**：OXE 的自然语言指令经过模板化与清洗。
- **批大小 / 硬件**：在 64× A100 (80GB) 上训练约 **14 天**，总训练量 ~1.2 万亿 token（含视觉 patch）。
- **学习率 / 调度**：cosine + warmup，专门为 7B 量级调过。

> 这是**社区第一次把"RT-2 训练日志"复刻成可复现配方**——每个超参都写在 README 里。

### 4. 微调：LoRA 与全量两条路

OpenVLA 提供 `prismatic` 训练栈的两条微调路径：

| 模式 | 显存 (A100) | 时间 (50 demos) | 适用 |
| --- | --- | --- | --- |
| **LoRA (rank=8/16)** | ~16 GB | ~30 min | 快速适配新本体 / 小数据 |
| **全量微调** | ~80 GB（多卡） | ~数小时 | 大数据、追求上限 |
| **冻结 backbone + 只训 head** | 极低 | 极快 | 仅换动作空间 |

LoRA 实现基于 PEFT，插入 Llama 全部 attention / MLP 线性层；推理时合并权重，**无额外延迟**。

### 5. 真机部署：高频推理

OpenVLA 用 **NVIDIA TensorRT-LLM** 做了推理加速，单卡 A100 上能跑到 **~6–8 Hz**（比 RT-2 的 1–3 Hz 快 3 倍）。这是它"真正可用"的关键——很多 manipulation 任务在 5 Hz 以上才能闭环。

### 6. 训练目标与损失

OpenVLA 本质是一个**自回归 token 分类**问题（next-token prediction），损失对动作 token 与语言 token 一视同仁：

$$
\mathcal{L} = -\sum_{t} \log p_\theta\!\big(y_t \mid y_{<t}, I, \ell\big)
$$

其中 $y_t$ 在机器人样本上是动作 token，在 VLM 样本上是自然语言 token。共享一个交叉熵损失，无加权——这是 co-finetune 的标准做法。

### 7. 与 RT-2 的具体工程差异

| 维度 | RT-2 (PaLI-X 55B) | OpenVLA (7B) |
| --- | --- | --- |
| 视觉编码器 | 单路 ViT-G | **双路 SigLIP + DINOv2** |
| 语言模型 | mT5 (encoder-decoder) | Llama 2 (decoder-only) |
| 动作维度 | 7 (固定 Google Robot) | 可变（OXE 多本体） |
| 训练数据 | 私有 + Google 内部 | OXE 公开 |
| 推理 | TPU 集群 | A100 单卡 + TensorRT |
| 微调 | 不支持 | LoRA + 全量 |

---

## 实验

### 1. 泛化评测（Generalist 评测，无微调）

直接在 BridgeData V2、RT-1 等评估集上跑 zero-shot：

| 模型 | 参数量 | BridgeData 任务平均 | 备注 |
| --- | --- | --- | --- |
| RT-1 | 35M | 专门训练 | 强 |
| RT-2 (PaLI-X 55B, 闭源) | 55B | ≈ RT-1 但泛化强 | 不可复现 |
| **OpenVLA (zero-shot)** | **7B** | **在多数 OXE 任务上 ≥ RT-1，接近 RT-2 5B 档** | 开源 |

> 在 zero-shot 多本体评测上，OpenVLA 是**当前最强开源 VLA**，并在多个任务上击败 RT-2 5B 闭源版本。

### 2. 微调评测（在 WidowX、Franka、UR5 真机）

这是 OpenVLA 的核心卖点——**微调后的下游任务**：

| 任务 | 本体 | 微调数据 | OpenVLA (LoRA) | OpenVLA (全量) | RT-1-X / 基线 |
| --- | --- | --- | --- | --- | --- |
| Bridge V2（7 任务） | WidowX | OXE 子集 | — | **+9% 相对 RT-1-X** | baseline |
| RT-1 Drawer/Cube | Google Robot | 50–100 demos | 接近全量 | 强 | — |
| Franka 精细插入 | Franka | 100 demos | LoRA 微调成功 | 全量最优 | 简单 BC 失败 |

**结论**：LoRA 微调在 50–200 条数据上即可取得接近全量的效果——这是让 OpenVLA 在普通实验室"装上就能跑"的根因。

### 3. 与 RT-2 / RT-X 的对比（论文主表）

| 维度 | RT-2 | RT-X / RT-1-X | **OpenVLA** |
| --- | --- | --- | --- |
| 开源 | ✗ | 部分（数据开，模型闭） | ✅ 全量权重 + 代码 |
| 参数量 | 5B / 55B | 35M / 多个 | 7B |
| 可微调 | ✗ | ✗ | ✅ LoRA + 全量 |
| 推理频率 | 1–3 Hz | 较快 | **6–8 Hz (A100)** |
| 多本体 zero-shot | ✗ | ✅ | ✅（OXE） |
| 数据 | 私有 | OXE（开源） | OXE（开源） |
| 真机部署示例 | ✗ | ✅ | ✅ WidowX/Franka/UR5 |

### 4. 消融

- **视觉编码器**：只用 SigLIP → 精细任务掉点；只用 DINOv2 → 语义指令理解差；**两者融合最优**。
- **模型规模**：7B 是经过 scaling 实验的甜蜜点（3B 太弱，13B 边际收益小且贵）。
- **微调数据量**：50 条 → 200 条提升明显；> 500 条边际收益递减（针对单任务）。
- **动作分桶数**：256 bin 是默认；降到 64 bin 在精细任务上明显掉点。

---

## 贡献与意义

1. **开源 VLA 的"事实标准"**：发布后所有 VLA 论文（π₀、RDT、CogACT 等）都默认跟 OpenVLA 比。它是新的 RT-1。
2. **微调工具链**：LoRA + 全量双路径 + 真机部署脚本，把"用 VLA"门槛从"50 人团队 + TPU"降到"1 个 PhD + 1 张 A100"。
3. **双视觉编码器范式**：SigLIP + DINOv2 的融合成为后续大量 VLA 的标配（Octo、π₀ 都借鉴）。
4. **训练透明**：数据混合、超参、训练日志全公开，是社区复现 VLA 训练的"教科书"。
5. **频率突破**：TensorRT-LLM 把 VLA 推到 6–8 Hz，让 VLA 首次能做半精细操作。

---

## 局限

1. **7B 仍偏大**：消费级显卡（24 GB）只能跑 LoRA 推理，全量微调要 4–8 卡。
2. **动作精度受 token 化限制**：和 RT-2 一样，256 bin 在亚毫米任务上有损。
3. **多本体泛化但非跨本体微调王者**：OXE 上 zero-shot 强，但**比不过在单本体上专门训练的 Diffusion Policy**——精细任务 DP 仍领先。
4. **指令理解弱于 RT-2 (55B)**：规模差距导致符号推理、链式思考能力明显弱一截。
5. **训练成本**：64× A100 × 14 天对绝大多数实验室不现实——只能复用其权重，无法从头训。
6. **没有显式的动作序列预测**：单步 token 自回归，比 DP / ACT 的 chunk 预测效率低，对多模态动作建模也弱。

---

## 个人评价

OpenVLA 之于 VLA，相当于 LLaMA 之于 LLM——它没有改变范式，但它把范式**变成了公共财产**。读这篇论文最大的感受是"工程克制"：作者没有追求 SOTA 数字，而是追求**可复现性、可微调性、可部署性**这三件事，每一件都做到了 90 分。

对我自己的研究启发：

1. **基座选 OpenVLA 而不是从零训 VLA**：做任何 VLA 改进，先用 OpenVLA 做 LoRA 微调基线，省半年工程。
2. **SigLIP + DINOv2 双编码器值得复用**：在 manipulation 上的几何理解差异是真的，不只是论文卖点。
3. **LoRA 是 VLA 微调的甜蜜点**：50–200 条 demo + LoRA = 实验室级真机任务的标配流水线。
4. **不要迷信 zero-shot**：OpenVLA zero-shot 强，但**真正干活仍要微调**——这呼应了"专用任务用专用策略"的规律。

唯一遗憾是 7B 模型仍跑不进消费级显卡做全量训练；未来 1B–3B 的蒸馏 OpenVLA（已有 Octo-Small 等跟进）才是真正"装上就能跑"的方向。

---

## 相关概念互链

- [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md) — VLA 总论
- [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md) — OpenVLA 训练本质是 token 分类 BC
- [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md) — OpenVLA 的训练数据
- [`2023-brohan-rt2.md`](2023-brohan-rt2.md) — 闭源前辈，OpenVLA 直接对标
- [`../surveys/../../surveys/2023-firoozi-foundation-models-survey.md`](../../surveys/2023-firoozi-foundation-models-survey.md) — 综述中的 VLA 章节
- [`../diffusion-policy/2023-chi-diffusion-policy.md`](../diffusion-policy/2023-chi-diffusion-policy.md) — OpenVLA 的弱项（精细多模态）由 DP 补
- [`../imitation-learning/2023-zhao-act.md`](../imitation-learning/2023-zhao-act.md) — ACT 的动作分块思想可改进 OpenVLA
