# 论文笔记 · Papers

> 语料库的**核心**：具身智能重要论文的结构化笔记。一篇论文一个文件，按主题归类到第二层目录。

---

## 主题分类

| 子目录 | 收录 | 状态 |
| --- | --- | --- |
| [`vla/`](vla/) | 视觉-语言-动作模型（RT-1/2/X、OpenVLA、Octo、π₀） | 🟡 充实中 |
| [`diffusion-policy/`](diffusion-policy/) | 扩散/生成式策略（DP、DP3、EquiBot） | 🟡 充实中 |
| [`imitation-learning/`](imitation-learning/) | 模仿学习方法（ACT、DAgger、GAIL、ALOHA） | ⚪ 待建 |
| `reinforcement-learning/` | RL 方法（PPO/SAC 应用、Decision Transformer） | ⚪ 待建 |
| `world-models/` | 世界模型（Dreamer 系列、UniSim、Genie） | ⚪ 待建 |
| `foundation-models/` | 基础模型（PaLM-E、RT-X、LVM） | ⚪ 待建 |
| `manipulation/` | 操作任务专项（抓取、装配、灵巧操作） | ⚪ 待建 |
| `navigation/` | 导航与探索（视觉导航、ObjectNav、探索策略） | ⚪ 待建 |
| `locomotion/` | 运动控制（四足/人形行走、强化学习 locomotion） | ⚪ 待建 |
| `sim-to-real/` | Sim-to-Real 迁移（域随机化、域适应） | ⚪ 待建 |
| `datasets-papers/` | 数据集论文（OXE/RT-X、DROID、BridgeData） | ⚪ 待建 |
| `scaling/` | 数据/模型 scaling 与涌现（机器人 scaling law） | ⚪ 待建 |

## 命名

`年份-首作者-关键词.md`，例：`2023-brohan-rt2.md`、`2023-chi-diffusion-policy.md`。

## 笔记结构（统一模板）

见 [`docs/conventions.md`](../../docs/conventions.md#4-笔记结构模板)。每篇必含：
- 元信息（作者/机构/会议/年份/链接）
- BibTeX
- 核心问题 / 方法 / 实验 / 个人评价

## 论文索引（按主题）

### VLA 系列
- 🟢 [`vla/2023-brohan-rt2.md`](vla/2023-brohan-rt2.md) — RT-2：VLM + 机器人，涌现泛化
- 🟢 [`vla/2024-kim-openvla.md`](vla/2024-kim-openvla.md) — OpenVLA：开源 7B VLA
- ⚪ `vla/2022-brohan-rt1.md` — RT-1：首个机器人 Transformer
- ⚪ `vla/2023-octo.md` — Octo：开源通用策略
- ⚪ `vla/2024-pi-zero.md` — π₀：flow matching 动作头

### 扩散策略
- 🟢 [`diffusion-policy/2023-chi-diffusion-policy.md`](diffusion-policy/2023-chi-diffusion-policy.md) — Diffusion Policy 原文
- ⚪ `diffusion-policy/2024-ze-dp3.md` — 3D Diffusion Policy

### 模仿学习
- 🟢 [`imitation-learning/2023-zhao-act.md`](imitation-learning/2023-zhao-act.md) — ACT / ALOHA：双臂遥操 + 动作分块
- ⚪ `imitation-learning/` — DAgger / GAIL 待补

> 图例：🟢 完成 · 🟡 进行中 · ⚪ 待写。每完成一篇把 ⚪ 改为 🟢 并加链接。
