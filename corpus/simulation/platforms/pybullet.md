# PyBullet：零成本入门的纯 Python 机器人仿真器

> **一句话定位**：基于 Bullet 物理引擎、**纯 Python、`pip` 一行安装、零外部依赖**的轻量机器人仿真器——是机器人学习/教学的入门首选与快速原型工具，门槛极低但接触精度与渲染简陋，无 GPU 并行，不适合大规模 RL。

> 最后更新：2026-08

---

## 1. 概览与定位

**PyBullet** 是瑞士 **Erwin Coumans**（Bullet 物理引擎作者，Google 前员工）于 2016–2019 年间推出的 **Bullet 物理引擎的 Python 绑定 + 机器人仿真模块**，托管在 `bullet3` 仓库的 `examples/pybullet` 下。它是 **Bullet Physics**（最初用于游戏与影视，曾驱动《Toy Story 3》《Grand Theft Auto》等）的机器人化封装。

定位：

- **不是**高精度物理引擎（接触求解不如 MuJoCo），也**不是**大规模并行平台（无 GPU）；
- **是**一个**「`pip install pybullet` 即装即用、自带示例机器人、跨平台」的入门仿真器**，让初学者在 5 分钟内跑起一个机械臂或四足机器人。

> 历史背景：在 MuJoCo 还闭源（2021 前）、Isaac 还要 RTX GPU 与 Omniverse 的年代，PyBullet 几乎是「学生唯一能立刻上手」的机器人仿真器。即便今天 MuJoCo 已开源，PyBullet 仍是教学场景的最常见选择，因为它的 API 更直观、文档更友好、生态（如 `panda-gym`、`pybullet_robots`）更贴近学习者。

---

## 2. 平台规格与特性

| 维度 | 规格 / 取值 | 备注 |
| --- | --- | --- |
| 底层引擎 | **Bullet Physics**（C++，实时物理） | 同源游戏引擎 |
| 绑定 | **Python（SWIG 自动生成）** | `import pybullet as p` |
| 许可 | **Zlib**（宽松开源） | 商用友好 |
| 安装 | **`pip install pybullet`**（纯 wheel，含示例资产） | 零外部依赖 |
| 模型格式 | **URDF**（主要）、SDF、MJCF（部分） | URDF 最常用 |
| 渲染 | **OpenGL（ERP / EGL / TinyRenderer 三后端）** | 简陋 |
| 物理后端 | CPU 单进程 | **无 GPU 并行** |
| 求解器 | Sequential Impulse（迭代投影） | Bullet 标配 |
| 接触模型 | 软接触 + 金字塔摩擦锥 | 精度低于 MuJoCo |
| 集成方式 | 显式 + 约束求解 | 步长需较小 |
| 接口风格 | **命令式 / GUI 直连**（`p.loadURDF`、`p.stepSimulation`） | 类「遥控」API |
| 平台 | Linux / macOS / Windows（含 ARM） | 跨平台 |
| 内置机器人 | **Kuka iiwa、UR5、Minitaur（四足）、Jackaroo、R2、Atlas** 等 | 自带示例资产 |
| RL 接入 | gym / gymnasium（`panda-gym`、`gym-pybullet-drones` 等） | 社区封装 |

> 核心卖点：**装好即用、自带机器人、API 像写脚本一样直接**——这是它成为教学首选的现实原因。代价是物理与渲染都偏简陋。

---

## 3. 核心技术解析

### 3.1 Bullet 引擎：Sequential Impulse 与软接触

Bullet 用 **Sequential Impulse（顺序冲量）求解器**处理约束：

- 把接触、关节、摩擦统一表述为**约束（constraint）**；
- 在每个时间步内**多次迭代**地投影约束冲量，逐步收敛到满足约束的解；
- 这本质上等价于 Gauss-Seidel 求解一个 LCP（线性互补问题）的近似解。

接触为**软接触（soft contact）**：接触力由弹簧-阻尼（接触刚度 + 阻尼）参数化，配合误差修正（ERP, Error Reduction Parameter）缓解穿透。摩擦用**金字塔摩擦锥（pyramid approximation）**——把库仑圆锥近似为多面体，求解快但摩擦方向有量化误差。

> 与 MuJoCo 对比：Bullet 是**LCP + 金字塔锥 + 软接触**，MuJoCo 是**凸 QP + 椭圆锥 + 参数化互补**。后者在接触稳定性、摩擦方向精度、可微性上都更优。这就是为什么「严肃的接触任务」研究者更倾向 MuJoCo。

### 3.2 三种渲染后端：GUI / EGL / TinyRenderer

PyBullet 的渲染是其相对灵活的部分，但整体仍属「简陋」：

| 后端 | 用途 | 特点 |
| --- | --- | --- |
| **GUI（`p.GUI` / `SHARED_MEMORY_GUI`）** | 交互调试 | 用 Bullet 自带的 OpenGL 窗口，可拖拽相机 |
| **EGL（`p.EGL`，headless GPU）** | 服务器/容器无头渲染 | 借 EGL 直接在 GPU 上做 OpenGL，无需 X11 |
| **TinyRenderer（`p.DIRECT` + TinyRenderer）** | 纯 CPU 无头 | 软件渲染，最便携但最慢 |

可输出 RGB、深度、分割掩码（`getCameraImage`）。但光照模型简单（无光追、无 PBR、无软阴影），与真实相机成像差距大，**纯视觉策略的 sim-to-real 效果显著差于 Isaac RTX 光追**。

> 实践含义：PyBullet 适合「状态观测（关节 + 接触）」驱动的策略；若你的策略是「RGB→动作」并要上真机，建议迁到 Isaac 或 Genesis。

### 3.3 命令式 API：`p.stepSimulation` 的「遥控」风格

PyBullet 的 API 与 MuJoCo「声明 MJCF + `mj_step`」风格不同，是**命令式**：

```python
import pybullet as p
import pybullet_data

# 直连物理（无渲染）或 GUI（有窗口）
client = p.connect(p.GUI)             # 或 p.DIRECT 无头
p.setAdditionalSearchPath(pybullet_data.dataPath)

# 加载地面 + 机器人（URDF 来自 pybullet_data）
p.loadURDF("plane.urdf")
robot = p.loadURDF("franka_panda/panda.urdf", useFixedBase=True)

# 设重力、步长
p.setGravity(0, 0, -9.8)
p.setTimeStep(1.0 / 240)

# 位置控制：把 7 个关节目标发给 PD 伺服
for _ in range(240):
    p.setJointMotorControlArray(robot, range(7), p.POSITION_CONTROL,
                                targetPositions=[0]*7)
    p.stepSimulation()                # 推进 1/240 s

# 读取关节状态
states = p.getJointStates(robot, range(7))
print([s[0] for s in states])         # 关节位置
```

要点：

- `stepSimulation` 一次推 1 个时间步（默认 1/240 s），需要自己写循环；
- 控制模式有 `POSITION_CONTROL` / `VELOCITY_CONTROL` / `TORQUE_CONTROL`，对教学友好（直观看到三种控制的差异）；
- URDF 即资产，机器人模型在 `pybullet_data` 里随包分发，无需另下。

### 3.4 经典示例：Kuka、Minitaur、quadruped

PyBullet 自带一批「教学级」机器人示例（位于 `bullet3/examples/pybullet/examples/`）：

- **`kuka_iiwa`**：Kuka iiwa 14 自由度臂，常用来演示 IK 与抓取；
- **`Minitaur`**：Ghost Minitaur 直接驱动四足，是 PyBullet locomotion RL 教学主力；
- **`laikago` / `mini_cheetah`**：四足机器人示例，配合 gym 做 RL；
- **`panda`**：Franka Panda URDF，配合 `panda-gym` 做操作任务；
- **`racecar` / `duckiebot`**：导航示例。

这些示例的价值在于「**5 分钟跑通一个完整 demo**」，是其他平台难以匹敌的上手体验。

### 3.5 为什么没有 GPU 并行

PyBullet 物理在 CPU 上单进程跑，**没有 GPU 向量化能力**。要并行只能：

- 用 Python `multiprocessing` 起多个进程，每个跑一个环境（`SubprocVecEnv`）；
- 受限于 Python GIL 与进程间通信开销，扩展性远不如 Isaac 的「GPU 内向量化」。

这意味着 PyBullet 跑大规模 RL（如 PPO 需要数千环境）不现实。典型规模：**1–64 个环境**，用于算法验证或小任务训练。

---

## 4. 适用场景

| 场景 | 为什么选 PyBullet |
| --- | --- |
| **教学 / 课程作业** | `pip` 一行装好，自带机器人，API 直观，学生 5 分钟上手 |
| **快速原型 / 算法验证** | 想在半小时内验证一个想法是否 work |
| **小规模 RL 实验** | 单/数环境，PPO/SAC 验证算法正确性 |
| **URDF 调试** | 检查一个 URDF 是否加载正确、惯性是否合理 |
| **跨平台便携 demo** | 纯 Python，无 GPU 依赖，任何机器能跑 |

不擅长：大规模 RL（用 Isaac）、精确接触任务（用 MuJoCo）、纯视觉 sim-to-real（用 Isaac/Genesis）、生产级部署（PyBullet 不是为此设计）。

---

## 5. 与同类对比

| 维度 | PyBullet | MuJoCo | Isaac Sim/Lab |
| --- | --- | --- | --- |
| 安装 | **最简（pip）** | `pip install mujoco` | 重（Omniverse + RTX） |
| 物理精度 | 中低 | **最高** | 中 |
| 接触稳定性 | 中低 | **最高** | 中 |
| 渲染 | 简易 OpenGL | 简易 OpenGL | **RTX 光追** |
| GPU 并行 | **无** | 无（MX 有） | **数千环境** |
| 上手成本 | **最低** | 低 | 高 |
| 模型格式 | URDF | MJCF（强） | USD |
| 主导 | 社区（Erwin Coumans） | DeepMind | NVIDIA |
| 典型用户 | 学生、入门者 | 算法研究员 | 大规模 RL 研究者 |
| 开源 | Zlib | Apache-2.0 | 否（研究免费） |

一句话：**PyBullet 是「入门的脚手架」，MuJoCo 是「研究员的实验台」，Isaac 是「工业级 RL 工厂」**。常见路径：PyBullet 学概念 → MuJoCo 做算法 → Isaac 跑大规模。

> 详见 [`mujoco.md`](mujoco.md)、[`nvidia-isaac.md`](nvidia-isaac.md)。

---

## 6. 常见坑与实战经验

- **接触抖动 / 穿透**：Bullet 软接触 + 金字塔摩擦，重载或硬接触易抖。调 `p.changeDynamics` 的 `lateralFriction`、`contactStiffness`、`contactDamping`，或减小 `timeStep`。
- **`TORQUE_CONTROL` 要自己算重力补偿**：PyBullet 的力矩模式不会自动补偿重力，需 `p.calculateInverseDynamics` 算 `tau_gravity` 再叠加，否则臂会下垂。
- **GUI 在服务器上跑不了**：用 `p.connect(p.DIRECT)` 或 `p.EGL`（无头 GPU 渲染）。容器里记得装 `libegl1`。
- **URDF 惯性参数**：许多社区 URDF 的 `<inertial>` 不准，导致动力学奇怪。用 `pybullet` 的 `getDynamicsInfo` 检查。
- **大规模 RL 别用**：64 个环境已是 PyBullet 的舒适上限；要上千环境请用 Isaac。
- **`panda-gym` / `gym-pybullet-drones` 是好起点**：社区已封装好 gym 接口，别从零写。

---

## 7. 相关概念互链

- [`README.md`](../README.md)——仿真总览。
- [`nvidia-isaac.md`](nvidia-isaac.md)——大规模 RL 的进阶平台。
- [`mujoco.md`](mujoco.md)——精度更高的算法实验台。
- [`genesis.md`](genesis.md)——开源 GPU 并行新选择。
- [`../sim-to-real/domain-randomization.md`](../sim-to-real/domain-randomization.md)——PyBullet 上做的简单 sim-to-real。
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)——RL 入门与仿真的关系。
- [`../../hardware/arms/franka-panda.md`](../../hardware/arms/franka-panda.md)——`panda-gym` 是 PyBullet 上最常见的本体。

---

## 8. 参考链接

- PyBullet 官方文档（Quickstart）：https://docs.google.com/document/d/10sXEhzFRSnvFcl3XxNGjnD5Nvnegwih8GZm7bVxCGx0
- `bullet3` GitHub（含 PyBullet）：https://github.com/bulletphysics/bullet3
- `pybullet_data`（自带机器人资产）：https://github.com/bulletphysics/bullet3/tree/master/data
- `panda-gym`（PyBullet + Franka，gym 封装）：https://github.com/qgallouedec/panda-gym
- `gym-pybullet-drones`（无人机 RL）：https://github.com/utiasDSL/gym-pybullet-drones
- Bullet Physics 主页（历史）：https://pybullet.org/
- 论文：*PyBullet, a Python module for physics simulation* (Coumans, E., 2016–2019, 白皮书)

---

## 9. 历史注记：从游戏引擎到机器人仿真

Bullet 的演化路径本身就是一段值得记录的技术史：

- **2000s 初**：Erwin Coumans 在 Sony Computer Entertainment 开发 Bullet，用于游戏与影视，曾驱动《Toy Story 3》《Grand Theft Auto IV》等——它的基因是「**实时、稳定、够用**」而非「物理精确」。
- **2010s**：Bullet 开源（Zlib），加入 Soft Body、FEA 等特性，被 Google、Disney、Blender 等采用。
- **2016+**：Coumans 把 Bullet 包装成 PyBullet，加入机器人 URDF 加载、PD 控制、相机渲染等模块，定位转向机器人研究/教学。这一步决定了 PyBullet 的「**轻量教学优先、精度次要**」气质。
- **2021 后**：MuJoCo 开源、Isaac 成熟，PyBullet 的研究占比下降，但在教学/入门场景仍不可替代——因为它「装好即用」的体验至今无平台超越。

> 这段历史解释了 PyBullet 的本质定位：**它是一个游戏物理引擎被「机器人化」的产物**，因此继承了游戏引擎的稳定性与易用性，也继承了其在精确接触上的固有局限。理解这一点，就知道何时该用它、何时该换 MuJoCo/Isaac。
