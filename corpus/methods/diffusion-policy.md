# 扩散策略（Diffusion Policy）

> 把**扩散模型**用作策略头：不去回归单个动作，而是建模「未来一段动作序列」的**多模态分布**。Chi et al., 2023。

## 要解决的核心问题：多模态动作

经典行为克隆用 MSE 回归 $\hat{a} = \pi(o)$，面对**多模态**（同一观测下多种合理动作）会「平均」出不可行动作。

```
观测：桌上一个杯子，要把它放进左/右任一盒子。
专家动作：有时往左、有时往右（两种都对）。
MSE 回归：平均 → 直着推（必失败）。
扩散策略：建模 P(动作序列 | 观测) → 采样出「往左」或「往右」。
```

## 方法：把动作序列当成「图像」去噪

扩散模型原本用于图像生成（DDPM）：从纯噪声出发，一步步去噪还原图像。Diffusion Policy 把这个流程搬到**动作序列**上。

### 训练（加噪）

对专家动作序列 $\mathbf{a}_{0:K}$ 逐步加高斯噪声：

$$
\mathbf{a}_k = \sqrt{\bar{\alpha}_k}\,\mathbf{a}_0 + \sqrt{1-\bar{\alpha}_k}\,\boldsymbol{\epsilon}, \quad \boldsymbol{\epsilon}\sim\mathcal{N}(0,\mathbf{I})
$$

训练网络 $\epsilon_\theta(\mathbf{a}_k, k, o)$ 预测所加噪声，损失：

$$
\mathcal{L} = \mathbb{E}\big\| \boldsymbol{\epsilon} - \epsilon_\theta(\mathbf{a}_k, k, o) \big\|^2
$$

### 推理（去噪，生成动作）

从噪声 $\mathbf{a}_K \sim \mathcal{N}(0,\mathbf{I})$ 出发，迭代 $K$ 步去噪，得到动作序列：

$$
\mathbf{a}_{k-1} = \text{denoise}_\theta(\mathbf{a}_k, k, o)
$$

关键：去噪过程**以观测 $o$ 为条件**，所以生成的动作与当前场景一致。

## 关键设计

| 设计 | 选择 | 原因 |
| --- | --- | --- |
| 动作表示 | **相对增量**（delta pose） | 数值小、泛化好 |
| 序列长度 | 预测未来 $k$ 步（如 16） | 动作分块，降低漂移 |
| 视觉编码 | CNN/ViT（预训练冻结） | 不让视觉训练干扰动作 |
| 条件方式 | 重建型 / 目标型 | 注入观测/目标 |

## 优点

- ✅ 天然建模**多模态**动作分布。
- ✅ 动作分块 → 对误差累积鲁棒。
- ✅ 训练稳定，不易模式坍塌（对比 GAIL/GAN）。

## 局限

- ❌ **推理慢**：要去噪几十步，实时性受挑战（可用 DDIM 减到几步）。
- ❌ 对**高维长程任务**（需多阶段规划）仍吃力。
- ❌ 无语言条件版本不能理解复杂指令（可加文本条件扩展）。

## 影响与后续

Diffusion Policy 是 2023 年具身操作的里程碑，启发了大量工作：
- **3D Diffusion Policy**（DP3）：用点云替代图像，泛化更强。
- **EquiBot**：融入等变性，少样本提升。
- 结合语言：语言条件的扩散策略。

## 代表论文

- Chi et al., *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion*, RSS 2023.（论文：https://arxiv.org/abs/2303.04137，项目站：https://diffusion-policy.cs.columbia.edu/，访问于 2026-09-02）
- Ze et al., *3D Diffusion Policy*, RSS 2024.

## 相关概念

- [`imitation-learning.md`](imitation-learning.md)
- [`behavioral-cloning.md`](behavioral-cloning.md)（Diffusion Policy 解决了 BC 的多模态痛点）
- [`vision-language-action.md`](vision-language-action.md)（另一条技术路线：大模型即策略）
