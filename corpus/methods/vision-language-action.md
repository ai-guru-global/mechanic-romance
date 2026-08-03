# 视觉-语言-动作模型（Vision-Language-Action, VLA）

> 用**大模型架构**直接把图像 + 语言指令映射为机器人动作。本质上是「给多模态大模型装上机械臂」——让一个统一的 Transformer 同时做感知、推理与控制。

## 核心思想

传统机器人策略是「小模型 + 手工特征」。VLA 的赌注是：**大模型的通用理解与推理能力，能直接迁移到机器人控制**。

```
图像 I + 语言指令 ℓ ──► [ 大模型（VLM） ] ──► 动作 token ──► 解码为连续动作 a
```

关键洞察：把动作离散化为 token，机器人控制就变成了「另一种形式的文本生成」——和语言模型生成下一个词，在数学上没有本质区别。

## 与 Diffusion Policy 的对比

两条当前主流路线：

| 维度 | VLA（RT-2, OpenVLA） | Diffusion Policy |
| --- | --- | --- |
| 出发点 | 大模型泛化 + 推理 | 多模态动作建模 |
| 动作表示 | **离散 token** | 连续序列（扩散采样） |
| 指令理解 | ✅ 强（继承 LLM 能力） | 弱（需额外条件） |
| 推理速度 | 慢（大模型） | 中（去噪步数） |
| 数据需求 | 海量（跨本体预训练） | 中等（百级演示） |
| 擅长 | 长程、语义、泛化 | 精细、高频操作 |

## 动作的「语言化」

VLA 的工程核心是**把连续动作变成 token**，塞进大模型的词表：

1. 将每维动作（末端位置、姿态、夹爪）分桶离散化。
2. 每个桶对应一个 token（如 `<act_0_47>`）。
3. 大模型像「生成句子」一样生成一串动作 token。
4. 解码回连续值执行。

```
"pick up the red block" → <a_pos_x_3> <a_pos_y_7> <a_pos_z_2> <a_rot...> <gripper_open>
```

## 关键工作脉络

| 模型 | 年份 | 贡献 |
| --- | --- | --- |
| **RT-1** | 2022 | 谷歌首个机器人 Transformer，验证 BC + 大数据可行 |
| **RT-2** | 2023 | 把 VLM（PaLI/PaLM-E）与机器人数据共训，**涌现**语义泛化 |
| **RT-X / Open-X-Embodiment** | 2023 | 跨本体、跨机构的大规模数据集，证明跨本体泛化 |
| **OpenVLA** | 2024 | 开源 7B 通用 VLA，可微调 |
| **Octo** | 2024 | 开源、多本体、可条件化的 Transformer 策略 |
| **π₀ (Pi-Zero)** | 2024 | 用 flow matching 替代 token 解码，支持高频连续动作 |

## 为什么 VLA 重要

1. **语义泛化**：能理解「把水果放进篮子」这类抽象指令，无需为每类物体单独训练。
2. **跨本体**：理论上一个模型适配多种机器人（OXE 数据集的承诺）。
3. **推理能力**：链式思考可嵌入控制（"先拿 A，再拿 B"）。
4. **scaling**：模型越大、数据越多，能力越强——继承大模型的 scaling law。

## 开放挑战

- ❌ **频率太低**：大模型推理 1–5 Hz，难做高频力控。
- ❌ **精细操作差**：插钥匙、拧螺丝等需高频反馈的任务不如 Diffusion Policy。
- ❌ **数据饥渴**：高质量机器人演示仍稀缺。
- ❌ **动作离散化损失精度**（flow matching / diffusion head 在尝试解决）。

## 相关概念

- [`diffusion-policy.md`](diffusion-policy.md)（另一条主流路线）
- [`imitation-learning.md`](imitation-learning.md)（VLA 本质是 BC 训练）
- [`../concepts/foundations/embodied-ai.md`](../concepts/foundations/embodied-ai.md)
- [`../datasets/open-x-embodiment.md`](../datasets/open-x-embodiment.md)（VLA 的数据基石）
