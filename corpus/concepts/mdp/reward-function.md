# 奖励函数（Reward Function）

> 衡量智能体动作**好坏的标量信号**。记作 $r_t = R(s_t, a_t)$。它是强化学习的「老师」。

## 直觉

强化学习里，智能体不知道「正确动作」是什么（没有演示），只知道每个动作得了多少分。奖励函数告诉它「什么该做、什么不该做」——**奖励设计直接决定学出的行为**。

```
动作 a ──► 环境 ──► 奖励 r ──► 智能体（据此调整策略）
```

## 形式化

强化学习的目标是最大化**累计折扣奖励**：

$$
J(\pi) = \mathbb{E}_\pi \left[ \sum_{t=0}^{T} \gamma^t r_t \right], \quad \gamma \in [0,1)
$$

- $\gamma$：折扣因子，越关注长远 $\gamma$ 越接近 1。
- $r_t = R(s_t, a_t)$（或 $R(s_t, a_t, s_{t+1})$）。

## 稀疏 vs 稠密奖励

| 类型 | 特点 | 例 |
| --- | --- | --- |
| **稀疏** | 只在结束时给信号 | 抓到物体 +1，否则 0 —— 难学（多数 episode 0 分，无梯度） |
| **稠密** | 每步都有信号 | 末端到目标的距离（越近分越高）—— 易学，但设计困难 |

具身任务天然**稀疏**（成功与否只在最后判定），这是 RL 在机器人上难用的主因之一。

## 具身任务的常见奖励组成

$$
r = w_1 \cdot r_{\text{任务进度}} + w_2 \cdot r_{\text{姿态正则}} - w_3 \cdot r_{\text{能耗}} - w_4 \cdot r_{\text{碰撞惩罚}}
$$

- 任务进度：到目标距离的减少。
- 姿态正则：避免奇异姿态/抖动。
- 能耗惩罚：鼓励顺滑省力。
- 碰撞惩罚：安全约束。

## 奖励设计两大陷阱

1. **奖励投机（reward hacking）**：智能体找到漏洞刷分，而非完成任务。例：奖励「球进入框」→ 智能体把框翻过来扣住球。
2. **奖励塑造（reward shaping）的代价**：加稠密引导能加速学习，但易引入人为偏差。

> 这也是为什么**模仿学习**（不要奖励、用演示）在机器人操作中更流行。见 [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)。

## 相关概念

- [`policy.md`](policy.md)（策略，被奖励评估）
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)（强化学习，奖励驱动）

## 参考

- Sutton & Barto《Reinforcement Learning: An Introduction》第 2 版——奖励假设与奖励塑造（来源：http://incompleteideas.net/book/the-book-2nd.html，访问于 2026-09-02）
