# Figure 02 + Helix·端到端人形服务 · Figure 02 + Helix End-to-End Humanoid Service

> **一句话描述**：在 Figure 02 商用 humanoid 上跑 Helix（System 1 + System 2 双系统 VLA），完成"开冰箱拿可乐递给用户"的端到端家庭服务任务。

| 字段 | 内容 |
| --- | --- |
| 场景编号 | `05` |
| 创建日期 | 2026-08-04 |
| 状态 | 🟡 设计中 |
| 负责人 | 待认领 |
| 关联课题 | [roadmap M1 · 语料沉淀](../../docs/roadmap.md) |

---

## 1. 任务目标（Task）

[Figure 02](../../corpus/hardware/humanoids/figure-02.md) 站在厨房区域，接收语音指令 **"给我从冰箱拿一瓶可乐"**，自主完成：
1. 导航到冰箱
2. 打开冰箱门（双指夹爪拉把手）
3. 识别可乐瓶
4. 抓取并取出
5. 转身递给用户（可人脸识别确认）
6. 等待用户接过后关冰箱门

- **关键创新**：**Helix 双系统 VLA**（System 1 快速反应 VLA + System 2 慢速规划 LLM）。
- **成功判据**：可乐完整递到用户手中，无掉落、无碰撞。
- **本场景定位**：⚠️ **商用平台，无开源代码**。重在**任务编排复盘 + 思路学习**。

---

## 2. 观测空间（Observation）

| 模态 | 来源 | 说明 |
| --- | --- | --- |
| RGB 图像 | 双 RGB 头盔相机 | 1280×720×3 × 2 路 |
| 深度 | 结构光 | 可选 |
| 触觉 | 五指灵巧手 × 2 | 6-DoF 力 + 触觉阵列 |
| 语音 | 麦克风阵列 | 经 Whisper 转文本 |
| 本体感知 | 全关节编码器 | 35-DoF（双臂 14 + 双腿 12 + 躯干 4 + 头 3 + 手 10） |
| 文本指令 | 语音识别后 | 经 Helix System 2 编码 |

---

## 3. 动作空间（Action）

- **控制模式**：**全身 35-DoF**（全关节角 + 灵巧手 + 底盘）
- **动作维度**：35
- **控制频率**：200 Hz（关节控制）/ 50 Hz（策略推理）
- **动作表示**：**全身相对增量**

---

## 4. 评估指标（Evaluation）

| 指标 | 定义 | 目标值 |
| --- | --- | --- |
| 端到端成功率 | 全流程一次跑通的占比 | ≥ 80%（公司宣传） |
| 任务分解步数 | 6 个子任务的总耗时 | ≤ 90 s |
| 摔倒率 | 任务中摔倒次数 | 0 |
| 安全性 | 撞人 / 夹伤 / 异常力矩 | 0 |
| 语音识别 WER | 嘈杂环境下的词错率 | ≤ 5% |

- **评估 episodes 数**：商用场景下 50+ 次
- **变体**：
  - 换物体（雪碧 / 橙汁）
  - 换用户（不同身高 / 语言）
  - 异常（可乐被挪动 / 冰箱门被挡住）

---

## 5. 硬件与平台（Hardware）

| 项目 | 规格 |
| --- | --- |
| 机器人 | **[Figure 02](../../corpus/hardware/humanoids/figure-02.md)**（双足 humanoid，35-DoF） |
| 灵巧手 | 第四代 5 指灵巧手（16-DoF × 2） |
| 头部 | 6 摄像头（双目 RGB + 红外 + 结构光） |
| 计算 | **机载 GPU**（Figure 自研 SoC，疑似 NVIDIA Orin + 自定义 FPGA） |
| 网络 | WiFi 6 / 5G，与云端 VLA 推理低延迟 |
| 仿真器 | 内部仿真栈（未公开） |

---

## 6. 方法（Method）

- 策略：**Helix = System 1 (VLA, 80 Hz 推理) + System 2 (LLM 规划, 7-9 Hz 推理)**
- System 2 负责：「听懂指令 → 拆解为子任务 → 决定策略」
- System 1 负责：「图像 → 关节动作」的快速反应
- 训练数据：Figure 内部数据 + 互联网 VLM 知识
- 论文笔记：[industry/figure-ai](../../corpus/industry/humanoid-overseas/figure-ai.md) + Helix 技术博客（2025-02 发布）

**为什么是它**：
1. **首个公开的"端到端 VLA + 人形"商用方案**（2025-02 Helix 发布）
2. 验证了 **"机器人基础模型"** 路线的可行性
3. 视觉冲击力强（家庭场景拿可乐），是讲解"具身智能时代来了"的最好 demo

---

## 7. 运行方式（How to Run）

```bash
# ⚠️ 无开源代码。本场景为"复盘 + 思路学习"。
# 1. 看 Helix 官方视频与技术博客
#    https://www.figure.ai/blog/helix

# 2. 在 [industry/figure-ai](../../corpus/industry/humanoid-overseas/figure-ai.md) 笔记中整理思路

# 3. 对照开源方案
#    - 思路：参考 [03-RT2](../03-RT2-自然语言指令分拣/README.md) + [04-OpenVLA-7B](../04-OpenVLA-7B-开源VLA/README.md)
#    - 全身控制：参考 H1 / Optimus 的开源运控栈
#    - 灵巧手：参考 [ALOHA + ACT](../02-ACT-双手叠衣/README.md) 改装

# 4. 在仿真中尝试"双系统 VLA"思路
#    - System 1: 训练 [Diffusion Policy](../../corpus/methods/diffusion-policy.md) 做单步反应
#    - System 2: 用 LLM 拆解指令为子任务
#    - 用 [Genesis](../../corpus/simulation/platforms/genesis.md) 跑家庭场景
```

- 关键脚本：无（商用）
- 学习材料：技术博客 + Figure 02 视频集

---

## 8. 结果与记录（Results）

> 跑通后补充。计划在 [research/experiments/](../../research/experiments/) 下建 `2026-XX-XX-figure02-helix-analysis/` 复盘报告。

- 首次成功演示：商用层面已达成（Figure 官方视频）
- 个人复现：N/A（硬件门槛）

---

## 9. 相关语料（References）

- 硬件：[figure-02](../../corpus/hardware/humanoids/figure-02.md)
- 行业：[figure-ai](../../corpus/industry/humanoid-overseas/figure-ai.md) · [tesla-optimus](../../corpus/industry/humanoid-overseas/tesla-optimus.md) · [physical-intelligence](../../corpus/industry/foundation-models/physical-intelligence.md)
- 概念：[embodied-ai](../../corpus/concepts/foundations/embodied-ai.md) · [embodiment](../../corpus/concepts/foundations/embodiment.md) · [degrees-of-freedom](../../corpus/concepts/foundations/degrees-of-freedom.md)
- 方法对照：[vision-language-action](../../corpus/methods/vision-language-action.md)
- 评测：[real-world-eval](../../corpus/benchmarks/vla-eval/real-world-eval.md)

---

## 10. 备注

- **为什么选作 `05`**：
  - 是**当前最热的商用 humanoid demo**（2025 Q1）
  - 体现 "**VLA + 全身控制 + 双系统**" 的前沿设计
  - 演讲 / 教学 / 客户演示 都好用
- **学习路径**：
  1. 先看 Helix 官方视频
  2. 再看 [figure-ai](../../corpus/industry/humanoid-overseas/figure-ai.md) 笔记
  3. 对照 [03-RT2](../03-RT2-自然语言指令分拣/README.md) 学 VLA 原理
  4. 对照 [04-OpenVLA-7B](../04-OpenVLA-7B-开源VLA/README.md) 学开源复现
  5. 未来：等 Helix 出技术细节，写到 `corpus/methods/`
- **商业对照**：
  - **Figure 02 + Helix**：双系统 VLA 路线
  - **1X Neo + Redwood**：纯端到端 VLA 路线（无 System 2 拆解）
  - **Tesla Optimus + FSD**：自动驾驶技术迁移路线
  - **Unitree H1 + 智元 A2**：硬件优先 + 学术合作路线
- **个人复现建议**：用 [Isaac Sim](../../corpus/simulation/platforms/nvidia-isaac.md) + [Genesis](../../corpus/simulation/platforms/genesis.md) 在仿真里跑简化版（无灵巧手 → 平行夹爪）。
