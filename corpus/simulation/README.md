# 仿真 · Simulation

> 具身智能的**廉价试错场**。强化学习需要百万步交互，真机不可能；仿真让数据近乎免费。但仿真若与真实物理对不上（Sim-to-Real Gap），策略迁不过去。

---

## 子分类

| 子目录 | 收录 | 状态 |
| --- | --- | --- |
| [`platforms/`](platforms/) | 仿真平台（Isaac Sim / Isaac Lab / MuJoCo / PyBullet / Genesis / Gazebo） | 🟡 充实中 |
| [`assets/`](assets/) | 仿真资产（URDF / MJCF / USD 资产库、物体网格） | ⚪ 待建 |
| [`sim-to-real/`](sim-to-real/) | Sim-to-Real 方法（域随机化 / 系统辨识 / 域适应） | ⚪ 待建 |
| [`data-gen/`](data-gen/) | 仿真数据生成（合成演示、自动标注、无限数据） | ⚪ 待建 |

## 主流仿真平台对比（概览）

| 平台 | 物理引擎 | 渲染 | 并行 | 适用 | 主人 |
| --- | --- | --- | --- | --- | --- |
| **Isaac Sim / Lab** | PhysX | RTX 光追 | GPU 数千并行 | 研究主流、RL 大规模 | NVIDIA |
| **MuJoCo** | MuJoCo | 简易 | CPU | 精确接触、算法原型 | Google DeepMind |
| **PyBullet** | Bullet | 简易 | CPU | 入门、轻量 | 开源 |
| **Genesis** | 多引擎 | 光追 | GPU 超大规模 | 2024 新秀、生成式 | CMU/麻省 |
| **Gazebo** | ODE/Bullet | 中 | 弱 | 传统机器人、SLAM | OSRF |

> 详细对比见 [`platforms/`](platforms/)。

## 为什么仿真不可或缺

1. **成本**：真机采集 1 条演示 ≈ 数十分钟人工；仿真可并行生成百万条。
2. **安全**：强化学习要探索，破坏性动作用真机会损坏设备。
3. **可控**：可任意指定物体位姿、光照、物理参数，便于消融。
4. **可标注**：仿真有完美 ground truth（深度、分割、位姿）。

## 核心难题：Sim-to-Real Gap

仿真的物理参数（摩擦系数、质量分布、齿轮间隙）与真机不完全一致 → 策略过拟合仿真 → 真机失效。解决路径见 [`sim-to-real/`](sim-to-real/)。

## 与其他模块的关系

- 为 [`methods/reinforcement-learning.md`](../methods/reinforcement-learning.md) 提供训练环境。
- 建模 [`hardware/`](../hardware/) 中的本体。
- 生成 [`benchmarks/`](../benchmarks/) 的评测场景。
