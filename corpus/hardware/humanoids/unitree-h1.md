# 宇树科技 Unitree H1 人形机器人

> **一句话定位**：杭州宇树科技（Unitree Robotics）的全尺寸双足人形机器人——身高约 1.8 m、19 DoF 主体（可扩展灵巧手）、电机直驱、开源 SDK，凭借约 9 万美元的售价与可量产交付，成为 2024–2026 年学术人形 locomotion 与全身控制研究的**事实首选本体**。

> 最后更新：2026-08

---

## 1. 概览

Unitree H1 于 2023 年底发布、2024 年量产交付，是宇树继四足（A1/Go1/Go2/B1）之后的第一款**全尺寸人形（full-size humanoid）**。它继承了宇树在四足上积累的**高功率密度准直驱（quasi-direct-drive, QDD）电机**与强化学习 locomotion 经验，把这套技术栈搬到了双足平台上。

H1 在学术界的地位来自三个「可及性」：

1. **买得到**：不像 Figure / Tesla Bot 那样封闭自用，H1 面向研究机构公开发售。
2. **价格可承受**：整机约 **9 万美元**量级（远低于 Figure / Tesla 估算的百万级），对标一台豪华车的钱就能拥有一台人形。
3. **开源 SDK**：提供关节级控制接口、ROS 封装、URDF/MJCF 模型，可直接对接 Isaac Gym/Lab、MuJoCo、Genesis。

这三点使它成为 **RL locomotion 真机部署（sim-to-real）**的主力平台——大量「在 Isaac Gym 训练、在 H1 上跑」的论文在 2024–2025 年井喷。

> 后继型号 **G1**（2024 年发布，约 1.3 m、23+ DoF、价格更低至 1.6 万美元起）面向更便宜的小尺寸研究，与 H1 形成「大尺寸旗舰 + 小尺寸普及」的双产品线。本文以 H1 为主。

---

## 2. 整机规格

| 参数（Parameter） | 数值（Value） | 备注 |
| --- | --- | --- |
| 身高 Height | **约 1.80 m**（含头部） | 全尺寸人形 |
| 自重 Weight | 约 47 kg（整机） | 含电池/结构 |
| 主体自由度 DoF | **19**（双臂 4×2 + 双腿 5×2 + 躯干 1） | 不含灵巧手 |
| 含灵巧手 DoF | 19 + 灵巧手（可选配） | H1-2 升级版含三指/灵巧手 |
| 行走速度 Walking Speed | **约 1.5 m/s**（标称） | 实验室 demo 更快可达 ~3 m/s |
| 末端有效载荷 | 每臂约 3 kg（单臂） | 双臂协调更低 |
| 续航 Battery | 约 2 小时（轻载行走） | 可换电 |
| 电机类型 | **准直驱（QDD）无框力矩电机** | 高功率密度 |
| 感知传感器 | IMU、关节编码器、（可选）RGB-D / LiDAR | 基础款不含多模态 |
| 计算平台 | 板载 IPC + 可选 Jetson / 外接 GPU | 策略推理 |
| 通信 | EtherCAT / CAN / 以太网 | 关节级实时 |
| 防护/工作温度 | 室内/实验室 | 非户外防水 |

> 与四足 Go2 共享大量软件栈（`unitree_sdk2`、ROS 封装、Isaac 模型），降低了从四足迁移到人形的学习成本。

---

## 3. 关节自由度分布与电机参数

H1 的 19 个自由度按「腿-腰-臂」分布。各关节采用**准直驱（QDD）**设计：高扭矩密度电机 + 低减速比（通常 < 10:1），从而保留**反向驱能力（backdrivability）**与高带宽力控——这是做 RL locomotion 与柔顺控制的关键。

### 3.1 自由度分布

| 部位 | 关节 | DoF 数 | 说明 |
| --- | --- | --- | --- |
| 每条腿 | 髋偏航（hip yaw）/ 髋俯仰（hip pitch）/ 髋侧倾（hip roll）/ 膝（knee）/ 踝（ankle） | 5 × 2 = 10 | 双足支撑与行走主力 |
| 躯干 | 腰偏航/俯仰 | 1–2 | H1 标配 1（腰偏航），增强转向 |
| 每条臂 | 肩俯仰/侧倾/偏航 + 肘 | 4 × 2 = 8 | 上肢操作 |
| **合计（主体）** | | **≈19** | 不含腕/手 |
| 灵巧手（选配） | 三指 / 多指 | 视配置 | H1-2 可装 |

### 3.2 关键关节电机扭矩（标称，量级）

H1 各关节峰值扭矩随部位差异较大（腿部承重远大于手臂）：

| 部位 | 峰值扭矩（量级） | 额定扭矩（量级） |
| --- | --- | --- |
| 膝 / 髋 pitch | **~360 N·m**（峰值） | ~120 N·m |
| 髋 roll / yaw | ~120 N·m | ~40 N·m |
| 踝 | ~120 N·m | ~40 N·m |
| 肩 pitch / roll / yaw | ~120 N·m | ~40 N·m |
| 肘 | ~120 N·m | ~40 N·m |

> 注：宇树在不同批次/固件下公布的扭矩值略有差异，且「峰值」通常是短时（<1 s）瞬态。做 RL 训练时应以**额定扭矩**作为安全边界，峰值仅作过载余量。

**为什么 QDD 重要**：传统高减速比（如谐波 100:1）关节反向驱动阻力大、带宽低，难以做爆发性跳跃与柔顺接触；QDD 低减速比 + 力矩电流反馈，使人形能做跑、跳、受扰恢复等高动态行为——这是 H1 区别于早期液压人形（如 Atlas 早期版）的核心。

---

## 4. 关键技术解析

### 4.1 为什么 H1 成为学术人形首选：可及性三连

学术界选本体本质看「**能不能用得起 + 能不能用得动 + 能不能改**」。H1 在这三项上同时达标：

| 维度 | H1 | Figure 02 / Tesla Optimus |
| --- | --- | --- |
| 价格 | **~9 万美元**（G1 更低至 1.6 万） | 估算百万级 / 不对研究发售 |
| 可购买性 | 公开发售，研究机构可下单 | 封闭自用 / 合作方限定 |
| 开源程度 | SDK + URDF + MJCF + ROS 封装 | 闭源，无公开控制接口 |
| 社区规模 | 大量论文/复现/开源策略 | 极少公开技术细节 |

结果：一个博士生用 H1 + Isaac Lab 就能复现一篇顶会 locomotion 论文；用 Figure/Tesla 做不到。

### 4.2 RL Locomotion 的标准范式（Sim-to-Real）

H1 是当前「**Isaac Gym 训练 → 真机部署**」范式的事实载体。典型流程：

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────────┐
│ Isaac Lab/Gym   │ ──▶ │ 策略蒸馏/量化     │ ──▶ │ H1 真机部署   │
│ 大规模并行仿真   │     │ (ONNX/TorchScript)│     │ 板载推理      │
│ 域随机化训练     │     │                  │     │ 实测恢复/行走 │
└─────────────────┘     └──────────────────┘     └──────────────┘
```

关键技术点：

1. **观测空间**：关节角 `q`、关节速度 `dq`、IMU（基座姿态/角速度）、上一时刻动作（延迟补偿）、（可选）高度扫描/深度图。
2. **动作空间**：目标关节角 / PD 目标 / 转矩前馈，通常以 `a = a_prev + Δ` 形式做平滑。
3. **域随机化（domain randomization）**：摩擦、质量、电机扭矩增益、延迟、外力扰动——确保策略鲁棒。
4. **课程学习（curriculum）**：从平地走到加扰动、上下台阶、跑步、跳跃，逐步加大难度。
5. **真机部署**：策略导出为 ONNX/TorchScript，板载 IPC/Jetson 以 50–100 Hz 推理，关节级 PD/QDD 力控以 1 kHz 跑。

这套范式在 H1 上被反复验证，使其成为 locomotion RL 的**benchmark 本体**。详见 [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)。

### 4.3 准直驱与力控带宽

QDD 的低减速比带来：

- **高反向驱能力**：人推机器人，关节能顺势让动（柔顺），这对人机接触与跌倒缓冲至关重要。
- **高带宽力控**：电机电流→转矩近似线性，带宽可达数十 Hz，支持跳跃落地缓冲（landing）与快速平衡修正。
- **高效率**：传动损耗小，整机续航优于液压方案。

代价是**散热挑战**：大电流持续输出易过热，因此 H1 在长时高动态任务里需要主动风冷与电流限幅。

### 4.4 全身控制（WBC）与 RL 的关系

H1 的控制栈并非纯 RL 端到端。常见两种架构：

- **纯 RL 策略**：策略直接输出关节目标，简单但泛化与可解释性弱。
- **RL + WBC 分层**：RL 策略输出任务空间目标（如躯干姿态、步长），底层 WBC（Whole-Body Control，基于 QP 的优先级控制）解算关节转矩。后者结合了 RL 的鲁棒性与经典控制的可解释性，是近年主流。

---

## 5. 控制接口与软件生态

### 5.1 官方 SDK

| 组件 | 说明 |
| --- | --- |
| **`unitree_sdk2` / `unitree_sdk2_python`** | C++ / Python 关节级控制接口，含 low-level（关节 PD/力矩）与 high-level（速度/位姿） |
| `unitree_ros` / `unitree_ros2` | ROS1/ROS2 封装 |
| `unitree_rl_gym` | 官方/社区 Isaac Lab RL 训练示例，含 H1/G1 配置 |
| URDF / MJCF | 仿真模型，MuJoCo / Isaac / Genesis 均有 |

### 5.2 控制层级

| 层级 | 接口 | 频率 |
| --- | --- | --- |
| 关节力矩（torque） | low-level `motor_ctrl` | ~1 kHz |
| 关节 PD 目标 | low-level `q_des, dq_des, kp, kd` | ~500 Hz–1 kHz |
| 全身/步态高层 | high-level velocity / pose | ~50–100 Hz |
| RL 策略推理 | 板载 IPC/Jetson | ~50–100 Hz |

### 5.3 训练仿真生态

H1 在以下仿真器都有官方或社区高保真模型，是 sim-to-real 闭环的基础：

| 仿真器 | 角色 |
| --- | --- |
| **Isaac Lab / Isaac Gym** | RL 训练主力（GPU 并行） |
| **MuJoCo** | 高精度物理，验证与 WBC |
| **Genesis** | 新兴并行/可微仿真，社区适配中 |
| **Gazebo / ROS** | 与真机接口一致的仿真验证 |

---

## 6. 应用场景

| 场景 | 为什么选 H1 |
| --- | --- |
| **RL locomotion 研究** | sim-to-real 范式标准载体，社区复现多 |
| **全身控制 / 跌倒恢复** | QDD 高带宽力控 + 19 DoF，可做高动态行为 |
| **双臂操作（mobile manipulation）** | 全尺寸人形 + 双臂，研究「走到桌前抓物体」 |
| **人机交互（HRI）原型** | 类人外观与身高，适合交互 demo |
| **算法 benchmark** | 论文间可比（越来越多工作以 H1 为标准本体） |
| **教育/竞赛** | G1 低价版进入高校机器人课与竞赛 |

不擅长：长时间工业作业（续航/散热有限）、高精度装配（关节反向间隙与柔顺性使精度受限）、户外恶劣环境（非防水防尘）。

---

## 7. 与同类对比

### 7.1 vs Figure 02

| 维度 | Unitree H1 | Figure 02 |
| --- | --- | --- |
| 定位 | 研究级人形，公开发售 | 商业级人形，工厂落地优先 |
| 身高 | ~1.8 m | ~1.7 m |
| DoF（主体） | ~19 | ~41（含灵巧手） |
| 价格 | **~9 万美元** | 不对研究发售（估算极高） |
| 开源 | SDK + 模型 | 闭源 |
| 学术可得性 | **高** | 极低 |

> Figure 02 详情见 [`figure-02.md`](figure-02.md)。

### 7.2 vs Tesla Optimus / Bot

| 维度 | Unitree H1 | Tesla Optimus（Gen 2） |
| --- | --- | --- |
| 可购买 | 是（研究机构） | 否（内部/合作方） |
| 开源 | 是 | 否 |
| 价格透明 | 是 | 不透明（Elon 估「<2 万美元」量产目标，未兑现研究可得价） |
| 学术论文密度 | **高** | 极低 |

### 7.3 vs 宇树 G1（同族小尺寸）

| 维度 | H1 | G1 |
| --- | --- | --- |
| 身高 | ~1.8 m | ~1.3 m |
| DoF | ~19 | 23+（更灵活） |
| 价格 | ~9 万美元 | **~1.6 万美元起** |
| 定位 | 旗舰研究 | 普及/教育 |

**一句话**：要全尺寸与论文可比性 → H1；要便宜与高 DoF 实验 → G1。

---

## 8. 常见坑与实战经验

- **过热保护**：长时大电流（如持续跑跳）会触发关节过热降扭矩甚至停机，训练时设电流限幅与散热。
- **跌落损坏**：双足摔倒对结构与关节冲击大，建议在低压软垫上做高风险实验，并加跌倒检测策略。
- **sim-to-real gap**：电机扭矩增益、关节摩擦、IMU 延迟是主要差异来源；务必在仿真做足域随机化。
- **板载算力**：复杂视觉策略可能超出板载 IPC，需外接 GPU 或用 Jetson Orin。
- **急停与安全**：必须配物理急停按钮（e-stop），RL 策略发散时秒停。
- **关节限位**：QDD 虽柔顺但超限位仍会撞硬限，训练时在奖励里加限位惩罚。

---

## 9. 相关概念互链

- [`../../concepts/foundations/embodiment.md`](../../concepts/foundations/embodiment.md)——人形是通用本体野心。
- [`../../concepts/foundations/degrees-of-freedom.md`](../../concepts/foundations/degrees-of-freedom.md)——19+ DoF 的冗余与控制难度。
- [`../../concepts/foundations/embodied-ai.md`](../../concepts/foundations/embodied-ai.md)——具身智能的物理载体。
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)——RL locomotion 是 H1 的主战场。
- [`../../simulation/`](../../simulation/)——Isaac Lab / MuJoCo 是 H1 训练仿真器。
- [`figure-02.md`](figure-02.md)——商业人形对比。

---

## 10. 参考链接

**官方来源**：

- Unitree H1 官方产品页（来源：https://www.unitree.com/h1，访问于 2026-09-02）

- Unitree 官方（H1）：https://www.unitree.com/h1/
- Unitree H1 技术参数（官方 PDF）
- `unitree_sdk2`（C++）：https://github.com/unitreerobotics/unitree_sdk2
- `unitree_sdk2_python`：https://github.com/unitreerobotics/unitree_sdk2_python
- `unitree_ros2`：https://github.com/unitreerobotics/unitree_ros2
- `unitree_mujoco`（MuJoCo 模型）：https://github.com/unitreerobotics/unitree_mujoco
- `legged_gym` / Isaac Lab 人形示例：https://github.com/leggedrobotics/legged_gym
- 宇树 G1 产品页：https://www.unitree.com/g1/
- 学术代表：Unitree 在 IEEE/ICRA/IROS 的 locomotion 与全身控制论文集
