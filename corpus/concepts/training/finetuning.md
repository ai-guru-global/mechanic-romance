# 微调（Fine-tuning）

> 在**大规模预训练模型**基础上，用**任务/机器人/场景专属的小批量数据**继续训练，把通用基座适配到具体下游。是「预训练-微调」范式在具身智能的核心落地环节。

> 最后更新：2026-08

---

## 1. 直觉

现代具身智能策略（尤其 VLA）走的是「**先通用、后专用**」两段式：

```
   阶段1：预训练 (pretrain)            阶段2：微调 (fine-tune)
   互联网图文 + OXE 百万 episode  ──►  你这台机器人 / 这个任务的 几十~几千 demo
   学到通用视觉-语言-动作先验          适配到具体场景，性能跃升
```

预训练赋予模型「**知道杯子能抓、抽屉能拉**」的常识；微调告诉它「**在 Franka 上、这张桌子上、用这个夹爪，具体怎么抓**」。没有微调，预训练模型在特定任务上泛化有余而精准不足；没有预训练，微调数据太少学不出鲁棒策略。两者缺一不可。

---

## 2. 为什么 VLA 必须微调

视觉-语言-动作模型（VLA）的典型生命周期：

| 阶段 | 数据 | 目标 |
| --- | --- | --- |
| **预训练** | Open-X-Embodiment（OXE，100 万+ episode，跨 20+ 本体） | 学通用动作原语与视觉-语言对齐 |
| **微调** | 单台机器人、特定任务的几十~几千条 demo | 适配本体物理、任务细节、场景外观 |

必须微调的原因：

1. **OXE 是「杂食」**：22 种机器人、各种任务混训，模型学到的是平均分布，落到你的 Franka 抓取任务上有**精度损失**。
2. **本体差异**：预训练见过的本体 ≠ 你的本体（自由度、夹爪、工作空间），需微调对齐动作空间。
3. **场景特异**：你的实验室光照、桌面、物体与 OXE 不同，视觉分布偏移需微调修正。
4. **任务语言**：你的指令模板（「把红色方块放进蓝碗」）与预训练措辞不同，需微调对齐。

> 见 [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)、OpenVLA 论文 [`../../papers/vla/2024-kim-openvla.md`](../../papers/vla/2024-kim-openvla.md)——其核心实验就是「OXE 预训练 → 单任务微调」的消融。

---

## 3. 全量微调 vs 参数高效微调

拿到预训练 VLA 后，更新多少参数？

| 方式 | 更新范围 | 显存 | 速度 | 性能上限 | 适用 |
| --- | --- | --- | --- | --- | --- |
| **全量微调（full FT）** | 所有参数 $\theta$ | 高 | 慢 | 最高（完全适配） | 数据充足、追求 SOTA |
| **LoRA** | 在权重上挂低秩矩阵 $\Delta W = BA$，只训 $A,B$ | **极低** | 快 | 接近全量（差距通常 < 5%） | **当前 VLA 主流** |
| **Adapter** | 在层间插小 MLP 模块，只训模块 | 低 | 快 | 接近全量 | 多任务共享底座 |
| **Prompt / Prefix tuning** | 只学少量软 token | 极低 | 极快 | 略低 | 极少数据快速试 |
| **冻结 + 仅训头** | 编码器冻结，只训策略头 | 最低 | 最快 | 有限（表征不能适配） | 极少数据 baseline |

### LoRA 形式化

原始权重 $W_0 \in \mathbb{R}^{d\times k}$ 不动，挂上低秩更新：

$$
W = W_0 + \Delta W = W_0 + B A, \quad B \in \mathbb{R}^{d\times r},\; A \in \mathbb{R}^{r\times k},\; r \ll \min(d,k)
$$

只训 $A, B$，可训练参数从 $dk$ 降到 $r(d+k)$（典型 $r=8$，参数量减少数百倍）。LoRA 在 OpenVLA、$\pi_0$ 等微调脚本里是默认选项。

---

## 4. 数据配比（Data Mixing）

微调时通常不只用下游 demo，而是**混入一部分预训练数据**防遗忘：

$$
\mathcal{D}_{\text{FT}} = \alpha \cdot \mathcal{D}_{\text{task}} + (1-\alpha) \cdot \mathcal{D}_{\text{pretrain}}, \quad \alpha \in [0.3, 0.8] \text{ 常见}
$$

- $\alpha$ 太高（纯任务数据）→ 灾难性遗忘，丢掉通用能力。
- $\alpha$ 太低（预训练占比大）→ 任务适配不充分。
- 实践：从 $\alpha=0.5$ 起调，监控任务成功率和通用 benchmark 的权衡。

> OXE 微调的标准做法是 replay 一部分 OXE 数据，详见 [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md)。

---

## 5. 灾难性遗忘（Catastrophic Forgetting）

微调最大的坑：在新任务上越训越好，**在旧任务/通用能力上越来越差**。

| 成因 | 神经网络权重被新数据「覆写」，旧知识存在权重里的分布被冲掉 |
| --- | --- |
| 表现 | 微调抓取任务后，模型忘了「倒水」；或视觉编码器丢失通用语义 |
| 缓解 1 | **数据回放（replay）**：微调时混入预训练数据（见上节配比） |
| 缓解 2 | **参数高效微调**：LoRA/Adapter 只动增量参数，原权重保留，天然抗遗忘 |
| 缓解 3 | **弹性权重巩固（EWC）**：对重要参数加正则，限制其漂移 |
| 缓解 4 | **课程 / 多任务持续微调**：避免单任务过拟合 |

> 实践经验：**LoRA + 数据 replay** 是当前 VLA 微调抗遗忘最简单有效的组合。

---

## 6. 少样本 / 演示微调（Few-shot / Demonstration Fine-tuning）

真机 demo 昂贵，常见场景是**只有 10~50 条人类演示**就要适配新任务：

| 技术 | 思路 | 代表 |
| --- | --- | --- |
| **少样本模仿学习** | 用预训练 VLA 做 backbone，少量 demo 微调策略头 | OpenVLA few-shot |
| **元学习（meta-learning）** | 预训练时学「快速适应」的初始化，几步梯度即适配 | MAML 式机器人学习 |
| **In-context learning** | 不更新参数，把 demo 作为上下文喂给 VLA | RT-2 / VIMA 的 prompt 式 |
| **演示 + RL 微调** | demo 做行为克隆初始化，再用 RL 在仿真里精修 | IL + RL 混合（见 [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)） |

> 关键洞察：预训练质量决定**少样本下限**，微调策略决定**数据效率上限**。OXE 预训练好的 VLA，几十条 demo 就能解锁新任务——这是「基础模型」范式的核心承诺。

---

## 7. 微调在具身智能中的特殊之处（vs NLP/CV）

| 维度 | NLP/CV 微调 | 具身智能微调 |
| --- | --- | --- |
| 数据形式 | 文本/图像对 | (图像, 语言, 本体感知, 动作) 多模态元组 |
| 动作空间 | 无 | 必须对齐本体（自由度、控制频率） |
| 闭环评估 | 静态指标（acc/BLEU） | **必须真机/仿真闭环 rollout** 才知好坏 |
| Sim-to-real | 无 | 微调可在仿真做，但最终要迁真机（见 [`../foundations/sim-to-real.md`](../foundations/sim-to-real.md)） |
| 安全性 | 答错无所谓 | 动作错了可能撞坏设备 |

因此具身微调的评估比 NLP/CV 复杂得多——**不能只看 loss，必须 rollout 测成功率**。

---

## 8. 典型微调流水线（VLA 示例）

```text
1. 选预训练基座（如 OpenVLA，OXE 预训练完成）
2. 采集下游 demo（真机遥操，50~500 条）
3. 配比：α=0.5 混入 OXE replay 数据
4. 选择微调方式：LoRA（r=16，仅训 attention 投影）
5. 训练：AdamW，lr=2e-5，3~10 epoch
6. 评估：仿真 + 真机闭环 rollout，监控成功率与通用 benchmark
7. 抗遗忘检查：跑预训练时见过的任务，确认未退化
8. 迭代：成功率不足 → 加数据 / 调 α / 解冻更多层
```

---

## 9. 主要挑战与前沿

- **自动微调**：如何免去人工调 α / lr，自动搜索最优微调配方。
- **跨本体微调**：从 Franka 的 demo 微调到 UR5e（不同动作空间）——需本体无关的动作表征。
- **持续学习**：不断学新任务而不忘旧任务，终极目标。
- **高效真机微调**：几分钟真机数据即适配，是产品化的关键。

---

## 10. 相关概念互链

- [`../foundations/embodied-ai.md`](../foundations/embodied-ai.md)——预训练-微调是基础模型范式在具身智能的体现。
- [`../perception/visual-representation.md`](../perception/visual-representation.md)——微调视觉编码器（冻结 vs 解冻）的核心决策。
- [`../foundations/sim-to-real.md`](../foundations/sim-to-real.md)——真机微调是 few-shot sim-to-real 的最后一步。
- [`../mdp/policy.md`](../mdp/policy.md)——微调的对象就是策略参数 $\theta$。
- [`../foundations/world-model.md`](../foundations/world-model.md)——世界模型预训练 → 下游策略微调。

## 11. 代表方法 / 论文

- VLA 预训练-微调范式 → [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)
- 模仿学习（微调常以 BC 为损失）→ [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)
- OpenVLA（OXE 预训练 + LoRA 微调的标杆）→ [`../../papers/vla/2024-kim-openvla.md`](../../papers/vla/2024-kim-openvla.md)
- 代表论文：
  - Hu et al., *LoRA: Low-Rank Adaptation of Large Language Models* (2021, arXiv:2106.09685)
  - OpenVLA, *Open-Source Vision-Language-Action Models* (2024, arXiv:2406.09246)
  - OXE 数据集论文 → [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md)

---

## 12. 参考链接

- 训练目录总览：[`./README.md`](README.md)
- 方法目录：[`../../methods/`](../../methods/)
- 综述：*Fine-tuning Large Vision-Language Models for Robotics* (2024)
- LoRA（Hu et al. 2021）——低秩适配微调的代表方法（来源：https://arxiv.org/abs/2106.09685，访问于 2026-09-02）
