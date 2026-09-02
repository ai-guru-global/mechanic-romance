# 模仿学习（Imitation Learning, IL）

> 从**人类演示**中学习策略，无需奖励函数。也叫「从示范中学习」（Learning from Demonstration, LfD）。

## 核心思想

强化学习要设计奖励（极难），模仿学习换了个思路：**直接给智能体看「专家怎么做」**，让它模仿。

```
人类演示  τ = (o₀,a₀), (o₁,a₁), …, (oₙ,aₙ)
                              │
                              ▼
            学习策略 π_θ，使 π_θ(oₜ) ≈ aₜ
```

## 与强化学习的对比

| 维度 | 模仿学习 IL | 强化学习 RL |
| --- | --- | --- |
| 信号来源 | 专家演示 $(o, a)$ | 环境奖励 $r$ |
| 需要奖励？ | ❌ | ✅ |
| 数据成本 | 高（人工遥操） | 低（仿真自采集） |
| 上限 | ≤ 专家水平 | 可超越人类 |
| 主流场景 | 真机操作 | 仿真、导航、 locomotion |

> 机器人操作任务**几乎都用 IL**，因为奖励难设计、真机探索危险。

## 主要分支

- **行为克隆（BC）**：纯监督学习，最简单。→ [`behavioral-cloning.md`](behavioral-cloning.md)
- **逆强化学习（IRL）**：从演示反推奖励函数，再据此做 RL。
- **DAgger**：在线收集「策略犯错时」的演示，迭代修正误差累积。
- **GAIL**：用对抗学习（GAN 思想）让策略分布逼近专家分布。

## 核心难题：协变量偏移（Covariate Shift）

BC 训练时策略看到的是**专家轨迹上的观测**，但测试时策略一旦犯错，就会进入「专家从没见过的状态」，越错越远（误差雪崩）。

**对策**：
- DAgger：边跑边请专家标注新状态。
- 数据增广：加入随机扰动下的「恢复」演示。
- 学习鲁棒表征（如 Diffusion Policy 的多模态建模）。

## 代表工作

- ACT（Action Chunking Transformer）：动.action chunking 降低误差累积。
- Diffusion Policy：用扩散建模多模态动作分布。→ [`diffusion-policy.md`](diffusion-policy.md)
- ALOHA / Mobile ALOHA：双臂遥操采集 + BC。

## 相关概念

- [`behavioral-cloning.md`](behavioral-cloning.md)
- [`../concepts/mdp/policy.md`](../concepts/mdp/policy.md)
- [`reinforcement-learning.md`](reinforcement-learning.md)

## 参考

- ALVINN（Pomerleau 1988）——模仿学习最早的端到端驾驶系统（来源：https://proceedings.neurips.cc/paper_files/paper/1988/hash/812b4ba287f5ee0bc9d43bbf5bbe87fb-Abstract.html，访问于 2026-09-02）
- An Algorithmic Perspective on Imitation Learning（arXiv 2018，模仿学习综述）（来源：https://arxiv.org/abs/1811.06711，访问于 2026-09-02）
