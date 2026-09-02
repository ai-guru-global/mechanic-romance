# 行为克隆（Behavioral Cloning, BC）

> 模仿学习最简单的形式：把学策略当成**监督学习**——输入观测，标签是专家动作。

## 核心思想

给定演示数据集 $\mathcal{D} = \{(o_i, a_i)\}_{i=1}^{N}$，直接拟合一个映射：

$$
\min_\theta \sum_{(o,a) \in \mathcal{D}} \mathcal{L}\big(\pi_\theta(o),\, a\big)
$$

- 回归损失（连续动作）：MSE / L1 / smooth L1。
- 也可以把动作离散化用分类损失（VLA 常用）。

```
观测 o ──► [ 神经网络 π_θ ] ──► 预测动作 â ≈ 专家动作 a
```

## 为什么是机器人操作的事实标准

1. **实现简单**：就是个监督学习，无需奖励、无需环境交互。
2. **数据高效**：几十到几百条演示就能训出可用策略。
3. **真机友好**：不在真机上探索，安全。

## 致命弱点：协变量偏移

训练分布（专家状态）≠ 测试分布（策略自身状态）。一旦策略输出有微小偏差，下一帧观测就偏离专家轨迹，进入「未见状态」→ 误差累积 → 失败。

**缓解方法**：
- **动作分块（Action Chunking）**：一次预测未来 $k$ 步动作，减少「重新决策」次数，降低漂移。代表：ACT。
- **DAgger**：迭代收集策略自身轨迹并由专家标注。
- **加噪训练**：演示时人为扰动，并教策略「如何恢复」。

## 输入输出典型设置（操作任务）

| | 典型 |
| --- | --- |
| 输入 | 第三人称 RGB + 手腕 RGB + 本体感知 + 语言指令 |
| 输出 | 末端位姿增量（7 维：位置3 + 姿态3 + 夹爪1） |
| 频率 | 10–20 Hz |

## 单模态问题

标准 BC 用 MSE 回归，会**平均掉多模态动作**（如「从左抓」和「从右抓」都合理时，回归出中间不可行动作）。
→ 这是 Diffusion Policy 要解决的核心问题，见 [`diffusion-policy.md`](diffusion-policy.md)。

## 代表工作

- **ACT**（2023）：Transformer + 动作分块。
- **RT-1 / RT-2**：Google 的机器人 Transformer，BC 训练。
- **Diffusion Policy**（2023）：扩散建模动作。

## 相关概念

- [`imitation-learning.md`](imitation-learning.md)（BC 的上位概念）
- [`diffusion-policy.md`](diffusion-policy.md)（解决 BC 多模态问题）
- [`../concepts/mdp/policy.md`](../concepts/mdp/policy.md)

## 参考

- ALVINN: An Autonomous Land Vehicle in a Neural Network（Pomerleau 1988）——行为克隆的最早实践（来源：https://proceedings.neurips.cc/paper_files/paper/1988/hash/812b4ba287f5ee0bc9d43bbf5bbe87fb-Abstract.html，访问于 2026-09-02）
- ACT: Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware（Zhao et al. 2023）——现代 BC 代表工作（来源：https://arxiv.org/abs/2304.13705，访问于 2026-09-02）
