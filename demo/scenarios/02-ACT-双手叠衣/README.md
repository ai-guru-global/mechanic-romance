# ACT·双手叠衣 · Action Chunking Transformer for Bimanual Laundry Folding

> **一句话描述**：在 ALOHA 双手平台上，让双臂协同学习「毛巾对折」任务，复现 Zhao et al. 2023 提出的 ACT（Action Chunking Transformer）。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `02` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M2 · 首个可演示场景](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

双臂机器人（ALOHA 平台）协同完成**毛巾对折**任务：把摊在桌面上的方巾沿中线对折一次，最终两块布完全重叠。**关键观察是「动作分块（action chunking）」**——一次预测未来 K 步动作，让策略避免抖动、加速训练收敛。

- **输入指令示例**：无显式指令（任务固定），通过演示传达"对折"语义。
- **成功判据**：折叠后两半布的角点 IoU ≥ 0.85，停留 ≥ 1 s。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| RGB 图像 | 双腕相机 + 顶视相机（共 3 路） | 640×480×3，30 Hz |
| 本体感知 | 双臂关节编码器 | 14-DoF（每臂 6 + 夹爪 1） |
| 末端位姿 | 前向运动学 | 14 维 |

> 3 路相机对**小物体的精细操作**至关重要。

---

## 3. 动作空间（Action）

- **控制模式**：双臂**关节角**（14-DoF） + 双夹爪（2 维）
- **动作维度**：16
- **控制频率**：50 Hz（高频率）
- **动作表示**：**绝对关节位置**（论文方案），但预测时使用**相对动作分块**

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| 成功率 | 50 次试验中"两半完全重叠"占比 | ≥ 70%（论文 90% 区间） |
| 完成时间 | 从起始到完成秒数 | ≤ 30 s |
| 抖动度 | 相邻动作差分 L2 范数 | 越低越好 |
| 安全性 | 关节力矩 / 碰撞 | 0 |

- **评估 episodes 数**：50 次
- **变体**：测对**不同材质**（棉、丝、纱布）、**不同起始位姿**的鲁棒性。

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机械臂 | ALOHA 双臂（每臂 6-DoF）或 [Franka Panda](../../corpus/hardware/arms/franka-panda.md) × 2 |
| 末端执行器 | ALOHA 平行夹爪（低成本 3D 打印） |
| 相机 | 2× 腕部 Logitech C270 + 1× 顶部 |
| 计算 | 1× RTX 4090（训练 + 推理） |
| 仿真器 | [MuJoCo](../../corpus/simulation/platforms/mujoco.md)（ALOHA 官方 sim）+ ALOHA Station |
| 真机 | 自主搭建（参考 ALOHA 开源 BOM，约 2 万美元） |

---

## 6. 方法（Method）

- 策略：**ACT = CVAE + Transformer Decoder**，核心是「**动作分块 + 时序集成（temporal ensembling）**」
- 预训练：无（任务级训练）
- 训练数据：50 条人工遥操作演示（论文 200 条；最低 50 条已可收敛）
- 论文笔记：[2023-zhao-act](../../corpus/papers/imitation-learning/2023-zhao-act.md)
- 方法笔记：[act](../../corpus/methods/act.md)

**为什么是它**：ACT 解决了模仿学习的两大痛点——**抖动**（chatter）和**复合误差**（compounding error），是入门双手操作最实用的方法。

---

## 7. 运行方式（How to Run）

```bash
# 1. 安装 ALOHA 官方环境
git clone https://github.com/tonyzhaozh/aloha.git
cd aloha && pip install -e .

# 2. 收集遥操作演示
python record_episodes.py --task fold_towel --num_episodes 50

# 3. 训练 ACT
python imitate_episodes.py --task fold_towel --num_epochs 5000 --ckpt_dir ckpt/

# 4. 评估 + 可视化
python eval_imitate.py --ckpt ckpt/policy_best.ckpt --save_video
```

- 关键脚本：`aloha/{record_episodes,imitate_episodes,eval_imitate}.py`
- 模型权重：HF Hub `aloha-act-fold-towel`（本仓库 release 链接）

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-act-bimanual-fold/` 实验记录。

- 首次成功演示：待定
- 复现指标：动作分块 K=10 时抖动能降低 50%（参考论文）

---

## 9. 相关语料（References）

- 论文：[2023-zhao-act](../../corpus/papers/imitation-learning/2023-zhao-act.md)
- 方法：[act](../../corpus/methods/act.md) · [imitation-learning](../../corpus/methods/imitation-learning.md) · [behavioral-cloning](../../corpus/methods/behavioral-cloning.md) · [dagger](../../corpus/methods/dagger.md)
- 仿真：[mujoco](../../corpus/simulation/platforms/mujoco.md)
- 概念：[sim-to-real](../../corpus/concepts/foundations/sim-to-real.md) · [embodied-ai](../../corpus/concepts/foundations/embodied-ai.md)

---

## 10. 备注

- **为什么选作 `02`**：是**双手协同 + 软体操作**的最佳入门 demo，对应 M2 场景扩展。
- **可扩展方向**：叠衣 → 叠纸盒 → 系鞋带（更难）；Mobile ALOHA 增加移动底盘。
- **已知坑**：
  - 遥操作演示采集非常耗时（单条 30 s × 50 条 ≈ 25 分钟）
  - 仿真 → 真机差距主要在**布料物理**，需用 NVIDIA [Isaac Sim](../../corpus/simulation/platforms/nvidia-isaac.md) 高级布料求解
- **对照参考**：[Diffusion Policy](../../corpus/methods/diffusion-policy.md) 在 ALOHA 上也有报告，可做 A/B。
