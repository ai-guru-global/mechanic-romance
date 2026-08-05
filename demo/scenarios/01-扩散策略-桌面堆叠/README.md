# 扩散策略·桌面堆叠 · Diffusion Policy Push-T

> **一句话描述**：用扩散模型学习一段「桌面推方块」演示轨迹，复现 Chi et al. 2023 提出的 Diffusion Policy 经典 Demo（Push-T）。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `01` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M2 · 首个可演示场景](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

机械臂在 20×20 cm 桌面区域内，根据**单张第三人称 RGB** 观测，把一个红色 T 形推块推到指定绿色目标位姿。任务关键在于**闭环多模态动作分布**——同一图像可能对应多种合理轨迹，扩散模型能更好刻画这种多模性。

- **输入指令示例**：目标点坐标可来自「图像中绿色标记」或「文本指令把 T 块推到这里」。
- **成功判据**：T 形推块与目标位姿重合度 IoU ≥ 0.9，停留 ≥ 1 s。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| RGB 图像 | 第三人称顶视相机 | 84×84×3，10 Hz |
| 本体感知 | 关节编码器 | 7-DoF 关节角 + 末端 6-DoF 位姿 |
| 目标 | 图像中绿色 mask / 文本条件 | 可选，扩到条件生成时使用 |

> 详细概念见 [observation-space](../../corpus/concepts/mdp/observation-space.md)。

---

## 3. 动作空间（Action）

- **控制模式**：末端位姿（6-DoF） + 夹爪开合（1 维）
- **动作维度**：7（x, y, z, roll, pitch, yaw, gripper）
- **控制频率**：10 Hz（与观测对齐）
- **动作表示**：**相对增量**（增量式位姿预测），避免绝对位姿漂移

> 对比「动作分块（action chunking）」在 [imitation-learning](../../corpus/methods/imitation-learning.md) 中的讨论。

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| 成功率 | 50 次试验中完成任务的占比 | ≥ 80%（论文 86%） |
| 平均完成步数 | 从推到位的总步数 | ≤ 25 步 |
| 路径效率 | 实际轨迹长度 / 最短轨迹 | ≥ 0.7 |
| 安全性 | 关节越限 / 异常力矩 | 0 |

- **评估 episodes 数**：50 次（与论文保持一致）
- **变体**：测在**新视觉外观**（不同桌面颜色 / 背景）下的零样本泛化。

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机械臂 | [Franka Panda](../../corpus/hardware/arms/franka-panda.md)（7-DoF） |
| 末端执行器 | [Robotiq 2F-85](../../corpus/hardware/end-effectors/robotiq-2f-85.md) 平行夹爪 |
| 相机 | Intel RealSense D435 顶视安装 |
| 计算 | 1× RTX 4090（训练）/ 推理可在 3060 上跑 |
| 仿真器 | [MuJoCo](../../corpus/simulation/platforms/mujoco.md)（官方推荐）或 [Genesis](../../corpus/simulation/platforms/genesis.md) |
| 真实平台 | Push-T 真实硬件套件（Stanford 官方） |

---

## 6. 方法（Method）

- 策略：[Diffusion Policy](../../corpus/methods/diffusion-policy.md)（DDPM 风格条件扩散）
- 预训练：无（从 0 训练）
- 训练数据：约 100 条专家演示（论文 206 条；本场景可缩到 100 条做最小复现）
- 论文笔记：[2023-chi-diffusion-policy](../../corpus/papers/diffusion-policy/2023-chi-diffusion-policy.md)

**为什么是它**：模仿学习场景下的「**多模态动作分布**」典型问题，扩散策略是当前最强 baseline，门槛低、效果好、易讲解。

---

## 7. 运行方式（How to Run）

```bash
# 1. 安装依赖（仿真）
pip install mujoco diffusers torch torchvision

# 2. 下载预训练权重（可选）
# 论文官方权重：https://diffusion-policy.cs.columbia.edu/

# 3. 仿真中跑预训练策略
python eval.py --task push-t --checkpoint checkpoints/dp_pusht.ckpt

# 4. 训练自己采集的演示
python collect_demos.py --robot franka --out data/pusht_demos.hdf5
python train.py --task push-t --data data/pusht_demos.hdf5 --epochs 500
```

- 关键脚本：项目内 `scripts/{collect_demos,train,eval}.py`
- 模型权重：外部存储（HF Hub / 项目 release），不入库

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-diffusion-policy-pusht/` 实验记录。

- 首次成功演示：待定
- 复现指标：与论文 86% / 95% 区间对比

---

## 9. 相关语料（References）

- 论文：[2023-chi-diffusion-policy](../../corpus/papers/diffusion-policy/2023-chi-diffusion-policy.md)
- 方法：[diffusion-policy](../../corpus/methods/diffusion-policy.md) · [imitation-learning](../../corpus/methods/imitation-learning.md) · [behavioral-cloning](../../corpus/methods/behavioral-cloning.md)
- 硬件：[franka-panda](../../corpus/hardware/arms/franka-panda.md) · [robotiq-2f-85](../../corpus/hardware/end-effectors/robotiq-2f-85.md)
- 仿真：[mujoco](../../corpus/simulation/platforms/mujoco.md)
- 概念：[embodied-ai](../../corpus/concepts/foundations/embodied-ai.md) · [observation-space](../../corpus/concepts/mdp/observation-space.md) · [action-space](../../corpus/concepts/mdp/action-space.md)

---

## 10. 备注

- **为什么选作 `01`**：是模仿学习最经典的入门 demo，仿真+真机都有完整开源，对应 M2 里程碑。
- **可扩展方向**：换成 Diffusion Policy 在 [LIBERO](../../corpus/benchmarks/manipulation/libero.md) 上的多任务版本（30+ 任务）。
- **已知坑**：Diffusion Policy 推理慢（10-15 Hz），实时部署需用 DDIM + 动作分块。
