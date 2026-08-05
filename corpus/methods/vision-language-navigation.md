# 视觉语言导航（Vision-Language Navigation, VLN）

> 让机器人（**移动底盘**）根据**自然语言指令**，在**3D 室内环境**中自主导航到目标位置。是具身智能中「**语言 → 动作**」在**导航**任务上的完整范式，和 [VLA（视觉-语言-动作）](vision-language-action.md) 共同构成具身智能的两条主线。

## 核心思想

传统导航是「**点 A 到点 B**」的几何问题（SLAM + 路径规划）。VLN 加了一层**自然语言接口**——机器人要听懂「走到厨房的冰箱前」这样的指令，并在探索中把语言短语和视觉场景对齐。

```
图像 I_t + 语言指令 L ──► [ 策略 π ] ──► 动作 a_t (前进/转向/停止)
                                       ──► 状态 s_{t+1}
                                       ──► 文本匹配分 (评测)
```

**关键挑战**：
1. **语言接地（grounding）**：「厨房」「冰箱前」这些短语在 3D 场景中如何定位？
2. **长程记忆**：指令常跨多步（如「走过客厅，左转进入厨房，停在冰箱前」），策略必须记住历史。
3. **模糊性**：同一指令有多种合理路径。
4. **Sim-to-Real**：仿真训 → 真机部署。

## 任务形式

| 任务 | 缩写 | 描述 | 典型数据集 |
| --- | --- | --- | --- |
| **Vision-and-Dialog Navigation** | **VDN** | 带多轮对话的导航 | CVDN |
| **Room-to-Room** | **R2R** | 走完指令描述的路径 | R2R |
| **Room-for-Room** | **RxR** | 多语言 + 时间对齐指令 | RxR |
| **Remote Embodied Visual Exploration** | **REVERIE** | 指令 + 对象定位 | REVERIE |
| **Continuous Environments** | **VLN-CE** | 连续动作（不是节点跳转） | R2R-CE / RxR-CE / REVERIE-CE |
| **ObjectNav** | — | 「找到某对象」不指定位置 | MP3D / HM3D |
| **AudioNav** | — | 用声音线索导航 | SoundSpaces |

## 关键数据集

| 数据集 | 场景 | 指令 | 特点 |
| --- | --- | --- | --- |
| **R2R** | MP3D 90 楼 | 7k 英文 | 最经典，4-7 步指令 |
| **R2R-CE** | MP3D | 同 R2R | 连续动作版 |
| **REVERIE** | MP3D | 21k 指令 | 极简指令 + 对象 bbox |
| **REVERIE-CE** | MP3D | 同 REVERIE | 连续动作版 |
| **RxR** | MP3D | 26k 多语言 | 英 / 印地 / 孟加拉 / 中文 |
| **HM3D** | 1200 楼 | 1M+ 指令 | 最新 SOTA 基准（2024+） |
| **CVDN** | MP3D | 7k 对话 | 带提问 / 回答的对话导航 |
| **Touchdown** | Google Street View | 9k 指令 | 室外导航 |

## 方法族演化

VLN 方法 7 年演化大致分 5 代：

```
2018 — Seq2Seq baseline
  │
  ├── 2019 — Speaker-Follower（数据增强）
  │     ├── 2020 — EnvDrop / RCM（dropout 增强）
  │     ├── 2020 — Self-Monitor（SAM）
  │     └── 2020 — ORG / AuxRN（辅助任务）
  │
  ├── 2020 — Pre-training 范式
  │     ├── 2020 — VLN-BERT
  │     ├── 2021 — AirBERT
  │     └── 2021 — PREVALENT
  │
  ├── 2021 — Transformer 全栈化
  │     ├── 2021 — HAMT（History Aware Multimodal Transformer）★
  │     ├── 2022 — HOP / HOP+（跨模态 grounding）
  │     └── 2022 — TD-STP（拓扑图）
  │
  ├── 2022 — 拓扑图 + 预训练大模型
  │     ├── 2022 — DUET（Dual-scale graph + Transformer）★
  │     └── 2023 — GridMM / KOLT
  │
  └── 2024+ — 基础模型路线
        ├── 2024 — NaVid（视频 LLM 直接导航）
        ├── 2024 — MapGPT（LLM 决策 + 地图）
        └── 2025 — NaVILA / NOLT（视觉语言基础模型）
```

### 关键方法详解

#### 1. Seq2Seq baseline（2018）

```python
# 简化版
img_feat = CNN(obs.rgb)
text_feat = LSTM(obs.instruction)
action = MLP(concat([img_feat, text_feat, hidden_state]))
```

**问题**：无长程记忆，碰到长指令就崩。

#### 2. Speaker-Follower（2019）

**两阶段**：
1. **Speaker**：生成新指令（数据增强，从 7k → 数十万）
2. **Follower**：基于增强数据训练导航策略

**贡献**：数据增强成为 VLN 标准技巧（EnvDrop、Sub-Instruction 等都源于此）。

#### 3. HAMT（2021，ICCV 2021 Best Paper）★

**核心**：**History Aware Multimodal Transformer**

- **三个输入流**：
  - 视觉：ViT 编码当前帧
  - 文本：BERT 编码指令
  - **历史**：Past Action Transformer（PAT）编码过去 100+ 步的视觉+动作
- **跨模态注意力**：文本 token ↔ 视觉 patch ↔ 历史 token
- **动作输出**：双线性注意力 → 2-DoF 连续动作

**贡献**：用 Transformer 把「历史」作为一等公民，**长程指令成功率从 30% → 55%+**。

#### 4. DUET（2022，TPAMI）★

**核心**：**拓扑图 + Transformer 双尺度**

- 拓扑图（topological map）维护「已访问节点 + 可达节点」
- 在「局部（细粒度图像-文本对齐）」和「全局（拓扑图决策）」两个尺度上做 Transformer
- 在线构图，边走边建图

**贡献**：**结合「图搜索」和「学习表征」**，在 RxR-CE 上 SPL 提升 10%+。

#### 5. NaVid / MapGPT（2024）—— 基础模型路线

把导航当**视频问答**：
- NaVid：直接喂第一人称视频 + 文本 → 视频 LLM（如 VideoLLaMA）输出动作
- MapGPT：用 LLM 决策 + 单独建图模块

**承诺**：零样本泛化 + 自然语言解释（"我选了这条因为看起来更亮"）。

## 评估指标

| 指标 | 缩写 | 定义 | 目标值（HM3D） |
| --- | --- | --- | --- |
| **成功率** | **SR** | 终点 3 m 内 + 朝向正确占比 | ≥ 60% |
| **路径长度加权成功率** | **SPL** | SR × (最优路径/实际路径) | ≥ 45% |
| **导航误差** | **NE** | 终点与目标距离 (m) | ≤ 5 |
| **Oracle 成功率** | **OSR** | 路径上是否经过目标邻域 | ≥ 75% |
| **Oracle SPL** | **oSPL** | 假设到达即停的最优 SPL | ≥ 60% |

> 详见 [VLN-CE Leaderboard](https://eval.ai/web/challenges/challenge-page/1612/leaderboard)。

## 与 VLA 的关系

| 维度 | VLN | VLA（[RT-2 / OpenVLA](vision-language-action.md)） |
| --- | --- | --- |
| **任务** | 导航（底盘控制） | 操作（机械臂控制） |
| **动作空间** | 2-DoF（前后转） + 1 STOP | 7-DoF（末端） + 夹爪 |
| **观测** | RGB + Depth + 指令 | RGB + 指令 |
| **语言作用** | 强（整个任务围绕指令） | 强（语义指令） |
| **代表模型** | HAMT、DUET、NaVid | RT-2、OpenVLA、π₀ |
| **数据集** | R2R、HM3D | Open X-Embodiment、Bridge |

**本质同源**：都是「**图像 + 语言 → 动作**」的统一范式，只是动作空间不同。**2024+ 的趋势是统一**：
- **NaVid / NOLT** = VLM-based 导航
- **π₀** = VLM-based 操作
- **GO-1**（具身基础模型）= 操作 + 导航统一

## 开放挑战

1. **零样本泛化**：训在 MP3D，测在 HM3D（或反向），成功率掉 20-30%
2. **长程指令**：20+ 步指令仍困难（当前 7 步最稳）
3. **多轮对话**：VDN 类任务的成功率比 R2R 低 20%
4. **真机部署**：LoCoBot / Stretch 上的 sim-to-real 仍有 10-15% 差距
5. **数据规模**：相比 VLA（97 万 episode），VLN 数据集小一个数量级

## 适用场景

- ✅ 家用服务机器人（家庭环境导航）
- ✅ 仓储 AGV（语音指令调度）
- ✅ 辅助机器人（视障导盲 / 老人陪护）
- ❌ 高速运动场景（需 100+ Hz 控制，VLN 主流 10 Hz）
- ❌ 室外长距离（VLN 主流室内）

## 相关语料

- [`vision-language-action.md`](vision-language-action.md) — VLA 是 VLN 的姊妹范式
- [`imitation-learning.md`](imitation-learning.md) — VLN 早期方法都是 BC
- [`reinforcement-learning.md`](reinforcement-learning.md) — RCM 等用 RL 微调
- [`../simulation/platforms/habitat.md`](../simulation/platforms/habitat.md) — VLN 主流仿真器
- [`../concepts/foundations/embodied-ai.md`](../concepts/foundations/embodied-ai.md) — 具身智能总论
- [`../concepts/mdp/observation-space.md`](../concepts/mdp/observation-space.md) / [`../concepts/mdp/action-space.md`](../concepts/mdp/action-space.md) — 观测 / 动作
- [demo 07-视觉语言导航-VLN-CE](../../demo/scenarios/07-视觉语言导航-VLN-CE/README.md) — 直接落地场景

## 参考论文

- **Anderson 2018** — R2R + Seq2Seq baseline
- **Fried 2018** — Speaker-Follower
- **Majumdar 2020** — VDAN（VDN 基准）
- **Qi 2020** — EnvDrop（数据增强）
- **Majumdar 2021** — AirBERT（预训练）
- **Chen 2021** — HAMT ★
- **Chen 2022** — DUET ★
- **Krantz 2020** — Beyond the Nav-Graph（R2R-CE 起点）
- **Liu 2024** — NaVid（视频 LLM）
- **Team 2025** — HM3D 1M+ 指令（最新基准）
