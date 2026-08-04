# 仿真到真机鸿沟（Sim-to-Real Gap）

> 在仿真里训练好的策略，部署到真实机器人上**性能骤降甚至完全失效**——这种「仿真物理 / 视觉 / 控制与真实世界的系统性差异」统称为 Sim-to-Real Gap（仿真到真机鸿沟）。

> 最后更新：2026-08

---

## 1. 直觉

强化学习在机器人上几乎只能在仿真里训（真机采百万步不现实）。但仿真器是**对真实世界的近似**：摩擦系数、物体质量、相机噪声……总有些对不上。策略在仿真里学到的「依赖摩擦=0.8 的步态」，到了真机（摩擦其实是 0.5）就摔。**Sim-to-Real Gap 就是「仿真的近似误差」在策略层面被放大成的失效**。

```
   仿真策略 π_sim          真实环境 f_real
      (在 f_sim 上学得)   ──►  性能塌陷：f_sim ≠ f_real
```

> 命名澄清：本篇讲的是**问题**（the gap）。**解决方法**（域随机化、系统辨识、域适应）收录在 [`../../simulation/sim-to-real/`](../../simulation/sim-to-real/) 目录，本篇负责讲清「问题是什么、为什么存在、为什么是最大障碍」，并指向那些方法。

---

## 2. 为什么存在：Gap 的三类来源

### 2.1 物理 / 动力学 Gap（最致命）

仿真器的物理引擎对真实物理做了简化与近似：

| 参数 | 仿真假设 | 真实情况 | 后果 |
| --- | --- | --- | --- |
| **摩擦** $\mu$ | 一个固定常数 | 依赖接触史、湿度、温度，且静摩擦≠动摩擦 | 抓取打滑 / 足式打滑 |
| **质量 / 转动惯量** | CAD 标称值 | 实物有加工误差、负载未知 | 动态响应不符 |
| **接触 / 柔性** | 刚体或简单弹簧 | 真实接触是非线性、含变形与粘连 | 推、插、装配任务尤其敏感 |
| **关节摩擦 / 齿轮间隙** | 常被忽略 | 静摩擦、回程间隙真实存在 | 精密定位误差 |

### 2.2 传感器 / 观测 Gap（视觉策略尤其痛）

- **相机**：仿真渲染（OpenGL/光追）的色彩、噪声、畸变与真实相机不同；曝光、白平衡、镜头脏污这些仿真常忽略。
- **本体感知**：关节编码器噪声、IMU 偏置、力矩传感器温漂。
- **延迟**：仿真 `step()` 瞬时生效；真机要经总线、驱动器、电机响应，几十毫秒延迟。

### 2.3 控制 / 执行器 Gap

- **执行器模型**：仿真常把电机当理想力矩源；真实电机有电感、磁滞、扭矩饱和。
- **控制频率抖动**：仿真 timestep 严格等间隔；真机实时性不保证。

> 直觉总结：**仿真里的物理是「干净、确定、即时」的；真实物理是「脏、随机、滞后」的**。策略若过度依赖前者，必然在后者上崩。

---

## 3. 形式化：把 Gap 写进 MDP

真实 MDP 转移函数 $f_{\text{real}}$ 与仿真转移函数 $f_{\text{sim}}^\xi$（$\xi$ 是仿真物理参数）的差异，可写成分布偏移：

$$
\epsilon_{\text{gap}} = \mathbb{E}_{(s,a)} \big[ \,\big\| f_{\text{real}}(\cdot \mid s,a) - f_{\text{sim}}^\xi(\cdot \mid s,a) \big\| \,\big]
$$

策略在仿真上的期望回报 $J_{\text{sim}}(\pi)$ 与真机回报 $J_{\text{real}}(\pi)$ 之间的差距，随 rollout 步数 $H$ **指数放大**（模型误差累积）：

$$
\big| J_{\text{real}}(\pi) - J_{\text{sim}}(\pi) \big| \;\lesssim\; O\big(H \cdot \epsilon_{\text{gap}}\big) \quad \text{（一阶界）}
$$

这就是为什么**长 horizon 任务（如灵巧操作）sim-to-real 比短 horizon（如定点抓取）难得多**。

---

## 4. 为什么是 RL 在机器人上的最大障碍

| 原因 | 说明 |
| --- | --- |
| **RL 必须在仿真训** | 真机无法承受 RL 的百万步探索（慢、贵、危险），所以 sim-to-real 是 RL 路线的**必经之路**，绕不开 |
| **策略对仿真特异性的过拟合** | 仿真环境的「bug」（如数值不稳定、特定纹理）会被策略当成真实规律学进去 |
| **接触丰富任务尤其难** | 抓取、装配、插拔等任务对接触物理极敏感，恰恰是 gap 最大的地方 |
| **视觉策略双重 gap** | 既要抗物理 gap，又要抗渲染 gap，难度叠加 |
| **没有银弹** | 每个任务、每台机器人的 gap 不一样，方法要 case-by-case 调 |

对比：模仿学习（IL）直接在真机上采演示，**绕开了 sim-to-real**，这也是真机操作任务更偏好 IL 的根本原因（见 [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)）。但 IL 受限于演示质量与覆盖，无法超越人类；RL 想突破上限就必须解决 sim-to-real。

---

## 5. 解决路径（指向方法目录）

Sim-to-Real 不是单一算法，而是一组互补策略。**核心方法笔记落在 [`../../simulation/sim-to-real/`](../../simulation/sim-to-real/)**，这里给出地图：

| 路径 | 一句话 | 笔记 |
| --- | --- | --- |
| **域随机化（Domain Randomization）** | 训练时把物理/视觉参数当随机变量，让策略对 gap **不敏感** | [`../../simulation/sim-to-real/domain-randomization.md`](../../simulation/sim-to-real/domain-randomization.md) |
| **系统辨识（System Identification）** | 测量真机参数，把仿真**调准**逼近真实，缩小 gap | 待建 `system-identification.md` |
| **域适应（Domain Adaptation）** | 用少量真机数据把仿真分布**迁到**真实分布 | 待建 `domain-adaptation.md` |
| **Sim+Real 联合训练** | 仿真海量探索 + 真机少量精修，互补 | 见各 method 笔记 |
| **Real2Sim2Real** | 用真机数据校准仿真，再回真机部署的闭环 | 近年趋势 |

实践口诀：**先域随机化兜底**（保证策略能跨一大片参数空间），**再系统辨识收紧**（用真机数据把分布收到真实参数附近），**最后少量真机微调**（用 [`../training/finetuning.md`](../training/finetuning.md) 收尾）。

---

## 6. 三种迁移模式

部署时按是否还接触真机数据，分三档：

| 模式 | 含义 | 难度 | 代表 |
| --- | --- | --- | --- |
| **Zero-shot Sim-to-Real** | 仿真训完直接上真机，不再调 | 最难 | OpenAI 魔方手（重 DR） |
| **Few-shot / Fine-tune** | 真机上少量交互微调 | 中 | RMA、Walk-These-Ways |
| **Sim+Real 联合** | 训练时就混入真机数据 | 较易 | 多数工业部署 |

> Zero-shot 是「浪漫目标」，Few-shot 是「工程现实」。

---

## 7. 常见失败模式（排查清单）

- **机械臂抖动 / 振荡** → 多半是关节摩擦 / 增益 gap，加执行器模型随机化。
- **抓取总抓空或捏碎** → 摩擦 / 接触 gap，加摩擦随机化 + 力感知。
- **视觉策略换光照就失效** → 渲染 gap，加视觉随机化（光照/纹理/背景），或换光追渲染。
- **同代码不同机表现差异大** → 本体参数 gap，逐台做系统辨识。
- **仿真完美、真机首步就崩** → 延迟 gap，加动作/观测延迟随机化。
- **真机前几秒正常随后漂移** → 累积误差（闭环策略能纠一段，但模型偏差逐渐压过纠正能力）→ 缩短 effective rollout horizon 或加在线适应（RMA 式）。

## 7.1. 任务难度分级：哪些任务 sim-to-real 容易

| 任务类型 | Gap 敏感度 | 备注 |
| --- | --- | --- |
| **Locomotion（行走）** | 中 | 容错高，DR 幅度可大，zero-shot 常成功 |
| **抛/扔、跳跃** | 中 | 开环为主，接触少 |
| **定点抓取（rigid 物体）** | 中高 | 接触短暂，几何主导 |
| **插孔 / 装配** | **极高** | 接触丰富、容差毫米级，gap 几乎必致命 |
| **柔性物（布、绳）** | **极高** | 仿真柔性物理本身不准 |
| **与人交互** | 极高 | 人行为不可建模 |

直觉：**接触越丰富、容差越紧、物理越复杂，sim-to-real 越难**。这也是为什么抓取能 sim-to-real，而穿针引线至今做不到。

---

## 8. 相关概念互链

- [`embodied-ai.md`](embodied-ai.md)——具身智能闭环，sim-to-real 是其工程落地核心难题。
- [`embodiment.md`](embodiment.md)——本体物理参数的建模误差是 gap 的主要来源。
- [`../mdp/observation-space.md`](../mdp/observation-space.md)——视觉 / 本体感知的 gap。
- [`../mdp/policy.md`](../mdp/policy.md)——策略需对参数鲁棒。
- [`world-model.md`](world-model.md)——世界模型的模型误差累积是同一类问题的镜像。
- [`../training/finetuning.md`](../training/finetuning.md)——真机微调是 few-shot sim-to-real 的最后一步。

## 9. 代表方法 / 论文

- 域随机化方法 → [`../../simulation/sim-to-real/domain-randomization.md`](../../simulation/sim-to-real/domain-randomization.md)
- 仿真器载体 → [`../../simulation/platforms/nvidia-isaac.md`](../../simulation/platforms/nvidia-isaac.md)、[`../../simulation/platforms/mujoco.md`](../../simulation/platforms/mujoco.md)
- RL 与 sim-to-real 的依存 → [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md)
- 经典论文：
  - OpenAI, *Solving Rubik's Cube with a Robot Hand* (2019, arXiv:1910.07113)
  - Tobin et al., *Domain Randomization for Sim-to-Real Transfer* (2017, arXiv:1703.06907)
  - Peng et al., *Sim-to-Real Transfer of Robotic Control with Dynamics Randomization (ADR)* (2018, arXiv:1710.06537)
  - 综述：*Sim-to-Real Transfer in Deep RL for Robotics: a Survey* (Wenshuai et al., 2020)

---

## 10. 参考链接

- 仿真目录总览：[`../../simulation/README.md`](../../simulation/README.md)
- Sim-to-Real 方法目录：[`../../simulation/sim-to-real/`](../../simulation/sim-to-real/)
- OpenAI 魔方手博客：https://openai.com/research/solving-rubiks-cube
