# 灵巧手数据采集专题 · Dexterous Hand Data Collection

> **灵巧手 RL 的"数据鸿沟"**——5 指 16+ DoF 需要数万到数百万条演示，单条 1-2 分钟，靠纯人工遥操作 = 不可行。2023+ 出现了 **4 大数据采集范式**：人手重定向、VR 示教、动捕重定向、视触觉自监督。

> 最后更新：2026-08

---

## 1. 为什么数据采集是灵巧手"圣杯"

### 1.1 数据需求 vs 成本

| 任务 | 演示数 | 单条时长 | 总工时 | 成本 |
| --- | --- | --- | --- | --- |
| 单臂 push-t | 100 | 30 s | **0.8 h** | 低 |
| 双手叠毛巾 | 50 | 60 s | 0.8 h | 低（ALOHA） |
| **5 指旋转立方体** | **5000** | 60 s | **83 h** | **高** |
| **5 指工具使用** | **10000** | 90 s | **250 h** | **极高** |
| **类人复杂任务** | 100000 | 120 s | **3333 h** | **不可能** |

> 灵巧手数据需求是**单臂的 50-1000 倍**——必须用新范式。

### 1.2 灵巧手特有挑战

- **示教者疲劳**：5 指协同 + 高频控制 = 人类 30 min 后质量骤降
- **自由度爆炸**：16-24 DoF 难以精确控制
- **触觉信息**：人工无法感知指尖压力
- **本体差异**：人手 → 灵巧手需要 retargeting

---

## 2. 4 大数据采集范式

### 2.1 范式 1：人手直接戴灵巧手（最直接）

**代表**：DexCap（Stanford 2024）、CyberGlove + 灵巧手

```
人手 ──► [手套式灵巧手] ──► 关节角 + 触觉
        (人手直接操作)
```

| 优势 | 劣势 |
| --- | --- |
| 直观、灵活 | 自重疲劳（LEAP 0.9 kg，戴 30 min 手累） |
| 触觉"真实" | 灵巧手与人手本体差异需 retarget |
| 实时反馈 | 数据质量与操作者经验强相关 |

**代表实现**：
- **DexCap**（Stanford 2024）：人手戴 LEAP + VR 摄像头 → 灵巧手 + 视觉数据
- **CyberGlove** + 灵巧手：手套 + 触觉
- **CyberHands**：人手操作 → 灵巧手重放

### 2.2 范式 2：VR 示教 + 灵巧手仿真（最具扩展性）

**代表**：ALOHA、Mobile ALOHA、TeleVision

```
操作员 ──► VR 控制器 ──► 仿真灵巧手 ──► 真机灵巧手 retarget
              │
              └─► 多相机捕捉 → 视觉策略
```

| 优势 | 劣势 |
| --- | --- |
| 操作员舒适 | 仿真 → 真机 Sim-to-Real gap |
| 远程操作（家庭场景） | 控制器自由度有限（2-6 DoF） |
| 多机同步 | VR 触觉差 |

**代表实现**：
- **ALOHA + Spacemouse**：Stanford 2023，2× 6-DoF 控制器
- **Mobile ALOHA**：加移动底盘
- **TeleVision**（Stanford 2024）：VR 头显 + 灵巧手
- **GELLO**（UC Berkeley 2024）：低成本 VR 遥操作
- **AnyTeleop**（Stanford 2024）：通用 VR 遥操作

### 2.3 范式 3：动捕 + 重定向（最精确）

**代表**：PhaseSpace（OpenAI Dactyl 用）、OptiTrack、ARKit

```
人手 ──► 动捕手套 ──► 人手姿态 ──► IK retarget ──► 灵巧手
   │
   └─► PhaseSpace markers（8 摄像头 + 主动 LED）
```

| 优势 | 劣势 |
| --- | --- |
| 精度高（亚毫米） | 设备贵（PhaseSpace $50k+） |
| 实时 | 穿戴麻烦 |
| 关节角直出 | 受光学遮挡 |

**代表实现**：
- **OpenAI Dactyl 2018**：PhaseSpace 8 摄像头 + Allegro
- **斯坦福 ALOHA Phase 1**：PhaseSpace + 灵巧手
- **Xsens MVN**（惯性动捕）：次选

### 2.4 范式 4：视触觉自监督（最有前景）

**代表**：DIGIT + 自监督学习、人类视频预训练

```
摄像头 ──► [人类操作视频] ──► 视频模型 ──► 动作 / 触觉预测
   │
   └─► DIGIT 触觉 → 滑移检测 / 力估计
```

| 优势 | 劣势 |
| --- | --- |
| **零人工示教** | 模型泛化弱 |
| 数据可互联网规模 | 触觉预测需大量训练 |
| 视频 / 触觉融合 | sim-to-real 仍存 |

**代表实现**：
- **Pi0-FAST**（Physical Intelligence 2024）：自监督动作 token
- **R3M / VIP**（Stanford）：人类视频预训练
- **DROID / Open X-Embodiment**：大规模混合数据
- **MimicPlay**（2024）：人类视频 + 机器人混合训练

---

## 3. 5 大主流方法深度对比

### 3.1 DexCap（Stanford 2024）

**全名**：DexCap: Dexterous Capture of Human Hand

**原理**：
1. 人手戴 LEAP Hand + 摄像头
2. VR 控制器标定人手 → 灵巧手关节映射
3. 操作员执行任务 → 关节角 + 视觉 + 触觉记录
4. 离线 retargeting 到 5 指灵巧手

**成果**：
- 10 任务 / 50 演示 / 90% 成功率（in-hand rotation 等）
- 用 1/10 数据超过纯遥操作

**代表引用**：[demo 08](../../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md) 灵巧手任务

### 3.2 ALOHA / Mobile ALOHA（Stanford 2023-2024）

**全名**：A Low-cost Open-source Hardware System for Bimanual Manipulation

**原理**：
1. 操作员用 2× 6-DoF 控制器（Spacemouse + 自制）
2. 同步记录 3 相机 + 关节角
3. **遥操作 → 训练 ACT**（动作分块）
4. ACT 直接迁移到真机

**成果**：
- **50 演示学会叠毛巾**（[demo 02](../../../demo/scenarios/02-ACT-双手叠衣/README.md)）
- Mobile ALOHA：5-50 演示学会复杂厨房任务

**硬件**：
- 双臂：$20k（自建）
- 控制器：$200 × 2
- 相机：$100 × 3

### 3.3 OpenAI Dactyl 2018-2019

**全名**：Learning Dexterity（OpenAI）

**原理**：
1. PhaseSpace 8 摄像头 + 主动 LED
2. 人手戴 26 标记球
3. 操作员戴 Allegro v3 + 摄像头
4. **域随机化** + 视觉伺服
5. **仿真训练 → 真实迁移**

**成果**：
- **2018**：单手玩魔方（20% 真实成功率）
- **2019**：单手堆方块（视觉伺服）

**方法论贡献**：
- 域随机化系统化
- 仿真中加视觉噪声 → 真实迁移
- 训练 100M 步

### 3.4 GELLO（UC Berkeley 2024）

**全名**：GELLO: A General, Low-Cost, and Easy-to-Make Teleoperation System

**原理**：
- **3D 打印的"机器人外骨骼"**——按主从方式镜像机器人结构
- 操作员戴外骨骼 → 灵巧手跟着动
- 极低成本：$300

**优势**：
- **遥操作 + 真机绑定**（无 sim-to-real gap）
- 即用即学（5 min 上手）
- 适合工业示教

**对比**：
| 维度 | GELLO | ALOHA | VR 示教 |
| --- | --- | --- | --- |
| 成本 | **$300** | $20k | $1k |
| 上手 | 5 min | 30 min | 30 min |
| 自由度 | 灵巧手同 | 6+6 DoF | 受限 |
| 触觉 | 真实 | 真实 | 弱 |

### 3.5 MimicPlay（2024 创新）

**全名**：MimicPlay: Long-Horizon Imitation Learning by Watching Human Play

**原理**：
1. **人类玩游戏的视频**（YouTube 上 100+ 小时）
2. 视频模型预测**高层计划**（如"先抓 A，再放 B"）
3. 灵巧手只学**底层动作**

**突破**：
- **无需机器人演示**：仅人类视频
- **长程任务**：5-10 步

---

## 4. 触觉数据采集专题

### 4.1 触觉数据有什么用

| 任务 | 触觉作用 |
| --- | --- |
| **滑移检测** | 触觉图像 → CNN 预测是否滑 → 重新夹紧 |
| **力估计** | 触觉 → 力大小 → 阻抗控制 |
| **纹理识别** | 触觉 → CNN 分类 → "这是纸 vs 布" |
| **抓取稳定性** | 多触觉点 → 综合评分 |
| **工具使用** | 触觉 → 握把力调整 |

### 4.2 DIGIT 触觉数据集

| 数据集 | 规模 | 内容 |
| --- | --- | --- |
| **DIGIT-360** | 11 GB | 320 类物体触觉 |
| **Touch100** | 100 类 | 视触觉分类 |
| **YCB-Slide** | - | 滑移检测 |
| **F-TAC** | - | 100 类操作触觉 |

### 4.3 触觉自监督学习

**关键洞察**：触觉数据**不需人类标注**——抓握 / 接触 / 滑移 都是**物理状态**：

```python
# 自监督信号
gripper_pressure = tactile_cnn(tactile_image)
slip_prob = slip_detector(tactile_seq)
reward = -slip_prob  # 越少滑越好
```

---

## 5. 选型决策

### 5.1 选什么数据采集方式

| 场景 | 推荐 | 原因 |
| --- | --- | --- |
| **学生 / 快速验证** | **GELLO** | $300，5 min 上手 |
| **研究 / 数据集发布** | **ALOHA + ACT** | 经典，论文最常用 |
| **5 指灵巧手** | **DexCap** | 人手重定向 + VR |
| **工业示教** | **主从外骨骼** | 真机直接示教 |
| **大规模数据** | **人类视频 + MimicPlay** | 无需机器人 |
| **跨人形公司** | **VR + 自定义 retarget** | 类似 Figure 方案 |

### 5.2 数据量 vs 任务复杂度

```
任务复杂度
   ▲
100k │              MimicPlay / Internet
     │              Video Pretrain
     │
10k  │          DAgger + Human-in-loop
     │          (VLM-as-Expert)
     │
1k   │     DDEX / ACT（100-1000 演示）
     │     Diffusion Policy
     │
100  │  ALOHA + ACT（50-100 演示）
     │  经典 BC
     │
  0  └──────────────────────────────────► 数据成本
```

---

## 6. 未来趋势（2025-2026）

1. **VLM-as-Expert 普及**：GPT-4V 当"在线示教导师"
2. **人类视频 + 机器人联合训练**：MimicPlay 类方法的扩展
3. **触觉 + 视觉 + 力融合**：基础模型路线
4. **众包数据采集**：FARM（家庭农场类）模式
5. **强化学习辅助**：DAgger + RL 微调混合

---

## 7. 相关笔记

- 单手：[allegro.md](allegro.md) · [leap-hand.md](leap-hand.md) · [shadow-hand.md](shadow-hand.md)
- 专题：[全景](dexterous-hands-landscape.md) · [控制](dexterous-hands-control.md) · [Sim-to-Real](dexterous-hands-sim2real.md) · [算法](dexterous-hands-algorithms.md)
- 方法：[imitation-learning](../../methods/imitation-learning.md) · [behavioral-cloning](../../methods/behavioral-cloning.md) · [dagger](../../methods/dagger.md) · [act](../../methods/act.md) · [diffusion-policy](../../methods/diffusion-policy.md)
- 触觉：[digit-tactile.md](../sensors/digit-tactile.md)
- Demo：[02-ACT-双手叠衣](../../../demo/scenarios/02-ACT-双手叠衣/README.md) · [08-灵巧手-五指旋转立方体](../../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)

## 8. 参考

**官方来源**：

- LEAP Hand 官方站（DexCap 采集载体）（来源：https://leaphand.com/，访问于 2026-09-02）
- Allegro Hand 官方站（Dactyl 采集载体）（来源：https://www.allegrohand.com/，访问于 2026-09-02）

- Stanford ALOHA 论文（2023）
- Stanford DexCap 论文（2024）
- Mobile ALOHA（2024）
- OpenAI Dactyl 博客（2018-2019）
- GELLO 论文（2024）
- MimicPlay（2024）
- Physical Intelligence π0（2024）
- 触觉综述：https://arxiv.org/abs/2401.05000
