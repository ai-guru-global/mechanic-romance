# 感知层 · Concepts / Perception

> 具身智能策略的**输入端**：如何把图像、点云、本体感知、语言指令编码成策略可用的特征。这里收录感知相关的**概念名词**（是什么），具体编码方法/网络架构见 [`../../methods/`](../../methods/)。

---

## 收录范围

- 视觉表征（编码器、预训练、多视角、点云）
- 本体感知（关节角、力矩、IMU）
- 语言条件（指令理解、多模态对齐）
- 表征学习范式（自监督、对比学习、冻结 vs 微调）

> 与 [`../mdp/observation-space.md`](../mdp/observation-space.md) 的区别：那里讲「观测空间是什么」（MDP 要素定义），这里讲「如何把观测变成好特征」（表征技术）。

---

## 文件索引

| 概念 | 关键词 | 状态 |
| --- | --- | --- |
| [视觉表征](visual-representation.md) | 编码器、CLIP/DINOv2、冻结 vs 微调、多视角、点云 | 🟢 完成 |
| 本体感知（proprioception） | 关节角、力矩、IMU | ⚪ 待写 |
| 语言条件（language-conditioned） | 指令理解、CLIP 对齐、VLM grounding | ⚪ 待写 |
| 点云表征（point-cloud） | PointNet、3D-aware、几何先验 | ⚪ 待写 |

> 图例：🟢 完成 · 🟡 进行中 · ⚪ 待写

---

## 命名规范

概念英文名，小写连字符：`visual-representation.md`、`language-conditioned.md`。

## 笔记结构

```markdown
# 概念名（中文 · English）

> 一句话定义。

## 直觉
## 形式化（如适用）
## 为什么重要 / 在具身智能的角色
## 代表方法 / 工作（互链 methods/ 与 papers/）
## 相关概念互链
## 参考链接
```

---

## 返回

- [概念层总目录](../README.md)
- [机械浪漫语料库首页](../../README.md)
