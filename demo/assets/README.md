# 场景素材 · Assets

存放 Demo 场景的**图片、视频、URDF / 网格、配置**等素材。

---

## 组织方式

- **通用素材**：放本目录，按类型分子目录（见下）。
- **场景专属素材**：放对应 `scenarios/NN-场景名/assets/`。

```
assets/
├── images/      # 场景示意图、截图
├── videos/      # 演示视频（小文件；大视频存外部并在此留链接）
├── models/      # URDF / MJCF / 网格（.obj/.stl）
└── configs/     # 场景配置（物体位姿、灯光等）
```

---

## 注意

- ⚠️ **大文件不入库**：视频、高精度网格等见 [`.gitignore`](../../.gitignore)；本目录只放轻量素材，大文件注明外部存储链接（网盘 / 仓库 release）。
- 引用素材时用相对路径，例如 `![任务示意](images/task-overview.png)`。
