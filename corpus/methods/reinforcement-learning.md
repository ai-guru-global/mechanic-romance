# 强化学习（Reinforcement Learning, RL）

> 智能体通过**与环境试错交互、最大化累计奖励**来学习策略。无需演示，靠奖励信号自我进化。

## 核心思想

模仿学习是「跟人学」，强化学习是「自己试」：没人告诉它正确动作，只告诉它做得好不好。

```
策略 π ──► 动作 a ──► 环境 ──► 奖励 r + 新观测 o ──► 更新 π
                              (循环，越试越好)
```

目标是最大化折扣累计奖励：

$$
\max_\pi \mathbb{E}\left[\sum_{t} \gamma^t r_t\right]
$$

## 主要分支

| 分支 | 数据来源 | 代表算法 |
| --- | --- | --- |
| **基于价值** | 学习 $Q(s,a)$ | DQN、Rainbow |
| **策略梯度** | 直接优化 $\pi_\theta$ | REINFORCE、PPO、SAC |
| **Actor-Critic** | 两者结合 | PPO、SAC、TD3 |
| **离线 RL** | 只用历史数据，不交互 | CQL、IQL、Decision Transformer |

具身智能里，**PPO / SAC** 最常用（连续动作友好），**离线 RL** 近年因能用大规模演示数据而崛起。

## RL 在机器人上的难点

1. **样本效率极低**：策略要数百万步探索；真机上不可能，所以**几乎都在仿真里训**。
2. **奖励难设计**：操作任务稀疏成功信号。见 [`../concepts/mdp/reward-function.md`](../concepts/mdp/reward-function.md)。
3. **Sim-to-Real Gap**：仿真训的策略迁到真机会失效（摩擦、质量、传感器噪声对不上）。

## Sim-to-Real（仿真到真机）

主流路径：
- **域随机化（Domain Randomization）**：训练时随机化物理参数（摩擦、质量、光照），让策略对参数不敏感。
- **系统辨识（System Identification）**：测量真机参数，把仿真调到接近真实。
- **域适应（Domain Adaptation）**：用少量真机数据微调。

## RL vs IL：何时用谁

| 场景 | 推荐 |
| --- | --- |
| 真机操作（抓取、装配） | **IL**（数据贵、探索危险） |
| 仿真 locomotion（行走、奔跑） | **RL**（可海量试错） |
| 导航、避障 | 两者皆可，RL 在仿真中训得多 |
| 有演示但想超越人类 | **IL 预训练 + RL 微调** |

## 代表算法速览

- **PPO**（2017）：on-policy，稳定好调，OpenAI 标配。
- **SAC**（2018）：off-policy，样本效率高，连续控制强。
- **Decision Transformer**（2021）：把 RL 重构为序列建模，能用离线数据。

## 相关概念

- [`../concepts/mdp/reward-function.md`](../concepts/mdp/reward-function.md)
- [`../concepts/mdp/policy.md`](../concepts/mdp/policy.md)
- [`imitation-learning.md`](imitation-learning.md)
