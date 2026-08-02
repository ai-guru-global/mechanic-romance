# 贡献指南 · Contributing

感谢你为「机械浪漫」语料库添砖加瓦。本指南约定如何组织内容，保证语料可检索、可复现、可讨论。

---

## 1. 总则

- **语言**：中文为主；专有名词、模型名、数据集名保留英文原文（如 `Vision-Language-Action (VLA)`、`Open-X-Embodiment`）。
- **格式**：统一 GitHub Flavored Markdown；数学公式用 `$$...$$`；图放 `assets/` 并用相对路径引用。
- **一个文件讲清一件事**：一篇论文一个笔记，一个场景一个文件。

---

## 2. 命名规范

| 类型 | 规则 | 示例 |
| --- | --- | --- |
| 论文笔记 | `年份-首作者-关键词.md` | `2024-brohan-open-x-embodiment.md` |
| 方法笔记 | `方法名.md`（小写连字符） | `diffusion-policy.md` |
| 数据集说明 | `数据集名.md` | `droid.md` |
| 实验记录 | `YYYY-MM-DD-关键词.md` | `2026-08-05-dp-ablation.md` |
| 场景 | `NN-中文场景名/`（两位序号） | `01-桌面物体抓取/` |

> 中文文件名可用，但**目录/脚本名优先用英文+连字符**以避免跨平台路径问题。

---

## 3. 引用与出处

- 每篇论文笔记**顶部**附 BibTeX 条目，置于代码块中：
  ```bibtex
  @article{rt22023,
    title  = {RT-2: Vision-Language-Action Models},
    author = {Brohan, Anthony and others},
    year   = {2023}
  }
  ```
- 正文行内引用统一用 `[@key]` 形式（如 `[@rt22023]`）。
- 引用网页/博客需附 URL 与访问日期。

---

## 4. 各目录贡献要点

- **`corpus/papers/`**：先建主题子目录（如 `manipulation/`、`navigation/`），再放笔记。
- **`corpus/methods/`**：讲清「是什么 / 适用场景 / 关键公式 / 代表工作」。
- **`research/experiments/`**：记录**可复现**信息——数据、超参、环境、结果、结论。
- **`demo/scenarios/`**：**必须**基于 [`_template.md`](demo/scenarios/_template.md) 起草，保证字段齐全。

---

## 5. 提交流程

1. 新建分支：`git checkout -b feat/<简述>`。
2. 提交信息用中文动词开头，如「新增 RT-2 论文笔记」「完善抓取场景评估指标」。
3. PR 描述说明：新增/修改了什么、归属哪个目录、是否有外部引用。

---

## 6. 不纳入版本控制的内容

见 [`.gitignore`](.gitignore)，主要包括：模型权重（`*.pt/*.safetensors`）、原始数据集（`data/raw/`）、Python 虚拟环境、编辑器配置。大文件请用外部存储并在笔记中注明链接。
