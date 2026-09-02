# 观测空间（Observation Space）

> 智能体在每一步**能从环境接收的全部信号**的集合。记作 $\mathcal{O}$。它是策略的输入。

## 直觉

「观测」就是机器人「看到/感觉到的东西」。策略 $\pi(a \mid o)$ 拿观测做决策，所以**观测空间决定了智能体能利用的信息上限**——看不见的东西，再聪明的策略也用不上。

```
环境状态 s ──(传感器)──► 观测 o ──► 策略 π ──► 动作 a
```

## 形式化

- 观测函数（POMDP 中）：$o_t = \Omega(s_t)$，可能带噪声。
- 观测空间 $\mathcal{O}$：所有可能观测的集合。

## 具身智能的常见观测模态

| 模态 | 符号 | 典型来源 | 维度 |
| --- | --- | --- | --- |
| RGB 图像 | $I$ | 第三人称/手腕相机 | $H\times W\times 3$ |
| 深度图 | $D$ | 深度相机 | $H\times W\times 1$ |
| 点云 | $P$ | 激光/结构光 | $N\times 3$ |
| 本体感知（proprioception） | $q$ | 关节编码器 | DoF |
| 力/力矩 | $f$ | 力矩传感器 | 6 |
| 语言指令 | $\ell$ | 文本输入 | token 序列 |

一个完整观测通常是这些模态的拼接：$o = (I_{\text{ext}}, I_{\text{wrist}}, q, \ell)$。

## 关键设计抉择

1. **图像 vs 状态向量**：图像信息丰富但维度高、训练慢；状态向量（物体位姿）紧凑但需标注。现代方法多用图像。
2. **历史 vs 当前**：单帧观测常不足以推断速度/意图，需堆叠多帧或用 RNN/Transformer 编码历史。
3. **第三人称 vs 手腕视角**：前者看全局，后者看细节，常**双视角融合**。

## 观测与「可观测性」

- **完全可观**（MDP）：$o_t = s_t$，观测即真实状态。
- **部分可观**（POMDP）：观测有噪声或不全，需从历史推断状态——这是具身任务的常态（如物体被遮挡）。

## 相关概念

- [`action-space.md`](action-space.md)（动作空间）
- [`policy.md`](policy.md)（策略，吃观测输出动作）
- [`../foundations/embodiment.md`](../foundations/embodiment.md)（本体决定传感器配置）

## 参考

- Sutton & Barto《Reinforcement Learning: An Introduction》第 2 版——状态、观测与部分可观测性的经典论述（来源：http://incompleteideas.net/book/the-book-2nd.html，访问于 2026-09-02）
