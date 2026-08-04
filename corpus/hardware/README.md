# 硬件 · Hardware

> 具身智能的**物理底座**：机械臂/人形等本体、传感器、执行器、计算平台。脱离硬件谈具身智能是空谈——形态决定能力上限。

---

## 子分类

| 子目录 | 收录 | 状态 |
| --- | --- | --- |
| [`arms/`](arms/) | 机械臂本体（Franka / UR / KUKA / xArm 等） | 🟡 充实中 |
| [`humanoids/`](humanoids/) | 人形机器人（Figure / Tesla Bot / 宇树 H1 / Optimus 等） | 🟡 充实中 |
| [`quadrupeds/`](quadrupeds/) | 四足机器人（Spot / ANYmal / 宇树 Go2） | ⚪ 待建 |
| [`end-effectors/`](end-effectors/) | 末端执行器（夹爪 / 灵巧手 / 吸盘 / 柔性手） | ⚪ 待建 |
| [`sensors/`](sensors/) | 传感器（RGB-D / LiDAR / 触觉 / 力矩 / IMU） | ⚪ 待建 |
| [`compute/`](compute/) | 计算平台（Jetson / 工作站 GPU / 边缘部署） | ⚪ 待建 |

## 硬件选型的核心维度

| 维度 | 关键参数 | 影响 |
| --- | --- | --- |
| 自由度 | DoF 数 | 任务复杂度上限 |
| 负载 | 末端有效载荷 | 能搬多重的物体 |
| 重复精度 | ±mm 级 | 装配/精密操作可行性 |
| 工作空间 | 臂展/可达范围 | 任务场景尺度 |
| 控制频率 | Hz | 是否能做力控/动态任务 |
| 传感器接口 | 是否带力矩/触觉 | 能否做接触式任务 |

## 与其他模块的关系

- 硬件决定 [`concepts/foundations/embodiment.md`](../concepts/foundations/embodiment.md) 与 [`concepts/mdp/action-space.md`](../concepts/mdp/action-space.md)。
- 在 [`simulation/`](../simulation/) 中被建模（URDF/MJCF）。
- [`industry/`](../industry/) 里的公司是这些硬件的制造者。
