# 策略（Policy）

> 从**观测到动作的映射**。记作 $\pi$。策略就是智能体的「大脑」——看到什么，就做什么。

## 直觉

```
观测 o ──► [ 策略 π ] ──► 动作 a
```

无论是手写控制器还是神经网络，本质上都是一个策略：输入世界状态，输出要执行的动作。**具身智能的核心目标，就是学到一个好策略。**

## 形式化

- **确定性策略**：$a = \pi(o)$。一个观测对应唯一动作。模仿学习常用。
- **随机性策略**：$a \sim \pi(\cdot \mid o)$。输出动作分布。强化学习标准设定，便于探索。

## 策略的参数化

策略通常由带参数 $\theta$ 的函数（神经网络）实现：

$$
a_t = \pi_\theta(o_t)
$$

「学习」就是找一组好的参数 $\theta$。根据更新信号来源分两大范式：

| 范式 | 更新信号 | 见 |
| --- | --- | --- |
| 模仿学习（IL） | 人类演示（专家动作） | [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md) |
| 强化学习（RL） | 环境奖励 | [`../../methods/reinforcement-learning.md`](../../methods/reinforcement-learning.md) |

## 开环 vs 闭环

- **闭环策略**（closed-loop）：每步都看新观测 $a_t = \pi(o_t)$，能纠错。具身任务**必须是闭环**。
- **开环**（open-loop）：只看初始观测，预录整条轨迹。容易因扰动失败。

## 现代策略的典型结构

```
观测 (图像+本体感知+语言)
   │
   ├─ 视觉编码器 (ViT / ResNet)
   ├─ 语言编码器 (预训练 LM)
   │      ↓ 融合
   ├─ 策略头 (MLP / Transformer / Diffusion)
   │      ↓
   └─ 动作 a
```

代表：
- Diffusion Policy（扩散模型做策略头）→ [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)
- VLA（大模型即策略）→ [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)

## 相关概念

- [`observation-space.md`](observation-space.md)（输入）
- [`action-space.md`](action-space.md)（输出）
- [`reward-function.md`](reward-function.md)（强化学习中评估策略好坏）
