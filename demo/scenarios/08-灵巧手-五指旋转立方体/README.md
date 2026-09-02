# 灵巧手·五指旋转立方体 · Dexterous Hand In-Hand Cube Reorientation

> **一句话描述**：用 [Isaac Lab](../../../corpus/simulation/platforms/nvidia-isaac.md) 训练 **5 指灵巧手（Allegro / LEAP Hand）** 在掌内原地旋转一个立方体，使指定目标面朝上——具身智能灵巧操作的天花板级 demo。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `08` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M3 · 实验复现](../../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

5 指灵巧手在**无桌面、空中**（in-hand manipulation）旋转一个**立方体**（3 cm 边长），使**目标面**（带特定颜色 / 标记）朝上。**关键难点**：
- **高维动作空间**（16-24 DoF）
- **无外部约束**（所有接触只在手指-立方体之间）
- **稀疏奖励**（必须稳定到目标位姿才给奖励）

- **输入指令示例**：目标面颜色（红 / 蓝 / 绿 / 黄 / 黑 / 白 6 选 1）
- **成功判据**：立方体目标面法向量与世界 +Z 的夹角 < 15°，停留 ≥ 1 s。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| 关节角 | 灵巧手编码器 | 16-DoF（Allegro）/ 16-DoF（LEAP） |
| 立方体位姿 | 仿真器读取 | 7 维（位置 + 四元数） |
| 立方体角速度 | 仿真器读取 | 3 维 |
| 触觉 | 仿真指尖传感器 | 16×3 维（每指 3D 力） |
| 目标面 | 文本 / one-hot | 6 维 |
| 可选：腕部相机 | 手腕外相机 | 84×84×3（增加视觉观测） |

> 概念见 [observation-space](../../../corpus/concepts/mdp/observation-space.md)。**注意**：通常**不用图像**——直接读仿真器位姿（特权信息），避免视觉噪声干扰。

---

## 3. 动作空间（Action）

- **控制模式**：**关节角增量**（每步 ±0.05 rad）
- **动作维度**：**16-DoF**（Allegro）/ **16-DoF**（LEAP Hand） / **20-DoF**（Shadow Hand）
- **控制频率**：120 Hz（仿真）
- **动作表示**：每关节相对增量

> 详见 [action-space](../../../corpus/concepts/mdp/action-space.md)。这是 16+ 维连续动作空间，PPO 等 on-policy 算法需**大批量+长时间**训练。

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| 成功率 | 目标面朝上 ≥ 1 s 占比 | ≥ 80%（OpenAI Rubik's Cube 80% 区间） |
| 平均步数 | 完成旋转所需步数 | ≤ 200 步 |
| 掉落率 | 立方体掉出掌心次数 | 0 |
| 样本效率 | 达到 70% 成功率所需仿真步数 | ≤ 50M 步（4× A100） |
| 真实部署 | 真机成功率 | ≥ 50%（Sim-to-Real 难） |

- **评估 episodes 数**：100 × 6 目标面 = 600
- **变体**：
  - **新形状**：训练外物体（球、圆柱）
  - **新质量 / 摩擦**
  - **增加视觉观测**（无特权信息）

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 灵巧手 | **Allegro Hand**（16-DoF，Wonik Robotics，$15k）/ **LEAP Hand**（开源 16-DoF，Stanford，$3k）/ **Shadow Hand**（20-DoF，$100k+） |
| 视觉 | 可选腕部 RGBD |
| 触觉 | 仿真合成（Isaac Lab 内置）/ 真机需 DIGIT 等触觉传感器 |
| 计算 | **4× A100**（PPO 训练）/ 1× RTX 4090（推理可） |
| 仿真器 | [NVIDIA Isaac Lab](../../../corpus/simulation/platforms/nvidia-isaac.md)（PhysX 5 GPU 并行） |
| 真机 | Allegro Hand + 桌面臂挂载 |

> 灵巧手规格对比见 [allegro](../../../corpus/hardware/hands/allegro.md) / [leap-hand](../../../corpus/hardware/hands/leap-hand.md) / [shadow-hand](../../../corpus/hardware/hands/shadow-hand.md)。

---

## 6. 方法（Method）

- 算法：**PPO**（标准 on-policy RL，对高维动作友好）
- 预训练：无
- 训练数据：仿真自采（域随机化立方体位置、目标面、摩擦）
- 关键技术：
  - **域随机化**（DR）：见 [domain-randomization](../../../corpus/simulation/sim-to-real/domain-randomization.md)
  - **课程学习**：从固定目标 → 随机目标
  - **特权信息**（privileged info）：训练时给真位姿，推理时只给触觉+本体
  - **Sim-to-Real**：见 [sim-to-real](../../../corpus/concepts/foundations/sim-to-real.md)
- 代表论文：
  - **OpenAI Rubik's Cube (2019)**：单手解魔方，开创灵巧手 in-hand manipulation
  - **DexCap (2024)**：灵巧手数据采集（人手示教 → 灵巧手执行）
  - **Dexterous Diffusion Policy (DDEX, 2024)**：把 [Diffusion Policy](../../../corpus/methods/diffusion-policy.md) 扩展到灵巧手

**为什么是它**：
1. **灵巧手 in-hand manipulation** 是具身智能**最难、最炫酷**的任务之一
2. **可与 01-04 串联**：Diffusion Policy → DDEX 是当前最热方向
3. 演讲 / 客户演示的**视觉爆点**

---

## 7. 运行方式（How to Run）

```bash
# 1. 安装 Isaac Lab（同 06）
docker pull nvcr.io/nvidia/isaac-lab:2.0.0
docker run --gpus all -it nvcr.io/nvidia/isaac-lab:2.0.0

# 2. 下载 Allegro Hand 资产（Isaac Lab 内置）
# 或 LEAP Hand URDF：https://leaphand.stanford.edu/

# 3. 编写任务（参考官方 AllegroHand 任务）
# scripts/environments/allegro_cube_reorient.py

# 4. 启动 PPO 训练（4× A100，约 24-48 h）
python scripts/reinforcement_learning/rl_games/train.py \
  --task Allegro-CubeReorient \
  --num_envs 4096 \
  --headless \
  --max_iterations 20000

# 5. 可视化评估
python scripts/reinforcement_learning/rl_games/play.py \
  --task Allegro-CubeReorient \
  --num_envs 64 \
  --checkpoint logs/allegro_cube_ckpt.pth

# 6. 真机部署（Sim-to-Real 需大量调参）
python deploy_allegro.py --checkpoint logs/allegro_cube_ckpt.pth
```

- 关键脚本：`scripts/environments/allegro_cube_reorient.py`
- 模型权重：CKPT 入库；Allegro 资产需从 Wonik 申请

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../../research/experiments/) 下建 `2026-XX-XX-allegro-cube-reorient/` 实验记录。

- 首次成功演示：待定
- 复现指标：Allegro 立方体旋转 SR 80%+（参考 OpenAI / Stanford 报告）

---

## 9. 相关语料（References）

- 仿真：[nvidia-isaac](../../../corpus/simulation/platforms/nvidia-isaac.md) · [domain-randomization](../../../corpus/simulation/sim-to-real/domain-randomization.md)
- 方法对照：[reinforcement-learning](../../../corpus/methods/reinforcement-learning.md) · [diffusion-policy](../../../corpus/methods/diffusion-policy.md)（DDEX 灵巧版）· [imitation-learning](../../../corpus/methods/imitation-learning.md)
- 概念：[sim-to-real](../../../corpus/concepts/foundations/sim-to-real.md) · [observation-space](../../../corpus/concepts/mdp/observation-space.md) · [action-space](../../../corpus/concepts/mdp/action-space.md) · [policy](../../../corpus/concepts/mdp/policy.md) · [reward-function](../../../corpus/concepts/mdp/reward-function.md)
- 硬件：[allegro](../../../corpus/hardware/hands/allegro.md) · [leap-hand](../../../corpus/hardware/hands/leap-hand.md) · [shadow-hand](../../../corpus/hardware/hands/shadow-hand.md)



---

## 10. 备注

- **为什么选作 `08`**：
  1. 具身智能**天花板级 demo**（OpenAI 2019 单手解魔方震惊业界）
  2. 5 指灵巧手 = 完整人手能力的最小子集
  3. 与 01-05 形成完整 **「平面 → 双手 → 全身 → 灵巧」** 梯度
- **梯度关系**：
  | 场景 | 动作维度 | 难度 | 视觉冲击力 |
  | --- | --- | --- | --- |
  | [01-扩散策略](../01-扩散策略-桌面堆叠/README.md) | 7-DoF | 🟢 | ⭐⭐ |
  | [02-ACT-双手](../02-ACT-双手叠衣/README.md) | 16-DoF | 🟡 | ⭐⭐⭐ |
  | [05-Figure02](../05-Figure02-端到端人形/README.md) | 35-DoF | 🔴 | ⭐⭐⭐⭐ |
  | **08-灵巧手** | **16-DoF（手）+ 7（臂）** | **🔴** | **⭐⭐⭐⭐⭐** |
- **与 01 关联**：
  - 01 用 **Diffusion Policy** 做平面操作
  - 08 是 **Dexterous Diffusion Policy (DDEX)** 的前身
  - 数据流：人手示教 → 灵巧手重定向（Hand Retargeting）
- **学习路径**：
  1. 看 OpenAI Rubik's Cube 视频
  2. 读 DDEX / DexCap 论文
  3. 在 Isaac Lab 里跑 Allegro 立方体任务
  4. 加域随机化做 Sim-to-Real
- **已知坑**：
  - 灵巧手 sim-to-real **极难**（触觉+摩擦+接触建模对不齐）
  - Allegro 真机约 $15k，加上臂 ≈ $50k
  - 训练 4× A100 × 24h 是基础门槛
- **对应里程碑**：M3 · 实验复现
