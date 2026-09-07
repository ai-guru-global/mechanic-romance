# RT-2：视觉-语言-动作模型与涌现的机器人泛化（Vision-Language-Action Models: Web-Scale Knowledge Applied to Robotic Manipulation）

> 把**预训练视觉-语言大模型（VLM）**与机器人动作数据**联合微调**，让一个统一 Transformer 同时吃图像、读指令、吐动作——并因此获得从网络海量数据中"涌现"出的**语义泛化**能力，能执行训练时从未见过的指令（如"把水果移到杯子"、理解图标、初等符号推理）。

## 元信息

- **标题**：Vision-Language-Action Models: Web-Scale Knowledge for Robotic Manipulation
- **作者**：Anthony Brohan, Noah Brown, Justice Carbajal, Yevgen Chebotar, ... (Google DeepMind 团队，约 80+ 作者)
- **机构**：Google DeepMind（含原 Google Research Robotics 团队）
- **会议 / 年份**：Conference on Robot Learning (CoRL) 2023；arXiv 预印本 2023-07-28
- **论文链接**：https://arxiv.org/abs/2307.15818
- **项目主页**：https://robotics-transformer2.github.io/
- **关键词**：VLA、co-finetuning、emergent capability、PaLI-X、PaLM-E、Google Robot

## BibTeX

```bibtex
@inproceedings{brohan2023rt2,
  title     = {{RT-2}: Vision-Language-Action Models Transfer Web-Scale Knowledge to Robotic Control},
  author    = {Brohan, Anthony and Brown, Noah and Carbajal, Justice and Chebotar, Yevgen and Chen, Xi and Choromanski, Krzysztof and others},
  booktitle = {Conference on Robot Learning (CoRL)},
  year      = {2023},
  note      = {arXiv:2307.15818}
}
```

> 注：CoRL 2023 收录标题以 "Transfer Web-Scale Knowledge..." 为正标题；早期 arXiv 标题为 "Vision-Language-Action Models: Web-Scale Knowledge Applied to Robotic Manipulation"。同一篇。

## 一句话概括

RT-2 把机器人动作变成文本 token，与网络图文数据**共训练（co-finetune）**进同一个 VLM，使 55B/5B 量级模型既能聊天、识图，又能驱动机器人，并把网络世界中学到的**概念理解与符号推理**"免费"迁移到真机操作上——首次证明大模型可以成为机器人的**端到端大脑**。

---

## 核心问题

### 1. 机器人数据"贫乏"，网络数据"富有"

通用机器人策略的最大瓶颈是**数据**：真机演示采集昂贵，2023 年单实验室数据量普遍在十万级 episodes；而 VLM 预训练用的网络图文对动辄数十亿。两者数量级差了 3–4 个数量级。

直接在小规模机器人数据上训练专用策略（如 RT-1），能学会**特定技能**，但**无法泛化到新概念**：训练时没见过"Star Wars 人物"，机器人就拿不起尤达大师玩具。

### 2. 两条相互割裂的"智能"流水线

2023 之前的典型架构是"模块化"：

```
VLM (感知+理解) ──► 自然语言"目标" ──► 技能库 / 小策略 (执行)
```

问题是：**高层理解无法反哺底层控制**。VLM 说"把可乐拿给我"，但底层小策略根本不懂"可乐"以外的物体，错误就在传递中累积。

### 3. RT-2 的赌注

> 能否让**一个**模型同时做感知、推理、控制，从而把网络世界知识**直接**注入运动指令？

这要求把动作"翻译"成 VLM 能处理的形式——这就是**动作 token 化（action tokenization）**的由来。

---

## 方法

### 1. 整体范式：Co-finetuning（联合微调）

不是"先训练 VLM 再训练机器人"，而是把机器人数据**当作另一种语言数据**混进 VLM 的微调集：

```
原始 VLM 训练数据  =  网络图文 (VQA、captioning、推理)   ┐
                                                          ├──►  统一 Transformer
机器人数据         =  (图像, 指令) → 动作 token 序列        ┘
```

权重共享、损失共享，唯一区别是机器人样本多了**动作 token**作为目标。

### 2. 动作 Token 化（Action Tokenization）

这是把连续控制塞进离散文本框架的**工程关键**。

机器人动作 $\mathbf{a} \in \mathbb{R}^d$（典型 $d=7$：末端 $x,y,z$，旋转 rpy 或 quat，夹爪开合）：

1. 每一维独立**分桶离散化**为 256 个 bin。
2. 每个 bin 映射为词表里的一个 token，复用 VLM 已有的、低频自然语言 token（避免扩充词表、保留预训练知识）。
3. 一条动作变成 $d$ 个 token 的"小句子"。

$$
\mathbf{a} = (a_1, \ldots, a_d) \;\longrightarrow\; \big(\text{tok}(a_1), \ldots, \text{tok}(a_d)\big), \quad a_i \in [a_i^{\min}, a_i^{\max}]
$$

$$
\text{tok}(a_i) = \text{Vocab}\!\left[\;\text{round}\!\left(255 \cdot \frac{a_i - a_i^{\min}}{a_i^{\max} - a_i^{\min}}\right)\right]
$$

解码时把 token 反查回连续值。**精度损失≈动作范围的 1/512**（量化误差），在 Google Robot 的 7 维动作空间上可忽略。

> 工程含义：动作 token 化让 RT-2 **无需改动 VLM 架构**——decoder 还是那套 decoder，只是学了一个新的"方言"。这是它能在 55B 模型上跑通的前提。

### 3. 架构：两种 VLM 基座

RT-2 实验了两个底座：

| 变体 | 基座 | 视觉编码 | 语言模型 | 参数量 |
| --- | --- | --- | --- | --- |
| **RT-2 (PaLI-X)** | PaLI-X | ViT-G/14（224² 或 565² 输入） | mT5 encoder-decoder | **55B**（实际 55B 全量；论文亦给 5B 版本） |
| **RT-2 (PaLM-E)** | PaLM-E | ViT 分支 | PaLM decoder-only | 12B（也用到 PaLM 540B 报告） |

两者都把图像 patch 与文本 token 串成一条序列输入 Transformer。动作作为 token 出现在输出端。

```
                  ┌──────────────────────────────────────────────┐
   图像 I ──ViT──►│                                              │
   指令 ℓ ────────►   统一 Transformer (PaLI-X / PaLM-E)         │──► a_x_tok, a_y_tok, ..., gripper_tok
   历史 obs ──────►│   (自回归解码)                                │     (7 个动作 token)
                  └──────────────────────────────────────────────┘
```

### 4. 训练数据配比

- **VLM 原始数据**：PaLI-X 的 web-scale 图文（数十亿量级）。
- **机器人数据**：Google Robot 的演示（约 13 万 episodes 的**RT-1 数据 + RT-1 仿真数据**），全部来自 Everyday Robots 的真机 + 仿真。指令覆盖约 **552 个技能语义**。
- **关键比例**：机器人样本在 co-finetune 阶段约占 50%（论文强调：太少→学不会动作；太多→遗忘 web 知识）。

### 5. 推理与"动作链式思考"

RT-2 可以把**链式思考（chain-of-thought）**用到动作上：先生成一段语言推理，再生成动作 token。这让它能执行**多阶段语义任务**（"拿起已消失的物体" → 先推理物体位置 → 再行动），论文把它叫 *Action Chain-of-Thought*。

### 6. 训练目标

RT-2 的损失就是标准 VLM 的 next-token 交叉熵，机器人样本的动作 token 与语言样本的文字 token 共享同一个 loss：

$$
\mathcal{L}_{\text{RT-2}} = -\mathbb{E}_{(I,\ell,\mathbf{a}) \sim \mathcal{D}_{\text{robot}}}\!\left[\sum_{i=1}^{d} \log p_\theta\!\big(\text{tok}(a_i) \mid I, \ell, \text{tok}(a_{<i})\big)\right] -\mathbb{E}_{\text{VLM data}}[\log p_\theta(\cdot)]
$$

两个期望项加权求和，权重由数据采样比例（约 50%）隐式决定。

### 7. 与 RT-1 的对比

| 维度 | RT-1 (2022) | RT-2 (2023) |
| --- | --- | --- |
| 架构 | 35M Transformer（专用） | 5B/55B VLM |
| 训练数据 | 13 万 episode（私有） | 同上 + web 图文 |
| 动作表示 | 离散 token (tokenized) | 离散 token（沿用） |
| 已见任务 | **76%** | **62%**（略低） |
| 未见指令 | ~32% | **62%** |
| 指令覆盖 | 552 个 | 552 + Emergent |

---

## 实验

### 1. 任务规模：从 552 到数千条指令

Google Robot 原有评测集覆盖 **552 条**基础指令（pick、move、open drawer 等）。RT-2 的关键评估是**Emergent Evaluation**——给训练中从未出现过的指令：

- **Object categories**：「pick up the bag」 vs 训练只见具体零食袋。
- **Human/celebrity**：移动"Harry Potter"玩具、识别"Spider-Man"。
- **Reasoning**：「move the object nearest to the red one」。
- **Symbol understanding**：理解图标/符号（如数字、emoji）。
- **Multilingual**：中文、法语指令。
- **Chain-of-thought**：复合指令。

### 2. 主结果（真机成功率，Google Robot）

| 模型 | 原始 552 任务 | Emergent（未见指令） |
| --- | --- | --- |
| RT-1（专用基线） | ~76% | ~32%（基本不行） |
| RT-2 (PaLI-X 55B) | **62%**（与 RT-1 相当/略低） | **62%** |
| RT-2 (PaLI-X 5B) | — | 显著低于 55B，但 > RT-1 |

> 关键洞察：在**已见任务**上 RT-2 与专用 RT-1 **打平**（这是惊喜：大模型没有因为多任务而退化）；在**未见任务**上 RT-2 **碾压**所有基线——这就是"涌现"。

注意 55B 模型的原始任务略低于 RT-1，是因为模型太大、推理慢、且部分精细动作不如专用策略。但**泛化是数量级提升**。

### 3. 涌现能力（Emergent Capability）

论文用一张图归纳——这些能力都**不在训练数据中**，但模型因为吃透了网络知识而具备：

| 能力类别 | 示例 | RT-1 | RT-2 (55B) |
| --- | --- | --- | --- |
| 概念理解 | "move the apple to the cup with numbers on it" | ✗ | ✓ |
| 人/角色 | "move the Spider-Man figure" | ✗ | ✓ |
| 推理 | "push the object closest to the green one" | ✗ | ✓ |
| 符号 | "move object with 3 written on it" | ✗ | ✓ |
| 多语言 | 中文/法语指令 | ✗ | ✓ |

> 这是论文标题里 "Web-Scale Knowledge Transfer" 的实证：网络世界的知识确实**流到了**机器人末端执行器。

### 4. 消融实验

- **模型规模**：5B → 55B，emergent 成功率单调上升（scaling 在起作用）。
- **数据比例**：机器人样本占比 50% 是甜蜜点；降到 10% 机器人学不会动作，升到 90% 网络知识丢失。
- **动作表示**：分桶 vs 直接回归数值——分桶（token 化）配合大模型效果显著更好。
- **视觉输入分辨率**：565² 高分辨率对小物体抓取有帮助（如带数字的物体）。
- **Co-finetune vs from-scratch**：从头训 VLM+机器人远不如 co-finetune——预训练 web 知识是泛化的根。

### 5. 速度与部署

55B 模型在 Google 内部 TPU 集群上跑，端到端推理约 **1–3 Hz**（每秒 1–3 个动作）。**这是 VLA 路线早期最被诟病的点**：频率太低，精细高速任务做不了。后续 OpenVLA、π₀ 都在解决这个问题。

---

## 贡献与意义

1. **范式确立**：第一次让"一个 50B+ 大模型直接控制真机"从论文变成可重复实验。此后所有 VLA 工作（OpenVLA、Octo、π₀）都在这条延长线上。
2. **Co-finetune 这一名词**：把机器人数据当作"另一种语言"喂给 VLM，是后续 VLA 训练的标准范式。
3. **动作 token 化**：连续控制 → 离散 token 的工程套路，被广泛沿用（OpenVLA、RT-X 等）。
4. **"涌现"概念引入机器人**：首次系统论证机器人策略能从 LLM 预训练中获得**训练数据里没有**的能力，把 NLP 的 scaling 信仰带进了 manipulation。
5. **闭环了高层语义与底层控制**：一举结束了"VLM 出目标 + 小策略执行"的割裂架构。

---

## 局限

1. **速度慢**：1–3 Hz 无法做插销、拧螺丝等高频反馈任务。
2. **闭源**：权重、数据、训练细节都不公开，社区无法复现或改进。
3. **动作精度受量化影响**：256 bin 的离散化对精细任务（亚毫米）有损。
4. **只验证了一种本体**（Google Robot，单臂 + 平面夹爪）：跨本体能力未在该论文证明（留给 RT-X）。
5. **数据集中在 Google 内部**：泛化到底层机器人数据稀缺机构不可复制。
6. **原文 552 任务成功率没超过 RT-1**：说明大模型在"已掌握"任务上未必更优，泛化的代价是专门能力的轻微退化。

---

## 个人评价

RT-2 是**具身智能的 GPT-3 时刻**——它没有发明任何全新算法（VLM、动作 token、co-finetune 都不是新概念），但它用惊人的工程规模（55B、80 人、内部 TPU）证明了一个信念：**通用智能可以下沉到机器人控制**。论文里那张"emergent capability"的图，是 2023 年整个领域最被反复引用的图之一，因为它第一次让"机器人也能涌现"这件事变得可信。

对我自己的研究启发有三：

1. **数据混合比（co-finetune ratio）是被低估的工程杠杆**——后续做任何 VLA 微调，这个 50% 都该作为起点去 sweep。
2. **emergent evaluation 是必须设计的**：只在"训练分布内"评测会掩盖 VLA 的真正价值；要刻意构造未见指令、未见物体、符号推理。
3. **不能迷信大模型**：RT-2 在原始任务上没赢 RT-1，提醒我们**专用任务仍是专用策略的天下**，VLA 的红利在"长尾泛化"——这正是它和 Diffusion Policy 互补的地方。

读这篇论文最大的收获不是公式（公式很简单），而是**世界观**：当动作只是另一种 token，机器人就和聊天机器人共享同一个大脑。这个 abstraction 是革命性的。

---

## 相关概念互链

- [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md) — VLA 总论，RT-2 是其中里程碑
- [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md) — RT-2 本质是 BC 训练（动作 token 的分类损失）
- [`../../concepts/foundations/embodied-ai.md`](../../concepts/foundations/embodied-ai.md) — 具身智能的定义
- [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md) — RT-2 之后跨本体的延伸
- [`2024-kim-openvla.md`](2024-kim-openvla.md) — RT-2 思想的开源复刻与平民化
- [`../../surveys/2023-firoozi-foundation-models-survey.md`](../../surveys/2023-firoozi-foundation-models-survey.md) — 综述中的 VLA 章节

---

## 参考

- 论文：*RT-2: Vision-Language-Action Models Transfer Web-Scale Knowledge to Robotic Control*，CoRL 2023（来源：https://arxiv.org/abs/2307.15818，访问于 2026-09-07）
- 项目主页：https://robotics-transformer2.github.io/（访问于 2026-09-07）
