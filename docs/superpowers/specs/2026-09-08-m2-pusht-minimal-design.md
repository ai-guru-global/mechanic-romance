# M2 设计：Diffusion Policy · Push-T 最小复现

- 日期：2026-09-08
- 状态：已批准（对话内逐节设计确认）
- 关联：[docs/roadmap.md](../../roadmap.md)（M2 · 首个可演示场景）、[场景 01 README](../../../demo/scenarios/01-扩散策略-桌面堆叠/README.md)、[gtm/README.md 可信度门禁](../../../gtm/README.md)

## 背景与目标

mechanic-romance 当前 M1 完成、M2 进行中：8 个场景均为设计文档，仓库零行 Python。本设计是仓库首个可跑代码——把 Diffusion Policy（Chi et al. 2023）在 Push-T 任务上做最小复现，对应 roadmap M2「首个场景推进到最小可跑复现」。gtm 门禁对该项引用的是 roadmap M3 定义（复现 1 个基线方法并记录结果）；本复现的产物（可跑管线 + 实验记录）预期同时满足两个定义的字面要求，里程碑勾选在实验完成后于 roadmap 按实际结果校准。

已确认的关键决策：

| 决策点 | 结论 |
| --- | --- |
| 实现路径 | 方案 A：自建精简实现 + 官方环境/数据（弃 LeRobot 全栈与官方仓库原样跑） |
| 深度 | 管线全通：环境 → 数据 → 训练 → 评估 → 指标记录，每一环可运行 |
| 验收 | 双口径：lowdim 观测版 ≥80% 成功率（硬门禁）；pixel 观测版训练+评估+记录（不设门禁） |
| 数据源 | 官方 `pusht_cchi_v7_human.zarr`（206 条人类演示）为主源；自采演示仅作环境自检工具 |
| 硬件 | 本机 Mac M4（MPS），纯仿真，无真机环节 |

非目标：真机 Franka 部署、LIBERO 多任务扩展、脚本专家作为主数据源、LeRobot/官方仓库依赖、GUI 可视化界面。

## 总体架构

```
官方数据 zarr ──► pusht_data.py ──► train.py ──► checkpoint (.pt, 不入库)
                                      │                    │
                                      ▼                    ▼
官方环境移植 ◄── pusht_env.py ◄──── eval.py ──► metrics.json + 成功/失败 episode 帧
                                                    │
                                                    ▼
                              research/experiments/2026-09-XX-diffusion-policy-pusht/
                              （曲线、指标、偏差说明）→ 回填场景 01 README 结果节
```

四模块三入口，全部单文件、可独立讲解。文件布局（对齐场景 01 README 承诺的 `scripts/{collect_demos,train,eval}.py`）：

| 文件 | 职责 | 规模预估 |
| --- | --- | --- |
| `scripts/pusht_env.py` | Push-T 环境：pymunk 物理、渲染、覆盖率 score、seed；移植自官方 MIT 代码并标注来源 | ~300 行 |
| `scripts/pusht_data.py` | zarr 读取、episode 切分、滑窗采样、归一化；首次加载校验 schema（keys/shapes） | ~150 行 |
| `scripts/pusht_model.py` | 条件 DDPM + 1D 时间卷积 UNet（FiLM 条件注入）；lowdim 用 MLP 观测编码器，pixel 用小 CNN | ~350 行 |
| `scripts/train.py` | 训练入口 `--variant {lowdim,pixel}`，EMA + checkpoint 保存 | ~200 行 |
| `scripts/eval.py` | 评估入口：50 episodes，输出 metrics.json + episode 帧（--episodes/--checkpoint 可调） | ~200 行 |
| `scripts/collect_demos.py` | 鼠标交互采集工具（环境自检 / 对照数据，非主数据源） | ~100 行 |

## 环境与动作空间

官方 Push-T 环境（论文原 demo 本体，2D 仿真）：

- **观测**
  - lowdim 口径：agentpos 5 维（推子 x/y、速度 x/y、T 块角度）
  - pixel 口径：96×96×3 RGB 顶视图 + agentpos 5 维
- **动作**：2 维——推子的目标位置，环境内部完成移动。场景 01 README 写的 7-DoF 末端是真机 Franka 口径；最小复现按论文仿真任务走 2 维，偏差表注明
- **成功判据**：episode 内 max 覆盖率 score ≥ 0.9（官方 env 提供 score 计算）。场景 01 README 的「IoU ≥ 0.9 停留 ≥ 1 s」不采用，偏差表注明
- 采样频率 10 Hz（与论文一致）

## 模型

一套骨干、两种观测编码器：

- **骨干**：1D 时间卷积 UNet，在动作块上做扩散预测。论文原配置：obs horizon 2 / action horizon 8 / pred horizon 16（10 Hz）
- **条件注入**：FiLM（scale/shift）由观测编码器输出调制 UNet 各层
- **扩散**：DDPM 100 步训练；推理 DDIM 16 步加速（对应场景 README「已知坑：推理慢」的讲解点）
- **EMA**：衰减 0.75，评估使用 EMA 权重

## 训练与评估（双口径）

| | lowdim（硬门禁） | pixel（记录不设门） |
| --- | --- | --- |
| 数据 | 官方 206 条人类演示 | 同左 |
| 训练 | 论文配置全量训练（具体 epochs/超参在实现计划定值，MPS 预计数小时） | ≤ 24 小时墙钟预算，训练到平台期即收 |
| 评估 | 50 episodes，固定 seed | 同左 |
| 通过线 | success rate ≥ 80%（论文 86%） | 如实报告，不设门 |

- 对照基线：随机策略跑一次 50 episodes（success ≈ 0），写进实验记录，证明 eval 管线本身有效
- 指标：success rate、mean/median 覆盖率、完成步数分布
- 输出：`metrics.json` + 成功/失败各 3 条 episode 帧序列（帧序列同时是 `demo/assets` 门禁项的素材候选）

## 诚实偏差表

写入实验记录，并回填场景 01 README 结果节：

| 维度 | 论文/场景 README | 本复现 |
| --- | --- | --- |
| 平台 | 2D 仿真（官方 env）| 相同（官方 env 移植） |
| 硬件 | RTX 4090 / Franka 真机 | Mac M4 MPS，纯仿真，无真机 |
| 动作 | 7-DoF 末端（真机口径） | 2 维推子目标位（仿真口径） |
| 图像分辨率 | 84×84（README 口径） | 96×96（官方数据原生） |
| 演示数 | README 写 100 | 用满官方 206 条 |
| 成功判据 | IoU ≥ 0.9 停留 1 s | max 覆盖 ≥ 0.9（官方判据） |
| 成功率 | 论文 86% | 目标 ≥ 80%；低于则如实记录并分析 |

## 测试与验证策略

冒烟级（随实现逐步验证）：

- env：固定 seed 确定性（同 seed 两次 rollout 状态一致）、100 步渲染不崩溃、score 计算合理（初始 ≈ 0，手动对齐目标 ≈ 1）
- 数据：schema 校验、episode 数 206、滑窗样本形状正确、归一化统计量有限
- 模型：CPU 上前向/反向形状正确、DDIM 采样输出形状 = 动作块形状
- 随机策略 eval：产出合法 metrics.json（success ≈ 0）

集成级：

- lowdim 全量训练 → 50 eps 评估 → 门禁判定（≥80%）
- pixel 预算内训练 → 50 eps 评估 → 记录

## 工程基建

- uv 管理，Python 3.11 固定（本机 3.14 与 torch 不兼容）；`pyproject.toml` + `uv.lock` 入库
- 依赖最小集：torch（MPS）、numpy、pymunk、zarr（锁 v2）、tqdm、pillow、matplotlib
- `checkpoints/` gitignore；权重不入库，最终权重视价值上传 HF Hub 或 GitHub release（对齐场景 README「模型权重不入库」）
- 数据目录 `data/` gitignore；`scripts/train.py --data` 指定路径，README 写明官方下载地址

## 风险与降级路径

| 风险 | 兜底 |
| --- | --- |
| MPS 精度/速度不足 | fp32 兜底；必要时缩 batch 或减 epoch（如实记录） |
| zarr v3 API 变更 | 锁 zarr v2 |
| pymunk 版本 API 差异 | 锁版本（与官方代码匹配的 6.x） |
| DDPM 不收敛 / 成功率 < 80% | 降级路径：调 seed → 缩 horizon → 缩模型；仍不过则如实记录差距与归因分析（诚实标注优先于凑数字） |

## 产物与后续

1. `scripts/` 六个文件 + `pyproject.toml`/`uv.lock` + README 运行说明更新
2. `research/experiments/` 实验记录目录（目录名按实际启动日，形如 `2026-09-12-diffusion-policy-pusht/`）：metrics、训练曲线、偏差说明
3. 场景 01 README：状态 🟡→🟢（视门禁结果）、结果节回填、运行方式节更新为实际命令
4. `demo/assets/`：至少 1 个真实素材（episode 帧/GIF），勾掉 gtm 门禁第 5 项
5. gtm 可信度门禁：勾「场景 01 达成最小可跑复现」（以 roadmap M3 定义为准）
