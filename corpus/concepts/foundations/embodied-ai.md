# 具身智能（Embodied AI）

> 通过**身体**（物理或仿真的）与**环境**交互来感知、理解并完成任务的智能体。

## 直觉

传统 AI（如语言模型）在「数字世界」里处理符号：文本进、文本出。
具身智能强调智能体必须**有身体、在世界中**——它用传感器看，用执行器动，动作又会改变世界，从而形成「感知 → 决策 → 行动 → 新感知」的闭环。

```
        感知 (perception)
      ○─────────────────────►  智能体 (agent)
      ▲                          │
      │                          │ 动作 (action)
      │                          ▼
     环境 (environment) ◄─────────
```

这个闭环是具身智能区别于「纯推理 AI」的本质特征。

## 三个关键要素

1. **本体（embodiment）**：智能体的身体形态决定它能做什么。机械臂与轮式机器人能完成的任务截然不同。见 [`embodiment.md`](embodiment.md)。
2. **环境（environment）**：任务发生的物理/仿真世界，含物体、几何、物理规律。
3. **任务（task）**：智能体要达成的目标，通常用自然语言或目标状态定义。

## 与相邻领域的关系

| 领域 | 关系 |
| --- | --- |
| 机器人学（Robotics） | 具身智能的**载体**和**硬件基础**；具身智能更强调「学习」与「通用性」 |
| 强化学习（RL） | 具身智能常用的**学习范式**之一，但具身智能不限于 RL |
| 计算机视觉（CV） | 提供感知能力（视觉是主要观测模态） |
| 自然语言处理（NLP） | 提供指令理解能力（语言条件任务） |

## 为什么现在火

- **基础模型**（大语言/视觉模型）提供了强大的表征与推理底座。
- **数据规模**（如 Open-X-Embodiment）首次让跨本体训练成为可能。
- **算力**让端到端策略学习可行。

## 代表方向

- 操作（Manipulation）：抓取、装配、工具使用
- 导航（Navigation）：移动、探索、避障
- 视觉-语言-动作（VLA）：用大模型驱动机器人

## 相关概念

- [`embodiment.md`](embodiment.md)（本体）
- [`agent.md`](agent.md)（智能体）
- [`degrees-of-freedom.md`](degrees-of-freedom.md)（自由度）

## 代表方法 / 论文

- [视觉-语言-动作模型](../../methods/vision-language-action.md)
- 综述：*Foundation Models for Robotics*（见 [`../surveys/`](../../surveys/)）

## 参考

- RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control（Google DeepMind 2023）——具身大模型代表作（来源：https://arxiv.org/abs/2307.15818，访问于 2026-09-02）
- Open X-Embodiment: Robotic Learning Datasets and RT-X Models（2023）——大规模跨本体机器人学习（来源：https://arxiv.org/abs/2310.08864，访问于 2026-09-02）
