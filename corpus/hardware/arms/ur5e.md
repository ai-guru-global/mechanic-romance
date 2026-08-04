# Universal Robots UR5e 6-DoF 协作机械臂

> **一句话定位**：丹麦 Universal Robots（UR）旗下的中量级**协作机器人（cobot）**——工业部署最广、学术与产线双用的 6-DoF 臂，以 PolyScope 图形示教、e 系列力矩传感与 RTDE 实时接口著称，是「能用而非能研」的代表。

> 最后更新：2026-08

---

## 1. 概览

UR5e 是 Universal Robots「e 系列（e-Series）」的中型机型，2018 年取代初代 UR5。「e」相对前代的关键升级是：内置**力矩传感（FT300 级）**、改进的关节设计、更高精度与新的 RTDE 实时数据接口。UR 系列（UR3e / UR5e / UR10e / UR16e / UR20 / UR30）共享同一套软件栈（PolyScope + URScript + RTDE），换机型只改参数。

定位：**协作机器人（collaborative robot, cobot）**——设计上允许与人共享工作空间，靠**力限制（power-and-force limiting）**与**碰撞检测**保证安全，无需传统工业臂的安全围栏。这使它既能进工厂做机器看护/螺丝/装配，也能进实验室做操作任务。

- 工业属性强：图形化示教器 PolyScope 让产线工人也能编程；
- 学术属性中强：RTDE + URScript 提供 500 Hz 实时通道，足够做 IL/BC，但力控带宽不如 Franka。

---

## 2. 整机规格

| 参数（Parameter） | 数值（Value） | 备注 |
| --- | --- | --- |
| 自由度 DoF | 6 | 全旋转关节 |
| 末端有效载荷 Payload | **5 kg** | e 系列；初代 UR5 为 5 kg |
| 最大工作半径 Reach | **850 mm** | 末端到基座 |
| TCP 最大线速度 | 1 m/s | 笛卡尔空间 |
| 重复精度 Repeatability | **±0.05 mm** | ISO 9283；初代 UR5 为 ±0.1 mm |
| 自重 | 20.6 kg（含线缆） | 可单人搬运 |
| 关节范围 | ±360°（多数关节） | 远超常规臂，可达性极好 |
| 安装姿态 | 任意（地面/倒挂/壁挂/倾斜） | |
| 力矩感知 | **末端力矩传感器（内置 FT）** + 关节电流估算 | e 系列特性 |
| TCP 力测量范围 | ±200 N（力）/ ±30 N·m（力矩） | 标称 |
| 安全功能 | 力/功率限制、碰撞检测、安全 I/O、安全停止（STO） | EN ISO 10218 / TS 15066 |
| 通信 | TCP/IP（URScript socket）、**RTDE @ 500 Hz**、EtherNet/IP、PROFINET、Modbus TCP | |
| 防护等级 | IP54（标准）/ IP67 可选 | 可用于轻工业环境 |
| 示教器 | **PolyScope 3+**（触摸屏） | 图形化编程 |

> 关键差异点：UR5e 的精度（±0.05 mm）**优于** Franka Panda（±0.1 mm），且关节范围达 ±360°，但自由度只有 6（无冗余）。

---

## 3. 六关节模组参数表

UR5e 的 6 个关节为 `shoulder_pan`、`shoulder_lift`、`elbow`、`wrist_1`、`wrist_2`、`wrist_3`。各关节运动范围与转矩：

| 关节 | 名称 | 运动范围 | 最大速度 | 最大转矩 |
| --- | --- | --- | --- | --- |
| joint1 | shoulder_pan（肩旋转） | ±360° | 2.094 rad/s (120°/s) | 150 N·m |
| joint2 | shoulder_lift（肩俯仰） | ±360° | 2.094 rad/s (120°/s) | 150 N·m |
| joint3 | elbow（肘） | ±360° | 3.141 rad/s (180°/s) | 150 N·m |
| joint4 | wrist_1（腕俯仰） | ±360° | 3.141 rad/s (180°/s) | 28 N·m |
| joint5 | wrist_2（腕翻滚） | ±360° | 3.141 rad/s (180°/s) | 28 N·m |
| joint6 | wrist_3（末端翻滚） | ±360° | 3.141 rad/s (180°/s) | 28 N·m |

要点：

- **所有关节 ±360° 无限回转（infinite rotation within range）**——这是 UR 的标志性设计，意味着关节可以「绕过去」走更短路径，避奇异能力极强（代价是没有 7-DoF 冗余）。
- **肩肘大力矩、腕部小力矩**：与 Panda 同构，符合重力分布。
- 关节内置**谐波减速器 + 编码器 + 制动器**，每个关节都是独立模组，可现场更换。
- **力矩感知不在关节而在末端工具法兰**：UR5e 在法兰处内置 FT 传感器，测量 TCP 受力；关节级转矩靠电机电流估算。这与 Panda「每关节一路力矩」是两种不同哲学。

---

## 4. 关键技术解析

### 4.1 协作机器人安全模型（Power-and-Force Limiting）

UR5e 是 TS 15066（协作机器人安全标准）意义上的**力限制型 cobot**。其安全机制包括：

1. **力/功率限制**：碰撞时机械功率被限制在人体可承受阈值内（按身体部位分类，如手/胸/头阈值不同）。
2. **碰撞检测与停机/退让**：检测到外力超过阈值即触发安全停止（category stop）或柔顺退让。
3. **安全 I/O 与 STO**：安全扭矩关闭（Safe Torque Off）、安全停止（Safe Stop），满足 SIL3/PLd。
4. **限速与限位（safety boundaries）**：可在 PolyScope 里设关节/TCP 速度与工作空间包围盒。

> 这意味着 UR5e 可以**不围安全围栏**直接与人并肩工作——这是它在 SME（中小企业）产线大杀四方的原因。Panda 没有这套认证，本质是研究臂。

### 4.2 PolyScope 图形界面

PolyScope 是 UR 的**触摸屏示教器图形界面**，允许用拖拽/录制方式编程：

- 录制路点（waypoint）→ 直线/关节插值 → 循环执行；
- 内置夹爪/传送带/力控等「模板（URCaps）」；
- 支持 If/Else、循环、变量——是个简化版脚本环境。

对非程序员友好，是「产线工人也能编程」的关键。研究用户通常跳过它，直接走 URScript/RTDE。

### 4.3 RTDE 实时数据接口

**RTDE（Real-Time Data Exchange）**是 e 系列引入的同步实时接口：

- **500 Hz** 周期性读写（控制环基频）；
- 用户可选择读写哪些字段（如 `actual_q`、`target_q`、`actual_TCP_force`、`speedj` 输入）；
- 通过 TCP/IP 同步，无需 EtherCAT 专用卡。

相比 Panda 的 1 kHz EtherCAT，UR5e 的 500 Hz + TCP/IP 带宽稍低，但**无需实时内核、跨平台友好**，对中等带宽的 IL/BC 任务足够。力控任务（高带宽阻抗）仍以 Panda 为优。

### 4.4 末端 FT 传感的取舍

UR5e 把力矩感知放在**末端**而非关节，带来两个后果：

- **TCP 力直接可读**：`actual_TCP_force` 直接给出末端 6 维力/力矩，做「推/拉/插」类接触任务时编程简单。
- **关节级动力学不透明**：无法像 Panda 那样做高保真整臂动力学补偿，因此**高带宽关节力控**能力弱于 Panda。

这是「工业易用」与「研究力控」的根本取舍。

---

## 5. 控制接口与软件生态

### 5.1 编程层级

| 层级 | 接口 | 说明 |
| --- | --- | --- |
| 图形 | **PolyScope**（示教器） | 录点/模板，零代码 |
| 脚本 | **URScript** | 机器人内置脚本语言，`movel`/`movej`/`speedl`/`force_mode` |
| 实时 | **RTDE**（500 Hz） | 周期读写控制/状态字段 |
| 套接字 | URScript `socket_open`/`socket_send` | 外部 PC 发指令 |
| ROS | `ur_robot_driver`（ROS1/ROS2） | 集成 MoveIt，社区主力 |
| 外设 | **URCaps** | 第三方插件（夹爪/视觉/力控） |

### 5.2 关键 URScript 指令

| 指令 | 作用 |
| --- | --- |
| `movej(q, a, v)` | 关节空间运动（joint move） |
| `movel(pose, a, v)` | 笛卡尔直线运动（linear move） |
| `speedj(qd, a, t)` | 关节空间速度控制（实时） |
| `speedl(xd, a, t)` | 笛卡尔速度控制（实时） |
| `force_mode(task_frame, selection, wrench, type, d)` | **力控模式**：沿指定轴施加目标力/位姿 |
| `get_actual_tcp_pose()` | 读末端位姿 |
| `rtde_set_watchdog()` | RTDE 看门狗，断连安全停机 |

`force_mode` 是 UR 力控的核心：可令末端在某轴上保持恒力（如恒定下压力做抛光），其余轴位控。这是工业力控任务的常用模式。

### 5.3 生态广度

UR 的第三方生态是协作机器人里**最庞大**的：Robotiq、OnRobot、Schunk、Zimmer 等夹爪/工具均有 URCaps；视觉（Pickit、Photoneo）、力控（ATI、Robotiq FT）、传送带跟踪、AGV 对接均有成熟方案。这对学术用户意味着「**今天到货明天能跑**」。

---

## 6. 应用场景

| 场景 | 为什么选 UR5e |
| --- | --- |
| **机器看护（machine tending）** | CNC 上下料，5 kg 够用，安全免围栏 |
| **螺丝/点胶/焊接** | PolyScope 易编程，力控模式保接触力恒定 |
| **电子装配/插件** | ±0.05 mm 精度 + force_mode，半柔顺插装 |
| **学术 IL/BC 实验** | RTDE + ur_robot_driver，ROS 集成成熟，开箱即用 |
| **教学（高校/职教）** | 安全 + 易用 + 廉价，是「第一台机器人」首选 |
| **医疗/实验室自动化** | IP54/67、力限制、安静（无大减速噪音） |
| **Pick-and-place** | 配 Robotiq 2F-85，电商分拣经典组合 |

不擅长：重载（>5 kg 用 UR10e/UR16e/UR20）、高带宽关节力控（用 Franka）、冗余避障（6-DoF 无冗余）。

### 6.1 学术里的特殊地位

虽然 Franka 在「操作任务研究」占比更高，但 UR5e 在两类研究里仍是首选：

1. **需要直接进产线/真工业场景的研究**（如工厂IL数据采集）——UR5e 是产线标配，复用现有工具链。
2. **强调整体易用与稳定性的教学型 IL**——不需折腾 libfranka 实时回调，Python 走 RTDE 即可。

---

## 7. 与同类对比：UR5e vs Franka Panda

| 维度 | UR5e | Franka Panda（FR3） |
| --- | --- | --- |
| 自由度 | 6（无冗余） | **7（冗余）** |
| 负载 | **5 kg** | 3 kg |
| 工作半径 | 850 mm | 855 mm（接近） |
| 关节范围 | **±360°（极广）** | ±166–175°（常规） |
| 重复精度 | **±0.05 mm** | ±0.1 mm |
| 力矩感知 | 末端 FT（内置）+ 电流估算 | **每关节力矩传感器** |
| 控制环 | 500 Hz（RTDE/TCP） | **1 kHz**（EtherCAT） |
| 力控能力 | 中（force_mode，工业级） | **强**（阻抗/导纳，研究级） |
| 用户界面 | **PolyScope 图形**（零代码） | 代码优先（libfranka） |
| 安全认证 | **TS 15066 cobot**（免围栏） | 研究臂（无完整 cobot 认证） |
| 第三方生态 | **极广**（URCaps） | 中（ROS 社区） |
| 工业部署 | **极广** | 少 |
| 学术占比 | 操作任务常见 | **操作任务 Top 1** |
| 单价（量级） | ~3.5 万美元起 | ~3–4 万美元 |

**一句话总结**：

- 要**产线稳定、易编程、安全合规、精度高** → UR5e。
- 要**冗余自由度、关节力控、研究带宽** → Franka Panda。

很多实验室两台都买：UR5e 做工程化 IL，Panda 做力控 RL。详见 [`franka-panda.md`](franka-panda.md)。

---

## 8. UR 系列谱系（选型参考）

| 型号 | 负载 | 臂展 | 定位 |
| --- | --- | --- | --- |
| UR3e | 3 kg | 500 mm | 桌面级，紧凑任务 |
| **UR5e** | **5 kg** | **850 mm** | **中量级主力，最通用** |
| UR10e | 12.5 kg | 1300 mm | 大件/码垛 |
| UR16e | 16 kg | 900 mm | 重载紧凑 |
| UR20 | 20 kg | 1750 mm | 新一代大臂展 |
| UR30 | 30 kg | 1300 mm | 重载中臂展 |

学术操作任务默认选 UR5e（与 Panda 同量级便于对比）；需要更大臂展或负载时上 UR10e/UR20。

---

## 9. 常见坑与实战经验

- **RTDE 同步与看门狗**：必须设 `rtde_set_watchdog`，否则 PC 卡顿会触发急停。
- **force_mode 的轴选择**：`selection_vector` 决定哪些轴力控、哪些轴位控，设错会把机器人推飞——务必仿真先验证。
- **碰撞阈值标定**：默认安全阈值偏保守，重负载时频繁误触发；可按 TS 15066 做碰撞测试后调参。
- **夹爪配套**：UR + Robotiq 2F-85 是黄金组合，URCaps 即插即用。见 [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md)。
- **TCP 校准**：换工具后必须 `set_tcp()`，否则位姿/力数据全错。
- **奇异点**：腕部对齐时 movel 会减速甚至停（「wrist singularity」），可用 `movej` 或设 `get_inverse_kin` 容错。

---

## 10. 相关概念互链

- [`../../concepts/foundations/degrees-of-freedom.md`](../../concepts/foundations/degrees-of-freedom.md)——为什么 UR5e 是 6-DoF 非冗余臂。
- [`../../concepts/foundations/embodiment.md`](../../concepts/foundations/embodiment.md)——本体决定能力上限。
- [`../../concepts/mdp/action-space.md`](../../concepts/mdp/action-space.md)——关节空间 vs 笛卡尔空间。
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) / [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md)——UR5e 是工业场景 IL 数据采集主力。
- [`franka-panda.md`](franka-panda.md)——同台对比（协作 vs 研究）。
- [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md)——UR5e 标配夹爪。

---

## 11. 参考链接

- Universal Robots 官方：https://www.universal-robots.com/products/ur5-robot/
- UR5e 技术参数（官方 PDF）：https://www.universal-robots.com/articles/ur/technical-specifications-ur5e/
- RTDE 协议文档：https://www.universal-robots.com/articles/ur/interface-communication/real-time-data-exchange-rtde-guide/
- URScript 手册（官方，需注册）
- `ur_robot_driver`（ROS1/ROS2）：https://github.com/UniversalRobots/Universal_Robots_ROS_Driver / https://github.com/UniversalRobots/ur_robot_driver
- `ur_simulation`（URSim / Gazebo）：https://github.com/UniversalRobots/Universal_Robots_ROS2_Gazebo_Simulation
- PolyScope / URCaps 开发者门户：https://plus.universal-robots.com/
- 安全标准：ISO 10218、ISO/TS 15066（协作机器人）
