# Meta DIGIT · 视触觉传感器（开源 $15）

> **Meta AI（FAIR）2020 开源的"低成本视触觉传感器"**——**$15/个**、**开源设计**、**USB 摄像头** + **硅胶触头** 的极简设计。**把高分辨率触觉传感器从 $1k+ 打到 $15**——是灵巧手研究民主化的关键。

> 最后更新：2026-08

---

## 1. 概览

| 项 | 详情 |
| --- | --- |
| **厂家** | Meta AI（FAIR，2020 开源） |
| **发布** | 2020-06（论文 + 硬件开源） |
| **价格** | **$15 自制**（商业版 $300） |
| **类型** | **视触觉（vision-based tactile）** |
| **应用** | **灵巧手 + 机器人 + 触觉研究** |
| **销量** | **数万个**（学术 + 工业） |

> DIGIT 让"**每根手指配触觉**"成为可能——之前是高端灵巧手才有。

---

## 2. 规格

| 参数 | 数值 | 备注 |
| --- | --- | --- |
| **触头** | 硅胶（可换） | 软硬度不同 |
| **成像** | **USB 摄像头**（640×480 60fps） | 极简 |
| **视场** | ~16×12 mm | 触头面积 |
| **空间分辨率** | ~0.05 mm | 视触觉高 |
| **力感知** | 间接（图像 → 力估计） | 用 CNN |
| **采样率** | 60 Hz | 视觉限制 |
| **接口** | USB 2.0 | 标准 |
| **尺寸** | 30×25×25 mm | 紧凑 |
| **重量** | < 10 g | 极轻 |
| **触头可换** | ✅ | 3D 打印 + 硅胶 |
| **寿命** | ~1-2 月（高强度） | 可重铸 |

---

## 3. 工作原理

### 3.1 视触觉机制

```
[硅胶触头]（接触物体）
     ↓
[底部摄像头]（看触头内部变形）
     ↓
[图像处理]（力 / 纹理 / 滑移）
     ↓
[CNN 估计]（力大小 / 滑动概率 / 纹理分类）
```

**关键洞察**：**触头变形 → 图像**——**等效于触觉的"视觉化"**。

### 3.2 触头设计

- **材质**：硅胶（硬度 10-30 Shore A）
- **表面**：有 / 无 pattern（梅花 / 平滑）
- **尺寸**：直径 14-16 mm（适配指尖）
- **3D 打印模具**：开源

### 3.3 标定（关键）

```python
# 标定数据采集
def collect_calibration():
    for force in [0.0, 0.5, 1.0, 1.5, 2.0, 2.5, 3.0]:  # N
        for position in [...]:  # 不同位置
            image = digit.get_frame()
            save(image, force, position)
    return dataset

# 训练 CNN
model = train_cnn(images, forces)
predicted_force = model.predict(digit.get_frame())
```

---

## 4. 触觉感知任务

| 任务 | 实现 | 准确率 |
| --- | --- | --- |
| **力大小估计** | 视触觉 CNN | ±0.1 N |
| **纹理识别** | 图像分类 | 95%+ |
| **滑移检测** | 帧差 CNN | 90%+ |
| **形状重建** | 接触图 + 几何 | 亚毫米 |
| **接触位置** | 图像分割 | 0.5 mm |
| **物体识别** | 触觉 + 视觉 | 99%+ |
| **抓取稳定性** | 多触觉融合 | — |

---

## 5. 软件栈

### 5.1 Python 接口

```python
from digit_interface import Digit

d = Digit("D00001", "Touch")  # 序列号 + 触头类型
d.connect()
frame = d.get_frame()  # numpy (240, 320, 3)
d.disconnect()
```

### 5.2 ROS2 集成

```bash
sudo apt install ros-${ROS_DISTRO}-digit-ros

ros2 launch digit_ros digit.launch.py
# 发布话题：
#   /digit/image_raw (sensor_msgs/Image)
#   /digit/info      (sensor_msgs/CameraInfo)
```

### 5.3 力估计 / 滑移检测

```python
# 力估计（迁移学习）
from torchvision.models import resnet18
model = resnet18(pretrained=True)
model.fc = nn.Linear(512, 1)  # 输出力值

# 训练数据：DIGIT 标定数据集
# 推理：force_n = model.predict(digit_image)

# 滑移检测（时序）
lstm = LSTM(input_size=512, hidden_size=128, num_layers=2, output_size=1)
slip_prob = lstm(sequence_of_features)  # 0-1
```

### 5.4 仿真

- **Taxim**（MIT）：D435-类触觉仿真
- **FOTS**（Stanford）：视触觉光学仿真
- **Isaac Lab**：内置 DIGIT 模型（部分）

---

## 6. 适配灵巧手

| 灵巧手 | DIGIT 集成 | 触点数 |
| --- | --- | --- |
| **LEAP Hand** | 自制支架 + 5 指 × 1 DIGIT | 5 |
| **Allegro v5** | 内置触觉 + DIGIT 可加 | 5+ |
| **Shadow Hand** | BioTac 原生 + DIGIT 改装 | 20+ |
| **Figure 02 手** | 自研触觉 | 16+ |
| **1X Neo 手** | 自研触觉 | 15+ |
| **宇树 G1 手** | 3 指 × 1 DIGIT（可加） | 3 |

> **LEAP Hand + DIGIT = 当前最便宜的"5 指 + 触觉"灵巧手**（$3000 + 5×$15 = $3075）

---

## 7. 与其他触觉对比

| 维度 | DIGIT | Tekscan FlexiForce | SynTouch BioTac | Xela 3D |
| --- | --- | --- | --- | --- |
| **价格** | **$15（最便宜）** | $100-300 | $1k-3k | $500 |
| **原理** | 视触觉 | 电阻 | 多种（电阻/电容/温度/振动） | 霍尔 |
| **空间分辨率** | **0.05 mm（高）** | 1 mm | 1 mm | 1 mm |
| **力估计精度** | ±0.1 N | ±0.5 N | ±0.1 N | ±0.2 N |
| **采样率** | 60 Hz | 100 Hz | 1 kHz | 100 Hz |
| **滑移检测** | ✅（CNN） | ❌ | ✅（振动） | ✅ |
| **温度** | ❌ | ❌ | ✅ | ❌ |
| **开源** | **✅ 完全** | ❌ | ❌ | ❌ |
| **可改造** | **高** | 低 | 低 | 低 |

> **DIGIT = 极低价格 + 高空间分辨率 + 开源**——是学术研究首选。**BioTac = 多模态 + 工业级**——是高端应用首选。

---

## 8. 触觉数据集（用 DIGIT）

| 数据集 | 规模 | 内容 |
| --- | --- | --- |
| **DIGIT-360** | 11 GB | 320 类物体触觉 |
| **Touch100** | 100 类 | 视触觉分类 |
| **F-TAC** | 100 类 | 操作触觉 |
| **YCB-Slide** | - | 滑移检测 |
| **ObjectFolder 2.0** | 100 类 | 触觉 + 视觉 + 声 |
| **SSVTP** | 1k+ 类 | 自监督视触觉预训练 |

---

## 9. 应用案例

### 9.1 学术研究

- **Stanford LEAP Hand + DIGIT**（[leap-hand.md](../hands/leap-hand.md)）—— 5 指灵巧手触觉
- **MIT Digger Finger**—— 视触觉挖掘
- **DeepMind Tactile-Sensor Fusion**—— 触觉 + 视觉融合

### 9.2 工业应用

- **Amazon Robotics**—— DIGIT 用于物流分拣（试点）
- **汽车装配**—— 触觉检测
- **食品加工**—— 软物质抓取

### 9.3 创意应用

- **艺术装置**（触觉绘画）
- **医疗康复**（触觉反馈）
- **教育**（触觉感知教学）

---

## 10. DIY 自制指南

### 10.1 所需材料

| 部件 | 数量 | 价格 |
| --- | --- | --- |
| USB 摄像头（小型） | 1 | $3 |
| 硅胶（Ecoflex 00-30） | 50 ml | $5 |
| 3D 打印触头模具 | 1 | $1 |
| LED 灯条 | 1 | $2 |
| 外壳（3D 打印） | 1 | $2 |
| 线缆 | 1 | $2 |
| **总计** | — | **$15** |

### 10.2 自制步骤

1. **3D 打印外壳**（FreeCAD 模型开源）
2. **浇注硅胶触头**（真空脱泡）
3. **安装 USB 摄像头**（带 LED 灯条）
4. **连接支架**（适配灵巧手）
5. **标定**（用标准砝码 + 视觉）

### 10.3 改造技巧

- **多模式触头**（梅花 / 平滑 / 沟槽）—— 适配不同任务
- **多层硅胶**—— 触头硬度可调
- **加宽视场**—— 适配大面积
- **多 DIGIT 组合**—— 触觉阵列

---

## 11. 选型决策

| 场景 | 推荐 | 原因 |
| --- | --- | --- |
| **学生 / 学术研究** | **DIGIT 自制** | $15 价格无敌 |
| **工业级应用** | BioTac | 稳定性 + 多模态 |
| **滑移检测** | **DIGIT** | 视触觉最擅长 |
| **微装配 / 精密** | Tekscan + DIGIT 组合 | 精度 + 视场 |
| **量产商用** | DIGIT 商业版 | 性价比 |
| **高频（1 kHz+）** | BioTac | DIGIT 仅 60 Hz |

---

## 12. 相关笔记

- 同类：[intel-realsense-d435.md](intel-realsense-d435.md) · [lidar-ouster-os1.md](lidar-ouster-os1.md)
- 灵巧手：[../hands/dexterous-hands-landscape.md](../hands/dexterous-hands-landscape.md) · [../hands/dexterous-hands-data-collection.md](../hands/dexterous-hands-data-collection.md) · [../hands/dexterous-hands-sim2real.md](../hands/dexterous-hands-sim2real.md) · [leap-hand.md](../hands/leap-hand.md)
- 概念：[../../concepts/mdp/observation-space.md](../../concepts/mdp/observation-space.md) · [../../concepts/perception/visual-representation.md](../../concepts/perception/visual-representation.md)
- 仿真：[../../simulation/platforms/nvidia-isaac.md](../../simulation/platforms/nvidia-isaac.md)
- Demo：[08-灵巧手-五指旋转立方体](../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)

## 13. 参考

- Meta DIGIT 论文（2020-06）：https://arxiv.org/abs/2005.14679
- DIGIT 开源仓库：https://github.com/facebookresearch/DIGIT
- 触觉综述：https://arxiv.org/abs/2401.05000
- Stanford DexCap（2024）：DIGIT + LEAP Hand
- MIT Taxim（视触觉仿真）
