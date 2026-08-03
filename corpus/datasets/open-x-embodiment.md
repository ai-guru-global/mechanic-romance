# Open-X-Embodiment（OXE）数据集

> 由全球 21 个机构联合发布的**跨本体机器人操作数据集**，是具身智能走向「通用模型」的里程碑。RT-X 论文，2023。

## 基本信息

- **规模**：100 万+ episodes / 22+ 种机器人本体。
- **任务**：操作为主（抓取、放置、开抽屉、推物等）。
- **格式**：RLDS（Reinforcement Learning Datasets），TensorFlow Datasets 生态。
- **许可**：各子集不同，多为研究用途许可。
- **下载**：[openx-embodiment.github.io](https://robotics-transformer-x.github.io/)

## 为什么重要

在 OXE 之前，机器人数据**碎片化**：每个实验室用不同机器人、不同格式、不同任务，无法联合训练。OXE 第一次把数据统一成「同一种格式、覆盖多种本体」，使得：

1. **跨本体泛化**：在 A 机器人上训练，迁移到 B 机器人成为可能。
2. **训练通用 VLA**：RT-2、OpenVLA、Octo 等都以 OXE 为预训练语料。
3. **Scaling**：数据规模首次达到「可训练大模型」的量级。

## 本体（Embodiment）多样性

涵盖从单臂到双臂、从夹爪到灵巧手的多种形态：

- 单臂 + 夹爪：Franka、UR5e、Kuka iiwa、xArm……
- 双臂：ALOHA、TidyBot。
- 移动机械臂： Everyday Robots。
- 仿真数据：RT-1 模拟、RoboMimimic 等。

> 见 [`../concepts/foundations/embodiment.md`](../concepts/foundations/embodiment.md)：本体多样性正是「跨本体泛化」的前提。

## 数据结构（RLDS）

每个 episode 是一条轨迹：
```
episode = {
  steps: [
    { observation: {image, natural_language_instruction, ...},
      action:      {world_vector, rotation_delta, gripper_closed, ...},
      reward, is_terminal },
    ...
  ]
}
```

- `world_vector`：末端位置增量。
- `rotation_delta`：姿态增量。
- `natural_language_instruction`：语言指令（VLA 的关键输入）。

## 使用注意

- **异构动作空间**：不同本体的动作维度、语义不一致，训练前需统一归一化（OXE 的 `dataset2dataset` 映射）。
- **质量不均**：子集质量差异大，混合训练需按比例采样。
- **下载量大**：全量数十 TB，可只下载子集（如 BridgeData V2、DROID）。

## 相关方法（都建立在 OXE 上）

- [视觉-语言-动作模型](../methods/vision-language-action.md)（VLA 的数据基石）
- RT-2 / OpenVLA / Octo

## 相关概念

- [`../concepts/foundations/embodiment.md`](../concepts/foundations/embodiment.md)
- [`../concepts/mdp/action-space.md`](../concepts/mdp/action-space.md)

## 论文

- *RT-X: Open Large-Scale Study of Generalist Robots*, 2023.（arXiv:2310.08864）
