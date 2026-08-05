# RT-2·自然语言指令分拣 · RT-2 Language-Conditioned Sorting

> **一句话描述**：用 RT-2（Vision-Language-Action 模型）让机器人听懂「把苹果放到红碗里」这类自然语言指令，并完成桌面分拣任务。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `03` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M1 · 语料沉淀](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

机器人在 6 类常见桌面物体（苹果、香蕉、杯子、勺子、积木、书）面前，根据**自然语言指令**把它们分拣到指定容器（红碗 / 蓝盘 / 绿盒）。任务是验证 **VLA（Vision-Language-Action）模型**对**新指令、新物体、新场景**的零样本泛化能力——核心是「语言→动作」的桥梁。

- **输入指令示例**：
  - 「把苹果放到红碗里」
  - 「把红色物体放进蓝盘」
  - 「Pick up the cup and put it in the green box」
- **成功判据**：物体 1 s 内到达指定容器且未掉落。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| RGB 图像 | 第三人称相机 | 320×320×3，3 Hz（RT-2 推理慢） |
| 文本指令 | 文字输入 | 经 PaLI-X（55B）语言编码 |
| 本体感知 | 关节编码器 | 7-DoF |

> RT-2 创新点是把**动作 token** 直接当语言 token 处理，模型无需修改主体架构。

---

## 3. 动作空间（Action）

- **控制模式**：**离散动作 token**（256 个 bin × 6 DoF + 1 夹爪 = 1792 token）
- **动作维度**：6 DoF + 1 夹爪
- **控制频率**：3 Hz（受限于 VLM 推理速度）
- **动作表示**：**绝对位姿 token**（与 RT-1 一致）

> 这是 RT-2 与 [Diffusion Policy](../../corpus/methods/diffusion-policy.md) 的核心区别：离散 vs 连续、单步 vs 多步。

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| 指令成功率 | 完成指定指令的占比 | ≥ 60%（论文 62%） |
| 零样本泛化 | 训练未见指令的成功率 | ≥ 50% |
| 符号理解 | 「the smallest object」这类指令成功率 | ≥ 40% |
| 推理延迟 | 单步决策时间 | ≤ 500 ms（生产级 1 s） |

- **评估 episodes 数**：200+ 次（论文 7000+ 真实评估）
- **变体**：
  - **新物体**：训练集外的玩具 / 工具
  - **新指令**：「red → blue」组合泛化
  - **新背景**：换桌面颜色 / 光照

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机械臂 | Google Everyday Robot（EDR）臂 / 等效 7-DoF 臂 |
| 末端执行器 | 平行夹爪 |
| 相机 | 单目 RGB |
| 计算 | **TPU v4**（训练，32+ pod）/ **TPU v5e**（推理）或 H100 |
| 仿真器 | **无原生仿真**（RT-2 主要真机） |
| 数据来源 | [Open X-Embodiment](../../corpus/datasets/open-x-embodiment.md) RT-1 数据集（13 万 episode） |

> ⚠️ **本场景重在"复现思路"而非真跑 RT-2**。完整 RT-2 训练需 14+ 天 × 32 TPU pod。复现建议用 [OpenVLA-7B](../04-OpenVLA-7B-开源VLA/README.md) 替代。

---

## 6. 方法（Method）

- 策略：**RT-2 = PaLI-X (55B VLM) + 动作头**，联合微调
- 预训练：PaLI-X 已在视觉-语言数据上预训练
- 训练数据：RT-1 演示数据 + 互联网视觉-语言数据
- 论文笔记：[2023-google-rt2](../../corpus/papers/vla/2023-google-rt2.md)

**为什么是它**：VLA 范式的**开山之作**（2023 年 7 月），首次把机器人动作当作"语言"处理，泛化能力跃迁。

---

## 7. 运行方式（How to Run）

```bash
# ⚠️ RT-2 完整训练在个人硬件上不可行（无 TPU pod 资源）
# 本场景主要做"概念复现 + 推理调用"

# 方案 A · 推理 Google Cloud 托管模型（若有权限）
python rt2_inference.py --image obs.png --instruction "pick up the apple" --bucket gs://rt2-ckpt/

# 方案 B · 用 OpenVLA 复现（推荐）
# 见 04-OpenVLA-7B-开源VLA/README.md

# 方案 C · 用 RT-2 论文公开 checkpoint 推理
git clone https://github.com/google-research/robotics_transformer.git
python inference.py --model rt2_55b --ckpt gs://rt2-weights
```

- 关键脚本：仅推理
- 模型权重：**仅授权下载**（Google 申请）

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-rt2-concepts/` 笔记。

- 首次成功演示：待定
- 复现指标：定性看「新指令泛化」即可，不强求复现 62%

---

## 9. 相关语料（References）

- 论文：[2023-google-rt2](../../corpus/papers/vla/2023-google-rt2.md)
- 方法：[vision-language-action](../../corpus/methods/vision-language-action.md)
- 数据集：[open-x-embodiment](../../corpus/datasets/open-x-embodiment.md)
- 评测：[real-world-eval](../../corpus/benchmarks/vla-eval/real-world-eval.md)
- 概念：[embodied-ai](../../corpus/concepts/foundations/embodied-ai.md) · [embodiment](../../corpus/concepts/foundations/embodiment.md)

---

## 10. 备注

- **为什么选作 `03`**：VLA 范式的**原点**，必读论文 + 必知 demo。
- **可执行版本**：直接看 [04-OpenVLA-7B-开源VLA](../04-OpenVLA-7B-开源VLA/README.md)，那是 RT-2 的开源复现版。
- **历史地位**：
  - RT-1（2022）→ 行为克隆 + 7-DoF token
  - **RT-2（2023）→ VLM + 动作 token，开 VLA 时代**
  - RT-X / RT-H（2024）→ 跨机器人数据混合
  - OpenVLA（2024）→ 开源 7B 复现
  - π0 / π0.5（2024-2025）→ 通用机器人基础模型
- **学习建议**：先精读论文 [2023-google-rt2](../../corpus/papers/vla/2023-google-rt2.md) 笔记，再看 [OpenVLA](../04-OpenVLA-7B-开源VLA/README.md) 的开源复现。
