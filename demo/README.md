# Demo 场景 · Demo

> 把研究结论落成**可演示的具身场景**。每个场景用统一模板定义任务、观测、动作、评估、硬件，保证可复现、可比较。

---

## 子目录

| 子目录 | 内容 | 入口 |
| --- | --- | --- |
| `scenarios/` | **具身场景定义**（核心产出），含 [_template.md](scenarios/_template.md) | [`scenarios/README.md`](scenarios/README.md) |
| `prompts/` | 提示词 / 交互脚本 / 对话流程 | [`prompts/README.md`](prompts/README.md) |
| `assets/` | 场景素材（图、视频、URDF、网格等） | [`assets/README.md`](assets/README.md) |

---

## 场景索引

> 每新增一个场景，在此登记一行。

| 编号 | 场景 | 状态 | 入口 |
| --- | --- | --- | --- |
| `01` | 扩散策略·桌面堆叠（Diffusion Policy） | 🟡 设计中 | [scenarios/01-扩散策略-桌面堆叠/](scenarios/01-扩散策略-桌面堆叠/README.md) |
| `02` | ACT·双手叠衣（Action Chunking Transformer） | 🟡 设计中 | [scenarios/02-ACT-双手叠衣/](scenarios/02-ACT-双手叠衣/README.md) |
| `03` | RT-2·自然语言指令分拣 | 🟡 设计中 | [scenarios/03-RT2-自然语言指令分拣/](scenarios/03-RT2-自然语言指令分拣/README.md) |
| `04` | OpenVLA·7B 开源 VLA 复现 | 🟡 设计中 | [scenarios/04-OpenVLA-7B-开源VLA/](scenarios/04-OpenVLA-7B-开源VLA/README.md) |
| `05` | Figure 02 + Helix·端到端人形服务 | 🟡 设计中 | [scenarios/05-Figure02-端到端人形/](scenarios/05-Figure02-端到端人形/README.md) |
| `06` | 强化学习·Franka 抓杯（Isaac Lab + PPO） | 🟡 设计中 | [scenarios/06-强化学习-Franka抓杯/](scenarios/06-强化学习-Franka抓杯/README.md) |
| `07` | 视觉语言导航·VLN-CE（Habitat + HAMT） | 🟡 设计中 | [scenarios/07-视觉语言导航-VLN-CE/](scenarios/07-视觉语言导航-VLN-CE/README.md) |
| `08` | 灵巧手·五指旋转立方体（Allegro / LEAP Hand） | 🟡 设计中 | [scenarios/08-灵巧手-五指旋转立方体/](scenarios/08-灵巧手-五指旋转立方体/README.md) |

> 配套文档：[`_demo-types-overview.md`](_demo-types-overview.md) — 我能做的 demo 类型全景 + 具身智能外的扩展方向。

### 8 大类覆盖度

| 类别 | 场景 | 状态 |
| --- | --- | --- |
| 🦾 操作（Manipulation）单臂 | 01 · 05（基座）· 06 | ✅ |
| 🦾 操作（Manipulation）双手 | 02 | ✅ |
| 🧭 导航（Navigation） | 07 | ✅ |
| 🧠 基础模型 / VLA | 03 · 04 · 05 | ✅ |
| 🔁 学习范式 - IL | 01 · 02 · 04 | ✅ |
| 🔁 学习范式 - RL | 06 | ✅ |
| 🔁 学习范式 - Sim-to-Real | 02（布料）· 04（LIBERO）· 06（域随机化）· 08（灵巧手） | ✅ |
| 🖐 灵巧手 | 08 | ✅ |

> 8 大类全部覆盖。后续要扩的话，往「跨模态 / 软体操作 / 长程任务规划」等纵深方向走。

---

## 工作流

1. 复制 [`scenarios/_template.md`](scenarios/_template.md) → `scenarios/NN-场景名/README.md`。
2. 填写全部字段，关联 [`corpus/`](../corpus/) 语料与 [`research/`](../research/) 实验。
3. 在本文件「场景索引」登记。
4. 跑通后更新状态为 🟢 可演示。
