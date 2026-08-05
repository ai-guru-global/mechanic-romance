# 视觉语言导航·VLN-CE · Vision-and-Language Navigation in Continuous Environments

> **一句话描述**：机器人在 3D 室内仿真环境（[Habitat](../../corpus/simulation/platforms/mujoco.md) Matterport3D）中，按自然语言指令（如"走到厨房的冰箱前"）**连续空间**导航到目标位置——视觉语言导航的 SOTA 基准。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `07` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M1 · 语料沉淀](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

机器人在 **Matterport3D 室内场景** 中，按 **自然语言指令**（英文 3-5 步指令）**自主导航**到目标位置。**连续环境**（VLN-CE）相比离散版（VLN）更接近真实：动作不再是「跳到下一个节点」，而是**真实连续控制**（前进 / 转向 / 停止）。

- **输入指令示例**：
  - 「Walk past the piano and turn left at the stairs. Stop in front of the painting on the wall.」
  - 「走出卧室，左转进入走廊，在浴室镜子前停下。」
- **成功判据**：在目标位置 3 m 范围内停下，且**面向正确方向**。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| RGB 图像 | 第一人称相机 | 256×256×3，10 Hz |
| 深度 | 深度相机 | 256×256，10 Hz |
| 文本指令 | 文字输入 | 经 CLIP / BERT 编码 |
| 本体感知 | 底盘里程计 | 位置 (x,y,z) + 朝向 (yaw) + 速度 |

> 概念见 [observation-space](../../corpus/concepts/mdp/observation-space.md)。**关键差异**：导航任务**不关心操作**（没有夹爪），但极度依赖**长程记忆**和**语言-视觉对齐**。

---

## 3. 动作空间（Action）

- **控制模式**：**连续底盘控制**（区别于离散 VLN）
- **动作维度**：2（线速度 forward ∈ [0, 0.5] m/s，角速度 turn ∈ [-1, 1] rad/s）+ 1（停止 STOP）
- **控制频率**：10 Hz
- **动作表示**：连续值 + 离散 STOP

> 详见 [action-space](../../corpus/concepts/mdp/action-space.md)。

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| **SR**（Success Rate） | 到达目标 3 m 内 + 正确朝向的占比 | ≥ 40%（HAMT SOTA 55%） |
| **SPL**（SR weighted by Path Length） | 成功率 ×（最优路径/实际路径） | ≥ 30% |
| **NE**（Navigation Error） | 终点与目标距离（m） | ≤ 5 m |
| **SR + Oracle Success** | 路径上是否经过目标邻域 | ≥ 60% |
| **oracle SPL** | 假设到达后立即停止的最优 SPL | ≥ 50% |

- **评估 episodes 数**：VLN-CE test 1400+（R2R-CE 标准 split）
- **变体**：
  - **R2R-CE**（Room-to-Room）
  - **REVERIE-CE**（对象定位 + 指令）
  - **RxR-CE**（多语言指令）
  - **HM3D / MP3D**（不同室内数据集）

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机器人 | 仿真 LoCoBot（底盘 + RGBD） / 真机 LoCoBot / Stretch |
| 相机 | 第一人称 RGB + 深度（RealSense D435） |
| 计算 | 1× RTX 3090（仿真 + 训练）/ Jetson AGX（真机部署） |
| 仿真器 | **Habitat-Sim**（基于 [BulletPhysics](../../corpus/simulation/platforms/pybullet.md) + 渲染） |
| 数据集 | **R2R-CE**（Matterport3D 室内 + 人类指令） |
| 真机 | LoCoBot（iRobot Create + Kobuki 底盘）/ Hello Robot Stretch |

> 仿真器见 [habitat](../../corpus/simulation/platforms/habitat.md)。

---

## 6. 方法（Method）

- 策略：**HAMT（History Aware Multimodal Transformer）** — 2022 SOTA，用 Transformer 编码「视觉 + 文本 + 历史轨迹」
- 预训练：CLIP / ViT-B/16
- 训练数据：R2R 训练集 ~14k episode
- 关键组件：
  - **视觉编码**：ViT
  - **文本编码**：BERT
  - **历史编码**：Past Action Transformer（PAT）
  - **动作预测**：双线性注意力 → 2-DoF 连续动作
- 算法家族：Seq2Seq → CMA（Cross-Modal Attention）→ **HAMT → DUET（2023 SOTA）**

**为什么是它**：
1. **导航是具身智能的另一半**（不只操作）
2. **HAMT 是 VLN-CE 公认 SOTA**，有开源实现
3. **仿真训练 → 真机部署** 链路清晰（Habitat-Sim → LoCoBot）
4. **视觉冲击力强**（机器人看图说话、听话走路）

---

## 7. 运行方式（How to Run）

```bash
# 1. 安装 Habitat-Sim + Habitat-Lab
conda install -c aihabitat -c conda-forge habitat-sim habitat-lab

# 2. 下载 R2R-CE 数据 + MP3D 场景
python -m habitat_sim.utils.datasets_download --uids habitat-api-datasets

# 3. 克隆 HAMT 官方实现
git clone https://github.com/cshizhe/VLN-HAMT.git
cd VLN-HAMT

# 4. 训练 HAMT（在 8× V100 / 4× A100 上约 1-2 天）
python train.py --config configs/r2r_ce/hamt.yaml

# 5. 评估（VLN-CE test set）
python eval.py --split test --checkpoint ckpt/hamt_r2r_ce.pth

# 6. 真机部署（LoCoBot / Stretch）
# 参考 RxR-Habitat 部署指南
```

- 关键脚本：`VLN-HAMT/{train,eval}.py`
- 模型权重：HF Hub `cshizhe/hamt_r2r_ce`
- 数据集：Matterport3D 申请（学术免费）

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-vln-ce-hamt/` 实验记录。

- 首次成功演示：待定
- 复现指标：HAMT R2R-CE SR 55% / SPL 51%（参考论文）

---

## 9. 相关语料（References）

- 概念：[embodied-ai](../../corpus/concepts/foundations/embodied-ai.md) · [embodiment](../../corpus/concepts/foundations/embodiment.md) · [observation-space](../../corpus/concepts/mdp/observation-space.md) · [action-space](../../corpus/concepts/mdp/action-space.md)
- 仿真：[habitat](../../corpus/simulation/platforms/habitat.md)（主）· [mujoco](../../corpus/simulation/platforms/mujoco.md)（参考）· [pybullet](../../corpus/simulation/platforms/pybullet.md)（参考）
- 方法对照：[vision-language-action](../../corpus/methods/vision-language-action.md)（VLA 是操作路线，VLN 是导航路线的姊妹）· [vision-language-navigation](../../corpus/methods/vision-language-navigation.md)（VLN 方法族）
- 评测：[real-world-eval](../../corpus/benchmarks/vla-eval/real-world-eval.md)（可参考真机评估流程）



---

## 10. 备注

- **为什么选作 `07`**：
  1. 具身智能的**另一半**（操作 + 导航 = 完整物理交互）
  2. **视觉语言导航** 是 VLA 范式在导航任务上的延伸
  3. 仿真训练成本低（不需要真机器人）
- **与方法论关联**：
  - **VLN-CE** = 「语言指令 → 导航动作」—— 和 [03-RT2](../03-RT2-自然语言指令分拣/README.md) 同源（语言 → 动作）
  - **HAMT** = Transformer + 历史注意力 —— 架构上和 [04-OpenVLA-7B](../04-OpenVLA-7B-开源VLA/README.md) 同根
  - 区别只在 **action space**（操作 7-DoF / 导航 2-DoF + STOP）
- **学习路径**：
  1. 先精读 R2R 论文 + HAMT 论文
  2. 在 Habitat 仿真里跑通 SOTA
  3. 部署到 LoCoBot / Stretch 真机
- **可扩展方向**：
  - **零样本导航**：用 CLIP 替代训练（OpenVLA 同源思路）
  - **多模态指令**：图像 + 文本混合指令
  - **移动操作**：导航 + 操作（Mobile ALOHA 类型）
- **已知坑**：
  - Matterport3D 学术申请流程约 1 周
  - HAMT 训练需 8× V100 起，单卡需用 gradient checkpointing
  - Sim-to-Real gap 主要在**噪声深度**和**滑移底盘**
- **对应里程碑**：M1 · 语料沉淀
