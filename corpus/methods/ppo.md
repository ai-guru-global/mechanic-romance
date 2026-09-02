# PPO · Proximal Policy Optimization（近端策略优化）

> **on-policy Actor-Critic 算法**的**事实标准**——通过**裁剪（clipping）目标函数**保证策略更新"近端"不剧变，2017 年由 OpenAI 提出后统治强化学习界 8 年，是具身 RL（[demo 06-强化学习-Franka 抓杯](../../demo/scenarios/06-强化学习-Franka抓杯/README.md) 和 [demo 08-灵巧手](../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)）的默认算法。

## 核心思想

策略梯度（PG）方法直接优化 $\pi_\theta(a|s)$ 最大化期望回报，**但步长难调**——步太大策略崩盘，步太小训练慢。PPO 用一个**简单巧妙的"裁剪"目标**解决这个问题：

$$
L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min\left( r_t(\theta) A_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) A_t \right) \right]
$$

其中：
- $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$：新旧策略的概率比
- $A_t$：优势函数（advantage）
- $\epsilon = 0.1 \sim 0.3$：裁剪范围
- **clip 之后**：当 $r_t$ 偏离 1 超过 $\epsilon$，梯度为 0 → 阻止剧变

## 关键机制

### 1. Actor-Critic 架构

```
状态 s_t ──► [ Actor π_θ ] ──► 动作 a_t ──► 环境 ──► r_t, s_t+1
       │
       └──► [ Critic V_φ ] ──► 状态价值 V(s_t) ──► 优势 A_t = R_t - V(s_t)
```

- **Actor**：策略网络，输出动作分布
- **Critic**：价值网络，估计 V(s_t)
- **优势 A_t**：动作比平均好多少（正 → 鼓励；负 → 抑制）

### 2. GAE（Generalized Advantage Estimation）

$$
A_t^{GAE(\lambda, \gamma)} = \sum_{l=0}^{T-t} (\gamma \lambda)^l \delta_{t+l}
$$

其中 $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$ 是 TD 残差。
- $\lambda \to 0$：低方差、高偏差（短视）
- $\lambda \to 1$：高方差、低偏差（长视）
- **$\lambda = 0.95$** 是 PPO 标配

### 3. 重要性采样（Importance Sampling）

PPO 用**旧策略采集的数据**训练**新策略**，靠 $r_t(\theta) = \pi_\theta/\pi_{\theta_{old}}$ 校正。

> 这一点让 PPO 可以**用一批数据更新多次**（PPO 通常 4 个 epoch），提高样本效率。

### 4. 多环境并行

PPO 几乎总是配合**大量并行环境**（Isaac Lab 1024+、IsaacGym 4096+）：

```
Env 1 ┐
Env 2 │  ──► shared rollout buffer ──► 1 次 PPO 更新
...  │      （并行采样，数据快）
Env N ┘
```

> **并行数量直接影响 PPO 性能**——4096 vs 1024 并行，相同训练步下性能可差 2×。

## 关键公式

### PPO-Clip 目标

$$
L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min\left( r_t(\theta) A_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) A_t \right) \right]
$$

### 总损失

$$
L(\theta) = L^{CLIP}(\theta) - c_1 L^{VF}(\theta) + c_2 S[\pi_\theta](s_t)
$$

- $L^{VF}$：价值函数 MSE 损失
- $S[\pi_\theta]$：策略熵（鼓励探索）
- $c_1, c_2$：平衡系数

## 关键超参数

| 参数 | 推荐值 | 备注 |
| --- | --- | --- |
| **并行环境数** | **1024-4096** | Isaac Lab 标配 |
| Rollout 步数 | 16-64 | 每个环境采多少步 |
| Mini-batch 大小 | 8-32 × 并行数 | 切 batch 训练 |
| Epochs per update | 3-10 | 数据重用次数 |
| Clip $\epsilon$ | **0.2** | 太严太松都不好 |
| 折扣 $\gamma$ | 0.99 | 长程任务可 0.995 |
| GAE $\lambda$ | **0.95** | 标准值 |
| 学习率 | **3e-4** | 线性衰减 |
| 价值损失系数 $c_1$ | 0.5 | 标配 |
| 熵系数 $c_2$ | 0.01-0.001 | 后期可降 |

## 关键工作脉络

| 模型 | 年份 | 贡献 |
| --- | --- | --- |
| **REINFORCE** | 1992 | 策略梯度起点 |
| **TRPO** | 2015 | 信赖域（KL 约束）→ 复杂 |
| **A2C / A3C** | 2016 | Actor-Critic + 异步 |
| **PPO** | **2017** | **简化 TRPO，clipping 替代 KL** |
| PPO + GAE | 2017 | 标准 PPO 全栈 |
| **PPO + LSTM** | 2017+ | 加循环网络处理部分可观测 |
| **Recurrent PPO** | Stable-Baselines3 | LSTM 版 |
| **IPPO / MAPPO** | 2022 | 多 agent 扩展 |

## 与其他 RL 算法对比

| 维度 | [PPO](ppo.md) | [SAC](sac.md) | DQN | TRPO |
| --- | --- | --- | --- | --- |
| 类型 | on-policy | off-policy | off-policy | on-policy |
| 样本效率 | 中 | **高** | 中 | 低 |
| 稳定性 | **高** | 中 | 中 | 高 |
| 连续动作 | ✅ | **✅**（首选） | ❌ | ✅ |
| 实现难度 | **低** | 中 | 中 | 高 |
| 训练速度 | 快 | 中 | 中 | 慢 |
| 探索 | 熵正则 | **最大熵** | ε-greedy | KL 约束 |
| 适用 | 通用首选 | 样本宝贵 | 离散动作 | 学术理论 |
| 具身智能 | **标配** | 备选 | 少用 | 少用 |

> **具身 RL 的实际选择**：
> - 有大规模并行仿真（Isaac Lab 1024+）→ **PPO**（首选）
> - 样本宝贵 / 真机交互 → **SAC**（off-policy）
> - 有演示数据 → **离线 RL**（CQL / IQL）

## PPO 在具身智能里的角色

PPO 是**操作 / 灵巧手 / locomotion** 的默认 RL 算法：

| 任务 | 状态-动作 | PPO 表现 |
| --- | --- | --- |
| **Franka 抓杯** | 7-DoF 关节 | SOTA（[demo 06](../../demo/scenarios/06-强化学习-Franka抓杯/README.md)） |
| **Allegro 立方体旋转** | 16-DoF 灵巧手 | SOTA（[demo 08](../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)） |
| **Humanoid 行走** | 30+ DoF | SOTA（Isaac Lab Humanoid） |
| **四足 locomotion** | 12 DoF | SOTA（Legged Gym） |
| **导航** | 2-DoF 底盘 | OK，VLN 主流是 IL |

> **PPO + Isaac Lab = 具身 RL 的"Linux + GCC"**。

## 优缺点

### 优点
- ✅ **实现简单**（相比 TRPO 无需共轭梯度 / 约束优化）
- ✅ **训练稳定**（clip 机制）
- ✅ **通用性强**（离散 / 连续都支持）
- ✅ **可与并行环境配合**（样本效率提升）
- ✅ **生态成熟**（Stable-Baselines3、CleanRL、tianshou 全有）

### 缺点
- ❌ **样本效率低**（on-policy，扔数据）
- ❌ **策略更新不稳定**（clip 之外仍有梯度）
- ❌ **连续动作效果不如 SAC**（off-policy 数据重用）
- ❌ **超参敏感**（clip、ent coef 要调）
- ❌ **大动作空间（如灵巧手）训练慢**

## 关键代码片段

```python
# Stable-Baselines3 PPO
from stable_baselines3 import PPO

model = PPO(
    "MlpPolicy",
    env,                          # gym env（建议并行包装）
    n_steps=2048,                 # rollout 步数
    batch_size=64,                # mini-batch
    n_epochs=10,                  # 数据重用
    gamma=0.99,                   # 折扣
    gae_lambda=0.95,              # GAE
    clip_range=0.2,               # clip
    ent_coef=0.01,                # 熵
    learning_rate=3e-4,
    verbose=1,
)
model.learn(total_timesteps=10_000_000)
```

## 相关语料

- 论文笔记：[demo 06-强化学习-Franka 抓杯](../../demo/scenarios/06-强化学习-Franka抓杯/README.md) 引用 PPO
- 方法：[reinforcement-learning](reinforcement-learning.md) · [sac](sac.md) · [dagger](dagger.md)
- 仿真：[nvidia-isaac](../simulation/platforms/nvidia-isaac.md)（PPO 标配平台）
- 概念：[policy](../concepts/mdp/policy.md) · [reward-function](../concepts/mdp/reward-function.md) · [sim-to-real](../concepts/foundations/sim-to-real.md) · [domain-randomization](../simulation/sim-to-real/domain-randomization.md)
- 评测：[rlbench](../benchmarks/manipulation/rlbench.md)

## 参考论文

- **Schulman et al. 2017** — *Proximal Policy Optimization Algorithms*（arXiv）（来源：https://arxiv.org/abs/1707.06347，访问于 2026-09-02）
- **Schulman et al. 2015** — *Trust Region Policy Optimization*（TRPO，PPO 前身）
- **Mnih et al. 2016** — *Asynchronous Methods for Deep Reinforcement Learning*（A3C）
- **Engstrom et al. 2020** — *Implementation Matters in Deep RL: A Case Study on PPO and TRPO*
- **Rudin et al. 2022** — *Learning to Walk in Minutes Using Massively Parallel Deep RL*（PPO + Isaac Gym）
