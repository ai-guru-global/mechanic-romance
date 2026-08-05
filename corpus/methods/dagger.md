# DAgger · Dataset Aggregation（在线纠错模仿学习）

> 行为克隆（BC）的**致命弱点是"协变量偏移（covariate shift）"**——训练时见专家状态，测试时见自己的错误状态，错误会越滚越大。**DAgger** 用**在线纠错**破解这个死结，是模仿学习从"离线 BC"走向"在线 IL"的奠基算法。

## 核心思想

**问题**：BC 训的策略 $\pi$ 在测试时进入**没见过的状态分布**（因为专家不会去那里），就崩了：

```
专家状态分布 ρ_π*(s) ≠ 策略状态分布 ρ_π(s)
                ↓
        BC 只见过 ρ_π* 的状态
                ↓
     策略走到 ρ_π 但 ρ_π 没训过 → 错误累积
                ↓
              崩溃
```

**DAgger 的解法**：让策略在**自己引发的状态**下**继续向专家问**：

```
1. 用 BC 训的策略 π 跑 rollout
2. 在每个状态 s_t，问专家"该做什么" a*_t
3. 把 (s_t, a*_t) 加到训练集 D
4. 用扩充的 D 重新训 π
5. 重复（这就是 "DAgger"——Dataset Aggregation）
```

> 关键：把**策略的真实状态分布**逐渐**覆盖进训练集**，消除协变量偏移。

## 算法流程

```python
# DAgger (Ross et al. 2011)
D = {}  # 初始数据集
π = BC(D)  # 初始 BC 策略

for iteration i in 1, 2, ..., N:
    D_i = run(π)  # 用当前策略 rollout
    for s in D_i:
        a* = expert(s)  # 在线问专家
        D = D ∪ {(s, a*)}  # 聚合
    π = BC(D)  # 重新训练
return π
```

> "在线问专家" = **专家全程看机器人走，实时给动作指令**。

## 关键机制

### 1. 协变量偏移（covariate shift）

BC 的统计学习假设：**训练分布 = 测试分布**。在模仿学习里这个假设被打破：

- 训练：$s \sim \rho_{\pi^*}(s)$（专家状态）
- 测试：$s \sim \rho_{\pi}(s)$（自己的状态）
- 当 $\pi \neq \pi^*$，$\rho_\pi \neq \rho_{\pi^*}$ → 错误累积

DAgger 把"测试分布"**也加进训练集**，让训练分布 = 真实运行分布。

### 2. 专家查询成本

DAgger 的代价是**专家必须在线**——但专家是稀缺的（人）：

| 任务 | 专家来源 | 成本 |
| --- | --- | --- |
| 棋类 | 程序 | 0 |
| 机器人 | 人类遥操作 | 慢（人疲劳） |
| 自动驾驶 | 人类司机 | 极高 |

> 2024+ 的研究转向**用 VLM 充当"在线专家"**（[Voyager](https://arxiv.org/abs/2305.16291)、[GenSim](https://arxiv.org/abs/2311.15161)），把 DAgger 思路复活。

### 3. DAgger 的几个变体

| 变体 | 关键差异 |
| --- | --- |
| **DAgger** (2011) | 朴素聚合所有数据 |
| **AggreVaTe** (2014) | 用梯度而非标签（cost-to-go） |
| **THOR** (2018) | 错误校正 + 混合专家数据 |
| **HG-DAgger** (2019) | 人类门控，只在必要时纠错 |
| **LBCP** (2019) | 用 cycle-consistency 减少专家查询 |
| **EIL** (2023) | 用 VLM 当"专家"自动标注 |

## 与其他 IL 方法对比

| 维度 | BC | [DAgger](dagger.md) | [Diffusion Policy](diffusion-policy.md) | [ACT](act.md) |
| --- | --- | --- | --- | --- |
| 数据采集 | **离线** | **在线** | 离线 | 离线 |
| 专家需求 | 一次性 | **持续** | 一次性 | 一次性 |
| 协变量偏移 | ❌ 严重 | ✅ 解决 | ⚠️ 缓解 | ⚠️ 缓解 |
| 样本效率 | 中 | **低**（多轮） | 高 | 高 |
| 实现难度 | **低** | 中 | 中 | 中 |
| 适合 | 简单任务 | 长程任务 | 复杂多模态 | 双手操作 |
| 2024+ 主流 | ❌ baseline | ⚪ 少见 | ✅ 主流 | ✅ 主流 |

> **DAgger 在 2020+ 不再主流**——Diffusion Policy / ACT 等离线方法用"动作分块"+"多模态建模"间接缓解了协变量偏移。**但 DAgger 的思想（"策略自己的状态 + 专家纠正"）在 2024+ 借 LLM 复活**（VLM-as-expert）。

## 为什么 DAgger 重要

1. **理论上**：是**第一个有理论保证**的模仿学习算法（Ross et al. 2011 证明：DAgger 性能随专家查询次数收敛到专家水平）
2. **历史上**：从 BC 到 IL 的**关键一步**
3. **思想复活**：2024+ VLM-as-expert 路线（Voyager、GenSim）本质是 DAgger

## 优缺点

### 优点
- ✅ **理论保证**（No-Regret）
- ✅ **解决协变量偏移**（根本问题）
- ✅ **通用框架**（不限于机器人）
- ✅ **可与 BC / DP / ACT 组合**（在 BC 后做 DAgger 微调）

### 缺点
- ❌ **专家查询成本高**（人疲劳）
- ❌ **训练多轮**（慢）
- ❌ **专家不一定知道答案**（新状态）
- ❌ **实践中已少用**（DP/ACT 更简单）

## 关键超参数

| 参数 | 推荐值 | 备注 |
| --- | --- | --- |
| 迭代轮数 N | 5-20 | 越多越接近专家 |
| 每轮 rollout 步数 | 100-1000 | 任务长度 |
| 每次更新数据比例 | 50-100% | 全量 vs 增量 |
| 专家查询比例 | 100% 早期 → 10% 后期 | 衰减减少专家负担 |

## 实际部署形态

### 1. 经典 DAgger（2011）

```
策略 π ──► rollout ──► 状态 s ──► 专家 ──► 动作 a*
                              ↓
                      D = D ∪ {(s, a*)}
                              ↓
                      重新训练 π
```

### 2. THOR（2018）—— 错误纠正式

- 策略走错时（检测：状态偏离预期）才问专家
- **专家查询减少 80%**（vs 朴素 DAgger）

### 3. HG-DAgger（2019）—— 人类门控

- **人类自主决定**何时介入（不依赖自动检测）
- 引入人类判断力，**更高效**

### 4. VLM-as-Expert（2024+）—— LLM 当专家

- 用 **GPT-4V / Claude Vision** 充当"专家"
- 在线给策略反馈
- Voyager / GenSim / ELLM 是代表

> **2024+ 趋势**：DAgger 的"在线专家"思想，结合 LLM 强大的视觉语言能力，让 **VLM-as-Expert IL** 成为新热点。

## 与现代 IL 方法的对比逻辑

| 方法 | 解决协变量偏移的方式 |
| --- | --- |
| **BC** | 不解决（baseline） |
| **DAgger** | 在线纠错（理论优） |
| **行为变换 / CDA** | 数据增强 |
| **DAgger + 视觉预训练** | 在线 + 视觉先验 |
| **Diffusion Policy** | 多模态建模间接缓解 |
| **ACT** | 动作分块 + 时序集成缓解 |
| **VLA** | 互联网数据 + 跨本体泛化 |
| **VLM-as-Expert** | DAgger 思想 + LLM |

> **DAgger 是"原理层"奠基**，Diffusion Policy / ACT 是"工程层"方案，**两者不冲突**——实践中可以**先用 DP 训，再用 DAgger 微调**。

## 关键代码片段

```python
# DAgger 简化版
import torch
import torch.nn as nn

class BCPolicy(nn.Module):
    """BC 策略网络"""
    def __init__(self, obs_dim, act_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(obs_dim, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, act_dim)
        )
    def forward(self, obs):
        return self.net(obs)

# DAgger 主循环
policy = BCPolicy(obs_dim, act_dim)
D = expert_dataset  # 初始离线数据

for i in range(N_ITER):
    # 1. 用当前策略 rollout
    states = rollout(policy, env)
    
    # 2. 在线问专家
    for s in states:
        a_star = expert(s)  # 关键：专家在线回答
        D.add(s, a_star)
    
    # 3. 重新训练
    policy = train_bc(D)

return policy
```

## 相关语料

- 方法：[imitation-learning](imitation-learning.md) · [behavioral-cloning](behavioral-cloning.md) · [diffusion-policy](diffusion-policy.md) · [act](act.md)
- 概念：[observation-space](../concepts/mdp/observation-space.md) · [action-space](../concepts/mdp/action-space.md) · [policy](../concepts/mdp/policy.md)
- 仿真：[mujoco](../simulation/platforms/mujoco.md) · [nvidia-isaac](../simulation/platforms/nvidia-isaac.md)
- Demo：[02-ACT-双手叠衣](../demo/scenarios/02-ACT-双手叠衣/README.md) · [01-扩散策略-桌面堆叠](../demo/scenarios/01-扩散策略-桌面堆叠/README.md)

## 参考论文

- **Ross et al. 2011** — *A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning*（AISTATS，DAgger 原始论文）
- **Ross & Bagnell 2014** — *Reinforcement and Imitation Learning via Interactive No-Regret Learning*（AggreVaTe）
- **Bousmalis et al. 2018** — *Using Simulation and Domain Adaptation to Improve Efficiency of Deep Robotic Grasping*（THOR 思想）
- **Kelly et al. 2019** — *HG-DAgger: Interactive Imitation Learning with Human Experts*（ICRA）
- **Wang et al. 2023** — *Voyager: An Open-Ended Embodied Agent with Large Language Models*（VLM-as-Expert 复兴）
