# 可供性（Affordance）

> 物体或环境**「能被怎样使用」的可能性**——由生态心理学家 J.J. Gibson 提出：可供性既非物体属性，也非感知者主观臆想，而是「动物的能力」与「环境结构」之间的**关系**。一把椅子「提供」坐的可能，一只把手「提供」拉的姿势。

> 最后更新：2026-08

---

## 1. 直觉

传统计算机视觉把任务定义为「**识别**」（这张图里是什么物体？杯子、椅子、刀）。但机器人要和物体**交互**，仅知道类名远远不够——它得知道：杯子的把手在哪里能抓、刀刃不能摸、椅子面能承重。这些「**物体在结构上邀请的动作可能性**」就是可供性（affordance）。

```
   物体识别 (recognition)         可供性 (affordance)
   "这是一个马克杯"               "这一侧的环形结构 → 可捏提"
   "这是一把锤子"                 "这个圆柱 → 可握，那头 → 可敲"
```

一句话：**识别回答「是什么」，可供性回答「能干什么」**。后者才是机器人操作真正需要的语义。

---

## 2. 形式化

可供性可形式化为一个**映射**：给定场景观测 $o$（图像/点云）和候选动作类型 $\mathcal{A}$（抓、推、拉、倒……），输出每个像素 / 每个空间位置上「该动作可行」的概率或得分：

$$
A: \mathcal{O} \times \mathcal{A} \to [0,1], \qquad A(o, a)(\mathbf{x}) = \text{在像素/3D点 } \mathbf{x} \text{ 处执行动作 } a \text{ 的成功概率}
$$

- 输入：观测 $o$（如 RGB-D 图像）、候选动作类别 $a$（如「抓」）。
- 输出：与 $o$ 同尺寸的 **affordance map**——每个空间位置一个分数，标出「哪里适合做这个动作」。

策略随后只需在 affordance map 的峰值处执行动作，相当于把「**动作生成分解为两步：先找在哪做（where），再决定怎么做（how）**」。

---

## 3. Affordance Map：连接感知与动作的桥梁

Affordance map 是具身智能里 affordance 最常见的工程化身：

| 形态 | 含义 | 典型用途 |
| --- | --- | --- |
| **2D 像素图** | 每个像素一个 affordance 分数 | 抓取：在图像上标出「可抓像素」，再投影到 3D 抓取位姿 |
| **3D 点云图** | 每个点一个分数 | 工具使用：标出刀刃/把手/功能部位 |
| **热力图 / 概率分布** | 动作参数上的分布 | 倒水：在杯子口上方画出倾倒目标分布 |

为什么是「桥梁」：感知模块输出 affordance map（**稠密、空间化、任务相关**），动作模块在 map 上采样/优化得到具体动作。这种解耦让**感知和动作可以分别学习**，比端到端黑箱更可解释、更样本高效。

---

## 4. 与抓取 / 操作的关系

Affordance 是现代机器人操作的核心抽象之一：

- **抓取（grasping）**：本质就是「预测 6-DoF 抓取位姿的 affordance」。代表工作 GraspNet / Dex-Net 都可看作「学一个抓取 affordance 模型」——输入点云/深度图，输出可达抓取分布。
- **工具使用（tool use）**：锤子的把手 vs 锤头 affordance 不同；学习 affordance 让机器人能迁移到没见过的工具（只要结构相似）。
- **物体部件（part-based）**：把物体分解为功能部件（刀刃、刀柄、杯口），每个部件对应一组 affordance——这是 PartNet-Mobility / ANETC 等 benchmark 的思路。

> 反例：纯端到端 VLA（见 [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)）不显式预测 affordance map，直接从像素到动作。Affordance 派认为显式中间表示更鲁棒、更可迁移；VLA 派认为大模型已隐式包含 affordance 知识。两者正在融合。

---

## 5. 学习 Affordance 的主要方法

### 5.1 监督学习（标注驱动）

用人工或仿真标注「每个像素/点的 affordance 类别」，训练分割/回归网络。

- 数据集：**AffordNet**、**PAD**（Part Affordance Dataset）、EPIC-Kitchens（动作-物体对）。
- 损失：逐像素交叉熵 / 回归 MSE。

### 5.2 自监督 / 交互式学习

让机器人在仿真或真机上**试**：执行候选动作，用成功与否作为 affordance 标签。

- 在仿真里无限试错收集 (位置, 动作, 成功) 三元组，训练 affordance 预测器。
- 与强化学习天然结合：affordance map 可作为策略的稠密引导信号。

### 5.3 大模型驱动（LLM/VLM affordance）

近年主流：用预训练视觉-语言模型「理解」物体的功能语义，再 grounding 到空间位置。

- **CLIP-based affordance**：用 CLIP 计算「可抓区域」文本与图像区域的相似度，得到 affordance heat map。
- **VLM affordance grounding**（如 VoxPoser、Code-as-Policies）：让 LLM 输出「物体哪个部位可交互」的代码，再生成 3D affordance 体素图引导策略。
- 这条线把 Gibson 的关系性定义（动物能力 × 环境结构）具象化为「**大模型提供功能常识，视觉模型提供空间定位**」。

---

## 6. 为什么 Affordance 是具身智能的核心抽象

| 维度 | Affordance 的作用 |
| --- | --- |
| **泛化** | 见过马克杯，迁移到从未见过的运动水壶——只要「环形把手」affordance 相似 |
| **可解释性** | affordance map 可视化，便于调试策略失败的原因（是没看见？还是 affordance 错？） |
| **样本效率** | 比端到端像素→动作更易学，因为中间表示压缩了动作空间 |
| **语言对齐** | 自然语言指令天然包含 affordance（「把杯子**提**起来」「**推**抽屉」），是 VLA 的语义接口 |
| **跨本体** | affordance 描述环境结构，与具体机械臂形态解耦，便于迁移 |

---

## 7. 主要挑战与坑

- **标注昂贵**：逐像素 affordance 标注人工成本高，自监督方法依赖仿真，又回到 sim-to-real（见 [`sim-to-real.md`](sim-to-real.md)）。
- **任务依赖**：同一物体在不同任务下 affordance 不同（杯子可抓、可倒、可放）——需条件化于任务/语言指令。条件化的形式：$A(o, a, \ell)$，其中 $\ell$ 是语言指令。
- **几何 vs 功能**：纯几何 affordance（如「任何细长物都可握」）常与功能语义脱节——笔和筷子都能握，但功能部位完全不同，需结合语义先验。
- **动态环境**：物体被移动后 affordance map 要重算，需高效在线推理（实时性要求常逼着用轻量网络）。
- **本体依赖**：Gibson 定义里 affordance 是「动物能力 × 环境结构」的关系——同一把手对机械夹爪（捏）和人形灵巧手（包握）affordance 不同。跨本体迁移时需重新评估。
- **不确定性**：现实接触有随机性，affordance 应输出分布而非点估计，否则策略过于自信导致硬碰撞。

## 7.1. 评估指标

| 指标 | 含义 |
| --- | --- |
| **Affordance mIoU** | 预测的 affordance 区域与标注的重叠度（分割式评估） |
| **Grasp success rate** | 在 affordance 峰值处执行动作的成功率（闭环评估，最重要） |
| **跨物体泛化率** | 在未见物体上的成功率，衡量 affordance 的抽象能力 |
| **Top-k 命中率** | affordance 排序前 k 个位置中含正确动作的比例 |

---

## 8. 相关概念互链

- [`embodied-ai.md`](embodied-ai.md)——具身智能闭环里，affordance 是感知到动作的中间表示。
- [`../mdp/observation-space.md`](../mdp/observation-space.md)——affordance map 是一种任务相关的观测变换。
- [`../mdp/action-space.md`](../mdp/action-space.md)——affordance 把巨大连续动作空间压缩到「可行子集」。
- [`../perception/visual-representation.md`](../perception/visual-representation.md)——视觉编码器是 affordance 预测的前端。
- [`sim-to-real.md`](sim-to-real.md)——自监督 affordance 学习受仿真-真机 gap 影响。

## 9. 代表方法 / 论文

- 抓取 affordance → [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)（VLA 的 grounding 一节）
- 操作方法总览 → [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)
- 经典论文与数据集：
  - Gibson, *The Ecological Approach to Visual Perception* (1979)——affordance 概念起源。
  - Myers et al., *Affordance Detection of Tool Parts from Geometric Features* (2015)——早期 affordance 分割。
  - Nguyen et al., *Object Affordances with Deep Learning* (2017, ICRA)。
  - PartNet-Mobility / PAD 数据集——部件级 affordance 标注。
  - VoxPoser (2023, arXiv:2207.05173) / Code-as-Policies (2023)——VLM 驱动 affordance grounding。

---

## 10. 参考链接

- Affordance 概念综述：*Affordance Detection and Application: A Survey* (2021)
- PartNet-Mobility 数据集：https://sapien.ucsd.edu/
- Gibson 原著中译与解读见相关教材
