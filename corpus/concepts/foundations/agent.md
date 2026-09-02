# 智能体（Agent）

> 在环境中**感知、决策、行动**的主体。具身智能中，智能体 = 一个有身体的决策者。

## 直觉

智能体是「闭环」里主动的一方。它接收来自环境的观测（observation），依据内部策略（policy）选择动作（action），动作再作用于环境。

```
观测 o ──► [ 智能体 π ] ──► 动作 a ──► 环境
```

- 在机械臂抓取里，智能体 = 控制器 + 策略网络。
- 在 LLM Agent 里，智能体 = 语言模型 + 工具调用循环。

## 形式化（MDP 视角）

在马尔可夫决策过程（MDP）中，智能体由**策略函数** π 定义：

$$
a_t = \pi(o_t) \quad \text{或} \quad a_t \sim \pi(\cdot \mid o_t)
$$

智能体的目标是最大化累计奖励：

$$
\max_\pi \; \mathbb{E}\left[ \sum_{t=0}^{T} \gamma^t r_t \right]
$$

## 两种策略表示

| 类型 | 形式 | 来源 |
| --- | --- | --- |
| 确定性策略 | $a = \pi(o)$ | 常用于模仿学习 |
| 随机性策略 | $a \sim \pi(\cdot\mid o)$ | 强化学习的标准设定 |

## 具身智能体的特殊之处

- **形态绑定**：策略的输出维度 = 本体自由度（见 [`degrees-of-freedom.md`](degrees-of-freedom.md)），换机器人往往要改动作空间。
- **多模态观测**：同时处理图像、本体感知、语言指令。
- **连续动作**：动作通常是连续的关节角/末端速度，而非离散 token。

## 相关概念

- [`embodied-ai.md`](embodied-ai.md)（具身智能）
- [`../mdp/policy.md`](../mdp/policy.md)（策略）
- [`../mdp/observation-space.md`](../mdp/observation-space.md)（观测空间）

## 参考

- Sutton & Barto《Reinforcement Learning: An Introduction》第 2 版——智能体-环境交互框架（来源：http://incompleteideas.net/book/the-book-2nd.html，访问于 2026-09-02）
