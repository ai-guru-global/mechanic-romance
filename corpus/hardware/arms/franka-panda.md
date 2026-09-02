# Franka Emika Panda（FR3 世代）7-DoF 机械臂

> **一句话定位**：学术界操作（Manipulation）任务最常用的固定式机械臂——7 个全关节模组、底座末端双力矩传感器、转矩控制（torque control）原生开放，是 IL/BC/RL 真机实验事实意义上的「参考本体（reference embodiment）」。

> 最后更新：2026-08

---

## 1. 概览

Franka Emika Panda 由德国 Franka Emika 公司（创始团队来自 TU Munich 的 SAMI / Albu-Schäffer 体系，传承 DLR 轻型臂技术）于 2016 年发布，2017 年量产。商品名 **Panda**；2023 年起后继型号 **FR3** 取代初代 FCI（Franka Control Interface）版本，但学术界「Panda」一名已成事实标准，论文里 `franka` / `panda` / `fr3` 通常指同一系列。

它是一台 **7 自由度（7-DoF）、串联旋转关节、拟人臂构型**的**力矩感知型轻型臂（torque-controlled lightweight arm）**。其核心卖点是：

1. **冗余度（redundancy）**：7-DoF > 末端位姿 6-DoF，天然具有 1 维零空间，可避奇异、避碰撞、优化姿态。
2. **关节级转矩传感**：每个关节都有转矩传感器，可做整臂导纳/阻抗控制与接触式操作。
3. **开放控制栈**：`libfranka`（C++）、`franka_ros` / `franka_ros2`、FCI（1 kHz EtherCAT）。

---

## 2. 整机规格

| 参数（Parameter） | 数值（Value） | 备注 |
| --- | --- | --- |
| 自由度 DoF | 7 | 全旋转关节 |
| 末端有效载荷 Payload | **3 kg**（FR3 同） | 含末端夹爪则更小 |
| 最大末端线速度 | 2 m/s | 笛卡尔空间 |
| 重复精度 Repeatability | **±0.1 mm**（典型） | ISO 9283；FR3 厂商标 ±0.1 mm |
| 最大工作半径 Reach | 855 mm | 末端到基座中心 |
| 自重 | 18 kg（臂本体） | 控制器/PC 外置 |
| 安装姿态 | 任意（地面/倒挂/壁挂） | 需相应负载核算 |
| 关节最大转矩（峰值） | 见 §3 各关节表 | 短时过载 |
| 关节传感器 | 关节位置编码器 + 关节力矩传感器 | 7 路力矩 |
| 安全功能 | 关节转矩/位置双重限幅、碰撞检测、反射式退让（reflex） | 协作机器人级 |
| 通信总线 | EtherCAT @ 1 kHz 控制环 | FCI |
| 防护等级 | IP30（标准） | 工业洁净环境 |

> 设计哲学：Panda 不是「协作机器人（cobot）」意义上的「碰人就停」，而是「**力控优先**」的研究臂——它假设你懂得如何写一个安全的阻抗控制器。

---

## 3. 七关节模组参数表

Panda 的 7 个关节命名为 `joint1` … `joint7`，从基座到末端。各关节的**运动范围**与**额定/峰值转矩**如下（厂商手册值，FR3 略有差异但量级一致）：

| 关节 | 运动范围 Range | 最大速度 | 额定转矩 | 峰值转矩 | 典型质量（连杆） |
| --- | --- | --- | --- | --- | --- |
| joint1（肩-1，基座俯仰） | ±166° | 2.375 rad/s | 87 N·m | 312 N·m | ~2.9 kg |
| joint2（肩-2，肩俯仰） | ±166° | 2.375 rad/s | 87 N·m | 312 N·m | ~2.7 kg |
| joint3（肩-3，肩旋转） | ±166° | 2.375 rad/s | 87 N·m | 312 N·m | ~2.4 kg |
| joint4（肘） | ±166° | 2.375 rad/s | 87 N·m | 312 N·m | ~2.0 kg |
| joint5（腕-1，腕翻滚） | ±166° | 3.707 rad/s | 12 N·m | 41 N·m | ~1.5 kg |
| joint6（腕-2，腕俯仰） | ±166° | 3.707 rad/s | 12 N·m | 41 N·m | ~0.8 kg |
| joint7（腕-3，末端翻滚） | ±175° | 3.707 rad/s | 12 N·m | 41 N·m | ~0.5 kg |

要点解读：

- **肩肘（j1–j4）承担重力与大力矩**：4 个关节额定 87 N·m，必须驱动整臂；这就是为什么肩部笨重、末端轻灵。
- **腕部（j5–j7）小力矩快响应**：额定仅 12 N·m，用于姿态调整。这也是 Panda 在腕部「精细但脆弱」的根源——做重切削/装配时腕部容易饱和。
- **7-DoF 冗余落在 j3/j7 的「肘部姿态」上**：同一个末端位姿，肘可以「朝外」或「朝内」，用于避障或避开奇异点（singularity）。
- 每个关节内置**谐波减速器（harmonic drive）** + 无刷电机 + 力矩传感器，关节模组高度集成。

---

## 4. 关键技术解析

### 4.1 为什么 7-DoF 是「黄金冗余」

任务空间（笛卡尔位姿）= 位置 3 + 姿态 3 = 6 维。一个 7-DoF 臂执行 6-DoF 任务时有 1 维**冗余（redundancy）**，对应零空间（null-space）运动：

- 同一末端位姿下，肘部可在「上方/下方/左/右」摆动；
- 用来**避开工作空间内的障碍**、**远离奇异位形**（如腕部对齐时的雅可比降秩）、**最小化重力转矩**以省电。

6-DoF 臂（如 UR5e）没有这层冗余，一旦遇到障碍或奇异点就只能改末端轨迹。详见 [`../../concepts/foundations/degrees-of-freedom.md`](../../concepts/foundations/degrees-of-freedom.md)。

### 4.2 关节力矩传感 → 真力控

Panda 在每个关节输出端（减速器之后、连杆之前）放置了**力矩传感器**，因此能直接读到真实的关节转矩 `τ_measured`，而不是用电机电流去估算。这带来三件事：

1. **阻抗/导纳控制**：可以令末端表现出「虚拟弹簧」特性 `F = K·Δx + D·Δẋ`，做擦玻璃、插销、装配等接触任务。
2. **动力学辨识**：`τ = M(q)q̈ + C(q,q̇)q̇ + G(q)` 的各参数可被辨识与补偿，整臂动力学模型精度高。
3. **碰撞检测与柔顺退让（reflex）**：检测到外力突变时，控制器可令臂「让一下」而非硬撞，是研究级安全策略的基础。

> UR5e 的力矩感知在**基座**（通过电流估算法），Panda 在**关节**——后者带宽与精度更高，但成本也更高。

### 4.3 1 kHz 控制环与 FCI

Panda 的控制环跑在 **1 kHz**（EtherCAT），即每 1 ms 接收一次新的关节转矩/位置目标。这意味着：

- 可以做**实时力控**（阻抗控制的带宽可达几十 Hz）；
- `libfranka` 的回调模型要求用户在 1 ms 内算完并返回下一个控制量，否则触发 `communication_constraints_violation` 错误并停机。

> 1 kHz 是真机学习里「为什么必须用 C++ 写底层、用 Python 写上层策略」的根本原因。

### 4.4 动力学模型与辨识

Panda 的整臂刚体动力学遵循标准机械臂方程：

$$
\boldsymbol{\tau} = \mathbf{M}(\mathbf{q})\ddot{\mathbf{q}} + \mathbf{C}(\mathbf{q},\dot{\mathbf{q}})\dot{\mathbf{q}} + \mathbf{G}(\mathbf{q}) + \boldsymbol{\tau}_{\text{fric}} + \boldsymbol{\tau}_{\text{ext}}
$$

其中 $\mathbf{M}$ 为惯性矩阵、$\mathbf{C}$ 含科氏/向心力、$\mathbf{G}$ 为重力项。`libfranka` 的 `Model` 类直接提供这些量的解析计算（基于 URDF 参数），因此做**重力补偿**（`tau_gravity = G(q)`）、**前馈力矩**（`tau_ff = M(q)q̈_des + ...`）非常方便。这比 UR5e（动力学参数不公开）对研究更友好，也是 Panda 成为动力学/力控研究载体的关键。

### 4.5 笛卡尔阻抗控制示例

Panda 的招牌行为是**笛卡尔阻抗控制**：令末端表现为一个 6-DoF 虚拟弹簧-阻尼系统，可写出期望行为：

$$
\mathbf{F}_{\text{ext}} = \mathbf{K}_x (\mathbf{x}_{\text{des}} - \mathbf{x}) + \mathbf{D}_x (\dot{\mathbf{x}}_{\text{des}} - \dot{\mathbf{x}})
$$

通过雅可比转置 $\boldsymbol{\tau} = \mathbf{J}^\top \mathbf{F}$ 映射到关节力矩。典型刚度取值：平移 $[600, 600, 600]$ N/m、旋转 $[30, 30, 30]$ N·m/rad，对应「末端柔顺、姿态稳定」。这是擦拭、插销、人机协作推拉的标准配方。

---

## 5. 控制接口与软件生态

### 5.1 官方栈

| 层级 | 组件 | 说明 |
| --- | --- | --- |
| 底层 C++ | **`libfranka`** | 与 FC I 通信的 C++ 库，提供 `Robot`、`Model`、回调式控制循环 |
| ROS1 | **`franka_ros`** | 把 `libfranka` 包装成 `hardware_interface`，集成 MoveIt |
| ROS2 | **`franka_ros2`** | FR3 时代主推，支持 `ros2_control` |
| 描述文件 | `franka_description` | URDF / xacro / meshes，仿真必备 |
| MoveIt 集成 | `franka_moveit_config` | 运动规划 / IK |
| Gazebo / Isaac | 官方与社区插件 | `franka_gazebo`、Isaac Sim/Mujoco 模型 |

### 5.2 控制模式

- **关节位置 / 速度控制**：经典 `MotionGenerator`。
- **笛卡尔位置 / 速度控制**：末端在基坐标系或工具坐标系下运动。
- **关节力矩控制（`torques`）**：直接给 7 维 `τ`，研究力控/RL 的主力模式。
- **外部力矩叠加**：可在位置控制基础上叠加一个力矩前馈。

### 5.3 学习生态约定

学术圈对 Panda 的「事实接口」已经固化：

- **观测（observation）**：`q`（7）、`dq`（7）、`O_T_EE`（末端齐次变换 4×4=16）、`K_F_ext_hat`（外力估计 6）、图像。
- **动作（action）**：通常为 `delta_ee`（末端 6-DoF 增量）或 `delta_q`（关节增量），配合 `torques` 做力控。
- **遥操（teleoperation）**：SpaceMouse、3Dconnexion、VR 手柄、Leader-Follower 双臂。

这套约定让 ALiA、Diffusion Policy、ACT、VLA 等工作能几乎「换 URDF 就跑」。详见 [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)。

---

## 6. 应用场景

| 场景 | 为什么选 Panda |
| --- | --- |
| **模仿学习 / 行为克隆** | 1 kHz 力控 + 7-DoF 冗余，能采集高保真演示；社区有成熟遥操栈 |
| **接触式操作（插销、开门、擦拭）** | 关节力矩传感 + 阻抗控制，UR5e 难以等价复现 |
| **强化学习真机部署** | `torques` 模式直接接策略网络输出，sim-to-real 通道成熟 |
| **双手协调研究** | 双 Panda 平台（如 ALOHA 的部分变种）是双臂 IL 主力 |
| **力觉辨识与系统辨识** | 力矩可读、动力学模型开源，是「教科书级」研究对象 |
| **教学（高校实验室）** | 单价相对可控（见 §7），文档/课程/论文极多 |

不擅长：重载装配（>3 kg）、户外/粉尘（IP30）、高速喷涂（需更大工作半径）。

### 6.1 仿真资产生态

Panda 是仿真器里模型最齐全的本体之一，这是它成为 sim-to-real 主力的另一半原因：

| 仿真器 | 资产 | 说明 |
| --- | --- | --- |
| MuJoCo | `robosuite` / `mujoco_menagerie` | MJCF，含可调摩擦/阻尼，RL 主力 |
| Isaac Sim / Isaac Lab | 官方 + NVIDIA 资产 | GPU 并行仿真，RL 大规模训练 |
| PyBullet | `panda-gym` | 轻量，教学/快速原型 |
| Gazebo | `franka_gazebo` | ROS 仿真，与真机接口一致 |
| Mujoco/Genesis | 社区维护 | 新兴可微/并行仿真 |

由于真机 URDF 与仿真 MJCF 高度一致，许多工作能实现「仿真训练 → 真机零误差部署」（zero-shot sim-to-real），这在 UR/KUKA 上更难复现。

---

## 7. 与同类对比：Panda vs UR5e

| 维度 | Franka Panda（FR3） | Universal Robots UR5e |
| --- | --- | --- |
| 自由度 | **7**（冗余） | 6（无冗余） |
| 负载 | 3 kg | **5 kg** |
| 工作半径 | 855 mm | **850 mm**（接近） |
| 重复精度 | ±0.1 mm | **±0.05 mm**（更精） |
| 力矩感知位置 | **每关节**（7 路） | 基座（电流估算 + FT 选项） |
| 控制环频率 | **1 kHz**（EtherCAT） | 500 Hz（RTDE） |
| 力控能力 | 强（阻抗/导纳原生） | 中（需外置 FT 或力控模式） |
| 用户界面 | 代码优先（libfranka） | **PolyScope 图形界面**（示教器） |
| 安全定位 | 研究臂（力控优先） | **协作机器人（cobot，碰人即停）** |
| 单价（学术，量级） | 中（~3–4 万美元级） | 中高（~3.5 万美元起） |
| 工业部署 | 少（多在实验室） | **极广**（装配/机器看护/螺丝） |
| 学术占比 | **操作任务 Top 1** | 操作任务常见但次于 Panda |

一句话：**UR5e 是「工业协作机器人里的研究员」，Panda 是「研究臂里的工业品」**。前者追求易用与安全合规，后者追求力控带宽与冗余度。

> 详见 [`ur5e.md`](ur5e.md)。

---

## 8. 常见坑与实战经验

- **`communication_constraints_violation`**：回调里干了重活（如 Python GIL 阻塞）。解决：重逻辑放上层线程，回调只做轻量计算并 `return` 控制量。
- **碰撞 reflex 频繁触发**：默认碰撞阈值偏保守。可在 `setCollisionBehavior` 里放宽 torque/force 阈值，但要先做风险评估。
- **末端夹爪**：官方 Franka Hand（两指，已停产/限量），社区多用 Robotiq 2F-85 或自定义。见 [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md)。
- **奇异点**：j5 接近 0 或 ±π 时腕部雅可比降秩，规划时用 `moveit` 的避奇异选项或利用 7-DoF 冗余偏置 j4。
- **重力补偿校准**：换工具/夹爪后必须重新 `setEE` 与负载参数，否则力控漂移。

---

## 9. 相关概念互链

- [`../../concepts/foundations/degrees-of-freedom.md`](../../concepts/foundations/degrees-of-freedom.md)——为什么 7-DoF 是冗余臂。
- [`../../concepts/foundations/embodiment.md`](../../concepts/foundations/embodiment.md)——本体决定能力上限。
- [`../../concepts/mdp/action-space.md`](../../concepts/mdp/action-space.md)——关节空间 vs 笛卡尔空间动作。
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) / [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md)——Panda 是 IL/BC 实验主力本体。
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)——多数 DP 真机实验跑在 Panda 上。
- [`ur5e.md`](ur5e.md) / [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md)——同台对比与配套夹爪。

---

## 10. 参考链接

**官方来源**：

- Franka Robotics 官方产品页（Panda 产品页已下线，由后继型号 FR3 承接）（来源：https://franka.de/franka-research-3，访问于 2026-09-02）

- Franka Emika 官方文档（FCI / libfranka / franka_ros）：https://frankaemika.github.io/docs/
- `libfranka` GitHub：https://github.com/frankaemika/libfranka
- `franka_ros2`：https://github.com/frankaemika/franka_ros2
- FR3 用户手册与规格表（官方 PDF，需账号）
- Franka Emika 论文 / 白皮书：*A Systems Engineering View on the Franka Emika Robot*
- 维基百科：Franka Emika（背景与历史）
- 学术 benchmark：`robosuite`（MuJoCo，含 Panda MJCF）、`panda-gym`、Isaac Lab Panda 资产
