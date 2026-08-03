# 动作空间（Action Space）

> 智能体每一步**可以选择的所有动作**的集合。记作 $\mathcal{A}$。它是策略的输出。

## 直觉

如果说观测是「输入」，动作就是「输出」。机器人能怎么动，完全由动作空间界定。

```
策略 π ──► 动作 a ──► 环境（执行）
```

## 形式化

策略输出动作：$a_t \sim \pi(\cdot \mid o_t)$，其中 $a_t \in \mathcal{A}$。

## 具身智能的常见动作空间

| 控制层级 | 动作内容 | 维度 | 频率 |
| --- | --- | --- | --- |
| **关节空间** | 各关节角速度/力矩 | DoF | 高（100–1000 Hz） |
| **末端位姿** | 末端位置 + 姿态（+ 夹爪） | 6–7 | 中（10–50 Hz） |
| **末端速度** | 末端笛卡尔速度 | 6 | 中 |
| **离散技能** | 「抓」「放」「开门」 | 离散 | 低（规划层） |

现代端到端策略（如 VLA）多采用 **末端位姿增量 + 夹爪开关**，维度低、易于从演示学习。

## 连续 vs 离散

- **连续动作空间**：$\mathcal{A} \subseteq \mathbb{R}^d$，具身任务的常态。
  - 优势：精细控制。
  - 挑战：探索困难，策略输出需回归而非分类。
- **离散动作空间**：$\mathcal{A} = \{a_1, \dots, a_K\}$，如棋类、Atari。
  - 适合高层规划（选项/options）。

## 动作表示（representation）

同样的末端运动，可表示为：
- **绝对位姿**（absolute）：$a_t = \text{pose}_t$。数值大，对初始位姿敏感。
- **相对增量**（relative / delta）：$a_t = \text{pose}_t - \text{pose}_{t-1}$。数值小、泛化好，**目前主流**。

## 动作空间由本体决定

- 7-DoF 机械臂 → 动作维度至少 7（关节）或 7（末端 6 + 夹爪 1）。
- 灵巧手 → 动作维度可达 20+。
- 见 [`../foundations/degrees-of-freedom.md`](../foundations/degrees-of-freedom.md)。

## 相关概念

- [`observation-space.md`](observation-space.md)（观测空间）
- [`policy.md`](policy.md)（策略）
- [`../foundations/embodiment.md`](../foundations/embodiment.md)（本体决定动作空间）
