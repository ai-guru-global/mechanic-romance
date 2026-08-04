# ACT：动作分块 Transformer 与 ALOHA 低成本双臂遥操系统（Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware）

> 一个**双重贡献**的工作：(1) **ALOHA**——一套约 2 万美元的双臂低成本遥操硬件系统；(2) **ACT（Action Chunking with Transformers）**——一个用 CVAE + Transformer 预测 $k$ 步动作 chunk 的策略，通过**动作分块（action chunking）**+ **CVAE 多模态建模**对抗 imitation learning 的协变量偏移（covariate shift），在 6 个精细双臂任务上把简单 BC 的成功率从 ~0% 拉到 80%+。RSS 2023，Zipeng Fu, Tony Z. Zhao, Chelsea Finn（Stanford）。

## 元信息

- **标题**：Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware
- **作者**：Zipeng Fu, Tony Z. Zhao, Chelsea Finn
- **机构**：Stanford University
- **会议 / 年份**：Robotics: Science and Systems (RSS) 2023；arXiv 预印本 2023-04
- **论文链接**：https://arxiv.org/abs/2304.13705
- **项目主页**：https://tonyzhaozh.github.io/aloha/
- **代码 / 硬件**：https://github.com/MarkFzp/act-plus-plus （ACT 训练栈）；ALOHA 硬件设计开源
- **关键词**：bimanual、action chunking、CVAE、covariate shift、teleoperation、低硬件成本

## BibTeX

```bibtex
@inproceedings{zhao2023learning,
  title     = {Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware},
  author    = {Fu, Zipeng and Zhao, Tony Z. and Finn, Chelsea},
  booktitle = {Robotics: Science and Systems (RSS)},
  year      = {2023},
  note      = {arXiv:2304.13705}
}
```

## 一句话概括

ACT 的核心论点只有一个：**别让策略一步一步生成动作，让它一次生成一整段（chunk）**——这既能让 Transformer 学到时序一致性，又能用 CVAE 容纳多模态，再加上 ALOHA 让采集双臂精细演示的成本降到 2 万美元，使"双臂精细操作 + 端到端学习"从大厂专利变成 PhD 论文级课题。

---

## 核心问题

### 1. 双臂精细操作的数据采集门槛

双臂任务（开抽屉、撕胶带、卷尺、转移立方块）需要**两只 6–7 DoF 手臂协同 + 双手夹爪**。2023 年前能做这事的主要是大厂（TRI、Google），硬件动辄几十万美元。社区缺乏**可复现的低成本遥操平台**。

### 2. 协变量偏移（Covariate Shift）—— BC 的根本病

经典 BC 训练时只见**专家轨迹**，推理时一旦动作稍偏，落到训练分布外的状态，策略就崩溃（错误雪球）：

```
训练：只看专家走过的状态 s ∼ D_expert
推理：策略犯错 → 进入 D_expert 之外的状态 → 没学过 → 越错越多
```

在精细双臂任务上，这个问题被**双臂的高维协同误差**放大——一只手偏 1cm，另一只手就接不住。

### 3. 多模态动作

同一观测下，专家可能用不同方式完成同一任务（左手先动 vs 右手先动）。简单 MSE BC 会平均化。

ACT 同时面对这三个问题，给出一个**统一架构答案**。

---

## 方法

### 1. ALOHA 硬件：低成本双臂遥操

| 组件 | 配置 |
| --- | --- |
| 从臂（slave） | 2 × ViperX 6-DoF 机械臂 + 双指夹爪（Interbotix） |
| 主臂（master） | 2 × 同型号 ViperX（leader 端） |
| 同步 | 主臂与从臂**反向运动学背靠背耦合**，操作员动主臂 → 从臂实时跟随 |
| 相机 | 4 个 Logitech C922 网络摄像头（顶视 + 双腕 + 侧视） |
| 控制 | 50 Hz 主循环，ROS noetic |
| 总成本 | **约 2 万美元**（对比：UR5e 双臂系统 5–10 万美元） |

> 关键工程：用**同款便宜臂做主端**而不是专用力反馈手柄——既降本又让演示与执行共享运动学，遥操手感直接。

### 2. ACT 模型架构（CVAE + Transformer）

ACT 是一个 **条件变分自编码器（CVAE）**，由 encoder 和 policy（decoder）组成：

```
                              训练时（KL + reconstruction）
   专家 chunk A ──► [Encoder] ──► z ~ q(z | o, A)
                                       │ 采样
                                       ▼
   当前 obs o ──► [Policy (Transformer decoder)] ──► 预测 chunk Â
                                       ▲
                                  style token z
```

#### Encoder（仅训练用）

输入：当前观测 $o$ + 专家动作 chunk $\mathbf{A}_{k} \in \mathbb{R}^{k \times d}$。

$$
\mathbf{z} = \mu_\phi(o, \mathbf{A}_k) + \sigma_\phi(o, \mathbf{A}_k) \odot \boldsymbol{\epsilon}, \quad \boldsymbol{\epsilon} \sim \mathcal{N}(0, \mathbf{I})
$$

$z$ 是个**低维 style 变量**（论文用 $d_z = 1$ 即可），编码"这次演示是哪种方式"——这是建模多模态的关键。

#### Policy（decoder，训练 + 推理都用）

输入：观测 $o$ + 采样（推理时从 $\mathcal{N}(0, I)$ 采）得到的 $z$。

输出：**一次性预测未来 $k$ 步动作 chunk** $\hat{\mathbf{A}}_{1:k} \in \mathbb{R}^{k \times d}$。

主干是 Transformer decoder，输入是 $k$ 个可学习 query token + 视觉特征（4 路 ResNet-18 编码）+ $z$ 注入。

### 3. 损失函数

$$
\mathcal{L}_{\text{ACT}} = \underbrace{\|\hat{\mathbf{A}} - \mathbf{A}\|_1}_{\text{reconstruction (L1)}} + \underbrace{D_{\mathrm{KL}}\!\big(q(z|o,\mathbf{A})\,\|\,\mathcal{N}(0, I)\big)}_{\text{KL 正则}}
$$

- L1 比 L2 对多模态更友好（少平均化）。
- KL 把 $z$ 拉向标准正态，**推理时直接 $z \sim \mathcal{N}(0, I)$ 采样**就能产出多模态 chunk。

### 4. 动作分块（Action Chunking）—— 反协变量偏移的核心

**为什么 chunk 能对抗协变量偏移？**

直观论证：单步 BC 推理时每个状态都要决策，每个决策都可能偏，错误**逐步累积**。Chunk 一次预测 $k$ 步，相当于把"决策频率"从每步降到每 $k$ 步：

```
单步 BC：  s0→a0, s1→a1, ...   每 1 步决策一次，错误每步放大
ACT (k=100)：s0→{a0..a99}, s100→{a100..a199}  每 100 步决策一次
```

**这意味着策略在每个 chunk 内部是开环执行专家近似轨迹**，绕过了"自己走入 OOD 状态再决策"的循环——这是 ACT 比 vanilla BC 鲁棒得多的根因。论文从理论上把这一条称为 *temporal ensemble* 与 *commitment*。

### 5. 时序集成（Temporal Ensemble）

为避免 chunk 边界跳变，ACT 用**指数平滑**融合重叠 chunk 的预测：

$$
\hat{a}_t = \sum_{i} w_i\, \hat{a}_t^{(i)}, \qquad w_i = \exp\!\left(-\frac{(t - t_i^{(\text{start})})^2}{2\sigma^2}\right)
$$

对来自不同起点的 chunk 预测，按时间距离做高斯加权平均。这让动作更平滑，**进一步降低 OOD 概率**。

---

## 实验

### 1. 任务：6 个 transbimanual 任务

1. **Transfer Cube**（双手传递立方块）
2. **Bungee Cord**（拉开/接合弹力绳）
3. **Dice Tray**（在托盘里拨骰子）
4. **Open Drawer**（拉开抽屉）
5. **Tape**（撕开胶带）
6. **Cube**（双手协作放方块）

每个任务采集 **50 条演示**（短短数小时，得益于 ALOHA 易用）。

### 2. 主结果（成功率 %，10 个 rollout 平均）

| 任务 | Vanilla BC (L1, 单步) | **ACT (chunk=100, z=1)** |
| --- | --- | --- |
| Transfer Cube | 25% | **80%** |
| Bungee Cord | 5% | **70%** |
| Dice Tray | 0% | **60%** |
| Open Drawer | 50% | **85%** |
| Tape | 0% | **60%** |
| Cube | 0% | **50%** |

**结论**：ACT 在所有任务上**远超** vanilla BC，平均提升 ~50 个百分点。在 Tape、Dice、Cube 上 BC 几乎为 0（精细任务），ACT 拉到 50–60%。

### 3. 协变量偏移实证

论文用 **"DAgger vs ACT"** 对照：DAgger 是经典反协变量偏移方法（推理时用专家纠正）。结果显示：

- 在这些任务上**专家无法做 DAgger**（ALOHA 主臂无法实时纠正从臂的偏移），所以 DAgger 不可行。
- **ACT 的 chunk 思路在不依赖 DAgger 的前提下达到了类似效果**——这是它的工程价值。

### 4. 消融实验

| 消融 | 结果 |
| --- | --- |
| 去掉 CVAE（仅 Transformer + L1，单模态） | 成功率掉 10–20%（多模态任务掉得更多） |
| chunk 大小 $k$ | $k=1$（退化为单步 BC）→ 接近 BC 基线；$k=100$ 最优；$k$ 过大无额外收益 |
| 去掉 temporal ensemble | chunk 边界跳变明显，成功率掉 5–15% |
| $d_z$（latent 维度） | $d_z=1$ 已足够；过大反而变差（过拟合） |
| 历史观测帧数 | 双臂协同任务对历史敏感；单帧显著掉点 |

### 5. Transfer 任务（跨物体/姿态泛化）

将 ACT 训练于固定姿态，迁移到**不同初始位置/不同物体颜色**的设置：

- Transfer Cube 在新位置上成功率 ~50–60%（训练位置 80%）；
- 显示 ACT 学到的是**相对动作策略**而非过拟合到绝对坐标——这得益于 chunk 的平滑性。

---

## 贡献与意义

1. **ALOHA 让双臂精细操作民主化**：2 万美元硬件 + 开源设计，催生后续 ALOHA 2、Gello、UMI（Universal Manipulation Interface）等一系列低成本采集系统。
2. **Action Chunking 成为 IL 标准技巧**：在 ACT 之后，**预测 chunk 而非单步**成为 Diffusion Policy、π₀、OpenVLA 微调的默认设计。
3. **CVAE 作为轻量多模态建模**：相比 diffusion 的多步采样，ACT 用 CVAE 一次前向产出多模态 chunk，推理极快（30+ Hz）。
4. **理论上的协变量偏移对策**：用 chunk 替代 DAgger，对**专家无法在线纠正**的真机任务尤其重要。
5. **示范了"低数据 + 好架构"的可行性**：50 条 demo 训出可用策略，与 DP 的百级、VLA 的万级形成对比。

---

## 局限

1. **任务相对窄**：6 个任务都在桌面、双臂可见；长程、跨场景泛化未验证。
2. **CVAE 多模态表达力弱于 Diffusion**：$d_z=1$ 只能建模少数模式，复杂多解任务不如 DP。
3. **依赖 ALOHA 风格遥操**：被动视觉采集的数据（如 OXE）不一定适合 ACT 直接训练。
4. **chunk 大小需任务定制**：$k$ 太小退化 BC，太大失去闭环；不是自适应的。
5. **没有显式语言条件**：纯 imitation，无法接受新指令——这是 Mobile ALOHA、π₀ 后续补的方向。
6. **真机频率受限于硬件**：ViperX 50 Hz 上限，超高频任务不行。

---

## 个人评价

ACT 是一篇**"简单到极致、却极其有效"**的论文。它的核心 idea——预测 chunk——一句话就能讲清，但工程上做到了 RSS best-paper 级别的扎实：50 条 demo、6 个任务、双臂真机、对照消融全齐。读这篇最大的感受是 Chelsea Finn 组的**问题驱动风格**：不是为了用 Transformer/VAE 而用，而是为了解决协变量偏移这个真问题，chunk + CVAE 恰好够用。

对我自己的研究启发：

1. **任何 BC baseline 都该先试 chunk**：哪怕不用 CVAE，单纯把单步回归改成 $k=10$ chunk 预测，通常就能涨 10+ 个点——这是 ACT 给我的最直接迁移经验。
2. **$d_z=1$ 已经够用**：别迷信复杂 latent，多模态建模用最小 latent 往往最稳。
3. **ALOHA 思路启发了"采集即演示"**：当采集系统便宜到 2 万美元，数据量瓶颈就从硬件变成了**任务多样性**——这影响了我对数据策略的优先级。
4. **ACT 与 DP 是互补的**：ACT 快、轻、小数据友好；DP 表达力强、多模态好。在真机任务上，我倾向于先试 ACT，跑不动再上 DP。
5. **Temporal ensemble 是免费午餐**：重叠 chunk 的高斯平滑几乎不增计算，应该作为 chunk 方法的默认配置。

后续 Mobile ALOHA（2024）把 ACT 装上移动底盘 + 协同训练（co-training），是这个工作的直接延伸，强烈建议连着读。

---

## 相关概念互链

- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) — IL 框架，ACT 是其中代表
- [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md) — ACT 本质是 BC + chunk + CVAE
- [`../../concepts/mdp/action-space.md`](../../concepts/mdp/action-space.md) — 动作空间与 chunk 的关系
- [`../diffusion-policy/2023-chi-diffusion-policy.md`](../diffusion-policy/2023-chi-diffusion-policy.md) — 同年并发、思路互补（多模态动作建模的两条路）
- [`../vla/2024-openvla.md`](../vla/2024-openvla.md) — OpenVLA 借鉴 ACT 的 chunk 思想
- [`../../hardware/arms/ur5e.md`](../../hardware/arms/ur5e.md) — 双臂任务的另一常用平台
