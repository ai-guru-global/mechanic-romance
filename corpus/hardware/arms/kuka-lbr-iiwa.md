# KUKA LBR iiwa · 力控 7-DoF 协作机械臂

> **KUKA LBR iiwa（intelligent industrial work assistant）**——**德国 KUKA** 的**协作机械臂代表**，**7-DoF + 内置力矩传感器**——是**协作机器人 / 安全 / 力控** 三大场景的**事实标准**。**研究 / 工业 / 医疗** 三栖全用。

> 最后更新：2026-08

---

## 1. 概览

| 项 | 详情 |
| --- | --- |
| **厂家** | KUKA（德国） |
| **首代** | LBR 4+ / LBR 4（2008） |
| **当前代** | LBR iiwa 7 R800 / LBR iiwa 14 R820（2014+） |
| **类型** | **7-DoF 协作机械臂** |
| **价格** | **$70,000 - $120,000** |
| **销量** | **数万台**（工业 + 研究） |

> LBR iiwa 是**"协作机械臂"的开山鼻祖之一**（与 Universal Robots UR 系列同期）。

---

## 2. 规格

| 参数 | LBR iiwa 7 R800 | LBR iiwa 14 R820 | 备注 |
| --- | --- | --- | --- |
| **自由度** | **7** | **7** | 同 |
| **负载** | 7 kg | 14 kg | |
| **工作半径** | 800 mm | 820 mm | |
| **重量** | 24 kg | 32 kg | |
| **重复精度** | ±0.1 mm | ±0.1 mm | 工业级 |
| **力矩传感器** | **每个关节内置** | **每个关节内置** | 关键差异 |
| **力矩分辨率** | 0.5 N·m | 0.5 N·m | |
| **控制频率** | 1 kHz | 1 kHz | |
| **功率** | 200 W | 350 W | |
| **防护** | IP54 | IP54 | |
| **通信** | EtherCAT / Profinet | 同 | 工业总线 |
| **碰撞检测** | **全关节** | **全关节** | 安全 |
| **编程** | KUKA.OfficeLite + Sunrise | 同 | 商业 / 学术 |

---

## 3. 关键技术

### 3.1 内置关节力矩传感器

**最核心差异**：**每个关节都集成高精度力矩传感器**：

- **7 关节 × 力矩传感** = 全手臂**精确力感知**
- **0.5 N·m 分辨率** —— 可检测 ±0.1 kg 的力
- **可做柔顺控制**（impedance / admittance）
- **碰撞检测**（比 UR 早 5 年）

> 这一点让 iiwa 在**装配 / 抛光 / 力控实验** 中无可替代。

### 3.2 7-DoF 设计

- **多 1 个自由度**（对比 6-DoF UR / Franka）
- **人手腕式冗余**——可绕过奇异点
- **更接近人手臂**（人肩-肘-腕 7-DoF）
- **代价**：运动学求解更复杂

### 3.3 协作 / 安全

- **碰撞检测**（关节力矩监控）
- **ISO 10218 / ISO/TS 15066** 认证
- **协作模式**：可与人同空间
- **紧急停止**（硬件 + 软件）
- **拖动示教**（hand-guiding）

### 3.4 ROS / 学术支持

- **ROS / ROS2** 官方驱动（KUKA 官方）
- **iiwa_stack**（ROS / ROS2）
- **PyTorch / TensorFlow** 集成
- **MoveIt** 集成
- **Gazebo / Isaac** 仿真

---

## 4. 与其他协作臂对比

| 维度 | KUKA iiwa | Franka Panda | UR5e | xArm 7 |
| --- | --- | --- | --- | --- |
| **DoF** | **7** | 7 | 6 | 7 |
| **负载** | 7 / 14 kg | 3 kg | 5 kg | 3.5 kg |
| **力矩传感** | **每个关节** | **每个关节** | 末端 | 末端 |
| **价格** | **$70-120k** | **$30-50k** | $35k | $5-10k |
| **重复精度** | ±0.1 mm | ±0.1 mm | ±0.03 mm | ±0.1 mm |
| **力矩分辨率** | 0.5 N·m | 0.5 N·m | 1 N·m | 1 N·m |
| **重量** | 24-32 kg | 18 kg | 20 kg | 12 kg |
| **控制频率** | 1 kHz | 1 kHz | 500 Hz | 1 kHz |
| **协作** | ✅ ISO | ✅ ISO | ✅ ISO | ✅ |
| **开源度** | 中 | **高（社区活跃）** | 中 | 中 |
| **生态** | 商业 | **学术 + 开源** | 商业 | 商业 |

> **iiwa = 工业级 + 早期 + 力控** · **Franka = 学术 + 开源 + 性价比** · **UR = 工业 + 易用**

---

## 5. 应用场景

### 5.1 工业

| 任务 | 状态 |
| --- | --- |
| **精密装配** | ✅（汽车、电子） |
| **力控抛光** | ✅（金属 / 玻璃） |
| **柔性测试** | ✅（汽车座椅） |
| **焊接** | ✅（钎焊、点焊） |
| **CNC 上下料** | ✅ |

### 5.2 学术

- **MIT / Stanford / CMU 等**用 iiwa 做研究
- 主要用：力控 + 7-DoF + 安全
- **Imitation Learning** 重要实验平台
- **Open-Dynamic-Robot-Initiative** 用 iiwa 做多指

### 5.3 医疗

- **骨科手术**（Mako 等）
- **康复**（KUKA 与康复设备集成）
- **辅助手术**

### 5.4 协作

- **人机共线**（生产线上）
- **手把手示教**

---

## 6. 软件栈

### 6.1 商业（KUKA 官方）

- **KUKA.OfficeLite**（编程 + 仿真）
- **KUKA Sunrise**（Java / Python）
- **KUKA.WorkVisual**（离线编程）
- **KUKA.Cockpit**（运行管理）

### 6.2 学术 / 开源

- **ROS / ROS2**：`iiwa_stack`
- **MoveIt** 集成
- **Gazebo / Isaac Sim** 模型
- **Python / C++** 控制接口

### 6.3 编程示例（ROS2）

```python
# ROS2 iiwa 控制
import rclpy
from iiwa_msgs.msg import JointPosition
from iiwa_msgs.srv import SetSpeed

rclpy.init()
node = rclpy.create_node('iiwa_controller')

# 拖动示教 → 录轨迹 → 回放
client = node.create_client(SetSpeed, '/iiwa/controller/speed')
# ...

# 控制到目标关节角
target = JointPosition()
target.position.a1 = 0.0
target.position.a2 = 1.0
# ...
pub = node.create_publisher(JointPosition, '/iiwa/command/JointPosition', 10)
pub.publish(target)
```

---

## 7. 商业模式

### 7.1 销售模式

```
硬件（$70-120k）
  + 软件许可（$10-30k）
  + 服务（$5-15k/年）
  + 培训（$3-5k）
  + 应用集成（$10-50k）
```

### 7.2 客户分布

- **汽车**（30%）：宝马、大众、奥迪
- **电子**（20%）：装配、测试
- **医疗**（15%）：骨科、康复
- **学术**（15%）：大学、研究机构
- **其他**（20%）：食品、零售

---

## 8. 优势与挑战

### 8.1 优势

- **关节力矩传感**（业界领先）
- **7-DoF 冗余**（接近人）
- **工业级稳定**
- **协作安全**
- **品牌效应**（KUKA 是德国工业代表）

### 8.2 挑战

- **价格高**（$70k+ vs Panda $30k+）
- **软件闭源**（比 Panda 弱）
- **社区弱**（Panda 学术社区更大）
- **重量大**（24-32 kg）

---

## 9. 选型决策

| 场景 | 推荐 | 原因 |
| --- | --- | --- |
| **工业力控** | **iiwa** | 关节力矩 + 协作 |
| **学术研究** | **Franka** | 开源 + 价格 |
| **消费 / 教学** | xArm 7 | 价格 + 简易 |
| **重型工业** | UR 16e / Fanuc | 负载 |
| **安全协作** | iiwa / UR | 协作安全 |
| **精密装配** | **iiwa** | ±0.1 mm |

---

## 10. 2025-2026 关注

1. **iiwa 新代**—— 加 VLA / 视觉集成
2. **价格策略**—— 应对 Panda / xArm
3. **医疗版**—— 临床 FDA 认证
4. **KUKA 与 NVIDIA 合作**—— 数字孪生
5. **欧洲市场扩展**

---

## 11. 相关笔记

- 同类：[franka-panda.md](franka-panda.md)（**学术研究首选**）· [ur5e.md](ur5e.md)
- 仿真：[../../simulation/platforms/nvidia-isaac.md](../../simulation/platforms/nvidia-isaac.md)
- 概念：[../../concepts/mdp/action-space.md](../../concepts/mdp/action-space.md) · [../../concepts/foundations/degrees-of-freedom.md](../../concepts/foundations/degrees-of-freedom.md)
- Demo：[01-扩散策略-桌面堆叠](../demo/scenarios/01-扩散策略-桌面堆叠/README.md) · [06-强化学习-Franka抓杯](../demo/scenarios/06-强化学习-Franka抓杯/README.md)
- 灵巧手：[../hands/allegro.md](../hands/allegro.md)

## 12. 参考

**官方来源**：

- KUKA LBR iiwa 官方产品页（来源：https://www.kuka.com/en-de/products/robot-systems/industrial-robots/lbr-iiwa，访问于 2026-09-02）

- KUKA LBR iiwa 官方
- KUKA Sunrise 文档
- iiwa_stack GitHub
- Open-Dynamic-Robot-Initiative
- IEEE / Springer 工业机器人综述
