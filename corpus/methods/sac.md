# SAC · Soft Actor-Critic（软 Actor-Critic）

> **off-policy + 最大熵**的连续控制 SOTA——在 [PPO](ppo.md) 统治 on-policy 仿真的同时，SAC 是**样本宝贵**场景（真机交互、expensive rollout）的首选。最大熵正则让策略**自动平衡探索与利用**，是连续控制最稳定的 off-policy 算法之一。

## 核心思想

SAC = **off-policy Actor-Critic + 最大熵正则**。

最大熵目标不仅奖励高回报，还奖励**策略的熵**（随机性）：

$$
\pi^* = \arg\max_\pi \mathbb{E}_{\tau \sim \pi} \left[ \sum_t r(s_t, a_t) + \alpha \mathcal{H}(\pi(\cdot | s_t)) \right]
$$

- $r(s, a)$：环境奖励
- $\mathcal{H}(\pi)$：策略熵（鼓励探索）
- $\alpha$：温度系数，权衡回报 vs 探索

**直观**：策略要"**既做得好，又不要过拟合单一动作**"——这种"软"探索是 SAC 成功的关键。

## 关键机制

### 1. 双 Q 网络（Clipped Double Q）

```
Q_net 1 (Q_θ1) ──┐
                  ├──► min(Q_θ1, Q_θ2) ──► 减少过估计
Q_net 2 (Q_θ2) ──┘
```

> 双 Q 取 min 是为了**避免 Q 值过估计**（DQN 的通病），稳定训练。

### 2. 软 Q 值更新

$$
\mathcal{T}^\pi Q(s_t, a_t) = r(s_t, a_t) + \gamma \mathbb{E}_{s_{t+1}} \left[ V(s_{t+1}) \right]
$$

其中软价值函数：

$$
V(s_t) = \mathbb{E}_{a_t \sim \pi} \left[ Q(s_t, a_t) - \alpha \log \pi(a_t | s_t) \right]
$$

### 3. 策略更新（重参数化技巧）

策略网络用**重参数化技巧**采样动作（不是直接 argmax）：

$$
a_t = \tanh(\mu_\theta(s_t) + \sigma_\theta(s_t) \odot \epsilon), \quad \epsilon \sim \mathcal{N}(0, I)
$$

这样梯度可以**直接反传到策略网络**。

### 4. 自动调节温度 $\alpha$

SAC 论文 2 提出**自动调整 $\alpha$**，让目标熵等于 $-\text{dim}(\mathcal{A})$（动作维度负值）：

$$
J(\alpha) = \mathbb{E}_{a_t \sim \pi_t} \left[ -\alpha \log \pi_t(a_t | s_t) - \alpha \bar{\mathcal{H}} \right]
$$

> 这是 SAC 与其他最大熵方法的关键差异——**不用手动调 alpha**。

### 5. Off-Policy 重放缓冲

SAC 用 Replay Buffer 存所有历史 transition，新数据混入**随机采样**训练：

```
[Buffer: 1M transitions]
新数据 ──► Buffer ──► 随机采样 batch ──► 更新 Q + π
```

> 这让 SAC 的样本效率比 PPO 高 5-10×，但并行性不如 PPO。

## 关键超参数

| 参数 | 推荐值 | 备注 |
| --- | --- | --- |
| **Replay buffer 大小** | 1M-10M | 越大越稳定 |
| Batch size | 256 | 标准 |
| 折扣 $\gamma$ | 0.99 | 长程 0.995 |
| 软更新 $\tau$ | 0.005 | target net 更新率 |
| 目标熵 $\bar{\mathcal{H}}$ | $-\text{dim}(\mathcal{A})$ | 自动 |
| 学习率（actor） | 3e-4 | |
| 学习率（critic） | 3e-4 | 通常等于 actor |
| 网络结构 | MLP 256×2 | 状态-动作小；图像加 CNN |
| 训练步数 | 1M-5M | 远少于 PPO |

## 关键工作脉络

| 模型 | 年份 | 贡献 |
| --- | --- | --- |
| **Soft Q-Learning** | 2017 | 最大熵 + Q-learning |
| **SAC** | **2018**（Haarnoja et al.） | Actor-Critic + 最大熵 + 自动 alpha |
| SAC v2 | 2018 | 自动温度调节 |
| **SAC + 视觉** | 2019 | RAD / DrQ / SAC-AE（视觉输入） |
| **CrossQ** | 2022 | 简化 SAC，移除 target net |
| **SAC + 演示** | 2018 | SAC-X（DeepMind 多任务） |
| **TD3** | 2018 | 双 Q + 延迟更新（与 SAC 平行） |

## 与其他算法对比

| 维度 | [SAC](sac.md) | [PPO](ppo.md) | TD3 | DDPG | DQN |
| --- | --- | --- | --- | --- | --- |
| 类型 | **off-policy** | on-policy | off-policy | off-policy | off-policy |
| 样本效率 | **高** | 中 | 中 | 中 | 中 |
| 稳定性 | **高** | 高 | 中 | 低 | 中 |
| 连续动作 | **首选** | OK | OK | OK | ❌ |
| 探索 | **最大熵** | 熵正则 | 高斯噪声 | OU 噪声 | ε-greedy |
| 实现难度 | 中 | **低** | 中 | 中 | 低 |
| 训练速度 | 中 | **快** | 中 | 中 | 快 |
| 适用 | 样本贵 | 仿真并行 | 简单任务 | 已较少用 | 离散动作 |
| 具身智能 | **样本宝贵时** | 仿真首选 | 较少用 | 较少用 | 离散动作 |

> **具身 RL 怎么选**：
> - **仿真 + 大规模并行** → PPO（首选）
> - **真机 / 样本贵** → **SAC**（首选）
> - **有演示 + 仿真** → BC + PPO/SAC 微调
> - **大规模历史数据** → 离线 RL（CQL / IQL）

## SAC 在具身智能里的角色

SAC 是**真机交互**和**数据效率**场景的首选：

| 任务 | SAC 表现 |
| --- | --- |
| **真机 RL** | ✅ 首选（样本贵） |
| **机械臂控制** | ✅ 替代 PPO |
| **SoftRobot 仿真** | ✅ 标配 |
| **多任务** | ✅ SAC-X（DeepMind） |
| **离线 RL** | ⚠️ 改用 CQL / IQL |
| **灵巧手** | PPO 更流行（Isaac 并行） |
| **locomotion** | PPO 更流行（同上） |

## 优缺点

### 优点
- ✅ **样本效率高**（off-policy，5-10× PPO）
- ✅ **稳定性好**（双 Q + 自动 alpha）
- ✅ **自动调探索**（目标熵）
- ✅ **真机 RL 首选**（样本成本）
- ✅ **离线数据可用**（replay buffer）

### 缺点
- ❌ **并行性差**（off-policy，buffer 难并行）
- ❌ **大动作空间不如 PPO**（如灵巧手 16-DoF）
- ❌ **图像输入需额外处理**（RAD/DrQ/SAC-AE）
- ❌ **实现比 PPO 复杂**（4 个网络 + 软更新）
- ❌ **温度调节不稳**（某些任务仍需手调 alpha）

## SAC vs PPO 选用决策

| 场景 | 推荐 |
| --- | --- |
| 仿真训练（Isaac Lab 1024+ 并行） | **PPO** |
| 真机交互（数据贵） | **SAC** |
| 大动作空间（灵巧手 16+） | **PPO**（并行） |
| 视觉输入（图像） | 两者都行（SAC + RAD/DrQ） |
| 有演示数据 | 两者都行，**先 BC 预训练** |
| 离线数据（只有历史） | **离线 RL**（CQL / IQL / DT） |
| 离散动作 | PPO / DQN（SAC 不支持） |

## 关键代码片段

```python
# Stable-Baselines3 SAC
from stable_baselines3 import SAC

model = SAC(
    "MlpPolicy",
    env,
    buffer_size=1_000_000,         # 1M replay buffer
    batch_size=256,
    gamma=0.99,
    tau=0.005,                     # 软更新率
    ent_coef="auto",               # 自动温度
    learning_rate=3e-4,
    verbose=1,
)
model.learn(total_timesteps=1_000_000)
```

## 相关语料

- 方法：[reinforcement-learning](reinforcement-learning.md) · [ppo](ppo.md) · [dagger](dagger.md)
- 概念：[policy](../concepts/mdp/policy.md) · [reward-function](../concepts/mdp/reward-function.md) · [observation-space](../concepts/mdp/observation-space.md) · [action-space](../concepts/mdp/action-space.md)
- 仿真：[nvidia-isaac](../simulation/platforms/nvidia-isaac.md) · [mujoco](../simulation/platforms/mujoco.md)
- 评测：[rlbench](../benchmarks/manipulation/rlbench.md) · [behavior-1k](../benchmarks/manipulation/behavior-1k.md)

## 参考论文

- **Haarnoja et al. 2018** — *Soft Actor-Critic: Off-Policy Maximum Entropy Deep Reinforcement Learning with a Stochastic Actor*（ICML 2018）
- **Haarnoja et al. 2018 v2** — *Soft Actor-Critic Algorithms and Applications*（自动 alpha）
- **Fujimoto et al. 2018** — *Addressing Function Approximation Error in Actor-Critic Methods*（TD3）
- **Laskin et al. 2020** — *Reinforcement Learning with Augmented Data*（RAD，视觉 SAC）
- **Sinha et al. 2022** — *CrossQ: Batch Normalization in Deep Reinforcement Learning for Greater Sample Efficiency*（简化 SAC）
