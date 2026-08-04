# 域随机化（Domain Randomization, DR）

> **一句话定位**：让策略在训练时「见识过」足够多的物理/视觉/控制扰动，从而对真实世界的未知参数**不敏感**——是 Sim-to-Real 迁移最主流、最朴素、也最有效的方法之一：把仿真当成「真实世界的采样器」，而非「真实世界的精确复刻」。

> 最后更新：2026-08

---

## 1. 概览与定位

**域随机化（Domain Randomization, DR）** 是 Sim-to-Real 的核心方法之一。它的核心思想可以用一句话概括：

> **训练时把仿真的物理/视觉/控制参数当作随机变量，从分布中采样，让策略学到一个对参数变化鲁棒的解；部署到真机时，真机只是这个分布里的「又一个样本」。**

形式化地说：仿真不再是「一个确定环境」，而是「一个环境分布」。策略优化的是在这个分布上的**期望回报**，而非单一环境上的回报。

历史脉络：DR 的概念可追溯到 **OpenAI 的 Quadrotor（2017）** 与 **Rubik's Cube Hand（2019）** 工作，后者的「单臂转魔方」zero-shot sim-to-real 是 DR 的标志性成果。此后 DR 成为 locomotion、dexterous manipulation、抓取等任务的标配，几乎所有 Isaac/Legged Gym 训练的 policy 都默认开 DR。

> 与系统辨识（System Identification）的根本区别：**系统辨识**试图把仿真「调准」逼近真机（缩小 gap）；**域随机化**承认 gap 不可消除，转而让策略「不在乎 gap」。两者常组合使用。

---

## 2. 核心规格与设计要点

| 维度 | 内容 | 备注 |
| --- | --- | --- |
| 解决的问题 | Sim-to-Real Gap（仿真与真实的参数失配） | 物理/视觉/控制三类 gap |
| 核心机制 | 训练时从参数分布采样，每 episode/每 reset 不同 | 期望意义下的鲁棒策略 |
| 随机化对象 | 物理、视觉、动力学、控制、传感器 | 见 §3 |
| 数学形式 | $\max_\pi \mathbb{E}_{\xi \sim p(\xi)} [J(\pi, \xi)]$ | $\xi$ 为随机化参数向量 |
| 代表框架 | Isaac Lab `EventTerm`、`gym.Env` wrapper、MuJoCo MJCF 参数扫描 | 主流仿真器均支持 |
| 典型收益 | zero-shot 或 few-shot sim-to-real | 视 gap 大小 |
| 主要代价 | 训练难度上升（任务在「最坏样本」上更难）、收敛变慢 | 需平衡随机化幅度 |
| 与系统辨识 | 互补，常组合 | 先辨识再在残余不确定性上随机化 |

---

## 3. 核心技术解析：随机化什么

域随机化的关键在于「**随机化哪些参数**」。按 gap 来源分三大类：

### 3.1 物理参数随机化（Physics / Dynamics Randomization）

针对「仿真物理与真实物理不一致」——这是最常见也最有效的一类。

| 参数（Parameter） | 典型分布 | 为什么随机化 |
| --- | --- | --- |
| **摩擦系数（friction）** $\mu$ | $\mathcal{U}(0.3, 1.2)$ | 接触摩擦真机难精确测，足地/抓取极敏感 |
| **质量 / 转动惯量（mass / inertia）** | $\pm 20\%$ 摄动 | CAD 模型与实物有偏差，负载未知 |
| **阻尼 / 关节摩擦（damping / joint friction）** | $\mathcal{U}(0.1, 2) \times$ nominal | 关节静摩擦真机显著 |
| **控制增益（PD gains）** $K_p, K_d$ | $\pm 30\%$ | 电机/驱动器特性漂移 |
| **外力扰动（external force）** | 随机推力 / 风扰 | 真机受气流、碰撞、人推 |
| **复位位姿（reset pose）** | 微小摄动 | 减少对精确初态的过拟合 |
| **重力方向 / 大小** | 微小摄动 | 安装倾斜、星球探索等 |
| **物体形状 / 尺寸** | $\pm 10\%$ | 同类物体形态各异 |

> 直觉：足式机器人足底摩擦从冰面（$\mu \approx 0.1$）到橡胶（$\mu \approx 1.5$）变化极大，若策略只在固定 $\mu=0.8$ 训练，换地面就摔。DR 让它在整个范围内都能走。

### 3.2 视觉随机化（Visual / Observation Randomization）

针对「仿真渲染的图像与真实相机图像不一致」——纯视觉策略（RGB→action）的痛点。

| 参数 | 典型做法 | 为什么 |
| --- | --- | --- |
| **光照（lighting）** | 随机光源位置、强度、色温 | 真实环境光照千变万化 |
| **纹理（texture）** | 随机替换物体/地面纹理 | 防策略依赖具体外观 |
| **相机位姿（camera extrinsic）** | $\pm$ 几度/几厘米摄动 | 安装误差、振动 |
| **相机内参（camera intrinsic）** | 焦距、畸变随机 | 标定误差 |
| **背景（background）** | 随机背景图/视频 | 防过拟合到仿真空背景 |
| **噪声（noise）** | 加高斯/泊松噪声 | 传感器噪声 |

经典做法（OpenAI Rubik's Cube Hand）：训练时给魔方和手随机贴上五颜六色纹理、随机光照、随机背景——策略被迫学到「**不依赖具体外观的、关于位姿与接触的几何特征**」，迁到真机时真实外观只是分布里的一个样本。

> 这就是为什么 Isaac 的 RTX 光追渲染重要：光追能产生接近真实的光照/反射，使视觉随机化的「分布」更接近真实图像分布，sim-to-real 更顺。

### 3.3 动力学 / 控制随机化（Dynamics / Control Randomization）

针对「仿真控制环与真机控制环不一致」——延迟、带宽、执行器动态。

| 参数 | 典型做法 | 为什么 |
| --- | --- | --- |
| **动作延迟（action latency）** | 随机延迟 1–5 个控制周期 | 真机通信/计算有延迟 |
| **观测延迟（observation latency）** | 随机延迟 | 传感器采样不同步 |
| **执行器动态（actuator dynamics）** | 串联弹性/电机模型参数随机 | 电机响应非理想 |
| **控制频率抖动（control rate jitter）** | $\pm 10\%$ timestep | 实时性不保证 |

> 直觉：仿真里 `step()` 立即生效；真机里你发的力矩要经过 EtherCAT/串口/驱动器才到电机，且电机有惯性与电感滞后。若策略在「零延迟」仿真里训，真机的几十毫秒延迟会让它失稳。**延迟随机化是 locomotion sim-to-real 的必选项**。

---

## 4. 数学形式化

把上述随机化参数收集成向量 $\xi \in \Xi$，从一个分布 $p(\xi)$ 采样。仿真的转移函数与奖励都依赖 $\xi$：

$$
\xi \sim p(\xi), \quad s_{t+1} = f_\xi(s_t, a_t), \quad r_t = r_\xi(s_t, a_t)
$$

策略优化目标是**在参数分布上的期望回报**：

$$
\max_\pi \; \mathbb{E}_{\xi \sim p(\xi)} \left[ \mathbb{E}_{\tau \sim \pi, f_\xi} \left[ \sum_{t} \gamma^t r_{\xi,t} \right] \right]
$$

这等价于一个**鲁棒强化学习（Robust RL）**问题：策略要对 $p(\xi)$ 支撑集内的所有环境都尽量好。当 $p(\xi)$ 覆盖了真实世界可能的参数范围时，真机就被包含在这个分布里，策略自然能泛化。

> 与最坏情况鲁棒（$\max_\pi \min_\xi J$）的区别：DR 优化**期望**而非最坏情况，更易优化但理论上不保证「最坏样本也好」。实践中常用**curriculum**（随机化幅度逐步增大）缓解。

---

## 5. 为什么有效：直觉与理论

### 5.1 直觉：学到「不依赖具体参数」的策略

如果策略在「摩擦=0.8」固定仿真里训，它会学到「**依赖摩擦=0.8**」的步态（例如某种特定的足底压力分布）。换到摩擦=0.4 的地面，这个依赖失效，摔。

如果策略在「摩擦∈[0.3,1.2]」随机化里训，它**不能依赖任何具体摩擦值**——它必须学到一种「**在任何摩擦下都work**」的步态（例如更保守的落脚、更大的稳定裕度）。这种步态对真机（摩擦=0.6，未知）也work。

形式化地：随机化迫使策略学到的是**不变特征（invariant features）**——对参数变化不敏感的状态-动作映射。

### 5.2 理论：部分可观测 / 贝叶斯视角

可以把「未知参数 $\xi$」视为环境的**隐变量（hidden variable）**，策略观测不到 $\xi$，只能从历史观测推断。这把问题变成一个 **POMDP（部分可观测 MDP）**：

- 真实状态 = $(s_t, \xi)$，但策略只看到 $o_t = h(s_t)$（不含 $\xi$）；
- 最优策略需要**隐式系统辨识（implicit system identification）**——从历史推断 $\xi$，再据此行动；
- 这就是为什么 **recurrent policy（LSTM/Transformer）在 DR 训练中常优于 feedforward**——它们能从历史积累对 $\xi$ 的信念。

> 实践含义：做 DR 训练时，给策略一个**历史窗口**（如最近 50 步的观测）或用 **RNN/Transformer 策略**，能显著提升 sim-to-real 成功率——它让策略「边做边学参数」。

---

## 6. 真实代码：在 Isaac Lab 里配置域随机化

Isaac Lab 把随机化抽象为 `EventTerm`，可在环境配置里声明式添加。下面是 Franka 抓取任务的典型 DR 配置（节选）：

```python
# franka_lift_dr.py —— Isaac Lab 域随机化配置（节选，API 随版本演进）
from isaaclab.utils import configclass
from isaaclab.envs import ManagerBasedRLEnvCfg
from isaaclab.managers import EventTermCfg as EventTerm
from isaaclab.envs.mdp import events as mdp_events

@configclass
class FrankaLiftDRCfg(ManagerBasedRLEnvCfg):
    # ... 基础配置省略 ...

    events: dict = {
        # 1) 物理随机化
        "robot_mass": EventTerm(
            func=mdp_events.add_random_mass,
            mode="reset",                           # 每个 episode reset 时触发
            params={"asset_cfg": ..., "mass_range": (-0.5, 1.0)},  # ±质量摄动
        ),
        "friction": EventTerm(
            func=mdp_events.randomize_rigid_body_material,
            mode="reset",
            params={"static_friction_range": (0.3, 1.2),
                    "dynamic_friction_range": (0.3, 1.2)},
        ),
        "joint_damping": EventTerm(
            func=mdp_events.randomize_actuator_parameters,
            mode="reset",
            params={"damping_distribution_params": (0.5, 2.0),
                    "stiffness_distribution_params": (0.8, 1.2)},
        ),
        # 2) 外力扰动
        "push_robot": EventTerm(
            func=mdp_events.apply_random_external_force,
            mode="interval",                       # 训练中周期性施加
            params={"force_range": (-5.0, 5.0), "interval_range_s": (5.0, 15.0)},
        ),
        # 3) 观测/动作延迟
        "action_delay": EventTerm(
            func=mdp_events.randomize_action_delay,
            mode="reset",
            params={"delay_steps_range": (1, 4)},   # 1-4 个控制周期延迟
        ),
        # 4) 视觉随机化（若有相机观测）
        "lighting": EventTerm(
            func=mdp_events.randomize_lighting,
            mode="interval",
            params={"intensity_range": (200.0, 800.0),
                    "color_temperature_range": (3000.0, 7000.0)},
        ),
    }
```

要点：

- `mode="reset"`：每个 episode 开始时随机化一次（适合质量/摩擦等不变参数）；
- `mode="interval"`：训练中周期性变化（适合光照/外力等时变扰动）；
- 配置驱动，无需改环境代码——这是 Isaac Lab `EventTerm` 设计的优势。

> 在 MuJoCo / PyBullet 上做 DR 则需手写 wrapper：在 `reset()` 时随机改 `model.geom_friction`、`model.body_mass`、`model.dof_damping` 等。

---

## 7. 代表工作

| 工作 | 平台 | 随机化重点 | 成果 |
| --- | --- | --- | --- |
| **OpenAI Rubik's Cube Hand（2019）** | MuJoCo（自研） | **视觉随机化为主**（纹理/光照/背景）+ 物理 | 单臂转魔方 **zero-shot sim-to-real**，DR 标志性成果 |
| **OpenAI Quadrotor（2017）** | 自研 | 物理（质量/惯性/气动）+ 延迟 | 室内飞行 zero-shot 迁移 |
| **Legged Gym / AnymalC（ETH, 2021+）** | Isaac Gym | 摩擦/质量/外力/延迟 | 四足户外行走 sim-to-real 事实标准 |
| **RMA（Rapid Motor Adaptation, 2021）** | Isaac Gym | DR + 在线隐式辨识 | 应对未见地形（泥/雪/楼梯） |
| **Walk These Ways（ETH, 2022）** | Isaac Gym | 步态指令 + 大幅 DR | Anymal 多种步态 sim-to-real |

> 共性：**locomotion 任务 DR 的随机化幅度可以很大**（摩擦变化 4 倍、质量变化 50%），因为任务本身容错高；**精细操作任务的 DR 幅度需更保守**，否则训不出来。

---

## 8. 与系统辨识对比

| 维度 | 域随机化（DR） | 系统辨识（System ID） |
| --- | --- | --- |
| 思路 | 让策略**不在乎** gap | 把仿真**调准**消除 gap |
| 需要真机数据 | 否（纯仿真） | **是**（测量真机参数） |
| 实现难度 | 中（配置随机化分布） | 高（参数辨识实验） |
| 残余 gap | 大（靠策略鲁棒吸收） | 小 |
| 泛化性 | 强（对未见参数也work） | 弱（只对辨识的真机准） |
| 典型组合 | 先 DR 兜底，再在关键参数上做辨识收紧分布 | 两者互补 |

实践路径：**先 DR 兜底**（确保策略能跨一大片参数空间），**再系统辨识**（用真机数据把分布收紧到真实参数附近，降低训练难度）。例如先随机化摩擦∈[0.3,1.2]，辨识后发现真机摩擦≈0.7，可把分布收紧到[0.5,0.9]。

---

## 9. 常见坑与实战经验

- **随机化幅度过大**：策略在「极端样本」上完全失败，导致训练不收敛或学出过于保守的策略。先小后大（curriculum），或用**自适应随机化（ADR, Automatic DR）**逐步增大。
- **随机化幅度过小**：分布没覆盖真机参数，sim-to-real 仍失败。需根据真机不确定性的实际范围设分布。
- **只随机化物理，忽略延迟**：延迟是 locomotion sim-to-real 失败的头号原因之一。务必加动作/观测延迟随机化。
- **Feedforward 策略 + DR**：对隐参数推断能力弱，常失败。加历史观测或用 RNN/Transformer 策略。
- **视觉 DR 没配光追渲染**：用 PyBullet 简易 OpenGL 做视觉 DR，分布与真实图像差太远，迁不过去。视觉 DR 建议用 Isaac RTX。
- **复位随机化被忽略**：每个 episode 复位到完全相同初态，策略会过拟合。加复位位姿摄动。

---

## 10. 相关概念互链

- [`../README.md`](../README.md)——仿真总览与 Sim-to-Real Gap 的位置。
- [`../platforms/nvidia-isaac.md`](../platforms/nvidia-isaac.md)——`EventTerm` 是 DR 的工程载体。
- [`../platforms/mujoco.md`](../platforms/mujoco.md)——MuJoCo 上手写 DR wrapper。
- [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)——RL 与 sim-to-real 的依存。
- [`../../concepts/mdp/policy.md`](../../concepts/mdp/policy.md)——策略为何要对参数鲁棒。
- [`../../concepts/mdp/observation-space.md`](../../concepts/mdp/observation-space.md)——视觉随机化改变观测分布。

> 待建：`system-identification.md`（系统辨识）、`domain-adaptation.md`（域适应）——本目录后续补充。

---

## 11. 参考链接

- 论文：*Solving Rubik's Cube with a Robot Hand* (OpenAI, 2019, arXiv:1910.07113)——DR 标志性工作。
- 论文：*Domain Randomization for Sim-to-Real Transfer* (Tobin et al., 2017, arXiv:1703.06907)——视觉 DR 经典。
- 论文：*Symbiotic Trajectory Optimization for Sim-to-Real* / *Learning to Walk in Minutes Using Massively Parallel Deep RL* (Rudin et al., 2021, Legged Gym)——locomotion DR 实践。
- 论文：*RMA: Rapid Motor Adaptation* (Kumar et al., 2021, arXiv:2107.04034)——DR + 在线适应。
- Isaac Lab 文档（EventTerm / Randomization）：https://isaac-sim.github.io/IsaacLab/
- 综述：*Sim-to-Real Transfer in Deep Reinforcement Learning for Robotics: a Survey* (Wenshuai et al., 2020)
