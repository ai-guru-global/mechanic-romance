# 基准测试 · Benchmarks

> **衡量具身智能体能力的标尺**。没有统一基准，方法之间无法比较。一个领域的成熟度，往往取决于其基准的成熟度。

---

## 子分类

| 子目录 | 收录 | 状态 |
| --- | --- | --- |
| [`manipulation/`](manipulation/) | 操作基准（CALVIN / LIBERO / RLBench / BEHAVIOR-1K） | 🟢 已收录核心 |
| [`navigation/`](navigation/) | 导航基准（HM3D / PointNav / ObjectNav） | ⚪ 待建 |
| [`locomotion/`](locomotion/) | 运动控制基准（IsaacGymLoco / Legged Gym） | ⚪ 待建 |
| [`vla-eval/`](vla-eval/) | VLA 评测（真机评测协议 / RT-2 / OpenVLA） | 🟢 已收录核心 |
| [`generalist/`](generalist/) | 通用基准（RT-X / BEHAVIOR / Habitat 多任务） | ⚪ 待建 |

## 基准的核心要素

一个合格基准需明确：
1. **任务集**：具体做什么（任务定义、变体数、难度分级）。
2. **观测/动作接口**：智能体看什么、能做什么（标准化 API）。
3. **评估指标**：成功率？完成步数？泛化到新物体/指令？
4. **基线**：已公开方法的分数，作为参考线。
5. **可复现**：代码/场景公开，他人能跑。

## 操作类基准现状（重灾区）

操作领域**缺乏公认统一基准**，是当前最大痛点：
- CALVIN：长程多任务，但仿真偏简单。
- LIBERO：5 套任务集，覆盖泛化维度，2024 流行。
- RLBench：19 个任务，经典但老。
- 真机评测：各家自定场景，几乎不可比较。

> VLA 时代更棘手：大模型推理慢、真机评测贵，社区急需标准化真机基准。

## 为什么基准重要

- **方法可比**：同基准下 A 方法 70% vs B 方法 85%，才有意义。
- **推动进步**：明确「下一个该攻克的难题」。
- **求职/发论文**：SOTA（state-of-the-art）是硬通货。

## 与其他模块的关系

- 依赖 [`simulation/`](../simulation/) 提供场景。
- 用 [`methods/`](../methods/) 的方法刷分。
- [`industry/`](../industry/) 的公司在这些基准上比拼。
