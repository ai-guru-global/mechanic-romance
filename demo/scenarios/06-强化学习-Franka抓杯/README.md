# 强化学习·Franka 抓杯 · RL Pick-and-Place with Isaac Lab

> **一句话描述**：在 [NVIDIA Isaac Lab](../../corpus/simulation/platforms/nvidia-isaac.md) GPU 并行仿真中，用 **PPO** 训练 [Franka Panda](../../corpus/hardware/arms/franka-panda.md) 学会把随机位姿的杯子抓起放到指定托盘——具身智能 RL 路线的「**Hello World**」。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `06` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M3 · 实验复现](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

仿真桌面环境中，Franka Panda 面对**随机位置**的马克杯，需把它抓起放到桌面上的**目标托盘**。与 [01-扩散策略](../01-扩散策略-桌面堆叠/README.md) 模仿学习路线对比——这次**没有演示数据**，纯靠**奖励信号**自己学出来。

- **输入指令示例**：任务无显式指令；目标托盘位姿由环境采样。
- **成功判据**：杯子在托盘上方 5 cm 内松手，且无掉落。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| 关节角 | 关节编码器 | 7-DoF |
| 末端位姿 | 前向运动学 | 6 维（x,y,z,r,p,y） |
| 杯子位姿 | 仿真器读取 | 6 维（绝对位姿） |
| 目标托盘位姿 | 仿真器读取 | 6 维 |
| 夹爪状态 | 夹爪编码器 | 1 维（开/合） |

> 概念见 [observation-space](../../corpus/concepts/mdp/observation-space.md)。**注意**：纯状态观测（无图像）——这是 RL baseline 与 VLA 的本质区别。

---

## 3. 动作空间（Action）

- **控制模式**：**关节速度控制**（更稳）或 **末端增量位姿**（更灵活）
- **动作维度**：7（每关节速度 / 末端 6-DoF + 1 夹爪）
- **控制频率**：60 Hz（仿真步长）
- **动作表示**：连续值，限幅 ±1

> 详见 [action-space](../../corpus/concepts/mdp/action-space.md)。

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| 成功率 | 100 个 episode 抓放完成占比 | ≥ 85%（RL baseline） |
| 平均奖励 | 单 episode 累计奖励 | 越大越好 |
| 样本效率 | 达到 80% 成功率所需环境步数 | ≤ 5M 步（Isaac Lab 4090） |
| Sim-to-Real Gap | 真机成功率 / 仿真成功率 | ≥ 0.7（域随机化后） |

- **评估 episodes 数**：100
- **变体**：
  - **新杯子形状**：训练外杯子（玻璃杯 / 塑料杯）
  - **新桌面颜色 / 光照**
  - **加域随机化**（摩擦 / 质量 / 视觉）

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机械臂 | [Franka Panda](../../corpus/hardware/arms/franka-panda.md)（7-DoF） |
| 末端执行器 | [Robotiq 2F-85](../../corpus/hardware/end-effectors/robotiq-2f-85.md) 平行夹爪 |
| 相机 | 仿真用（Isaac Lab 合成相机） |
| 计算 | **1× RTX 4090 / A100**（Isaac Lab 并行 1024+ 环境） |
| 仿真器 | [NVIDIA Isaac Lab](../../corpus/simulation/platforms/nvidia-isaac.md)（原 Orbit，2024 GA） |
| 真机部署 | sim-to-real 迁移到 Panda |

---

## 6. 方法（Method）

- 算法：**PPO**（Proximal Policy Optimization，on-policy Actor-Critic）
- 预训练：无
- 训练数据：仿真自采（无需演示）
- 关键参数：
  - 并行环境数：1024
  - 折扣因子 γ = 0.99
  - GAE λ = 0.95
  - 学习率 3e-4（线性衰减）
  - 总步数 50M
- 概念笔记：[reinforcement-learning](../../corpus/methods/reinforcement-learning.md) · [ppo](../../corpus/methods/ppo.md) · [sac](../../corpus/methods/sac.md) · [reward-function](../../corpus/concepts/mdp/reward-function.md) · [policy](../../corpus/concepts/mdp/policy.md)
- Sim-to-Real：[domain-randomization](../../corpus/simulation/sim-to-real/domain-randomization.md) · [sim-to-real](../../corpus/concepts/foundations/sim-to-real.md)

**奖励设计（关键）**：
```
r = -α·d_gripper_to_cup  # 末端到杯子的距离，越近越好
    - β·d_cup_to_goal    # 杯子到目标托盘的距离
    + γ·grasp_bonus      # 成功抓取 +1
    + δ·place_bonus      # 成功放置 +5
    - ε·collision_penalty # 碰撞桌面/自身 -0.5
```

**为什么是它**：
1. **RL 路线的「Hello World」**：所有 RL baseline（PPO/SAC）的标准对照
2. **Isaac Lab 官方 demo**（`IsaacLab/source/standalone/environments/franka_cabinet.py` 改写）
3. 对比 [01-扩散策略](../01-扩散策略-桌面堆叠/README.md) 模仿学习，看**有演示 vs 无演示**的差异

---

## 7. 运行方式（How to Run）

```bash
# 1. 安装 Isaac Lab（推荐 Docker 镜像）
docker pull nvcr.io/nvidia/isaac-lab:2.0.0
docker run --gpus all -it nvcr.io/nvidia/isaac-lab:2.0.0

# 2. 克隆并安装依赖
git clone https://github.com/NVlabs/IsaacLab.git
cd IsaacLab && ./isaaclab.sh -i rl  # 安装 RL deps

# 3. 写任务环境（基于 FrankaCabinet 模板）
# scripts/environments/franka_pick_cup.py

# 4. 启动 PPO 训练（1024 并行环境）
python scripts/reinforcement_learning/rl_games/train.py \
  --task Franka-PickCup \
  --num_envs 1024 \
  --headless \
  --max_iterations 5000

# 5. TensorBoard 监控
tensorboard --logdir logs/rl_games/franka_pick_cup/

# 6. 评估 + 可视化
python scripts/reinforcement_learning/rl_games/play.py \
  --task Franka-PickCup \
  --num_envs 64 \
  --checkpoint logs/.../checkpoint.pth
```

- 关键脚本：`scripts/environments/franka_pick_cup.py` + `train.py`
- 模型权重：CKPT 入库（小），大权重放 release

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-rl-franka-pick-cup/` 实验记录。

- 首次成功演示：待定
- 复现指标：80% 成功率对应 ~20M 仿真步（参考 Isaac Lab benchmark）

---

## 9. 相关语料（References）

- 方法：[reinforcement-learning](../../corpus/methods/reinforcement-learning.md) · [imitation-learning](../../corpus/methods/imitation-learning.md)
- 仿真：[nvidia-isaac](../../corpus/simulation/platforms/nvidia-isaac.md) · [domain-randomization](../../corpus/simulation/sim-to-real/domain-randomization.md)
- 硬件：[franka-panda](../../corpus/hardware/arms/franka-panda.md) · [robotiq-2f-85](../../corpus/hardware/end-effectors/robotiq-2f-85.md)
- 概念：[policy](../../corpus/concepts/mdp/policy.md) · [reward-function](../../corpus/concepts/mdp/reward-function.md) · [observation-space](../../corpus/concepts/mdp/observation-space.md) · [action-space](../../corpus/concepts/mdp/action-space.md) · [sim-to-real](../../corpus/concepts/foundations/sim-to-real.md)
- 评测：[rlbench](../../corpus/benchmarks/manipulation/rlbench.md)

---

## 10. 备注

- **为什么选作 `06`**：是 RL 路线在操作任务上的**入门 demo**，对应 M3 复现里程碑。
- **与模仿学习对比**：
  | 维度 | [01-扩散策略](../01-扩散策略-桌面堆叠/README.md)（IL） | 06-RL-Franka 抓杯 |
  | --- | --- | --- |
  | 数据 | 100+ 演示 | 0（自采） |
  | 训练时间 | 数小时 | 数小时（GPU 训练） |
  | 成功率 | 86% | 80-90% |
  | Sim-to-Real | 较易（演示里就有真机） | 较难（需域随机化） |
  | 上限 | 演示的策略 | 可超越人类 |
- **进阶方向**：
  - **Sim-to-Real**：加 [domain-randomization](../../corpus/simulation/sim-to-real/domain-randomization.md) 后部署到真 Panda
  - **离线 RL**：用 [Open X-Embodiment](../../corpus/datasets/open-x-embodiment.md) 演示训 IQL / CQL
  - **多任务**：换成 [Behavior-1k](../../corpus/benchmarks/manipulation/behavior-1k.md) 1000 任务 RL 训练
- **已知坑**：
  - Isaac Lab 2.0 与 Isaac Gym API 不兼容，老代码需重写
  - 奖励设计是核心难点，参考 [RLBench](../../corpus/benchmarks/manipulation/rlbench.md) 提供的 100 任务奖励模板
- **对应里程碑**：M3 · 实验复现
