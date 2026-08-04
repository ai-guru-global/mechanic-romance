# 视觉表征（Visual Representation）

> 把**原始图像**（RGB / RGB-D）编码成策略网络可用的**紧凑特征向量**的模块与过程。是具身智能策略的「眼睛」，也是当前机器人泛化能力的**主要瓶颈**。

> 最后更新：2026-08

---

## 1. 直觉

策略网络（无论是模仿学习还是 RL）不能直接吃百万像素——既慢又易过拟合。需要一个**视觉编码器** $e_\phi$ 把图像压成几百维特征：

```
   原始图像 o (H×W×3)  ──► [ 视觉编码器 e_φ ] ──►  特征 z ∈ ℝ^d  ──►  策略头 ──►  动作 a
```

这个 $z$ 必须保留与任务相关的信息（物体位置、位姿、几何、语言指令相关对象），同时扔掉无关细节（背景纹理、光照）。**视觉表征的好坏，直接决定策略上限**——编码器没看到的，策略再强也用不上。

---

## 2. 为什么视觉是具身智能的主要瓶颈

| 原因 | 说明 |
| --- | --- |
| **信息量巨大** | 一帧 224×224 RGB ≈ 15 万维，比本体感知（关节角，几十维）大 3 个数量级 |
| **任务相关信号稀疏** | 抓取任务里，真正有用的是物体位姿，但它淹没在背景里 |
| **泛化难题** | 新物体、新光照、新视角、遮挡——视觉上的微小变化常导致策略失效 |
| **数据成本** | 视觉策略需要海量多样化图像，真机采集贵，仿真渲染又有 sim-to-real gap |
| **多视角融合** | 第三人称 + 手腕相机 + 深度，如何对齐与融合是工程难题 |

对比：低维状态（关节角、力矩）的策略很容易学；**视觉表征这一步做好了，下游策略学习往往水到渠成**。这就是为什么 RT-1 / RT-2 / OpenVLA 都在视觉编码器上下大功夫。

---

## 3. 主干架构：CNN vs ViT

### 3.1 CNN（ResNet 系，经典主力）

- **代表**：ResNet-18 / ResNet-50，早期 RT-1、BC 标配。
- **优点**：卷积的平移等变性对空间推理友好；计算高效；小数据集不易过拟合。
- **缺点**：感受野有限，长程依赖弱；全局聚合靠池化，丢空间细节。

### 3.2 ViT（Vision Transformer，新主流）

- **代表**：ViT-Base / ViT-Large，RT-2、OpenVLA、$\pi_0$ 等用 ViT 做视觉端。
- **优点**：自注意力捕获全局关系；与语言 Transformer 同构，便于多模态融合；scaling 友好。
- **缺点**：数据饥渴（小数据易过拟合）；平移等变性弱（需大量增广弥补）。

> 趋势：**大模型时代 ViT 占优**——因为 (1) 与 LLM 同构易对齐；(2) 预训练 ViT 编码器（CLIP/DINOv2）质量已超过 ResNet。

---

## 4. 预训练编码器：CLIP / DINOv2 在机器人上的应用

现代具身智能几乎**不从零训视觉编码器**——站在大规模预训练肩膀上：

| 编码器 | 预训练方式 | 提供什么 | 机器人里的角色 |
| --- | --- | --- | --- |
| **CLIP** | 图文对比学习 | 语言对齐的视觉特征 | VLA 主力：语言指令能直接「找到」图像里对应区域 |
| **DINOv2** | 自监督蒸馏 | 强语义分割、部件一致性 | 操作任务：物体部件 / 位姿的鲁棒表征 |
| **R3M** | 人类视频自监督 | 适合机器人操作的「奖励/表征」 | Ego4d/人类视频预训练，迁移到机器人 |
| **VC-1 / VC-2** | 多任务视觉预训练 | 通用具身表征 | 跨任务/本体迁移的视觉底座 |

**为什么预训练重要**：机器人数据（OXE ≈ 百万 episode）相比互联网图文/视频（十亿级）仍是「小数据」。预训练编码器把互联网级视觉先验注入策略，是泛化的关键。

---

## 5. 冻结 vs 端到端训练

拿到预训练编码器后，是否更新它的权重？这是核心设计决策：

| 策略 | 做法 | 优点 | 缺点 | 适用 |
| --- | --- | --- | --- | --- |
| **冻结（frozen）** | 编码器权重不动，只训策略头 | 样本高效、不破坏预训练表征、显存小 | 表征不能适配任务特异需求 | 小数据、快速原型、LoRA 式 VLA |
| **端到端微调** | 编码器 + 策略头一起训 | 表征完全适配任务，性能上限高 | 数据需求大、易过拟合、显存大、**易灾难性遗忘** | 大数据（OXE 级）、追求 SOTA |
| **部分解冻 / LoRA** | 只解冻后几层或用低秩适配 | 折中：轻度适配 + 保留先验 | 需调超参 | 当前 VLA 微调主流 |

> 经验法则：**数据少 → 冻结；数据多 → 微调；不确定 → 先冻结跑 baseline，再解冻对比**。与 [`../training/finetuning.md`](../training/finetuning.md) 中的 LoRA/Adapter 思路一致。

---

## 6. 多视角融合：第三人称 + 手腕相机

真实机器人常配多相机，需把不同视角的特征融合：

| 视角 | 提供什么 | 典型配置 |
| --- | --- | --- |
| **第三人称（third-person / scene cam）** | 全局场景、物体相对机器人位姿 | 固定在外部或头顶 |
| **手腕相机（wrist / eye-in-hand）** | 末端精细视角、接触前特写 | 装在机械臂末端 |
| **深度（depth）** | 几何信息、抗光照变化 | RGB-D 相机 |

融合方式：

- **早期融合**：多视角图像拼成一张大图 / 多通道，送同一编码器。
- **晚期融合**：每个视角独立编码，特征 concat 或 attention 融合（RT-X、OpenVLA 的做法）。
- **3D 融合**：把多视角特征投影到统一 3D 坐标（见下节点云）。

> 工程坑：手腕相机视角随动作剧烈变化，策略易过拟合到特定视角；常用**视角增广**（随机裁剪/颜色抖动）缓解。

---

## 7. 点云表征（3D-aware Representation）

当任务强依赖 3D 几何（插孔装配、堆叠），2D 图像不够，需用**点云 / 体素**：

| 方法 | 思路 | 代表 |
| --- | --- | --- |
| **PointNet / PointNet++** | 直接在无序点集上做对称函数提取特征 | 经典 3D 表征 |
| **Voxel / 三维卷积** | 把点云栅格化成体素网格做 3D CNN | voxel-CNN 抓取 |
| **Transformer on points** | 点 token + 自注意力 | Point Transformer |
| **NeRF / 3D Gaussian** | 隐式 3D 表征，可渲染任意视角 | 场景重建式策略 |

**3D 表征的优势**：对视角变化天然不变（旋转/平移等变），抓取位姿预测更准；**代价**：点云采集需深度相机（噪声大）、计算更贵、数据规模小。当前趋势是「**2D 预训练 + 3D grounding**」混合（如 VoxPoser：2D VLM 给语义，点云给几何）。

---

## 8. 形式化：从图像到动作的特征流

完整数据流：

$$
z^{(v)} = e_\phi(o^{(v)}), \quad z^{(\ell)} = g_\psi(L), \quad z = \text{fuse}\big(z^{(v)}, z^{(\ell)}, z^{(\text{prop})}\big), \quad a_t = \pi_\theta(z_t)
$$

其中 $o^{(v)}$ 是图像（可多视角），$L$ 是语言指令，$z^{(\text{prop})}$ 是本体感知。视觉编码器 $e_\phi$ 是这条流水线的第一站，也是最容易成为瓶颈的一站。

---

## 9. 主要挑战与前沿

- **数据效率**：如何在千条级真机演示上学好视觉表征 → 少样本微调（见 [`../training/finetuning.md`](../training/finetuning.md)）。
- **泛化**：新物体 / 新背景 / 新光照——预训练 + 测试时增广。
- **多模态对齐**：视觉特征如何与语言指令对齐 → CLIP 式对比学习。
- **时序建模**：单帧不够，需多帧堆叠或 RNN/Transformer 时序编码（diffusion policy 的做法）。
- **可解释性**：特征到底编码了什么 → attention 可视化、affordance map（见 [`../foundations/affordance.md`](../foundations/affordance.md)）。

---

## 10. 相关概念互链

- [`../foundations/embodied-ai.md`](../foundations/embodied-ai.md)——视觉是具身智能的主要感知模态。
- [`../mdp/observation-space.md`](../mdp/observation-space.md)——视觉特征是观测空间的主要组成。
- [`../foundations/affordance.md`](../foundations/affordance.md)——affordance map 是任务相关的视觉表征。
- [`../foundations/world-model.md`](../foundations/world-model.md)——视觉编码器是世界模型的前端。
- [`../foundations/sim-to-real.md`](../foundations/sim-to-real.md)——渲染 gap 是视觉策略 sim-to-real 的痛。
- [`../training/finetuning.md`](../training/finetuning.md)——冻结 vs 微调编码器的决策。

## 11. 代表方法 / 论文

- VLA 主干 → [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)
- 扩散策略（视觉特征进扩散头）→ [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)
- 模仿学习（视觉特征进 BC）→ [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)
- 代表论文：
  - Radford et al., *CLIP* (2021, arXiv:2103.00020)
  - Oquab et al., *DINOv2: Learning Robust Visual Features without Supervision* (2023, arXiv:2304.07193)
  - Nair et al., *R3M: A Universal Visual Representation for Robot Manipulation* (2022, arXiv:2203.12601)
  - Majumdar et al., *Where are we in the search for an Artificial Visual Cortex for Embodied Intelligence? (VC-1)* (2023, arXiv:2303.18240)
  - RT-2 / OpenVLA 论文 → [`../../papers/vla/`](../../papers/vla/)

---

## 12. 参考链接

- 方法目录：[`../../methods/`](../../methods/)
- 数据集（视觉策略训练数据）：[`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md)
- 综述：*A Survey on Visual Representations for Robot Manipulation* (2024)
