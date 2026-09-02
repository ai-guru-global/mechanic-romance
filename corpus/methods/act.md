# ACT · Action Chunking Transformer

> 把**连续动作序列**切成固定长度的**「动作块（action chunk）」**，用一个 **CVAE + Transformer** 一次预测未来 K 步——模仿学习的**双手操作**范式，由 Stanford 2023 在 [ALOHA](https://mobile-aloha.github.io/) 上提出并一举成名。

## 核心思想

传统行为克隆（BC）一次预测**单步动作**，带来两个问题：
1. **抖动（chatter）**：单步预测方差大，机器人动作会**抖**
2. **复合误差（compounding error）**：走错一步后，状态分布偏离训练集，越走越偏

ACT 的核心洞察是**一次预测 K 步**（如 K=10）：

```
观测 o_t ──► [ CVAE 编码 ] ──► 风格变量 z ──► [ Transformer Decoder ] ──► 动作块 [a_t, a_t+1, ..., a_t+K-1]
                                                              │ 期间用
                                                              ▼
                                                   时序集成 (Temporal Ensembling) 平滑执行
```

两个关键技巧：

1. **动作分块（action chunking）**：一次预测未来 K=10 步动作 → 减少抖动 + 提供回旋余地
2. **时序集成（temporal ensembling）**：相邻时间步的动作块**重叠**，对重叠区取加权平均 → 执行更平滑

## 关键机制

### 1. CVAE 风格编码

ACT 用了**条件变分自编码器**架构：

- **编码器**：把演示轨迹 (o_1, a_1, ..., a_T) 编码为风格变量 z（捕获演示的"风格"——如速度、姿态）
- **解码器**：以 (o_t, z) 为条件，用 Transformer 自回归生成 K 步动作块

> 风格变量 z 让策略能**表达多模态演示**（"这次叠得快、那次叠得慢"），而不是平均成一种风格。

### 2. Transformer 解码器

```
input:  [z, o_t, query_1, query_2, ..., query_K]
                              ▼ (每 query 对应一个未来动作)
output: [a_t, a_t+1, ..., a_t+K-1]
```

- **查询向量**（learned queries）作为 K 个动作的"占位"
- Transformer 跨注意力看 (z, o_t) 后输出 K 个动作
- 训练用 L1 回归 + L2 正则

### 3. 时序集成

```
时间 t：动作块预测 [a_t, a_t+1, ..., a_t+K-1]
时间 t+1：动作块预测 [a_t+1, a_t+2, ..., a_t+K]
对 a_t+1 取两块的加权平均：0.8 × (t 块的) + 0.2 × (t+1 块的)
```

权重随预测时间远近衰减（近的权重大），让执行**平滑过渡**。

## 关键工作脉络

| 模型 | 年份 | 贡献 |
| --- | --- | --- |
| **ACT** (Zhao et al.) | **2023** | 首次系统提出「动作分块 + CVAE + 时序集成」 |
| ALOHA / Mobile ALOHA | 2023 | ACT 配套的**低成本双手硬件平台**（$20k 自建） |
| **Mobile ALOHA** | 2024 | 加移动底盘，ACT 训 50 演示即可学会复杂厨房任务 |
| **Crossformer** | 2024 | 把 ACT 思路扩到多相机多任务 |
| **DDEX** | 2024 | ACT + [Diffusion Policy](diffusion-policy.md) + 灵巧手 |
| **HATO** | 2024 | ACT + 触觉信号扩展 |

## 与其他 IL 方法对比

| 维度 | BC（单步） | [Diffusion Policy](diffusion-policy.md) | **ACT** |
| --- | --- | --- | --- |
| 一次预测 | 1 步 | K 步 | K 步 |
| 多模态建模 | ❌ 平均化 | ✅ 扩散去噪 | ✅ CVAE 风格变量 |
| 抖动 | **严重** | 中等 | **轻**（分块 + 时序集成） |
| 复合误差 | **严重** | 中等（分块缓解） | **轻**（分块） |
| 训练数据需求 | 100+ | 100+ | **50-100**（更少） |
| 推理速度 | 快（单步） | 慢（去噪） | 中（K=10 一次出） |
| 擅长 | 简单任务 | 高频精细 | **双手、长程** |
| 硬件 | 任意 | 任意 | **ALOHA 平台首选** |

## 为什么 ACT 重要

1. **解决 BC 两大痛点**（抖动 + 复合误差），是模仿学习里程碑
2. **双手 + 软体操作**的最实用方法（叠毛巾、拧瓶盖、装电池）
3. **Mobile ALOHA** 配合 ACT 创造了 2024 年的厨房家务 demo 现象级
4. **数据高效**：50 条演示即可学会复杂任务

## 优缺点

### 优点
- ✅ **数据高效**（50-100 演示）
- ✅ **执行平滑**（时序集成是杀手锏）
- ✅ **双手操作**最强 baseline
- ✅ **多模态演示**（CVAE 风格变量）
- ✅ **配套硬件**（ALOHA 设计简洁，DIY $20k）

### 缺点
- ❌ **CVAE 训练不稳定**（KL 项要小心调）
- ❌ **风格变量 z 的可解释性弱**
- ❌ **单相机输入不友好**（ALOHA 用 3 相机）
- ❌ **长程任务仍需更复杂架构**（HATO、Crossformer 在解决）

## 适用场景

- ✅ **双手操作**（叠衣 / 装配 / 料理）
- ✅ **长程任务**（多步操作）
- ✅ **演示数据少**（50-100 条）
- ✅ **需要平滑执行**（不允许抖动）
- ❌ 单步高频控制（ACT 的 K=10 滞后）
- ❌ 单目图像（需 3 相机才好）

## 关键超参数

| 参数 | 推荐值 | 备注 |
| --- | --- | --- |
| 动作块长度 K | **10-100** | 越短越灵活，越长越平滑 |
| KL 权重 β | 1e-5 | 太小 → 风格无意义；太大 → 单模态 |
| Transformer 层数 | 4-7 | 太深过拟合 |
| 学习率 | 1e-4 | AdamW |
| Epochs | 5000-20000 | 收敛慢，但稳 |

## 相关语料

- 论文笔记：[2023-zhao-act](../papers/imitation-learning/2023-zhao-act.md)
- 方法：[imitation-learning](imitation-learning.md) · [behavioral-cloning](behavioral-cloning.md) · [diffusion-policy](diffusion-policy.md) · [dagger](dagger.md)（在线纠错）
- 硬件：[franka-panda](../hardware/arms/franka-panda.md)（参考）
- 仿真：[mujoco](../simulation/platforms/mujoco.md)
- 概念：[sim-to-real](../concepts/foundations/sim-to-real.md)
- Demo：[02-ACT-双手叠衣](../../demo/scenarios/02-ACT-双手叠衣/README.md) · [08-灵巧手-五指旋转立方体](../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)（DDEX 路线）

## 参考论文

- **Zhao et al. 2023** — *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware*（RSS 2023）
- **Fu et al. 2024** — *Mobile ALOHA: Learning Bimanual Mobile Manipulation with Low-Cost Whole-Body Teleoperation*
- **DDEX 2024** — *Dexterous Diffusion Policy*
- **HATO 2024** — *HATO: Hand-Action Tactile-Object Manipulation*
