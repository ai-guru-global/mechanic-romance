# NVIDIA Jetson Orin · 具身智能边缘计算平台

> **NVIDIA Jetson Orin 系列**（2023 发布）是**具身智能 / 边缘 AI 的事实标准计算平台**——100 TOPS（INT8）AI 性能、70W 功耗、Arm + Ampere GPU 异构。**人形 / 四足 / 移动机器人 / VLA 边缘推理** 几乎都用 Orin。

> 最后更新：2026-08

---

## 1. 概览

| 项 | 详情 |
| --- | --- |
| **厂家** | NVIDIA |
| **发布** | 2023-03 |
| **当前代** | Orin Nano / NX / AGX 三档 |
| **类型** | **边缘 AI 模块** |
| **价格** | **$199 - $1,999** |
| **应用** | **机器人 / 无人机 / 智能相机** |

> Orin 是**"具身智能的 RTX 3090"**——把 GPU 算力塞进 5W-60W 移动平台。

---

## 2. 规格

| 型号 | Orin Nano 4GB | Orin Nano 8GB | Orin NX 8GB | Orin NX 16GB | Orin AGX 64GB |
| --- | --- | --- | --- | --- | --- |
| **价格** | **$199** | $299 | $399 | $699 | **$1,999** |
| **AI 性能** | 20 TOPS | 40 TOPS | 70 TOPS | 100 TOPS | **275 TOPS** |
| **GPU** | 512-core Ampere | 1024-core | 1024-core | 1024-core | **2048-core** |
| **CPU** | 6-core Arm | 6-core Arm | 6-core Arm | 8-core Arm | **12-core Arm** |
| **内存** | 4GB LPDDR5 | 8GB | 8GB | 16GB | **64GB** |
| **存储** | microSD | microSD | microSD | microSD | NVMe |
| **功耗** | 7-15 W | 7-15 W | 10-25 W | 10-25 W | **15-60 W** |
| **尺寸** | 70×45 mm | 同 | 同 | 同 | 100×87 mm |

> **Orin Nano 8GB**（$299）是**性价比首选**——40 TOPS + 8GB 够跑 VLA / Diffusion Policy / YOLO。

---

## 3. 关键技术

### 3.1 异构架构

```
[CPU] 6-12 core Arm Cortex-A78AE
   ├─ 实时控制（ROS2 / FreeRTOS）
   └─ 串行任务
   ↓
[GPU] Ampere 架构（512-2048 core）
   ├─ AI 推理（TensorRT）
   ├─ 视觉处理（VPI）
   └─ 并行计算
   ↓
[PVA] Programmable Vision Accelerator
   ├─ 图像预处理
   └─ 视频编解码
   ↓
[NVDEC / NVENC]
   └─ 视频编解码（4K）
```

### 3.2 软件栈

| 工具 | 用途 |
| --- | --- |
| **JetPack 6.0+** | 系统 + CUDA + TensorRT |
| **TensorRT** | 推理优化（10× 加速） |
| **VPI** | 视觉处理库 |
| **Isaac ROS** | ROS2 GEM 加速 |
| **DeepStream** | 视频流处理 |
| **Triton** | 模型服务化 |
| **ROS2 Humble** | 默认支持 |

### 3.3 AI 加速

```python
# PyTorch → TensorRT 转换
import torch
from torch2trt import torch2trt

model = MyModel().cuda()
model.load_state_dict(torch.load("model.pth"))
model.eval()

# 转 TensorRT
model_trt = torch2trt(model, [torch.randn(1, 3, 224, 224).cuda()])
model_trt.save("model.engine")

# 推理加速
output = model_trt(input_tensor)  # 5-10× 加速
```

---

## 4. 具身智能应用

### 4.1 VLA 边缘推理

| 模型 | Orin Nano 8GB | Orin NX 16GB | Orin AGX 64GB |
| --- | --- | --- | --- |
| **YOLOv8** | 30 FPS | 60 FPS | 120 FPS |
| **CLIP** | 5 Hz | 10 Hz | 25 Hz |
| **Diffusion Policy** | 1 Hz | 3 Hz | 10 Hz |
| **OpenVLA-7B（INT8）** | ❌ | 🟡（1 Hz） | ✅（3 Hz） |
| **π₀-3B（INT8）** | ❌ | 🟡 | ✅（5 Hz） |
| **RT-2 55B** | ❌ | ❌ | ❌（需 8× AGX） |

> **Orin AGX 64GB + TensorRT-INT8 = 边缘跑 OpenVLA-7B 1-3 Hz** —— **VLA 落地的最低门槛**。

### 4.2 灵巧手控制

- **LEAP Hand + Orin Nano**：Dynamixel 通信 + 触觉融合 + 策略
- **Allegro + Orin NX**：PPO 推理 + EtherCAT 主站

### 4.3 人形机器人

- **Tesla Optimus**：自研芯片（不用 Orin）
- **Figure 02**：自研 + Orin 备选
- **1X Neo**：Orin NX
- **宇树 G1**：Orin
- **智元 A2**：Orin AGX × 2
- **Apptronik Apollo**：Orin

> 80% 的 2024+ 人形机器人用 **Jetson Orin** 系列。

### 4.4 四足机器人

- **Unitree Go2 旗舰**：Orin Nano
- **Boston Dynamics Spot**：Intel NUC（不用 Orin）
- **ANYmal C**：Orin AGX

### 4.5 移动操作

- **Mobile ALOHA**：Orin（边缘推理）
- **Stretch**：Orin

---

## 5. 与其他边缘平台对比

| 平台 | AI 性能 | 功耗 | 价格 | 生态 | 适用 |
| --- | --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | 40 TOPS | 7-15 W | **$299** | 完整 | **学生 / 中小项目** |
| **Jetson Orin NX 16GB** | 100 TOPS | 10-25 W | $699 | 完整 | **VLA / 灵巧手** |
| **Jetson Orin AGX 64GB** | **275 TOPS** | 15-60 W | $1,999 | 完整 | **人形 / 重型 AI** |
| **Apple M2 MacBook** | 15 TOPS | 25 W | $1,500 | macOS | 开发 / 仿真 |
| **Intel NUC + RTX 4060** | 100 TOPS | 150 W | $1,500 | 完整 | **真机训练** |
| **Raspberry Pi 5** | 0.5 TOPS | 5 W | $80 | 通用 | 低端控制 |
| **Google Coral** | 4 TOPS | 2 W | $60 | TensorFlow | 嵌入式 |

> **具身智能的"甜蜜点"**：**Orin NX 16GB**（$699）—— 性能够、价格中、生态好。

---

## 6. 生态优势

### 6.1 Isaac ROS 加速

- **cuVSLAM**：GPU 加速 SLAM
- **cuMotion**：运动规划
- **DOPE**：6-DoF 物体位姿
- **AprilTag**：Tag 检测
- **FoundationPose**：6-DoF 跟踪

### 6.2 完整 VLA 栈

```
Jetson Orin
   ↓
Isaac ROS（视觉 + SLAM + 规划）
   ↓
TensorRT（VLA 模型推理）
   ↓
ROS2 控制（关节 + 夹爪 + 灵巧手）
   ↓
真实硬件执行
```

### 6.3 仿真 → 真机一致性

- **Isaac Sim** → **Isaac Lab** → **Isaac ROS** → **Orin**
- 同一套代码，仿真训练 + 真机部署

---

## 7. 软件栈

### 7.1 JetPack 6.0

```
Ubuntu 20.04 + Linux 5.10
   ├─ CUDA 12.2
   ├─ cuDNN 8.9
   ├─ TensorRT 10.0
   ├─ VPI 3.0
   ├─ OpenCV 4.10
   ├─ ROS2 Humble
   └─ Isaac ROS
```

### 7.2 容器化部署

```dockerfile
FROM nvcr.io/nvidia/l4t-jetpack:r36.2.0

# 安装 ROS2
RUN apt install ros-humble-desktop

# 复制 VLA 模型
COPY vla_model.engine /workspace/

# 启动命令
CMD ["ros2", "launch", "vla_package", "deploy.launch.py"]
```

```bash
# 部署到 Orin
sudo docker run --runtime nvidia -it vla-image
```

### 7.3 性能监控

```bash
# 实时监控 GPU / 内存 / 功耗
sudo tegrastats

# 性能分析
nsys profile -o profile.qdrep python3 vla_inference.py
```

---

## 8. 选型决策

| 场景 | 推荐 | 原因 |
| --- | --- | --- |
| **学生 / 入门** | **Orin Nano 8GB** | $299 性价比 |
| **VLA 边缘推理** | **Orin NX 16GB** | 100 TOPS + 16GB |
| **人形 / 重型 AI** | **Orin AGX 64GB** | 275 TOPS + 64GB |
| **SLAM + 视觉** | Orin NX 16GB | 够用 |
| **灵巧手控制** | Orin Nano 8GB | 性能充裕 |
| **移动底盘** | Orin Nano 4GB | 轻量 |
| **训练（不在边缘）** | RTX 4090 / H100 | 训练用 |

---

## 9. 常见坑

| 坑 | 原因 | 解决 |
| --- | --- | --- |
| **功耗超限** | 满载 25W 但设计 15W | 限频 + 散热 |
| **TensorRT 不支持某些算子** | ONNX 转换失败 | 自定义 plugin |
| **JetPack 版本不匹配** | ROS2 / PyTorch 版本 | 锁定 JetPack 6.0 |
| **散热不够** | 跑大模型 60°C+ 降频 | 加风扇 / 水冷 |
| **microSD 速度慢** | 启动慢 / 加载慢 | NVMe SSD |
| **USB 带宽** | 多相机同时接 | 用 MIPI / Ethernet |
| **内存不足** | 16GB 跑 7B 模型紧 | AGX 64GB |

---

## 10. 未来与替代

### 10.1 2025-2026 趋势

- **Jetson Thor**（2025-2026 计划）—— 1000 TOPS
- **Jetson Orin Refresh** —— 价格降
- **人形机器人定制 SoC**：Tesla Dojo、Figure 内部
- **国产替代**：华为 MDC、地平线 J5/J6、寒武纪

### 10.2 国产边缘 AI 芯片

| 厂家 | 型号 | AI 性能 | 价格 | 适用 |
| --- | --- | --- | --- | --- |
| **地平线** | J5 / J6 | 128 TOPS | $300+ | 自动驾驶 + 机器人 |
| **华为** | MDC 810 | 400+ TOPS | $1k+ | 智能汽车 |
| **寒武纪** | MLU370 | 256 TOPS | $500+ | 数据中心 |
| **比特大陆** | Sophon | 32 TOPS | $100+ | 嵌入式 |

> **国产替代在"人形机器人"场景中加速**——成本 / 供应链 / 国家安全。

---

## 11. 相关笔记

- 同类：[intel-realsense-d435.md](../sensors/intel-realsense-d435.md)
- 灵巧手：[../hands/leap-hand.md](../hands/leap-hand.md)
- 人形：[../humanoids/unitree-g1.md](../humanoids/unitree-g1.md) · [../humanoids/agibot-a2.md](../humanoids/agibot-a2.md)
- 仿真：[../../simulation/platforms/nvidia-isaac.md](../../simulation/platforms/nvidia-isaac.md)
- 概念：[../../concepts/training/finetuning.md](../../concepts/training/finetuning.md)
- Demo：[04-OpenVLA-7B-开源VLA](../demo/scenarios/04-OpenVLA-7B-开源VLA/README.md) · [05-Figure02-端到端人形](../demo/scenarios/05-Figure02-端到端人形/README.md)

## 12. 参考

- NVIDIA Jetson 官方文档
- JetPack 6.0 release notes
- Isaac ROS GEM 列表
- TensorRT 优化指南
- VLA 边缘部署案例（Physical Intelligence、OpenVLA）
