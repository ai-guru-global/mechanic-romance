# Intel RealSense D435 · 主动红外深度相机

> **Intel RealSense D435** 是**机器人 / 具身智能研究**最常用的 **RGBD 相机**——**$300 出头**、**主动红外立体**、**开源 SDK 完整**、**跨平台**。从 2018 发布至今，已是 ALIKE **"深度相机的事实标准"**——配合 ROS2 / Isaac / Habitat 生态完备。

> 最后更新：2026-08

---

## 1. 概览

| 项 | 详情 |
| --- | --- |
| **厂家** | Intel RealSense（已停产但生态完整） |
| **发布** | 2018 |
| **当前** | D435 / D435i / D455 等系列 |
| **类型** | **主动红外立体深度相机** |
| **价格** | **$300-400**（二手 $150-200） |
| **销量** | **数百万台**（机器人研究标配） |

> **2024+ 趋势**：D435 仍是研究主流，但 Intel 已停产，新代是 D405（小型）和 D555（工业）。**最新 D455** 仍是市场主力。

---

## 2. 规格（D435 经典款）

| 参数 | 数值 | 备注 |
| --- | --- | --- |
| **深度范围** | 0.3 - 3 m | 室外 0.3-10 m |
| **深度分辨率** | 最高 1280×720 | 默认 848×480 |
| **深度帧率** | 90 fps | |
| **RGB** | 1920×1080 @ 30 fps | |
| **视场角（深度）** | 86°×57° | 较广 |
| **视场角（RGB）** | 69°×42° | 较窄 |
| **深度精度** | ±2 mm @ 1 m | 精度良好 |
| **尺寸** | 90×25×25 mm | 紧凑 |
| **接口** | USB 3.0 | 必用 3.0 |
| **功率** | < 2.5 W | 低功耗 |
| **IMU（i 版）** | BMI055 | D435i 才有 |
| **同步** | 硬件时间戳 | 多机同步 |
| **防尘防水** | 室内 / 室外遮光 | 不防水 |

---

## 3. 关键技术

### 3.1 主动红外立体原理

```
[左红外投射] → 物体 → [左红外相机]
                           ↓
                  视差计算 → 深度
                           ↓
[右红外投射] → 物体 → [右红外相机]
                           ↑
                  三角测量计算
```

- **红外投射器**（散斑 pattern）→ 主动补光，弱光环境也工作
- **左右红外相机** → 立体视差
- **RGB 相机** → 独立彩色图像
- **校准**：出厂校准 + 内参可用

### 3.2 SDK 生态（强项）

| 平台 | 状态 |
| --- | --- |
| **librealsense2**（官方 C++ / Python） | ✅ 完整 |
| **ROS / ROS2** realsense2_camera | ✅ 官方包 |
| **Isaac Lab / Isaac Sim** | ✅ 直接集成 |
| **Habitat** | ✅ |
| **PyBullet / MuJoCo** | ✅ |
| **OpenCV** | ✅ |
| **MATLAB** | ✅ |
| **LabVIEW** | ✅ |

### 3.3 优势与局限

| 优势 | 局限 |
| --- | --- |
| ✅ 室内外都能用（红外补光） | ❌ 玻璃 / 高反光面失效 |
| ✅ 价格低（$300） | ❌ 室外强光深度退化 |
| ✅ 精度好（±2 mm） | ❌ 范围限制（10 m） |
| ✅ SDK 完整 | ❌ Intel 2024 停止消费级（部分停产） |
| ✅ ROS2 集成 | ❌ 不防水（IP54） |
| ✅ 紧凑 | ❌ 红外干扰（多机同时） |

---

## 4. 应用场景

### 4.1 机器人 / 具身智能

| 场景 | 用法 |
| --- | --- |
| **机械臂抓取** | 点云分割 → 6-DoF 抓取位姿 |
| **人形视觉** | 头戴 RGBD + SLAM |
| **导航** | 避障 + 占用栅格 |
| **仿真真实化** | 真实 D435 相机模型（Isaac） |
| **VLA 视觉** | 多视角 D435 → VLA 输入 |

### 4.2 其他领域

- **3D 扫描**（人 / 物体）
- **AR / VR**（手部追踪）
- **医疗**（康复）
- **教育**（机器人课程）
- **3D 打印**（建模）

---

## 5. RealSense 系列对比

| 型号 | 价格 | 深度范围 | 特色 | 适用 |
| --- | --- | --- | --- | --- |
| **D405** | $300 | 0.07-1 m | 超近距（亚毫米精度） | 微操作 / 装配 |
| **D415** | $300 | 0.3-3 m | 较 D435 视场窄 | 一般场景 |
| **D435** | **$300-400** | 0.3-3 m | **经典款 / 性价比最高** | **机器人研究主流** |
| **D435i** | $400-500 | 0.3-3 m | + IMU（BMI055） | SLAM / 状态估计 |
| **D455** | $500 | 0.6-6 m | 远距 / 大视场 | 户外 / 工业 |
| **D555** | $800+ | 0.5-6 m | IP65 防水 / 工业级 | 工业 / 户外 |
| **L515** | $500 | 0.25-9 m | LiDAR 深度 | 室外长距 |

> **首选 D435**（价格 + 性能平衡），**D435i** 用于 SLAM，**D455** 用于室外 / 远距。

---

## 6. 与其他深度相机对比

| 维度 | RealSense D435 | ZED 2 | Azure Kinect | OAK-D Pro |
| --- | --- | --- | --- | --- |
| **价格** | **$300（最便宜）** | $450 | $400 | $400 |
| **原理** | 主动红外立体 | 被动立体 | ToF | 主动红外立体 + VPU |
| **范围** | 0.3-3 m | 0.2-20 m | 0.25-5 m | 0.2-8 m |
| **精度** | ±2 mm | ±1 m 内 1 cm | ±5 mm | ±2 mm |
| **计算** | 主机 CPU | 主机 CPU | 主机 CPU | **板载 VPU（VPU 推理）** |
| **SDK** | **librealsense2（最佳）** | ZED SDK | Azure Kinect SDK | DepthAI |
| **ROS2** | ✅ 官方 | ✅ | ✅ | ✅ |
| **AI 能力** | ❌ | ❌ | 物体检测 | **VPU 上跑 YOLO** |
| **室外** | 中 | 强 | 中 | 强 |

> **D435 vs ZED 2**：D435 室内强、价格低、SDK 完整；ZED 2 远距、室外、CUDA 加速。
> **D435 vs OAK-D**：D435 通用强；OAK-D 板载 VPU 可跑神经网络。

---

## 7. 编程示例

### 7.1 Python 基础

```python
import pyrealsense2 as rs
import numpy as np

pipeline = rs.pipeline()
config = rs.config()
config.enable_stream(rs.stream.depth, 640, 480, rs.format.z16, 30)
config.enable_stream(rs.stream.color, 640, 480, rs.format.bgr8, 30)

pipeline.start(config)

try:
    while True:
        frames = pipeline.wait_for_frames()
        depth = frames.get_depth_frame()
        color = frames.get_color_frame()
        
        # 转 numpy
        depth_img = np.asanyarray(depth.get_data())  # 16-bit
        color_img = np.asanyarray(color.get_data())  # BGR
        
        # 中心点深度
        cx, cy = 320, 240
        dist = depth.get_distance(cx, cy)  # 米
        print(f"Center distance: {dist:.3f} m")
        
        # 保存点云
        pc = rs.pointcloud()
        points = pc.calculate(depth)
        # ...
finally:
    pipeline.stop()
```

### 7.2 ROS2 集成

```bash
# 安装
sudo apt install ros-${ROS_DISTRO}-realsense2-camera

# 启动
ros2 launch realsense2_camera rs_launch.py \
    depth_module.depth_profile:=640x480x30 \
    pointcloud.enable:=true \
    align_depth.enable:=true
```

### 7.3 Isaac Sim 集成

```
Isaac Sim → Replicator → Synthetic Data → D435 模型
   ↓
仿真 D435 输出
  - depth
  - RGB
  - semantic segmentation
  - normal
  - 3D 点云
```

---

## 8. 选型决策

| 场景 | 推荐型号 | 原因 |
| --- | --- | --- |
| **通用机器人研究** | **D435** | 价格 + 性能 + SDK 平衡 |
| **SLAM / 状态估计** | **D435i** | 加 IMU |
| **户外 / 远距** | D455 | 远距 6m / 室外 |
| **微装配** | D405 | 7 cm 近距 |
| **户外 / 工业** | D555 | IP65 防水 |
| **板载 AI** | OAK-D Pro | VPU 上跑神经网络 |
| **3D 扫描（人体）** | ZED 2 | 远距 + 高精度 |

---

## 9. 常见坑

| 坑 | 原因 | 解决 |
| --- | --- | --- |
| **USB 2.0 带宽不足** | D435 需要 2.5 Gbps | 用 USB 3.0 端口 |
| **玻璃反光失效** | 红外被反射 | 贴磨砂纸 / 改位置 |
| **强光深度退化** | 红外被淹没 | 室内用 / 加遮光罩 |
| **多机红外干扰** | 多 D435 同时投红外 | 时间分片 / 间距 |
| **深度对齐 RGB** | 物理位置不同 | `align_depth_to_color` |
| **点云下采样** | 深度 90 fps 太大 | voxelize 0.01 m |

---

## 10. 未来与替代

### 10.1 2024+ 趋势

- **D435 停产传闻**——Intel 2023 出售 RealSense 业务
- **新代 D405 / D455 / D555**——Intel 主推
- **替代品**：
  - **OAK-D Pro**（板载 VPU）
  - **ZED 2 / ZED X**（NVIDIA 生态）
  - **Photoneo**（工业级）
  - **Lucid Helios**（ToF）

### 10.2 仍推荐 D435 的原因

- **生态完整**（ROS2 / Isaac / Habitat 100% 兼容）
- **价格优势**（$300 vs 替代 $400+）
- **二手市场**（D435 大量库存）
- **文档丰富**（任何问题都有 Stack Overflow）

---

## 11. 相关笔记

- 同类：[digit-tactile.md](digit-tactile.md) · [lidar-ouster-os1.md](lidar-ouster-os1.md)
- 计算平台：[../compute/nvidia-jetson-orin.md](../compute/nvidia-jetson-orin.md)
- 仿真：[../../simulation/platforms/nvidia-isaac.md](../../simulation/platforms/nvidia-isaac.md) · [../../simulation/platforms/habitat.md](../../simulation/platforms/habitat.md)
- 概念：[../../concepts/mdp/observation-space.md](../../concepts/mdp/observation-space.md) · [../../concepts/perception/visual-representation.md](../../concepts/perception/visual-representation.md)
- 方法：[../../methods/vision-language-action.md](../../methods/vision-language-action.md)
- Demo：[05-Figure02-端到端人形](../demo/scenarios/05-Figure02-端到端人形/README.md) · [07-视觉语言导航-VLN-CE](../demo/scenarios/07-视觉语言导航-VLN-CE/README.md)

## 12. 参考

**官方来源**：

- RealSense D435 官方产品页（RealSense 已自 Intel 独立）（来源：https://realsenseai.com/products/stereo-depth-camera-d435/，访问于 2026-09-02）

- Intel RealSense 官方文档
- librealsense2 GitHub
- ROS2 realsense2_camera 包
- Isaac Sim Replicator
- Microsoft Azure Kinect（对比）
- Stereolabs ZED（对比）
