# Diffusion Policy：用扩散模型驱动的视觉运动策略（Diffusion Policy: Visuomotor Policy Learning via Action Diffusion）

> 把**扩散模型（DDPM）从图像生成搬到机器人动作序列生成**：不去回归"一个最优动作"，而是建模"未来一段动作序列的多模态分布"，用去噪采样产出连贯的、能容纳多解的策略输出。RSS 2023，Cheng Chi 等（哥伦比亚，Shuran Song 组）。

## 元信息

- **标题**：Diffusion Policy: Visuomotor Policy Learning via Action Diffusion
- **作者**：Cheng Chi, Siyuan Feng, Yilun Du, Zhenjia Xu, Eric Cousineau, Benjamin Burchfiel, Shuran Song
- **机构**：Columbia University（哥伦比亚大学）；MIT；Toyota Research Institute（TRI）
- **会议 / 年份**：Robotics: Science and Systems (RSS) 2023；arXiv 预印本 2023-03
- **论文链接**：https://arxiv.org/abs/2303.04137
- **项目主页**：https://diffusion-policy.cs.columbia.edu/
- **代码**：https://github.com/real-stanford/diffusion_policy
- **关键词**：diffusion model、visuomotor、behavior cloning、multimodal action、receding horizon

## BibTeX

```bibtex
@inproceedings{chi2023diffusion,
  title     = {Diffusion Policy: Visuomotor Policy Learning via Action Diffusion},
  author    = {Chi, Cheng and Feng, Siyuan and Du, Yilun and Xu, Zhenjia and
               Cousineau, Eric and Burchfiel, Benjamin and Song, Shuran},
  booktitle = {Robotics: Science and Systems (RSS)},
  year      = {2023}
}
```

## 一句话概括

Diffusion Policy 把"专家动作序列"当作"图像"去噪：用 DDPM 在观测条件下建模未来 $T_p$ 步动作的分布，推理时只执行前 $T_a$ 步再滚动（receding horizon），从而**优雅解决多模态动作、训练稳定、对动作维度的表达力远超 BC/IBC/LSTM-AE**——一跃成为精细操作的事实标准。

---

## 核心问题

### 1. 多模态动作（Multimodal Action）

这是 manipulation BC 的**头号难题**。同一观测 $o$ 下，专家可能给出**多种都正确**的动作（往左放、往右放）。传统 BC 用 MSE 回归：

$$
\hat{a} = \arg\min_a \mathbb{E}_{a^* \sim \text{data}}\|a - a^*\|^2 = \mathbb{E}[a^*]
$$

均值化的结果**既不在左也不在右**，是不可行的中间动作。这就是为什么 LSTM-AE / IBC 都试图建模分布，但前者表达力不足、后者训练不稳定。

### 2. 高维动作序列的联合建模

机械臂一条轨迹是 $T \times d$ 维（$d$=7，$T$=16+），传统生成模型（VAE、Flow）对这种高维、强时间相关的分布建模吃力。

### 3. 训练稳定性 vs 表达力

IBC（Implicit BC）能建模多模态，但**采样需要 MCMC，训练与推理都不稳**。Diffusion 给出了"训练稳 + 表达力强 + 采样可控"的三全解。

---

## 方法

### 1. 背景：DDPM 回顾

给定数据 $\mathbf{x}_0 \sim q(\mathbf{x}_0)$，定义前向加噪过程：

$$
q(\mathbf{x}_t \mid \mathbf{x}_0) = \mathcal{N}\!\big(\mathbf{x}_t;\, \sqrt{\bar{\alpha}_t}\,\mathbf{x}_0,\, (1-\bar{\alpha}_t)\mathbf{I}\big)
$$

其中 $\bar{\alpha}_t = \prod_{s=1}^{t} \alpha_s$ 是累积噪声调度。训练一个网络 $\epsilon_\theta$ 预测所加噪声：

$$
\mathcal{L}_{\text{simple}} = \mathbb{E}_{\mathbf{x}_0, t, \boldsymbol{\epsilon}}\Big[\big\|\boldsymbol{\epsilon} - \epsilon_\theta(\mathbf{x}_t, t)\big\|^2\Big]
$$

推理时从 $\mathbf{x}_T \sim \mathcal{N}(0, \mathbf{I})$ 用反向过程一步步去噪得 $\mathbf{x}_0$。

### 2. 把 DDPM 用在动作上

Diffusion Policy 把 $\mathbf{x}_0$ 替换为**未来 $T_p$ 步动作序列** $\mathbf{A} \in \mathbb{R}^{T_p \times d_a}$，并以观测 $O$ 为**条件**：

$$
\mathbf{A}_t = \sqrt{\bar{\alpha}_t}\,\mathbf{A}_0 + \sqrt{1-\bar{\alpha}_t}\,\boldsymbol{\epsilon}, \qquad \boldsymbol{\epsilon} \sim \mathcal{N}(0, \mathbf{I})
$$

$$
\mathcal{L} = \mathbb{E}_{\mathbf{A}_0, O, t, \boldsymbol{\epsilon}}\Big[\big\|\boldsymbol{\epsilon} - \epsilon_\theta(\mathbf{A}_t, O, t)\big\|^2\Big]
$$

**关键设计：条件作用（conditioning）**——把视觉特征 $f(O)$ 通过 FiLM / cross-attention 注入去噪网络。

```
观测 O ──► 冻结视觉编码器 ──► 特征 f(O) ─┐
                                          │  (FiLM / cross-attn)
噪声 A_T ──► 去噪网络 ε_θ ──► ... ──► A_0  ┘
                                          │
                                          ▼
                              取前 T_a 步执行 (receding horizon)
```

### 3. 两种网络实现

论文给出两种 backbone，各有优劣：

#### (a) CNN-based（基于 1D 时序 U-Net，借鉴 Janner 的 Diffuser）

- 把动作序列 $\mathbf{A} \in \mathbb{R}^{T_p \times d_a}$ 看作"伪图像"（时间×维度），用 1D 卷积 U-Net。
- 条件 $f(O)$ 通过 **FiLM** 逐层调制。
- **优势**：参数少、推理快；**劣势**：感受野与时间建模弱于 Transformer。

#### (b) Transformer-based（基于 Ho 的 DDPM Transformer）

- 动作序列展平为 token 序列，时间步 $t$ 用 sinusoidal embedding 加入。
- 条件 $f(O)$ 作为额外 prefix token / cross-attention key。
- **优势**：表达力强、长序列建模好；**劣势**：参数多、显存高。

> 论文默认推荐 **CNN 版本**——多数任务上和 Transformer 打平甚至更好，且推理快得多。这是经验性结论，反直觉但实用。

### 4. Receding Horizon 控制（滚动执行）

这是 Diffusion Policy **真正可用于真机**的工程关键：

- 一次去噪采样得到未来 $T_p$ 步预测（如 $T_p = 16$）。
- **只执行前 $T_a$ 步**（如 $T_a = 8$），然后**重新观测 + 重新去噪**。
- 滚动执行 = 闭环控制，纠正漂移与累积误差。

$$
\text{每 } T_a \text{ 步重新推理}:\quad O_{\text{new}} \rightarrow \mathbf{A}_0^{(1:T_p)} \rightarrow \text{执行 } \mathbf{A}_0^{(1:T_a)}
$$

> 没有 receding horizon，单次预测一长串开环执行会迅速发散。这是 DP 比传统开环 BC 鲁棒的核心机制。

### 5. 关键工程经验（论文 §5 全是宝藏）

| 设计 | 选择 | 原因 |
| --- | --- | --- |
| 视觉编码器 | **冻结 ResNet-18**（用 EPIC-Kitchens 预训练） | 解冻会过拟合且训练不稳 |
| 动作表示 | **相对动作**（相对于当前末端位姿的 delta） | 消除绝对坐标系偏置，迁移性更好 |
| 历史观测 | 用 $T_o$ 帧历史堆叠（如 $T_o=2$） | 提供速度信息，单帧丢失动态 |
| 噪声调度 | cosine schedule | 训练更稳 |
| 去噪步数 | 训练 $T=100$，**推理 DDIM 加速到 10 步** | 真机 10 Hz 可达 |
| $T_p$ / $T_a$ | $T_p=16$, $T_a=8$ | 论文经验默认 |
| 观测 | **深度 + RGB 双模态** | 深度对精细插入极重要 |

---

## 实验

### 1. 任务覆盖：11 个任务

涵盖仿真 + 真机，覆盖推动、抓取、插入、倒水、烹饪（Push-T、Can、Mug、Square、Transport、Cable 等）。

### 2. 主结果（任务成功率 %）

| 任务 | BC (L1) | IBC | LSTM-AE | **Diffusion Policy (CNN)** | **Diffusion Policy (Transformer)** |
| --- | --- | --- | --- | --- | --- |
| Push-T | 0% | 6% | 20% | **95%** | 96% |
| Can (抓取) | 12% | 41% | 58% | **96%** | 92% |
| Mug (挂杯子) | 0% | 18% | 53% | **97%** | 92% |
| Square | 0% | 0% | 33% | **84%** | 88% |
| Transport (双臂) | 12% | 19% | 45% | **78%** | 76% |
| Cable (软体) | 0% | 0% | 25% | **65%** | 60% |

**结论**：DP 在所有任务上**碾压**三个基线，尤其在多模态任务（Push-T、Square）上从 0% → 95%。这是论文的核心数据。

### 3. 真机实验

在 Franka Panda 上验证 4 个真机任务（推 T、抓 Can、双臂 Transport、Cable），全部成功跑通，部分任务（Push-T）真机成功率甚至与仿真持平。

### 4. 多模态能力可视化

论文给出 Push-T 的动作序列采样可视化：同一观测，多次采样得到**截然不同但都合理**的推动轨迹（往左上、往右下、绕弯）。这是 BC/IBC 做不到的——它们要么平均、要么模式坍塌。

### 5. 消融实验（关键经验）

| 消融项 | 结果 |
| --- | --- |
| 去掉 receding horizon（开环执行全部 $T_p$ 步） | 成功率掉 30–50%，证明滚动闭环至关重要 |
| 单帧观测（去掉历史） | 多数任务掉 10–20%，动态任务掉更多 |
| 绝对动作 vs 相对动作 | 相对动作稳定提升 5–15%，尤其在迁移任务 |
| 视觉编码器解冻 | 训练不稳，真机成功率下降 |
| $T_p$ 过短 (4) | 长程任务掉点；过长 (32) 推理慢且不增益 |
| DDIM 步数 10 vs 100 | 推理 10 步已接近 100 步性能，真机可用 |
| CNN vs Transformer | 大多数任务打平，CNN 更快；Transformer 在长序列略优 |

---

## 贡献与意义

1. **确立 diffusion-as-policy 范式**：此后 DP3、3D Diffusion Policy、EquiBot、π₀ 的 flow-missing 头都源自此。
2. **解决多模态动作的工程标准答案**：在 DP 之前，多模态是 BC 的痛点；之后，**用 DP 就够了**。
3. **开源代码库**（`diffusion_policy`，基于 robomimic）成为全社区的基线复现平台——后续所有 manipulation 论文都跟 DP 比。
4. **receding horizon + chunk 执行**这套范式被 ACT 等并发工作也采用，成为 imitation learning 的现代共识。
5. **细节论文**：§5 的工程经验（冻结视觉、相对动作、历史堆叠）被无数后续工作引用为"标准操作"。

---

## 局限

1. **推理速度**：即使 DDIM 10 步，单次前向仍慢于直接回归的 BC（毫秒级），高频任务（>30 Hz）吃力。
2. **训练数据需求**：百级到千级演示，比单步 BC 多。
3. **长程任务仍弱**：$T_p$ 有限，超长程需要层次化（Hierarchical DP 等后续工作）。
4. **条件作用简单**：原版没有显式语言条件，语义指令理解弱（不如 VLA）。
5. **多物体场景泛化差**：在 BridgeData 这类多物体 OXE 任务上不如 VLA——这是 OpenVLA 与 DP 互补的根源。
6. **去噪步数仍需调**：极端精细任务 10 步不够，需 50+ 步，影响实时性。

---

## 个人评价

Diffusion Policy 是我读过的**工程价值最高的 manipulation 论文之一**。它的理论并不新（DDPM 2020 就有了，Diffuser 2022 用过），但它把"用 diffusion 做动作"这件事做到了**真机可跑、可复现、可教学**——这才是 RSS best-paper 级别的功劳。

对我自己的研究启发：

1. **多模态动作默认上 DP**：在 manipulation 实验里，除非证明任务单模态，否则 DP 是更安全的默认选择，避免 MSE 平均化陷阱。
2. **receding horizon 是 BC 真机化的必备**：哪怕用普通回归 BC，也应该预测 chunk + 滚动执行，这是从 DP 学到的迁移经验。
3. **冻结视觉编码器 + 相对动作**是我现在所有 manipulation 实验的起手式——这两条经验省了无数调参。
4. **CNN 版本别小看**：很多人迷信 Transformer，但 DP 的 CNN 版本在多数任务上更稳更快，做实验先用 CNN 跑通再说。
5. **DP 与 VLA 互补**：DP 强在精细高频，VLA 强在语义长程——我倾向于 **VLA 做高层 + DP 做底层**的混合架构（这也是 π₀、RDT-1B 的思路）。

读这篇一定要配合跑它的开源代码——光读论文体会不到 receding horizon 在真机上的差别，跑一次 Push-T demo 立刻懂。

---

## 相关概念互链

- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md) — 方法总论（含更直观的多模态示意）
- [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md) — DP 是 BC 的高级变体
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) — IL 框架
- [`../imitation-learning/2023-zhao-act.md`](../imitation-learning/2023-zhao-act.md) — ACT：并发的"动作分块 + Transformer"方案，思路互补
- [`../vla/2023-google-rt2.md`](../vla/2023-google-rt2.md) — DP 与 VLA 的两条主流路线对比
- [`../vla/2024-openvla.md`](../vla/2024-openvla.md) — DP 在精细任务上领先 OpenVLA 的根源
- [`../../hardware/arms/franka-panda.md`](../../hardware/arms/franka-panda.md) — DP 真机验证平台
