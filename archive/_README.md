# 归档说明（Archive）

> 本目录存放本仓库**历史项目**，已停止维护，仅供留档参考。
> 仓库当前主线项目为「**机械浪漫**」——具身智能课题研究语料库与 Demo 场景，见仓库根目录 [`README.md`](../README.md)。

---

## 内容

| 原项目 | 说明 |
| --- | --- |
| **Piroom / 派屋**（appsforcoder.com） | 面向 Coder 的「物联网（Web of Things）」乐园，基于 CodeIgniter（PHP）的 Web 应用，含 Android / iOS 端占位目录。 |

## 目录结构

```
archive/
├── README.md        # 原 Piroom 项目 README
├── 中文版说明        # 原中文说明
├── web/             # CodeIgniter Web 应用（381 个文件）
├── doc/             # 原项目文档（占位）
└── mobile/          # 移动端占位（android / ios）
```

## 归档信息

- **归档日期**：2026-08-02
- **归档方式**：`git mv` 整体迁移，git 历史完整保留（可用 `git log --follow <file>` 追溯）。
- **状态**：不再维护，不接受针对本目录的更新。

如需查阅原始提交历史，可执行：

```bash
git log --oneline -- archive/README.md      # 追溯某一文件
git log --oneline --before="2026-08-02"     # 查看归档前的提交
```
