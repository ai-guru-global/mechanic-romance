# 方法笔记 · Methods

收录具身智能核心**方法 / 算法**的结构化梳理。一篇笔记讲清一个方法：是什么、解决什么、关键公式、代表工作、适用边界。

---

## 方法索引

| 方法 | 关键词 | 状态 |
| --- | --- | --- |
| [模仿学习](imitation-learning.md) | 从演示学习、无需奖励 | 🟢 完成 |
| [行为克隆](behavioral-cloning.md) | IL 基线、监督学习、协变量偏移 | 🟢 完成 |
| [强化学习](reinforcement-learning.md) | 试错、奖励、Sim-to-Real | 🟢 完成 |
| [扩散策略](diffusion-policy.md) | 扩散模型、多模态动作 | 🟢 完成 |
| [视觉-语言-动作模型](vision-language-action.md) | 大模型即策略、跨本体 | 🟢 完成 |
| ACT (Action Chunking Transformer) | 动作分块 / Transformer | ⚪ 待写 |
| PPO / SAC（具身 RL） | 经典 RL 算法细节 | ⚪ 待写 |
| DAgger | 在线纠错 IL | ⚪ 待写 |

> 图例：🟢 完成 · 🟡 进行中 · ⚪ 待写

---

## 文件命名

方法英文名，小写连字符：`diffusion-policy.md`、`behavioral-cloning.md`。

---

## 笔记结构建议

```markdown
# 方法名（中文 · English）

> 一句话定义。

## 核心思想
## 关键公式
## 输入 / 输出
## 代表论文（链接到 papers/）
## 优点 / 局限
## 适用场景
```
