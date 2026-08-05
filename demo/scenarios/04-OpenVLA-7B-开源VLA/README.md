# OpenVLA·7B 开源 VLA 复现 · OpenVLA 7B Open-Source VLA

> **一句话描述**：用 OpenVLA-7B（Prismatic VLM + 机器人动作头）做 BridgeData V2 上的微调，**单 GPU 就能跑**的 RT-2 开源替代方案。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `04` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M3 · 实验复现](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

把预训练 OpenVLA-7B 在 **BridgeData V2** 桌面倒水任务上做 LoRA 微调，验证「VLA 模型可在消费级 GPU 上微调并部署到真机」。这是 [RT-2](../03-RT2-自然语言指令分拣/README.md) 思路的**可复现版本**。

- **输入指令示例**：
  - 「pour the water from the cup into the bowl」
  - 「把杯子里的水倒进碗里」
- **成功判据**：水从杯中倒入目标容器 ≥ 80% 体积，无溢出。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| RGB 图像 | 第三人称相机 | 224×224×3（Prismatic VLM 输入尺寸） |
| 本体感知 | 关节编码器 | 7-DoF（Franka / WidowX） |
| 文本指令 | 文字输入 | 自由文本 |

---

## 3. 动作空间（Action）

- **控制模式**：**离散动作 token**（与 RT-2 一致，256 bin × 7 DoF）
- **动作维度**：7 DoF
- **控制频率**：3-5 Hz
- **动作表示**：**绝对位姿 token**

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| Bridge 成功率 | 倒水 / 推物 / 抓物 7 任务平均 | ≥ 50%（论文 78.3%） |
| 零样本泛化 | 新指令 / 新物体的成功率 | ≥ 40% |
| 推理延迟 | 单步决策 | ≤ 200 ms（A100） |
| LoRA 微调成本 | 显存 + 时间 | ≤ 24 GB + ≤ 6 h（A100） |

- **评估 episodes 数**：50 次 × 7 任务 = 350
- **变体**：
  - 7 类 Bridge 任务（含倒水、推物、擦桌、开关抽屉等）
  - 零样本 vs 微调对比

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机械臂 | [Franka Panda](../../corpus/hardware/arms/franka-panda.md) 或 WidowX 250（论文方案） |
| 末端执行器 | 平行夹爪 |
| 相机 | 单目 RGB（Intel RealSense） |
| 计算 | **1× A100 80GB**（LoRA 微调）/ 4090 24GB（推理可） |
| 仿真器 | [LIBERO](../../corpus/benchmarks/manipulation/libero.md)（前置仿真验证） |
| 数据集 | [Open X-Embodiment](../../corpus/datasets/open-x-embodiment.md) + BridgeData V2 |

---

## 6. 方法（Method）

- 策略：**OpenVLA-7B = Prismatic VLM (SigLIP + Llama 2 7B) + 动作头**
- 预训练：970K 演示 episode（Open X-Embodiment）
- 微调：LoRA（rank 32） 在 BridgeData V2 上
- 论文笔记：[2024-openvla](../../corpus/papers/vla/2024-openvla.md)

**为什么是它**：
1. **开源 + 权重可下载**（HF Hub，7B/13B 两档）
2. **单卡可微调**（LoRA），对个人研究者最友好
3. **复现 RT-2 思路**但门槛降 10×

---

## 7. 运行方式（How to Run）

```bash
# 1. 克隆 OpenVLA 官方仓库
git clone https://github.com/openvla/openvla.git
cd openvla && pip install -e .

# 2. 下载预训练权重（HF Hub）
python -c "from huggingface_hub import snapshot_download; snapshot_download('openvla/openvla-7b')"

# 3. 下载 BridgeData V2
python download_bridge.py --out data/bridge_v2/

# 4. LoRA 微调
python finetune.py \
  --base_vla_path openvla/openvla-7b \
  --data_root_dir data/bridge_v2 \
  --lora_rank 32 --lora_alpha 16 \
  --batch_size 8 --grad_accumulation 2 \
  --learning_rate 5e-4 --num_steps 50_000

# 5. 真机部署（或 LIBERO 仿真评估）
python deploy.py --checkpoint ckpt/openvla-bridge-ft --robot franka

# 6. 在 LIBERO 上评估
python experiments/robot/libero/run_libero_eval.py \
  --pretrained_checkpoint ckpt/openvla-bridge-ft
```

- 关键脚本：`openvla/{finetune,deploy}.py` + `experiments/robot/libero/run_libero_eval.py`
- 模型权重：HF Hub `openvla/openvla-7b` + `openvla/openvla-7b-bridge`（微调版）

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-openvla-bridge/` 实验记录。

- 首次成功演示：待定
- 复现指标：BridgeV2 78.3% / SimplerEnv 47.7%（参考论文）

---

## 9. 相关语料（References）

- 论文：[2024-openvla](../../corpus/papers/vla/2024-openvla.md)
- 方法：[vision-language-action](../../corpus/methods/vision-language-action.md)
- 数据集：[open-x-embodiment](../../corpus/datasets/open-x-embodiment.md)
- 评测：[libero](../../corpus/benchmarks/manipulation/libero.md)
- 概念：[embodied-ai](../../corpus/concepts/foundations/embodied-ai.md) · [finetuning](../../corpus/concepts/training/finetuning.md)

---

## 10. 备注

- **为什么选作 `04`**：是 [RT-2](../03-RT2-自然语言指令分拣/README.md) 的**开源复现版**，可执行性 100%，对应 M3 复现里程碑。
- **对照参考**：
  - **vs RT-2（55B / TPU）**：参数 8× 小，单卡可跑
  - **vs Octo（27M）**：OpenVLA 更通用、零样本更强
  - **vs π0（3B）**：OpenVLA 公开权重最完整
- **可扩展方向**：
  - 在 [Behavior-1k](../../corpus/benchmarks/manipulation/behavior-1k.md) 上测
  - 换 Llama 2 → Llama 3 backbone
- **已知坑**：
  - 显存峰值 22-24 GB（A100），4090 推理可行但微调需开启 gradient checkpointing
  - LIBERO 仿真与真机差距约 15-20%
