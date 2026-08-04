# RLBench 基准（Robot Learning Benchmark）

> **一句话定位**：基于 **CoppeliaSim** 的**经典视觉操作基准**——100 个手工精心设计的桌面任务、多视角 RGB-D、语言条件，是 3D 表征方法（PerAct / Act3D / GNFactor）的标准舞台。虽发布于 2020 年偏「老」，但因任务设计精良，至今仍被广泛沿用。James et al., RA-L 2020。

> 最后更新：2026-08

---

## 1. 概览与定位

RLBench 由 Imperial College London 的 Stephen James 等人发布于 IEEE RA-L 2020，是**视觉引导机器人操作（vision-guided manipulation）**领域的早期重量级基准。它的核心贡献是：在一个统一的、基于 CoppeliaSim（原 V-REP）的仿真框架里，提供**大量手工设计、视觉丰富、支持多视角 RGB-D**的操作任务，并配以标准化的演示采集与评测接口。

在 CALVIN（长程）和 LIBERO（泛化）出现之前，RLBench 是操作领域**任务设计最精细**的基准。即便今天，它在**3D 表征方法**圈子里仍是首选——因为它的多视角 RGB-D 设定天然适合体素（voxel）方法。

| 属性 | 数值 / 说明 |
| --- | --- |
| 发布年份 | 2020（RA-L） |
| 仿真平台 | **CoppeliaSim**（原 V-REP，物理为 Bullet/ODE） |
| 本体 | Franka Panda 7-DoF + 平行夹爪 |
| 任务总数 | **100** 个手工设计任务 |
| 常用子集 | **PerAct-18**（18 任务 / 249 变体，事实标准） |
| 观测 | **多视角 RGB-D**（前/左/右/腕等相机）+ 关节状态 |
| 语言条件 | ✅ 每任务配语言目标模板 |
| 动作 | 关键帧法（waypoint）→ 6-DoF next-best-pose |
| 演示采集 | 脚本化路径规划器，提供 100 条/任务 |
| 评测指标 | 任务成功率（每任务 25 rollout） |

---

## 2. 任务集：100 个手工任务

### 2.1 为什么「手工设计」是 RLBench 的招牌

与 LIBERO 的 PDDL 程序化生成不同，RLBench 的 100 个任务是**研究团队逐一手工设计、调试、配演示采集脚本**的。每个任务都有：

- **独特的物体几何**（钥匙、插头、杯子、碗、块、把手……）。
- **独特的动作结构**（插入、旋转、按压、倾倒、堆叠……）。
- **手工写的成功判定**（基于物体位姿/接触）。
- **手工写的演示采集脚本**（用 CoppeliaSim 的运动规划器）。

这意味着任务**质量高、语义清晰、难度梯度合理**，但也意味着**扩展成本极高**——加一个新任务要数天工程。这是 RLBench 任务数停在 100（而非像 LIBERO 那样程序化生成上千）的根本原因。

### 2.2 任务分类（示例）

| 类别 | 代表任务 | 动作类型 |
| --- | --- | --- |
| **抓放**（pick-and-place） | pick up cup, put in bowl | 抓取 + 移动 + 释放 |
| **插入/装配**（insertion） | put plug in socket, insert peg | 精细对位 + 插入 |
| **旋转/操作**（rotation） | turn tap, open drawer | 旋转/平移把手 |
| **按压**（press） | press button, push block | 定向推压 |
| **堆叠**（stacking） | stack blocks, place on shelf | 多步放置 |
| **倾倒**（pouring） | pour from jug | 容器操作 |

这些任务覆盖了**精细操作**的核心难点——尤其是插入、旋转这类**需要精确 6-DoF 位姿**的任务，是 CALVIN/LIBERO 的简单开/关动作所没有的。

### 2.3 PerAct-18：事实标准子集

由于 100 个任务全跑成本极高，**PerAct（Shridhar et al., CoRL 2022）** 选取了其中 **18 个任务（共 249 个变体）** 作为标准评测子集，此后成为 3D 方法的「主榜」：

- PerAct-18 包含：close_jar, drag_box, empty_dish, light_bulb_in, meat_off_grill, open_drawer, place_cups, place_wine, push_buttons, put_groceries, put_item_in_drawer, put_money_in_safe, reach_and_drag, slide_block_to_target, sort_shapes, stack_blocks, stack_cups, turn_tap 等。
- 每个任务有多个**变体**（不同物体位姿/颜色），共 249 个评测点。
- 评测：每任务 100 条演示训练，25 rollout 测成功率。

> 注意：社区偶有「RLBench-19」的说法，多指某个工作的具体子集选取（18 或 19 因含/不含某任务而异），但 PerAct 官方标准是 **18 任务**。完整 RLBench 仍是 100 任务。

---

## 3. 多视角 RGB-D 与 3D 方法

### 3.1 为什么 RLBench 是 3D 方法的舞台

RLBench 提供**多台 RGB-D 相机**（典型：前、左、右、腕部，或前+腕+顶），可重建**完整 3D 场景**。这天然适合：

- **体素方法**（PerAct）：把多视角深度融合成体素网格，用 3D Perceiver 预测 next-best-pose。
- **点云方法**（DP3、EquiBot）：从多视角深度反投影成点云，用 PointNet/Transformer 编码。
- **3D 关键点方法**（Act3D、Heat-3D）：在 3D 空间预测关键点/热力图。

相比之下，CALVIN（PyBullet，单/双视角低清）和 LIBERO（MuJoCo，主要是 2-3 视角 RGB）在 3D 表征的**信息量**上都不如 RLBench。

### 3.2 动作表示：next-best-pose

RLBench 的标准动作接口不是高频关节增量，而是**关键帧式的 6-DoF 末端位姿（next-best-pose）**：

```
策略输出 → 下一个目标位姿 (position 3D + rotation 6D, 夹爪开/关)
          → CoppeliaSim 的运动规划器生成轨迹并执行
          → 到达后策略再输出下一个目标位姿
```

这是一种**粗粒度、规划式**的动作空间，适合**长视野的抓放/插入**，但不适合高频力控。这也是 RLBench 与 Diffusion Policy（高频连续动作）路线不太兼容的原因。

---

## 4. 评估指标与协议

### 4.1 指标

| 指标 | 定义 |
| --- | --- |
| **Per-task success rate** | 单任务 25 rollout 成功比例 |
| **Multi-task average** | 18（或 100）任务的平均成功率（主榜） |
| **Per-variation** | 部分工作报每变体成功率 |

成功判定基于**物体最终状态**（位姿/接触），由每个任务的手写脚本判定。

### 4.2 评测流程（PerAct 协议）

1. 每任务用 **100 条专家演示**训练。
2. 在该任务的 **25 个评测 rollout**（不同初始状态）上测成功率。
3. 跨 18 任务平均，得 multi-task 平均成功率。
4. 可选：报 single-task（每任务单独训）vs multi-task（一个模型学所有任务）的对比。

> 25 rollout 的统计量比 LIBERO（20）和 CALVIN（504/任务）都小，方差相对大，这也是 RLBench 评测常被诟病的一点。

---

## 5. 基线分数参考

PerAct-18 子集上的代表性分数（multi-task 平均成功率，%）：

| 方法 | 类型 | 年份 | 平均成功率 | 备注 |
| --- | --- | --- | --- | --- |
| **PerAct** | 体素 + Perceiver | 2022 | ~43 | 3D 方法里程碑 |
| **GNFactor** | 体素 + NeRF 特征 | 2023 | ~50 | 语义 3D |
| **Act3D** | 3D 关键点 | 2022 | **~65** | 比 PerAct +22% |
| **EquiBot / EquAct** | SE(3) 等变 | 2024 | ~55–60 | 等变提升泛化 |
| **MimicGen** | 自动演示生成 | 2023 | 提升 demo 数 → 上限抬升 | 数据增强 |
| **RVT / RVT-2** | 多视角 Transformer | 2023/24 | ~55–65 | 视角融合 |
| **HDP / 3D Diffusion** | 3D 扩散 | 2024 | ~60+ | 扩散 + 3D |

> 关键观察：
> - **Act3D（65%）是 PerAct（43%）之后的标志性跃升**，说明「关键点 + 3D」比「体素分类」更高效。
> - RLBench-18 上的 SOTA 已被推到 65%+，但**仍有 35% 失败**——精细操作（插头、钥匙）仍是硬骨头。
> - 精确数字请查各论文与 PerAct 官方 repo。

---

## 6. 设计理念与价值

### 6.1 为什么 RLBench 至今仍有价值

1. **任务设计最精细**。100 个手工任务，每个都有独特几何与动作结构，是 CALVIN/LIBERO 的程序化任务无法替代的——尤其**插入、旋转、按压**这类需精确 6-DoF 的任务。
2. **3D 表征的黄金测试场**。多视角 RGB-D + 精细几何，让体素/点云方法能充分发挥；这是 PerAct/Act3D/GNFactor 选择它的原因。
3. **被 3D 方法生态深度绑定**。PerAct 的 18 任务子集已成为 3D 操作的「ImageNet」——所有 3D 方法都在它上报分，可比性强。
4. **语言条件原生支持**。每任务配语言模板，支持语言条件策略评测。
5. **演示质量高**。脚本规划器产出的演示干净、可复现，适合 BC。

### 6.2 它推动的技术方向

- **3D 表征**：体素（PerAct）、点云（DP3）、关键点（Act3D）、NeRF 特征（GNFactor）。
- **等变性**（equivariance）：SE(3) 等变网络在 RLBench 上证明能大幅提升少样本泛化。
- **自动演示生成**（MimicGen）：用少量人工演示自动生成跨变体的大量演示。
- **多视角融合**（RVT）：用 Transformer 融合多视角特征。

---

## 7. 局限

### 7.1 CoppeliaSim 慢

RLBench 基于 **CoppeliaSim**，它有几个问题：

- **速度慢**：CoppeliaSim 的渲染与物理较重，单机并行度低，训练/评测效率不如 MuJoCo（robosuite）或 Isaac。
- **接口复杂**：CoppeliaSim 的 API 较老派，与现代 RL/IL 栈（如 Gymnasium）集成不如 robosuite 顺滑。
- **GPU 并行弱**：无法像 Isaac Lab 那样数千并行，限制了大规模 RL。

这是 LIBERO（robosuite/MuJoCo）后来居上的重要原因之一。

### 7.2 任务数少且固定

- 100 个手工任务，扩展成本极高——加新任务要写规划脚本、成功判定、调试，数天起步。
- **无法程序化生成变体**（不像 LIBERO 的 PDDL），泛化评测受限。
- 任务集固定 → 容易过拟合到这 100 个任务的设计偏好。

### 7.3 不评长程、不评跨本体

- 每任务独立评测，无连续长程设定（不如 CALVIN）。
- 只有 Franka Panda 一个本体，不评跨本体（不如 OXE）。
- 无杂乱场景、无真机迁移通道。

### 7.4 动作空间局限

- next-best-pose 式动作**不适合高频力控**（接触任务如擦拭、插销的力觉反馈无法体现）。
- 与现代 VLA（离散 token）和 Diffusion Policy（连续序列）的动作范式都不完全兼容，需要适配。

### 7.5 评测方差

- 25 rollout/task 的统计量小，分数波动大；不同实现间复现差异常被诟病。

---

## 8. 与同类基准对比

| 维度 | **RLBench** | CALVIN | LIBERO | MetaWorld |
| --- | --- | --- | --- | --- |
| 仿真 | **CoppeliaSim** | PyBullet | robosuite/MuJoCo | MuJoCo |
| 任务生成 | **手工**（精细） | 脚本 | PDDL 程序化 | 手工 |
| 任务数 | 100（PerAct 用 18） | 34 原语 | 130 | 50 |
| 长程评测 | ❌ | **✅ 核心** | Long（多任务） | ❌ |
| 泛化维度 | 任务变体 | 环境布局 | **三正交维度** | 任务 |
| 3D 表征适配 | **✅ 强（RGB-D 多视角）** | 弱 | 中 | 弱 |
| 精细操作 | **✅ 强（插入/旋转）** | 弱 | 中 | 中 |
| 仿真速度 | 慢 | 中 | 快 | 快 |
| 主流用途 | **3D 方法（PerAct 系）** | 长程 VLA | VLA 泛化 | RL 算法 |
| 发布年 | 2020 | 2022 | 2023 | 2019 |

一句话：**RLBench 赢在任务精细与 3D 适配，输在仿真速度与可扩展性**。它是 3D 操作方法的「老家」，但 VLA 浪潮已更多转向 LIBERO/CALVIN。

---

## 9. 相关概念互链

- [`../README.md`](../README.md)——基准总览，RLBench 是经典操作基准。
- [`calvin.md`](calvin.md)——长程操作基准，与 RLBench 互补（长程 vs 精细）。
- [`libero.md`](libero.md)——泛化操作基准，2024 VLA 主流。
- [`../vla-eval/real-world-eval.md`](../vla-eval/real-world-eval.md)——仿真基准的局限与真机评测的挑战。
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)——3D Diffusion Policy（DP3）在 RLBench 上验证。
- [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)——VLA 与 3D 方法的路线对比。
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)——RLBench 的标准训练范式是 BC。
- [`../../simulation/README.md`](../../simulation/README.md)——CoppeliaSim 是 RLBench 的底座，速度是痛点。
- [`../../hardware/arms/franka-panda.md`](../../hardware/arms/franka-panda.md)——RLBench 使用的本体。

---

## 10. 参考链接

- 项目主页：https://sites.google.com/view/rlbench
- GitHub：https://github.com/stepjam/RLBench
- 论文：James et al., *RLBench: The Robot Learning Benchmark & Learning Environment*, IEEE RA-L 2020. arXiv:1909.12271
- PerAct：Shridhar et al., *Perceiver-Actor: A Multi-Task Transformer for Robotic Manipulation*, CoRL 2022. https://peract.github.io/
- Act3D：Gervet et al., *Act3D: 3D Feature Field Transformers for Multi-Task Robotic Manipulation*, CoRL 2022.
- GNFactor：Bharadhwaj et al., *Towards Scaling Multi-task Learning with Robot Learning*, 2023.
- MimicGen：Mandlekar et al., *MimicGen: A Data Generation System for Autonomous Robot Manipulation*, 2023.
