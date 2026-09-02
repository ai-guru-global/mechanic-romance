# LEAP Hand · 开源 16-DoF 灵巧手

> **一句话定位**：Stanford 2023 推出的**全开源、3D 打印、低成本（$3000）** 16-DoF 灵巧手——**「灵巧手研究的 iPhone」**，把硬件门槛从 $1.5 万打到 $3000，让任何学生实验室都能做灵巧手研究。

> 最后更新：2026-08

---

## 1. 概览

**LEAP Hand**（**L**ow-cost, **E**fficient, **A**daptable, **P**assive）是 Stanford 团队（Shen et al.）2023 年发表的**开源灵巧手**。核心动机是 Allegro / Shadow Hand 商用太贵、闭源、不可改，**研究民主化** 已是 2020+ 趋势（参考 LeRobot / Habitat / Isaac Sim 社区）。

| 维度 | 价值 |
| --- | --- |
| **开源** | CAD / BOM / 控制代码 / URDF 全部公开 |
| **低成本** | DIY **~$3000**（BOM 算上电机、3D 打印、PCB） |
| **可复制** | 任何 3D 打印 + 电机商家可组装 |
| **Sim-to-Real 友好** | 在 Isaac Sim 中训，迁移到 LEAP 真机差距小 |
| **16-DoF** | 与 Allegro 同级（4 指 4 关节） |
| **2024+ 主流** | 多家实验室用 LEAP 做博士论文、顶会工作 |

> **代表工作**：「LEAP Hand: Low-Cost, Efficient, and Adaptive Hands for Dexterous Manipulation」（2023 RSS, Best Paper Finalist）。

---

## 2. 规格

| 参数 | 数值 | 备注 |
| --- | --- | --- |
| **指段数** | 4（拇指 + 食指 + 中指 + 无名指） | 与 Allegro 同 |
| **自由度（DoF）** | **16**（每指 4 关节） | 与 Allegro 同 |
| **电机** | **Dynamixel XM430-W350**（4×4=16 个） | 商用现成电机 |
| **通信** | Dynamixel Protocol 2.0（TTL/RS-485） | 一根总线 16 电机 |
| **控制频率** | 100 Hz（关节级） | 较 Allegro 333 Hz 低 |
| **抓握力** | 单指 ~25 N | 比 Allegro 强 |
| **自重** | 约 0.9 kg（单手） | 略轻 Allegro |
| **指尖触觉** | 可外接（如 DIGIT / 触觉阵列） | 原生无触觉 |
| **腕部接口** | 标准法兰（与 Panda 兼容） | 即插即用 |
| **供电** | 12 V DC（Dynamixel） | 桌面电源 |
| **售价** | **~$3000**（BOM 全算） | Allegro 1/5 |
| **SDK** | **Dynamixel SDK + ROS2 + Isaac Sim** | 全开源 |

> 关键差异：**LEAP 用现成 Dynamixel 电机**（机器人社区普及），Allegro 用自制电机——这决定了 LEAP **可修可改**。

---

## 3. 设计哲学

LEAP 的三个核心原则（论文标题首字母 LEAP = 4 大原则）：

| 原则 | 含义 |
| --- | --- |
| **L**ow-cost | 全部元件可在 DigiKey / Amazon 买到，BOM < $3000 |
| **E**fficient | 4 指 16-DoF 平衡「控制难度」与「灵巧度」 |
| **A**daptable | 全 3D 打印，**指段可任意改造**（如换触觉、改指长） |
| **P**assive | **被动机械适应性**（如自适应指铰链），减少控制负担 |

> 「**P**assive」是 LEAP 的关键创新——通过**机构设计**（自适应包络、铰链）代替**主动控制**（算法适应）。这与 [Robotiq 2F-85](../../hardware/end-effectors/robotiq-2f-85.md) 的「自适应指」哲学一脉相承。

---

## 4. 工作原理与设计

### 4.1 机械设计

```
电机（指外） ──► 齿轮减速 ──► 丝杠 ──► 指段直线运动 ──► 关节转动
```

- **电机在手掌内**（4×4=16 个），与 Allegro 类似
- **丝杠 + 直线运动** 替代传统「电机 + 肌腱」（Shadow Hand 风格），**减少维护成本**
- **3D 打印指段**（PLA / ABS / 尼龙），可重印

### 4.2 关节联动

与 Allegro 类似，远端两关节通过 4-bar linkage 联动，减少独立控制变量。

### 4.3 与 Allegro 设计的核心差异

| 维度 | LEAP | Allegro |
| --- | --- | --- |
| **电机** | Dynamixel 商用 | 自制电机 |
| **控制** | 商用总线（TTL） | EtherCAT |
| **外壳** | 3D 打印 PLA | 工业塑料 + 金属 |
| **触觉** | 无（可外接） | v5 集成 |
| **开源** | **完全开源** | SDK 开源，硬件不开源 |
| **价格** | **$3000** | $1.5-2 万 |
| **可改造** | 高（换指段、加触觉） | 低（指段是工业件） |
| **耐用性** | 中（3D 打印件磨损） | 高（工业件） |

> **LEAP 像「Maker 风格的灵巧手」**，**Allegro 像「工业品」**。学生 / 研究者用 LEAP，工程化用 Allegro。

---

## 5. 控制接口与软件生态

### 5.1 通信

```python
# Dynamixel SDK
from dynamixel_sdk import PortHandler, PacketHandler
port = PortHandler('/dev/ttyUSB0')
port.openPort()
port.setBaudRate(57600)

# 写 16 关节角
for motor_id, position in enumerate(target_angles):
    packet.write4ByteTxRx(port, motor_id, ADDR_PRO_GOAL_POSITION, position)
```

### 5.2 ROS2

- `leap_hand_ros2`（官方）：https://github.com/leap-hand/ros2 leap
- 集成 `moveit2`：可直接用 MoveIt 做运动规划
- Isaac Sim 内置 LEAP 模型：开箱即用

### 5.3 Sim-to-Real

LEAP 的**最大优势**是**Sim-to-Real 迁移**：

```python
# Isaac Sim 训练
env = LEAPHandCubeEnv(num_envs=2048)
policy = train_ppo(env)  # 域随机化 + 真实 URDF

# 直接部署到真机（无修改！）
state = leap_hand.read_joint_positions()
action = policy(state)
leap_hand.write_joint_positions(action)
```

> 论文报告 **LEAP Sim-to-Real Gap < 5%**（成功率差异），远低于 Shadow Hand 的 20%+。

---

## 6. 应用场景

| 场景 | 为什么选 LEAP |
| --- | --- |
| **学生 / 实验室** | $3000 价格可承担 |
| **Sim-to-Real 研究** | 论文报告 gap < 5% |
| **快速迭代** | 改指段重新打印，1 天可换装 |
| **多手协作** | 4 手 $1.2 万，做 multi-hand 任务 |
| **Diffusion Policy 灵巧版** | LEAP + DP 是 2024 主流 |
| **DexCap 数据采集** | LEAP 戴手上更轻（0.9 kg vs Allegro 1.1 kg） |
| **教学** | 学生 DIY 经历对研究更深入 |

不擅长：长期生产（3D 打印件磨损）、极端环境（高温 / 振动）、5 指任务（无小指）。

---

## 7. 与其他灵巧手对比

| 维度 | LEAP | [Allegro](allegro.md) | [Shadow Hand](shadow-hand.md) |
| --- | --- | --- | --- |
| **DoF** | 16 | 16 | 24 |
| **指段** | 4 | 4 | 5 |
| **电机** | Dynamixel 商用 | 自制 | 直流伺服 + 肌腱 |
| **触觉** | 无（可外接） | 6×6 阵列 | 整手 100 触点 |
| **开源** | **全开源** | SDK 开源 | 不开源 |
| **价格** | **$3000** | $1.5-2 万 | $10-15 万 |
| **可改造** | **高** | 低 | 低 |
| **Sim-to-Real Gap** | **< 5%** | 10-15% | 20%+ |
| **耐用性** | 中 | 高 | 极高 |
| **代表工作** | LEAP 论文、Stanford MAIRA | OpenAI Dactyl、DDEX | OpenAI In-hand Cube |
| **2024+ 主流** | **⭐⭐⭐⭐⭐** | ⭐⭐⭐⭐ | ⭐⭐⭐ |

> **2024+ 趋势**：LEAP 凭「**开源 + 便宜 + Sim-to-Real 友好**」三杀，**正在取代 Allegro 成为新学生/研究首选**。

---

## 8. 常见坑

- **3D 打印精度**：指段间隙 0.1-0.2 mm，需 SLA 或高精度 FDM
- **Dynamixel 总线带宽**：16 电机共享 TTL，**100 Hz 是上限**
- **电机散热**：高负载下电机温度上升，建议间歇操作
- **力矩校准**：Dynamixel 默认电流 ≠ 力矩，需测静摩擦后标定
- **指段维修**：3D 打印件断裂可重印，但 1-2 天打印 + 组装时间
- **Isaac Sim 版本兼容**：LEAP 官方模型跟随 Isaac Sim 升级，**升级前先查兼容**
- **触觉集成**：DIGIT / 其他触觉需自己接线和校准

---

## 9. 在 IL/RL 里的典型用法

### 9.1 RL（Isaac Sim 训练）

```python
# Isaac Lab / IsaacGymEnvs 内置 LEAP 环境
from omni.isaac.lab.envs import ManagerBasedRLEnv
env = LEAPHandCubeEnv(num_envs=2048)        # 2048 并行
obs = env.reset()
for t in range(200):
    action = ppo_policy(obs)                 # 16-DoF 增量
    obs, reward, done, info = env.step(action)
# 论文报告：70%+ 成功率在 30M 步内
```

### 9.2 IL（DexCap + Diffusion Policy）

1. **人戴 VR 控制器 + 摄像头**（DexCap 系统）
2. **人手演示 50+ 任务**
3. **重定向人手姿态 → LEAP 关节角**
4. **训练 Diffusion Policy**

### 9.3 Sim-to-Real 关键

LEAP 论文报告的核心 Sim-to-Real 技巧：
- **域随机化**（摩擦 / 质量 / 关节阻尼）
- **细颗粒度接触建模**（指段柔性）
- **真实 URDF**（CAD 出图 → URDF）
- **小幅物理参数校正**（real-to-sim 标定）

---

## 10. 相关语料互链

- [`./allegro.md`](allegro.md) — 商用主流灵巧手
- [`./shadow-hand.md`](shadow-hand.md) — 工业级对比
- [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md) — 2 指夹爪对比
- [`../arms/franka-panda.md`](../arms/franka-panda.md) — LEAP 最常挂的臂
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md) — LEAP 主要用 RL 训
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md) — DDEX 灵巧手版
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) — DexCap 数据采集
- [`../../simulation/platforms/nvidia-isaac.md`](../../simulation/platforms/nvidia-isaac.md) — Isaac Lab 训练
- [`../../simulation/sim-to-real/domain-randomization.md`](../../simulation/sim-to-real/domain-randomization.md) — 灵巧手 sim-to-real
- [demo 08-灵巧手-五指旋转立方体](../../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md) — 直接落地场景

---

## 11. 参考链接

**官方来源**：

- LEAP Hand 官方站（Stanford / WCRI）（来源：https://leaphand.com/，访问于 2026-09-02）

- LEAP Hand 官网：https://leaphand.stanford.edu/
- 论文（RSS 2023 Best Paper Finalist）：https://leaphand.stanford.edu/papers
- GitHub（URDF / CAD / SDK）：https://github.com/leap-hand
- Isaac Sim 模型：Isaac Lab 内置
- Stanford MAIRA Lab：https://aira.stanford.edu/
- DexCap（2024）：基于 LEAP 的数据采集
- 学术 benchmark：IsaacGymEnvs、Isaac Lab LEAPHandCube
- 仿制案例：多家国内外实验室 DIY 复刻
