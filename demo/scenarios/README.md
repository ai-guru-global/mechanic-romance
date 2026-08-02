# 场景定义 · Scenarios

本目录是 Demo 的**核心产出区**：每个具身场景一个子目录，内含基于 [_template.md](_template.md) 填写的 `README.md`。

---

## 编写指南

1. **新建目录**：命名 `NN-中文场景名/`（`NN` 为两位序号，如 `01-桌面抓取/`）。
2. **复制模板**：把 [`_template.md`](_template.md) 复制为场景目录内的 `README.md`。
3. **逐项填写**：任务、观测、动作、评估、硬件、方法、运行方式……不留空字段。
4. **挂素材**：图片/视频放本场景目录内的 `assets/` 或顶层 [`../assets/`](../assets/)。
5. **关联**：用相对链接指向 [`../../corpus/`](../../corpus/) 语料与 [`../../research/`](../../research/) 实验。
6. **登记**：在 [`../README.md`](../README.md) 的「场景索引」加一行。

---

## 目录结构示例

```
scenarios/
├── _template.md
├── 01-桌面抓取/
│   ├── README.md          # 场景定义（基于模板）
│   └── assets/            # 本场景专属素材（可选）
└── 02-抽屉开合/
    └── README.md
```

---

## 场景清单

> 见 [`../README.md`](../README.md#场景索引)。
