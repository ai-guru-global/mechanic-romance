# MuJoCo：高精度接触动力学仿真器（DeepMind 开源）

> **一句话定位**：被 Google DeepMind 于 2021 年开源的**高精度多关节接触动力学仿真器**——以凸优化（convex optimization）求解接触约束，是机器人 RL 算法原型、精确接触/摩擦建模、学术 benchmark 的事实首选；快、准、轻量，且接触/摩擦建模比 PhysX/Bullet 更可信。

> 最后更新：2026-08

---

## 1. 概览与定位

**MuJoCo**（读作 *moo-joe-co*）全称 **Multi-Joint Dynamics with Contact**，由 **Roboti LLC**（Emo Todorov 团队，源自华盛顿大学 HIT 实验室）开发，最初作为商用闭源产品（2012 起，授权给 OpenAI、DeepMind 等）。**2021 年 10 月被 Google DeepMind 收购并开源**（Apache-2.0），从此免费，社区生态爆发。

它的定位非常清晰：

- **不是**全功能仿真平台（无内置 RL 框架、无光追渲染、无 USD 大场景编辑）；
- **是**一个**专注多关节刚体 + 接触动力学**的「物理引擎 + 极简 Python/C API」，目标是**又快又准地仿真机器人接触**。

> 与 Isaac 的本质区别：MuJoCo 是**单环境 CPU 仿真器**，追求每个时间步的精度；Isaac 是**GPU 大规模并行仿真器**，追求环境数量。两者常被组合使用：MuJoCo 调算法、Isaac 跑大规模。

---

## 2. 平台规格与特性

| 维度 | 规格 / 取值 | 备注 |
| --- | --- | --- |
| 全称 | Multi-Joint Dynamics with Contact | 凸优化求解接触 |
| 开发方 | Roboti LLC → **Google DeepMind**（2021 起） | 现维护在 google-deepmind/mujoco |
| 许可 | **Apache-2.0**（2021 起开源） | 此前商用闭源 |
| 物理求解 | **凸优化（凸 QP + complementarity-free）** | 见 §3.2 |
| 接触模型 | 软接触（soft contact，基于参数化补互补） | 可配硬/软 |
| 摩擦模型 | **椭圆摩擦锥（elliptic friction cone）** | 比金字塔锥更准 |
| 积分器 | Euler / RK4 / **隐式（implicit）** / implicit-fast | 隐式对刚性接触更稳 |
| 模型格式 | **MJCF**（XML） + URDF（只读导入） | MJCF 表达力远超 URDF |
| 渲染 | 内置 OpenGL（`mujoco.viewer`，基于 GLFW）；可外接 | 简洁但不华丽 |
| 并行 | **CPU 多进程**；`MuJoCo XLA`（MX，JAX/GPU） | 见 §3.5 |
| 接口语言 | **Python**（`mujoco`）、C（`mujoco.h`） | C API 是底层 |
| RL 框架 | gymnasium、`dm_control`、`robosuite`、`gymnasium-robotics` | 不绑定特定框架 |
| 安装 | `pip install mujoco`（一行，纯 wheel） | 极轻量 |
| 平台 | Linux / macOS / Windows（含 ARM） | 跨平台 |

> 关键卖点：**`pip install mujoco` 一行装好，无 Omniverse、无 RTX GPU、无容器**——这是它成为学术算法原型首选的现实原因之一。

---

## 3. 核心技术解析

### 3.1 MJCF：表达力远超 URDF 的模型格式

**MJCF（MuJoCo XML format）** 是 MuJoCo 的原生模型格式，基于 XML。它的设计哲学是「**声明式 + 默认值继承**」：

- **默认值机制（`<default>`）**：可定义关节/几何/执行器的默认属性（如摩擦、阻尼），子元素自动继承，避免 URDF 那种每个 `<joint>` 都写一遍冗长参数；
- **条件性激活（`<actuator>`）**：执行器可声明为电机、肌肉、气缸、位置/速度伺服，参数化程度远高于 URDF 的纯 `<actuator>`；
- **等式约束（`<equality>`）**：可定义关节耦合、连接（weld）、绳索等约束，URDF 完全不支持；
- **场景内嵌**：MJCF 可在一个文件里同时定义机器人 + 地面 + 物体 + 灯光，URDF 只描述机器人本体。

实践含义：**`mujoco_menagerie`（Google 维护的官方模型库）提供了 Franka、UR5e、Spot、Anymal、ShadowHand、H1、G1 等高质量 MJCF**，开箱即用。相比之下 URDF 常需要补 collision/inertia。

### 3.2 凸优化求解接触：MuJoCo 的数学心脏

这是 MuJoCo 与 PhysX/Bullet 最本质的区别。传统物理引擎用**非凸互补问题（non-convex LCP）**求解接触，存在多解、抖动；MuJoCo 把接触松弛化为**凸优化（convex optimization）**：

物理方程（拉格朗日方程 + 接触约束）被写成：

$$
\min_{v, f} \quad \tfrac{1}{2}(v - \hat{v})^\top M (v - \hat{v}) \quad \text{s.t. 接触约束}
$$

其中 $v$ 是下一时刻广义速度、$\hat{v}$ 是无约束预测速度、$M$ 是质量矩阵。MuJoCo 用**参数化互补（parameterized complementarity）**——即「软接触」——把硬性互补约束 `f ≥ 0, gap ≥ 0, f·gap = 0` 松弛为可微的接触力，从而**整个问题保持凸性**，可用高效的牛顿法/梯度投影求解，且解唯一、可微。

带来的好处：

1. **接触稳定**：堆叠、滚动、滑动不易出现 PhysX/Bullet 的「穿透」「抖动」；
2. **可微**：可计算 $\partial \text{next\_state}/\partial \text{action}$，支持可微仿真与基于梯度的优化（如系统辨识、轨迹优化）；
3. **快**：凸 QP 求解极快，单步仿真在毫秒以下（中等规模模型）。

> 代价：接触是「软」的（接触刚度参数化），极端刚性接触不如硬约束 LCP 精确；但通过调参（`solref`/`solimp`）可逼近硬接触。

### 3.3 椭圆摩擦锥与摩擦建模

MuJoCo 默认使用**椭圆摩擦锥（elliptic friction cone）**，而非 Bullet/PhysX 常用的**金字塔近似（pyramid approximation）**：

- 椭圆锥更接近真实库仑摩擦（$|f_t| \leq \mu f_n$，切向力在二维切平面上构成圆/椭圆）；
- 金字塔锥是凸多面体近似，求解快但摩擦方向量化误差大；
- 椭圆锥使整个接触 QP 仍为二阶锥规划（SOCP），可凸求解。

参数化：摩擦系数 $\mu$ 可对各接触对单独指定（`friction`），还有**切向/扭转/滚动**三种摩擦各独立可调。这使得 MuJoCo 能可信地仿真足式机器人足地接触、灵巧手抓取、轮子滚动——这些都是 PhysX/Bullet 容易失真的场景。

### 3.4 隐式积分与刚性接触

MuJoCo 提供多种积分器，其中 **implicit / implicit-fast** 对刚性系统（高刚度接触、高频关节）至关重要：

- 显式 Euler 在刚性接触下需要极小步长才稳定；
- 隐式积分把刚性项放在下一时刻求解，允许大步长（典型 1–5 ms）仍稳定；
- MuJoCo 的隐式积分与凸接触求解耦合，**即使有硬接触也能用较大 timestep**（如 0.002 s）保持稳定，而 PhysX 常需要更小步长。

这是为什么 MuJoCo 被广泛用于「双足 / 灵巧手 / 抓取」——这些任务对接触稳定性极敏感。

### 3.5 MuJoCo XLA（MX）：JAX JIT 加速的 GPU 版本

**MuJoCo XLA（MX）** 是 DeepMind 推出的 MuJoCo 的 **JAX 重写版**，目标是：

- **JIT 编译 + vmap**：用 JAX 的 `jit` 编译单步物理，用 `vmap` 在 GPU 上批量并行数百到数千环境；
- **可微**：MX 的前向仿真对参数可微，支持可微 RL、基于梯度的系统辨识；
- **端到端 GPU**：状态、动作、物理全部在 GPU 张量上，无 CPU↔GPU 拷贝。

定位：MX 不是替代 MuJoCo（C 仍是底层），而是**提供「MuJoCo 精度 + GPU 大规模并行」**的新选择，直接对标 Isaac 的 GPU 仿真，但保留 MuJoCo 的接触精度。截至 2026 仍在活跃开发，部分 MJCF 特性尚未完全支持。

> 现状：MX 已能跑 locomotion RL 任务（MJX 提供了 Anymal、Humanoid 等 benchmark），吞吐量与 Isaac 同量级，但生态与文档不如 Isaac 成熟。值得作为「精准 + 并行」的候选。

### 3.6 真实代码：加载模型、步进、读取状态

```python
# mujoco_minimal.py —— MuJoCo Python API 最小示例
# 依赖：pip install mujoco
import mujoco
import numpy as np

# 1) 加载 MJCF（也可加载 URDF，但 MJCF 表达力更强）
model = mujoco.MjModel.from_xml_path("franka.xml")  # 来自 mujoco_menagerie
data = mujoco.MjData(model)

# 2) 关键属性
print("DoF:", model.nq, model.nv)          # 广义坐标 / 广义速度维度
print(" actuators:", model.nu)             # 执行器数
print(" timestep:", model.opt.timestep)    # 物理步长（默认 0.002 s）

# 3) 步进物理 + 写入控制
mujoco.mj_step(model, data)                # 推进一帧
data.ctrl[:] = np.zeros(model.nu)          # 写零力矩（重力补偿需另算）
mujoco.mj_step(model, data)

# 4) 读取状态
print("qpos:", data.qpos)                  # 关节位置
print("qvel:", data.qvel)                  # 关节速度
print("contact:", data.ncon)               # 当前接触点数

# 5) 可视化（独立窗口）
import mujoco.viewer
mujoco.viewer.launch(model, data)
```

要点：

- 单进程、单环境、CPU；
- `mj_step` 是核心函数，其余围绕 `MjModel`（静态参数）与 `MjData`（动态状态）展开；
- 想做 RL，包一层 `gymnasium.Env`，或直接用 `dm_control` / `robosuite`。

---

## 4. 适用场景

| 场景 | 为什么选 MuJoCo |
| --- | --- |
| **RL 算法原型**（新算法快速验证） | 单步快、装好即用、API 简洁，迭代效率高 |
| **接触/摩擦敏感任务**（灵巧手、双足、抓取） | 凸接触 + 椭圆摩擦锥，接触可信度高 |
| **学术 benchmark 复现** | DeepMind Control Suite、Gym MuJoCo、robosuite 均基于它 |
| **可微仿真 / 系统辨识** | 接触可微，MX 进一步提供 JAX 自动微分 |
| **算法对比（vs PhysX）** | 当你需要「物理上正确」的对照基准 |

不擅长：大规模 GPU 并行 RL（用 Isaac 或 MX）、高质量视觉渲染（用 Isaac）、ROS 集成（用 Gazebo）、入门零配置（PyBullet 更简单，但 MuJoCo 也已 `pip` 化）。

---

## 5. 与同类对比

| 维度 | MuJoCo | Isaac Sim/Lab | PyBullet |
| --- | --- | --- | --- |
| 求解器 | **凸 QP（最准）** | PhysX TGS | Bullet Sequential Impulse |
| 接触稳定性 | **最高** | 中 | 中低 |
| 摩擦锥 | **椭圆（最准）** | 金字塔 | 金字塔 |
| 并行 | CPU 多进程；MX 提供 GPU | **GPU 数千环境（原生）** | 仅 CPU |
| 渲染 | 简易 OpenGL | **RTX 光追** | 简易 OpenGL |
| 上手 | `pip install mujoco` | 重（需 RTX + Omniverse） | **最简（pip）** |
| 模型格式 | MJCF（表达力强） | USD | URDF |
| 开源 | Apache-2.0 | 否 | Zlib |
| 主导 | Google DeepMind | NVIDIA | 社区 |

一句话：**MuJoCo 是「精度优先的算法实验台」，Isaac 是「规模优先的 RL 工厂」**。研究里常见做法是 MuJoCo 验证算法 → Isaac 跑大规模 → 真机。

> 详见 [`nvidia-isaac.md`](nvidia-isaac.md)、[`pybullet.md`](pybullet.md)。

---

## 6. 常见坑与实战经验

- **MJCF 的 `solref`/`solimp`**：接触刚度的两个核心参数。默认值对多数任务够用，但接触「太软」（穿透）或「太硬」（抖动）时需调。先理解 `solref`（时间常数 + 阻尼比）与 `solimp`（三段阻抗曲线）。
- **隐式积分器**：刚性任务（双足、硬接触）默认开 `implicit-fast`；若发现抖动，检查是否被改回了 Euler。
- **URDF 导入丢信息**：URDF 无执行器、无场景，导入后常需在 MJCF 里补 `<actuator>`、`<equality>`、地面、物体。
- **大规模 RL 别硬上**：MuJoCo 单进程跑 1 个环境，PPO 训百万步会慢。用 `SubprocVecEnv` 多进程（仍远慢于 Isaac），或切到 MX / Isaac。
- **`mujoco_menagerie` 优先**：别自己写 MJCF，先用官方模型库的现成资产，参数已调好。

---

## 7. 相关概念互链

- [`README.md`](../README.md)——仿真总览。
- [`nvidia-isaac.md`](nvidia-isaac.md)——大规模并行 RL 的对应方。
- [`pybullet.md`](pybullet.md)——更轻量的入门替代。
- [`genesis.md`](genesis.md)——可调用 MuJoCo 作为后端的新平台。
- [`../sim-to-real/domain-randomization.md`](../sim-to-real/domain-randomization.md)——MuJoCo 训完怎么迁真机。
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)——RL 与仿真的依存关系。
- [`../../hardware/arms/franka-panda.md`](../../hardware/arms/franka-panda.md)——MuJoCo 上建模最全的本体之一（robosuite / mujoco_menagerie）。

---

## 8. 参考链接

- MuJoCo 官方文档：https://mujoco.readthedocs.io/
- MuJoCo GitHub（DeepMind）：https://github.com/google-deepmind/mujoco
- `mujoco_menagerie`（官方模型库）：https://github.com/google-deepmind/mujoco_menagerie
- MuJoCo XLA（MX）：https://github.com/google-deepmind/mujoco/blob/main/mjx
- DeepMind Control Suite：https://github.com/google-deepmind/dm_control
- `robosuite`（MuJoCo 操作 benchmark）：https://github.com/ARISE-Initiative/robosuite
- 论文：*MuJoCo: A Physics Engine for Model-Based Control* (Todorov, E., IROS 2012)
- 开源公告：DeepMind Blog, *MuJoCo joins DeepMind, open-sourced*（2021-10）
