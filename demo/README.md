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
| — | （暂无，从 `scenarios/_template.md` 开始） | — | — |

---

## 工作流

1. 复制 [`scenarios/_template.md`](scenarios/_template.md) → `scenarios/NN-场景名/README.md`。
2. 填写全部字段，关联 [`corpus/`](../corpus/) 语料与 [`research/`](../research/) 实验。
3. 在本文件「场景索引」登记。
4. 跑通后更新状态为 🟢 可演示。
