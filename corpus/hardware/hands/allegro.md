# Allegro Hand · 商用四指 16-DoF 灵巧手

> **一句话定位**：韩国 Wonik Robotics（原 SimLab）研制的**四指 16-DoF 灵巧手**——研究界仅次于 Shadow Hand 的「商用灵巧手主力」，因**价格低 5× + 开源 Python 接口** 普及度反而更广，是 2018-2024 灵巧手 RL 研究的事实标准平台。

> 最后更新：2026-08

---

## 1. 概览

Allegro Hand 是 Wonik Robotics（韩国，原 SimLab）推出的**研究级灵巧手**，目标是为学术研究提供**「用得起、用得动」的灵巧操作平台**。与 [Shadow Hand](#对比) 百万美元级不同，Allegro 售价约 **1.5-2 万美元**，加上一只臂共 ~5 万美元——是大多数灵巧手实验室的「入门款」。

| 代际 | 发布年 | 关键改进 |
| --- | --- | --- |
| **Allegro Hand v3** | 2016 | 16-DoF、4 指、首批开源 |
| **Allegro Hand v4** | 2018 | 改进触觉、电机升级 |
| **Allegro Hand v5** | 2023 | 新增指尖高分辨率触觉阵列 |

> 2019 OpenAI 单手解魔方（**Dactyl**）的硬件就是 **Allegro v3** + PhaseSpace 动捕系统，**一举把「灵巧手 RL」从冷门推到台前**。

---

## 2. 规格

| 参数 | 数值 | 备注 |
| --- | --- | --- |
| **指段数** | 4（拇指 + 食指 + 中指 + 无名指） | 无小指 |
| **自由度（DoF）** | **16**（每指 4 关节，拇指 4） | 单指 = 4-DoF |
| **关节配置** | 拇指：CMC/ABD/ABD/ABD；其他指：ABD/ABD/ABD（近/中/远指间关节联动） | 非完全独立 |
| **执行器** | **电机内置指段**（无外置驱动） | 体积紧凑 |
| **通信** | EtherCAT / CAN / RS-485 | 主控用 PC/嵌入式 |
| **控制频率** | 333 Hz（关节级） | 高频 |
| **抓握力** | 单指 ~15 N | 4 指合力 ~60 N |
| **自重** | 约 1.1 kg（单手） | 较轻 |
| **指尖触觉** | 6×6 或 16×16 触觉阵列（v5） | 压电式 |
| **腕部接口** | 标准法兰（M5 / 4-M4） | 适配 Panda / UR / KUKA |
| **供电** | 24 V DC | 桌面电源 |
| **售价** | **~$1.5-2 万**（单手） | Shadow Hand 1/5 |
| **SDK** | **Python / C++ 官方 SDK（开源）** | ROS / ROS2 集成 |

> 关键参数：16-DoF 在「控制难度」和「灵巧度」之间取平衡——比 1-DoF 夹爪复杂 10×，但比 24-DoF Shadow Hand 简单 1.5×。

---

## 3. 工作原理与设计权衡

### 3.1 4 指 vs 5 指

Allegro 是**4 指设计**（无小指），这与 Shadow Hand 的 5 指不同：

| 维度 | 4 指（Allegro） | 5 指（Shadow / LEAP） |
| --- | --- | --- |
| 抓取稳定度 | 中（小指侧略弱） | 高 |
| 控制复杂度 | 低（16-DoF） | 高（20-24-DoF） |
| 制造成本 | 低 | 高 |
| 仿真难度 | 中 | 高 |
| 适用任务 | in-hand rotation、pinch、envelop | 复杂 dexterous tool use |

> 4 指在 in-hand rotation（[demo 08 引用](../../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)）等任务上**与 5 指差距小**，但样本效率**高 30-50%**。

### 3.2 关节联动（coupled joints）

单指的 4 个关节并非完全独立：

- **远端两关节（远指间 + 近指间）** 通过**耦合机构（tendon / gear）** 联动，类似人手（人手也是 PIP-DIP 联动）
- 优点：减少独立控制变量（4 → 3）、增加抓取稳定性
- 缺点：丧失「独立指节控制」（如按琴键做不到）

### 3.3 内置电机 vs 外置驱动

Allegro 电机在指段内部，无需外置「肌腱驱动箱」（如 Shadow Hand 那样），所以：
- ✅ 体积紧凑
- ❌ 单指坏了要换整指段
- ❌ 散热差，长时间高负载降速

---

## 4. 控制接口与软件生态

### 4.1 通信与驱动

| 接口 | 说明 |
| --- | --- |
| **EtherCAT** | 主控接口，1 kHz 实时控制 |
| **CAN** | 备选，慢一些 |
| **RS-485** | 调试用 |

### 4.2 SDK

```python
# 官方 Python SDK（github.com/SimLab-Research/allegro_hand_ros）
from allegro_hand import AllegroHand
hand = AllegroHand(hand_type='right', port='/dev/ttyUSB0')
hand.set_joint_positions([0.1, 0.2, ...])  # 16 维目标
state = hand.get_joint_positions()         # 16 维反馈
```

### 4.3 ROS / ROS2

- `allegro_hand_ros`（ROS1）：官方包
- `allegro_hand_ros2`（ROS2）：社区维护
- Isaac Lab / MuJoCo：内置 Allegro 模型

### 4.4 与主流机械臂的兼容性

| 机械臂 | 兼容方式 | 备注 |
| --- | --- | --- |
| [Franka Panda](../../hardware/arms/franka-panda.md) | 法兰 + EtherCAT 总线 | 主流方案 |
| KUKA LWR / iiwa | 法兰 + CAN | 工业方案 |
| [UR5e](../../hardware/arms/ur5e.md) | 法兰（需配驱动板） | 非主流 |
| Humanoid（如 H1） | 腕部集成 | 前沿探索 |

---

## 5. 应用场景

| 场景 | 为什么选 Allegro |
| --- | --- |
| **In-hand manipulation RL** | OpenAI Dactyl、Stanford LEAP 都用 Allegro 验证算法 |
| **Diffusion Policy 灵巧版（DDEX）** | 4 指 + DP，是当前 SOTA 灵巧手 IL 路线 |
| **DexCap 示教采集** | 戴在人手上做数据采集，比 Shadow 便宜 |
| **教学 / benchmark** | 5 万美元一套（含臂），学生实验室可负担 |
| **多手协作（双手）** | 两手 ~3 万美元，做 bimanual dexterous 比 Shadow 可行 |
| **灵巧手 sim-to-real** | 仿真 ↔ 真机差距比 Shadow 小（结构简单） |

不擅长：超精密工具使用（手表/医疗）、5 指专属能力（钢琴演奏）、高负载（>5 kg 抓持）。

---

## 6. 与其他灵巧手对比

### 6.1 vs [Shadow Hand](shadow-hand.md)

| 维度 | Allegro | Shadow Hand |
| --- | --- | --- |
| **DoF** | 16 | 24 |
| **指段** | 4 | 5 |
| **电机位置** | 内置指段 | 外置驱动箱 + 肌腱 |
| **触觉** | 6×6 阵列（v5） | 整手 ~100 触点 |
| **售价** | **$1.5-2 万** | **$10-15 万** |
| **控制难度** | 中 | 极高 |
| **可靠性** | 中（电机过热） | 高（军工级） |
| **代表工作** | OpenAI Dactyl、Stanford LEAP、DEX-ART | OpenAI In-hand Cube、DeepMind SAC-X |

> **Allegro 适合算法研究，Shadow Hand 适合「产品级」研究**。多数论文选 Allegro 是因为**可重复性**（5 万 vs 100 万）。

### 6.2 vs [LEAP Hand](leap-hand.md)

| 维度 | Allegro | LEAP Hand |
| --- | --- | --- |
| **DoF** | 16 | **16** |
| **设计** | 商用 | **开源 3D 打印**（Stanford 2023） |
| **售价** | $1.5-2 万 | **~$3000**（DIY 组装） |
| **SDK** | 官方 Python | 开源 Python + Isaac Sim |
| **设计哲学** | 工业稳定 | **Sim-to-Real 友好** |
| **代表工作** | Dactyl、DDEX | LEAP Hand 论文、KUKA 部署 |

> **LEAP 是 2023 后的「**研究民主化**」**——把灵巧手从 $1.5 万打到 $3000。详见 [leap-hand.md](leap-hand.md)。

### 6.3 vs [LEAP Hand / Shadow Hand] vs 2 指夹爪（[Robotiq 2F-85](../../hardware/end-effectors/robotiq-2f-85.md)）

| 维度 | 2F-85 | Allegro | LEAP | Shadow |
| --- | --- | --- | --- | --- |
| **DoF** | 1 | 16 | 16 | 24 |
| **能力** | 抓 / 放 | 抓 / 转 / 拧 / 捏 | 同 Allegro | 最强 |
| **控制难度** | 极低 | 中 | 中 | 极高 |
| **样本效率** | 极低（演示 10 条） | 高（数千万 RL 步） | 同 Allegro | 更高 |
| **价格** | $1-2 千 | $1.5-2 万 | $3000 | $10-15 万 |
| **研究角色** | IL / BC 标配 | 灵巧 RL / 灵巧 IL | 灵巧 RL（开源） | 产品级 / 长期 |

> 2F-85 是「**iPhone**」——人人用、能用、够用。Allegro / LEAP / Shadow 是「**Mac Pro**」——贵、但能干 iPhone 干不了的活。

---

## 7. 常见坑

- **电机过热**：连续高负载 5+ 分钟会降速甚至保护停机
- **指段损坏**：4 指独立可换，DIY 一指 ~$500
- **触觉校准**：v5 的触觉阵列需重新校准，校准脚本不公开
- **EtherCAT 主站**：需 Linux + EtherCAT 主站驱动（如 `igh-ethercat`）
- **仿真差距**：Isaac Lab 的 Allegro 摩擦 / 阻尼参数与真机差异较大
- **腕部线缆走线**：电缆从手背走，避免卡到场景物体
- **ROS2 兼容性**：官方 ROS2 包仍在 beta

---

## 8. 在 IL/RL 里的典型用法

### 8.1 RL（OpenAI Dactyl 路线）

```python
# Isaac Lab 训练
from omni.isaac.lab.envs import ManagerBasedRLEnv
env = AllegroCubeEnv(num_envs=4096)         # 4096 并行环境
obs = env.reset()
for t in range(200):
    action = ppo_policy(obs)                # 16-DoF 增量
    obs, reward, done, info = env.step(action)
```

### 8.2 IL（DexCap + Diffusion Policy 路线）

1. **戴 Allegro 在人手上**（DexCap 用 VR 控制器标定人手 → Allegro 关节）
2. **遥操作完成 50+ 任务演示**
3. **训练 DDEX**（Dexterous Diffusion Policy）模仿

### 8.3 数据约定

- **观测**：16 关节角 + 立方体位姿（特权信息，仿真）或指尖触觉（真机）
- **动作**：16 维关节位置增量
- **奖励**：稀疏（成功旋转给 1） 或 密集（每步距离减少给 0.01）

---

## 9. 相关语料互链

- [`../arms/franka-panda.md`](../arms/franka-panda.md) — Allegro 最常挂的臂
- [`../end-effectors/robotiq-2f-85.md`](../end-effectors/robotiq-2f-85.md) — 对比：2F-85 是 IL 标配
- [`./leap-hand.md`](leap-hand.md) — 开源更便宜的替代
- [`./shadow-hand.md`](shadow-hand.md) — 工业级对比
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md) — Allegro 主要用 RL 训
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md) — DDEX 灵巧手版
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) — DexCap 数据采集
- [`../../concepts/foundations/sim-to-real.md`](../../concepts/foundations/sim-to-real.md) — Allegro sim-to-real
- [`../../simulation/platforms/nvidia-isaac.md`](../../simulation/platforms/nvidia-isaac.md) — Isaac Lab 训练
- [`../../simulation/sim-to-real/domain-randomization.md`](../../simulation/sim-to-real/domain-randomization.md) — 灵巧手 sim-to-real 关键
- [demo 08-灵巧手-五指旋转立方体](../../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md) — 直接落地场景

---

## 10. 参考链接

**官方来源**：

- Allegro Hand 官方站（来源：https://www.allegrohand.com/，访问于 2026-09-02）

- Wonik Robotics 官方：https://www.wonikrobotics.com/
- 官方 SDK：https://github.com/SimLab-Research/allegro_hand_ros
- Isaac Lab Allegro 任务：https://github.com/NVlabs/IsaacLab
- OpenAI Dactyl（2019）：https://openai.com/blog/learning-dexterity/
- Stanford LEAP Hand 论文（2023）
- DDEX（2024 灵巧手 DP）：Dextrous Diffusion Policy
- DexCap（2024 数据采集）：Stanford
- 学术 benchmark：IsaacGymEnvs、Isaac Lab AllegroCube
