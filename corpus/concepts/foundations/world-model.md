# 世界模型（World Model）

> 智能体**内部对环境动态的预测模型**——给定当前状态与动作，预测下一步状态（及奖励、观测）。让 agent 能在「脑内」想象未来、规划动作，而不必每一步都与真实环境交互。

> 最后更新：2026-08

---

## 1. 直觉

强化学习的瓶颈是**样本效率**：真机交互昂贵，agent 要试错百万次才能学好。世界模型把「环境」搬进 agent 内部——学一个函数 $f$，输入「现在的状态 + 我要做的动作」，输出「下一刻会变成什么样」。有了它，agent 可以**在脑内反复推演**（imagine / dream），就像棋手在落子前在脑子里算十几步。

```
        真实环境 (real env)                     世界模型 (learned f)
   s_t ──► [ f_true ] ──► s_{t+1}          s_t,a_t ──► [ f_θ ] ──► ŝ_{t+1}
                                  脑内 rollout：可批量、可快放、零成本
```

类比：人类抓杯子前，脑子里已「看见」手握住杯子的样子；这种预测能力就是人脑的世界模型。机器人学一个等价物，就叫世界模型。

---

## 2. 为什么重要 / 在具身智能的角色

世界模型解决具身智能的三个根本矛盾：

| 矛盾 | 世界模型如何缓解 |
| --- | --- |
| **真机数据贵**（采集慢、有损耗、危险） | 在模型里 rollout 生成虚拟经验，把 1 条真机数据放大成上百条训练样本 |
| **奖励稀疏**（操作任务常常只有成功/失败） | 在模型里做 planning（MPC / 树搜索），把稀疏信号变成稠密的目标导向搜索 |
| **探索危险**（真机乱试会撞坏东西） | 在模型里「试错」，只把验证安全的动作发到真机 |

因此世界模型是**model-based RL**（基于模型的强化学习）的核心组件，也是当前通用机器人策略走向「自监督预训练」的关键基础设施（ VeLA / UniSim / Genie 的共同主线：把世界当作可预测的视频来学）。

---

## 3. 形式化

### 3.1 状态转移函数学习

标准 MDP 的转移是 $s_{t+1} \sim P(\cdot \mid s_t, a_t)$。世界模型用一个参数化函数 $f_\theta$ 去逼近它：

$$
\hat{s}_{t+1} = f_\theta(s_t, a_t) \quad \text{（确定性世界模型，如 PisDB / DreamerV1）}
$$

$$
\hat{s}_{t+1} \sim f_\theta(\cdot \mid s_t, a_t) \quad \text{（随机性世界模型，世界本身有随机性）}
$$

训练目标是让预测贴近真实转移（下一状态预测 / 观测重构）：

$$
\min_\theta \; \mathbb{E}_{(s_t,a_t,s_{t+1}) \sim \mathcal{D}} \left[ \ell\big( f_\theta(s_t,a_t),\; s_{t+1} \big) \right]
$$

### 3.2 隐空间世界模型（latent dynamics model）

直接在像素空间预测下一帧图像既贵又不准（高频细节难预测）。现代世界模型先编码到**紧凑隐空间** $z_t = e_\phi(o_t)$，再在隐空间学动态：

$$
z_t = e_\phi(o_t), \qquad \hat{z}_{t+1} = f_\theta(z_t, a_t), \qquad \hat{o}_{t+1} = d_\psi(\hat{z}_{t+1})
$$

这把「预测整个未来」从「预测百万像素」降为「预测几百维隐向量」，是 **PlaNet / Dreamer / DayDreamer** 系列的设计基石。

### 3.3 在世界模型上做 RL / Planning

学到 $f_\theta$ 后，有两种用法：

- **Planning（在线规划，如 MPC）**：每步用梯度或采样（CEM）在模型上搜索最大化回报的动作序列，再只执行第一步。
- **Dreaming（离策略 RL）**：在模型里 rollout 出大量「 imagined trajectories」，用标准 actor-critic（如 Dreamer）在这些虚拟数据上更新策略，**完全不碰真实环境**。

---

## 4. 代表方法谱系

### 4.1 经典 model-based RL（低维状态空间）

| 工作 | 思路 | 备注 |
| --- | --- | --- |
| **Pendulum / MuJoCo 上的 ME-TRPO、MB-MPO、MBPO** | 学神经网络动力学模型，短 horizon rollout 混入 off-policy 训练 | model-based 样本效率标杆 |
| **PETS / Probabilistic Ensembles** | 用集成模型量化认知不确定性，做概率 MPC | 接触式操作奠基 |

### 4.2 隐空间世界模型：Dreamer 家族（核心主线）

| 工作 | 年份 | 关键创新 |
| --- | --- | --- |
| **World Models**（Ha & Schmidhuber） | 2018 | VAE + RNN 学隐空间动态，「在梦里训练 agent」的原型 |
| **PlaNet** | 2019 | RSSM 循环状态空间模型 + CEM planning |
| **Dreamer** | 2020 | 在 imagined rollout 上做 actor-critic，不再每步 plan，效率飞跃 |
| **DreamerV2** | 2021 | 离散表征， Atari 上 SOTA |
| **DreamerV3** | 2023 | **固定超参跨域**——从 Crafter 到 Minecraft 到 proprioceptive 机器人，一套超参全打通，世界模型走向「通用基础设施」 |
| **DayDreamer** | 2022 | Dreamer 跑到真机（四足、机械臂），世界模型在真实世界首次大规模验证 |

### 4.3 生成式视频世界模型（大模型时代）

把「世界模型」从隐向量重构升级为**直接生成未来视频**：

| 工作 | 定位 |
| --- | --- |
| **UniSim**（2023） | 用互联网视频训一个通用「世界模拟器」，文本/动作条件生成物理合理的未来视频，可作机器人训练环境 |
| **Genie**（Google DeepMind, 2024） | 从无标注视频里隐式学出可控的交互式环境，输入一张图+动作即可生成可玩世界 |
| **Sora** 及后续 | 视频生成模型被视为「世界模拟器」的雏形（争议：是否真理解物理） |

---

## 5. 与 model-free RL 的对比

| 维度 | model-free RL（PPO/SAC） | model-based RL（世界模型） |
| --- | --- | --- |
| 学习对象 | 直接学策略 / 价值 $\pi, Q$ | 先学环境模型 $f_\theta$，再在其上学 $\pi$ |
| 样本效率 | 低（百万步交互） | **高**（1-2 个数量级提升） |
| 渐近性能 | 强（无模型偏差） | 受**模型误差**拖累，长 horizon rollout 容易发散 |
| 规划能力 | 无（只能反应式） | 有（可在脑内 search/lookahead） |
| 真机友好度 | 差（需海量交互） | 好（真机数据用于训模型，策略在模型里训） |
| 典型算法 | PPO、SAC、TD3 | Dreamer、MBPO、PETS |

一句话：**model-free 把世界当黑盒，model-based 想搞清楚世界的规律再决策**。世界模型的全部艺术，就是在「样本效率」和「模型误差累积」之间找平衡。

---

## 6. LLM / VLM 是不是隐式世界模型？

这是当前最具争议的问题之一。

- **乐观派**：大语言/视觉模型在预训练时见过了海量「状态转移」（文本/视频序列），内部必然编码了某种世界动力学。LeCun 等主张「**世界模型是通向 AGI 的必备组件**」，下一代架构（如 JEPA）应显式学预测。
- ** skeptics**：语言模型预测的是 **token 分布**，不是物理因果；它「知道」杯子掉下会碎，是因为语料里这么写过，而非它模拟了重力。形式上没有 $s_{t+1}=f(s,a)$ 的可干预结构。
- **具身智能的取舍**：实践中 VLA 模型（如 RT-2、OpenVLA）**不显式学世界模型**，直接端到端映射 observation→action；但 2024 起的趋势是「VLA + 显式世界模型」结合（如用视频预测做数据增广、做 lookahead 验证）。

> 参考：LeCun, *A Path Towards Autonomous Machine Intelligence* (2022)——JEPA 路线；下一篇世界模型综述见 [`../../surveys/`](../../surveys/)。

---

## 7. 具身智能里世界模型的典型用法

1. **样本增广**：真机采 100 条 demo，世界模型 rollout 出 10000 条 imagined trajectory 训策略。
2. **模型预测控制（MPC）**：高频规划，每步在模型上 search 未来 N 步最优动作（用于机械臂抓取、灵巧手 in-hand manipulation）。
3. **目标条件 planning**：给定目标图像，在世界模型里反推到达目标的动作序列（视频 planning / UniSim 式用法）。
4. **自监督预训练**：用「下一帧预测」作为无标注预训练任务，学到的表征迁移到下游策略（与 [`../perception/visual-representation.md`](../perception/visual-representation.md) 互补）。

---

## 8. 主要挑战与坑

- **模型误差累积（compounding error）**：单步预测误差小，rollout H 步后指数放大。缓解：短 horizon rollout、ensemble 不确定性加权（PETS）、在隐空间而非像素空间 rollout。
- **随机性建模**：真实世界有不可观测随机性（人推、风扰），确定性模型会「平均掉」这些。需概率模型（RSSM / 扩散式世界模型）。
- **长程一致性**：视频世界模型常出现物体凭空出现/消失，物理不一致——这是「是否真理解世界」争议的实证来源。
- **动作条件**：纯视频预测模型不接收动作输入，无法用于控制；**action-conditioned** 才是机器人可用的世界模型。

---

## 9. 相关概念互链

- [`embodied-ai.md`](embodied-ai.md)——具身智能闭环里世界模型的位置。
- [`../mdp/observation-space.md`](../mdp/observation-space.md)——世界模型的输入是观测。
- [`../mdp/policy.md`](../mdp/policy.md)——世界模型为策略提供 planning / imagination 能力。
- [`../mdp/reward-function.md`](../mdp/reward-function.md)——reward 也常并入世界模型一起预测。
- [`../perception/visual-representation.md`](../perception/visual-representation.md)——视觉编码器是世界模型的前端。
- [`../training/finetuning.md`](../training/finetuning.md)——世界模型预训练 → 下游策略微调的范式。

## 10. 代表方法 / 论文

- Dreamer 系列 → [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)（model-based RL 一节，待补专篇）
- VLA / 视频世界模型 → [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)
- 综述与经典论文：
  - Ha & Schmidhuber, *World Models* (2018, arXiv:1803.10122)
  - Hafner et al., *DreamerV3: Mastering Diverse Domains through World Models* (2023, arXiv:2302.03011)
  - Yang et al., *Learning Interactive Real-World Simulators (UniSim)* (2023, arXiv:2310.06114)
  - Bruce et al., *Genie: Generative Interactive Environments* (DeepMind, 2024, arXiv:2402.15391)

---

## 11. 参考链接

- Dreamer 官方实现与论文合集：https://danijar.com/project/dreamer/
- LeCun JEPA 路线文章：https://openreview.net/pdf?id=BZ5a1r-kVsf
- 世界模型综述：*A Survey on Visual World Models for Robotics* (2024)
