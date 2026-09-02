# 本体（Embodiment）

> 智能体的**物理形态**——它的身体结构、传感器布置与执行器能力。本体决定了智能体「能感知什么、能做什么」。

## 直觉

同一个「倒水」任务，对一台 7 自由度机械臂 + 平行夹爪，和一台双足人形机器人，难度与解法完全不同。**本体不是容器，而是约束与可能性的来源**：

- 两指夹爪 → 只能捏，不能包握。
- 没有手腕力矩传感器 → 难以做接触式装配。
- 轮式底盘 → 无法爬楼梯。

```
本体 = { 形态, 自由度, 传感器, 执行器, 工作空间 }
```

## 本体的关键属性

| 属性 | 含义 | 示例 |
| --- | --- | --- |
| 自由度（DoF） | 独立运动维度 | 6-DoF / 7-DoF 机械臂 |
| 末端执行器 | 末端与物体交互的部分 | 平行夹爪 / 灵巧手 / 吸盘 |
| 传感器配置 | 能感知的模态 | RGB / 深度 / 力矩 / 触觉 |
| 工作空间 | 末端可达的空间范围 | 机械臂臂展内的半球壳 |
| 运动学/动力学 | 关节→末端位姿的映射 | 正运动学（FK）/ 逆运动学（IK） |

## 为什么「本体」是核心议题

1. **跨本体泛化**：能否训练一个策略，迁移到不同形态的机器人？这是 Open-X-Embodiment 等工作的核心动机。
2. **形态决定学习难度**：高自由度灵巧手的策略空间远大于夹爪，样本效率更关键。
3. **仿真-真机差异**：本体的物理参数（质量、摩擦、齿轮间隙）若仿真建模不准，策略难以迁移（Sim-to-Real 问题）。

## 常见本体类型

- **固定臂**：Franka Panda、UR5e、KUKA —— 操作任务主力。
- **移动机械臂**：臂 + 移动底盘 —— 仓库、服务场景。
- **人形机器人**：双臂 + 双足 —— 通用但极难控制。
- **四足**：ANYmal、Spot —— 越野、导航。
- **专用本体**：灵巧手（Shadow Hand）、软体机器人。

## 相关概念

- [`degrees-of-freedom.md`](degrees-of-freedom.md)（自由度）
- [`embodied-ai.md`](embodied-ai.md)（具身智能）
- [`../mdp/action-space.md`](../mdp/action-space.md)（动作空间，由本体决定）

## 代表方法 / 数据

- 跨本体学习：Open-X-Embodiment 数据集 → [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md)

## 参考

- Modern Robotics（Lynch & Park）——机器人本体与运动学结构（来源：http://hades.mech.northwestern.edu/index.php/Modern_Robotics，访问于 2026-09-02）
- Open X-Embodiment: Robotic Learning Datasets and RT-X Models（2023）——跨本体数据与模型（来源：https://arxiv.org/abs/2310.08864，访问于 2026-09-02）
