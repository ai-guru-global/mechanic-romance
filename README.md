# 机械浪漫 · Mechanic Romance

> **具身智能（Embodied AI）课题研究语料库与 Demo 场景**
>
> 「机械浪漫」——把冰冷的机械、数据与算法，组织成可被理解、可被复现、可被讨论的研究语言。

本仓库是一个以 **Markdown** 为载体、面向具身智能研究的 **语料库（corpus）** 与 **Demo 场景**工作台。目标是为课题研究沉淀结构化的论文笔记、方法梳理、数据集说明、实验记录，并把研究结论落成可演示的具身场景。

---

## 这是什么 / 不是什么

- ✅ **是**：一个研究语料库——论文、方法、数据集的结构化笔记与索引。
- ✅ **是**：一个 Demo 场景工作台——用统一的场景模板定义具身任务。
- ✅ **是**：一个研究过程记录——选题、阅读、实验的留痕。
- ❌ **不是**：一个训练框架或仿真器本身（Demo 场景是「定义」与「脚本」，可对接外部仿真/硬件）。

---

## 目录导航

| 目录 | 用途 | 入口 |
| --- | --- | --- |
| `corpus/` | **语料库**：论文 / 方法 / 数据集 / 综述 | [`corpus/README.md`](corpus/README.md) |
| `research/` | **研究产出**：选题 / 阅读笔记 / 实验记录 | [`research/README.md`](research/README.md) |
| `demo/` | **Demo 场景**：具身任务定义与交互脚本 | [`demo/README.md`](demo/README.md) |
| `docs/` | 项目级文档：路线图、写作规范 | [`docs/roadmap.md`](docs/roadmap.md) |
| `archive/` | 历史项目归档（旧 Piroom 项目，已停止维护） | [`archive/_README.md`](archive/_README.md) |

---

## 快速上手

1. **了解课题方向** → 读 [`docs/roadmap.md`](docs/roadmap.md)。
2. **查语料** → 进 [`corpus/`](corpus/)，按 `papers / methods / datasets / surveys` 分类浏览。
3. **写一个新场景** → 复制 [`demo/scenarios/_template.md`](demo/scenarios/_template.md)，按字段填写。
4. **贡献内容** → 先读 [`CONTRIBUTING.md`](CONTRIBUTING.md) 了解命名与引用约定。

---

## 写作约定（摘要）

- **语言**：中文为主，关键术语保留英文原文（如 VLA、Diffusion Policy）。
- **命名**：论文笔记用 `年份-首作者-关键词.md`（如 `2024-rt-x-open-dataset.md`）。
- **引用**：每篇笔记顶部放 BibTeX，正文用 `[@key]` 行内引用。
- 详细规范见 [`docs/conventions.md`](docs/conventions.md)。

---

## 许可与状态

- 本仓库为研究语料库，**内容版权归原作者所有**；笔记部分遵循项目许可（待定）。
- 当前状态：**🟡 骨架搭建中**（2026-08-02 初始化）。
