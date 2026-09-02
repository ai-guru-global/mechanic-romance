# Shadow Hand · 工业级 24-DoF 灵巧手

> **一句话定位**：英国 Shadow Robot 公司的**24-DoF 五指灵巧手**——灵巧手研究「**黄金标准**」，**最贵（$10-15 万）但最接近人手能力**，是 OpenAI 2018 Dactyl、DeepMind SAC-X、跨模态灵巧操作等里程碑工作的硬件平台。

> 最后更新：2026-08

---

## 1. 概览

**Shadow Hand** 是 Shadow Robot Company（英国）从 2000 年代研发的**仿人灵巧手**，目标是「**复刻人手**」——5 指、24-DoF、20 多种触觉传感器、亚毫米精度。被 NASA、ESA、DeepMind、OpenAI 等顶级研究机构采用，是**灵巧手研究的事实高端基准**。

| 代际 | 发布年 | 关键改进 |
| --- | --- | --- |
| **Shadow Hand C3** | 2005 | 第一代研究版 |
| **Shadow Dexterous Hand E1** | 2010 | 模块化电机 |
| **Shadow Hand** | 2015 | 24-DoF 标配，20+ 触觉 |
| **Shadow Hand × 2** | 2018 | 双手机构（双臂系统） |

> **代表工作**：
> - OpenAI 2018 — Dactyl 单手玩魔方（Shadow Hand + 域随机化）
> - DeepMind 2020 — SAC-X（多任务灵巧手学习）
> - DeepMind 2021 — BC-Z（跨模态灵巧操作）
> - 各种 in-hand tool use 论文

---

## 2. 规格

| 参数 | 数值 | 备注 |
| --- | --- | --- |
| **指段数** | 5（拇指 + 4 指 × 4 关节） | 含小指 |
| **自由度（DoF）** | **24** | 5×4 + 4 wrist |
| **腕部** | 4-DoF（屈伸 / 尺桡 / 旋前旋后） | 与人手腕对应 |
| **执行器** | 20 个 Maxon DC 电机（外置驱动箱） | 肌腱驱动 |
| **通信** | **EtherCAT** | 1 kHz 实时控制 |
| **控制频率** | 1 kHz（关节级） | 工业级 |
| **抓握力** | 单指 ~30 N | 比 Allegro / LEAP 强 |
| **自重** | 约 4.2 kg（单手 + 驱动箱） | 较重 |
| **触觉传感器** | 整手 **20+ 触觉**（指尖、掌面、指段） | 全手覆盖 |
| **指尖传感器** | BioTac（压阻 / 温度 / 振动） | 高端 |
| **腕部接口** | 标准法兰 | 兼容 Panda / KUKA / 定制 |
| **供电** | 24V DC + 12V DC | 需独立电源 |
| **售价** | **$10-15 万** | 单手 |
| **SDK** | **ros_ethercat + Python / C++** | 半开源 |

> 关键参数：24-DoF 是「**逼近人手能力**」的下限——人手有 27-DoF，Shadow Hand 24 个覆盖 88%。

---

## 3. 工作原理与设计哲学

### 3.1 五指完整设计

Shadow Hand 是**完整 5 指**（含小指），区别于 [Allegro](allegro.md) / [LEAP Hand](leap-hand.md) 的 4 指：

| 维度 | 5 指（Shadow） | 4 指（Allegro / LEAP） |
| --- | --- | --- |
| 抓取稳定度 | **高**（小指侧支撑强） | 中 |
| 控制复杂度 | **高**（24-DoF） | 低（16-DoF） |
| 仿真难度 | **高** | 中 |
| 适用任务 | 复杂 dexterous tool use | in-hand rotation、pinch |
| 仿人程度 | **高** | 中 |

> 5 指在**乐器演奏、外科手术辅助、五指灵巧工具使用**等任务上是 4 指无法替代的。

### 3.2 肌腱驱动（tendon-driven）

Shadow Hand 电机**外置**在前臂的「**驱动箱**」，通过**类肌腱（tendon）** 连接关节：

```
电机（外置）──► 肌腱（柔索）──► 关节（远端）──► 指段
```

- ✅ **远端指段无电机**（轻量、紧凑、外观更仿人）
- ✅ **多电机驱动同一关节**（如 PIP 关节可同时由 2 个电机控制）
- ❌ **维护成本高**（肌腱会拉伸 / 断裂）
- ❌ **需要标定**（温度 / 湿度影响肌腱张力）

### 3.3 全手触觉感知

整手覆盖 **20+ 触觉传感器**：
- **指尖**：BioTac（压阻 + 温度 + 振动）
- **指段**：FSR 压阻阵列
- **掌面**：Fingertip 传感器
- **腕部**：碰撞 / 力矩传感器

> 这是 Shadow Hand 与 Allegro / LEAP **最大差异**——**触觉是「灵巧操作」的感知基石**，尤其在**滑移检测、力控拧螺丝、纹理识别**等任务上不可替代。

---

## 4. 控制接口与软件生态

### 4.1 通信

- **EtherCAT** 主控（1 kHz 实时）
- ROS / ROS2：`sr_ethercat` 驱动
- 仿真：MuJoCo（最准）、Isaac Lab、PyBullet

### 4.2 SDK

```python
# Shadow Hand ROS（开源）
import rospy
from sr_robot_msgs.msg import sendupdate, jointstate
# 发送 24 关节角目标
state = JointState(position=[0.1, 0.2, ...])  # 24 维
pub.publish(state)
```

### 4.3 仿真

- **MuJoCo**：最准（Shadow Robot 官方 MJCF）
- **Isaac Lab / Isaac Sim**：内置 Shadow Hand
- **PyBullet**：简化模型

> Shadow Hand 的 MuJoCo 模型是**研究界最被信任的灵巧手仿真模型**（摩擦、接触都标定过）。

---

## 5. 应用场景

| 场景 | 为什么选 Shadow Hand |
| --- | --- |
| **In-hand tool use** | 24-DoF + 触觉，能拧螺丝 / 拿钥匙 |
| **乐器演奏（钢琴/吉他）** | 5 指完整 + 触觉反馈 |
| **5 指专属任务** | 小指侧支撑、外科辅助 |
| **产品级灵巧手研究** | 军工级可靠性 |
| **基础科学问题** | 神经科学（人手控制机理） |
| **长期实验** | 工业级耐久 |

不擅长：成本敏感（预算 < $5 万）、快速迭代（不可改）、学生教学（门槛太高）。

---

## 6. 与其他灵巧手对比

| 维度 | Shadow | [Allegro](allegro.md) | [LEAP Hand](leap-hand.md) |
| --- | --- | --- | --- |
| **DoF** | **24** | 16 | 16 |
| **指段** | 5 | 4 | 4 |
| **电机位置** | 外置驱动箱 | 内置指段 | 内置指段 |
| **触觉** | **20+** | 6×6 阵列 | 无（可外接） |
| **开源** | 半开源（ROS） | SDK 开源 | **全开源** |
| **价格** | **$10-15 万** | $1.5-2 万 | $3000 |
| **耐用性** | **极高** | 高 | 中 |
| **维护成本** | 高（肌腱） | 中 | 低 |
| **Sim-to-Real Gap** | 中（10-15%） | 中（10-15%） | **低（< 5%）** |
| **代表工作** | Dactyl、SAC-X、BC-Z | Dactyl（2019）、DDEX | LEAP 论文 |
| **研究角色** | **黄金标准** | 商用主流 | 开源新锐 |

> **「研究标杆用 Shadow，研究民主化用 LEAP，工程化用 Allegro」**。LEAP 在 2024+ 势头最猛，但 Shadow 在**顶级会议 + 工业研发**仍不可替代。

---

## 7. 常见坑

- **价格 + 维护**：单手 $10-15 万 + 维护合同年费 10%+
- **肌腱标定**：每 6-12 月需重新标定肌腱张力
- **肌腱断裂**：高频实验中可能发生
- **重量**：4.2 kg（手 + 驱动箱）需高负载臂（Panda 满载、UR 不可）
- **触觉数据量大**：20+ 触觉 @ 1 kHz = 20k 采样/秒，需高带宽
- **仿真差距**：摩擦 / 接触标定难，**sim-to-real gap 比 LEAP 大**
- **5 指资源占用**：24-DoF 训练比 16-DoF 多 30-50% 算力

---

## 8. 在 IL/RL 里的典型用法

### 8.1 OpenAI Dactyl（2018-2019）

```python
# Shadow Hand + 域随机化 + 视觉伺服
env = ShadowHandCubeEnv(
    num_envs=8192,                  # 8192 并行
    domain_randomization=True,
    visual_servo=True,
    reward='sparse'                 # 稀疏奖励
)
# 训练 100M 步，约 1 周 8×V100
```

**成果**：单手玩魔方，**sim-to-real 迁移成功**（虽然真实成功率 20%）。

### 8.2 DeepMind SAC-X（2020）

- 多任务辅助奖励（sparse reward 难训，加辅助 reward）
- Shadow Hand 完成 9 个 dexterous 任务
- 代表「**max-entropy exploration + 任务无关**」路线

### 8.3 跨模态操作（BC-Z, 2022）

- 视觉语言指令 → Shadow Hand 动作
- 训练 100+ 任务，100k 演示

---

## 9. 相关语料互链

- [`./allegro.md`](allegro.md) — 商用灵巧手主流
- [`./leap-hand.md`](leap-hand.md) — 开源新锐
- [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md) — 2 指夹爪对比
- [`../arms/franka-panda.md`](../arms/franka-panda.md) — Shadow Hand 最常挂的臂
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md) — Shadow Hand 主要用 RL
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md) — DDEX 等灵巧手 DP
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) — BC-Z 等跨模态
- [`../../concepts/foundations/sim-to-real.md`](../../concepts/foundations/sim-to-real.md) — Shadow sim-to-real
- [`../../simulation/sim-to-real/domain-randomization.md`](../../simulation/sim-to-real/domain-randomization.md) — 灵巧手 sim-to-real
- [demo 08-灵巧手-五指旋转立方体](../../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md) — 直接落地场景

---

## 10. 参考链接

**官方来源**：

- Shadow Dexterous Hand 官方产品系列页（来源：https://shadowrobot.com/dexterous-hand-series/，访问于 2026-09-02）

- Shadow Robot 官方：https://www.shadowrobot.com/
- 论文：Shadow Dexterous Hand 系列（2005+）
- OpenAI Dactyl（2018-2019）：https://openai.com/blog/learning-dexterity/
- DeepMind SAC-X：https://arxiv.org/abs/1801.02892
- DeepMind BC-Z：https://arxiv.org/abs/2202.03347
- MuJoCo 模型：https://github.com/shadow-robot/sr_mujoco
- ROS 驱动：https://github.com/shadow-robot/sr_ethercat
- 学术 benchmark：DeepMind Control Suite、MuJoCo Benchmark
- 行业评测：IEEE Spectrum、The Robot Report
