# 综述：通用机器人的基础模型（Foundation Models in Robotics）

> Firoozi et al., 2023/2024 这篇综述（*Foundation Models for Generalist Robots*）是**具身智能 + 基础模型**交叉领域早期最系统的全景图之一，把"如何用 LLM/VLM/世界模型/扩散模型给机器人做基座"这件事分门别类、列举代表工作、点明关键挑战。本笔记按综述框架提炼，并加上对本课题（manipulation 学习）的具体启发。

## 元信息

- **标题**：Foundation Models for Generalist Robots: A Survey
- **作者**：Dawei Firoozi, Stephen Tian, Minho Heo, Xue Lin, Huazhe Xu, Roya Firoozi, Yi Ma, Tobias Kreiman, Lerrel Pinto, others
- **机构**：UC Berkeley、Boston Dynamics AI Institute、MIT、清华大学、Google DeepMind 等多机构合作
- **会议 / 年份**：预印本 2023-12（v1）；多次更新至 2024；目标投稿 IJRR / CoRL Review
- **论文链接**：https://arxiv.org/abs/2312.07843
- **项目主页**：https://generalerobots.github.io/
- **关键词**：foundation model、generalist robot、VLA、world model、LfD、survey

## BibTeX

```bibtex
@article{firoozi2024foundation,
  title   = {Foundation Models for Generalist Robots: A Survey},
  author  = {Firoozi, Dawei and Tian, Stephen and Heo, Minho and others and
             Xu, Huazhe and Kreiman, Tobias and Pinto, Lerrel and Ma, Yi},
  journal = {arXiv preprint arXiv:2312.07843},
  year    = {2024}
}
```

## 一句话概括

把"基础模型怎么帮机器人变通用"分成**感知（VLM）、规划（LLM 推理）、动作（VLA / 学习策略）、世界模型、数据与采集（LfD）**五大支柱，系统列举 200+ 代表工作，并指出**数据稀缺、跨本体泛化、长程任务评估**三大瓶颈——是入门具身智能 + FM 交叉领域的最佳路线图。

---

## 覆盖范围

综述把机器人基础模型生态组织成一个分层框架：

```
                         通用机器人 (Generalist Robot)
                                    │
        ┌──────────┬──────────────┼──────────────┬──────────────┐
        ▼          ▼              ▼              ▼              ▼
   ① 感知基座   ② 规划/推理     ③ 动作基座    ④ 世界模型      ⑤ 数据与采集
   (VLM)        (LLM-as-planner) (VLA / Policy) (World Model)  (LfD / Teleop)
   CLIP, SigLIP  SayCan, Code    RT-2, OpenVLA  Dreamer, UniSim  ALOHA, OXE
   DINOv2, SAM   as Policies    Diffusion-Policy, ACT
```

### 1. 感知基础模型（Perception Foundation Models）

- **目的**：提供通用、可迁移的视觉/几何表征，让下游策略不必从零学视觉。
- **代表**：CLIP / SigLIP（图文对齐）、DINOv2（自监督几何）、SAM（通用分割）、Depth Anything。
- **在机器人里的用法**：作为冻结的视觉编码器，给 VLA / Diffusion Policy / ACT 当 backbone。本综述重点讨论**冻结 vs 微调**的权衡。
- **趋势**：从单编码器（CLIP）→ 多编码器融合（SigLIP + DINOv2，OpenVLA 的设计）。

### 2. 规划与推理（LLM as Planner）

- **目的**：把 LLM 的常识、长程规划、符号推理用到任务分解上。
- **范式**：
  - **LLM + 技能库**：SayCan、Code as Policies——LLM 出高层计划，调用预定义技能。
  - **LLM + 视觉提示**：VoxPoser、ProgPrompt——LLM 输出代码/3D 提示驱动低层。
- **局限**：依赖预定义技能库，本身**不学新动作**——这是 RT-2 等 VLA 试图取代的方向。
- **意义**：证明 LLM 的推理可以"流"到机器人，是 VLA 的概念前身。

### 3. 动作基础模型（Action Foundation Models / VLA）

综述的核心章节，按动作表示再细分：

| 子类 | 动作表示 | 代表工作 |
| --- | --- | --- |
| **离散 token VLA** | 动作分桶 token | RT-2, OpenVLA, RT-X, Octo |
| **连续扩散策略** | 扩散采样动作序列 | Diffusion Policy, DP3, EquiBot |
| **Chunk / 自回归** | 多步动作 chunk | ACT, π₀ (flow matching), RDT |
| **世界模型驱动** | 由世界模型推 action | DreamerV3, DayDreamer |

讨论的**关键对比维度**：
- 推理频率（VLA 慢 1–10 Hz；DP/ACT 10–30 Hz）
- 多模态建模能力（DP > ACT > token-VLA）
- 语义泛化（VLA > DP > ACT）
- 数据需求（VLA 海量，DP/ACT 中等）

### 4. 世界模型（World Models）

- **目的**：学习环境的**动力学模型** $s_{t+1} = f(s_t, a_t)$，用于预测、规划、imagined rollout。
- **代表**：
  - 基于视频的：UniSim、Genie、Sora-style 模型（学通用世界动力学）。
  - 基于 RL 的：Dreamer 系列（在 latent 空间想象训练）。
  - 物理可微的：PhysGen、可微仿真器。
- **对 manipulation 的意义**：可在世界模型里**想象**动作后果，做 model-based planning；但**精细接触动力学**建模仍是大难题。
- **趋势**：从 latent world model → pixel-level generative world model（视频生成式）。

### 5. 数据采集与 LfD（Learning from Demonstration）

这是综述区别于其他纯方法综述的亮点——**强调数据本身是一等公民**：

- **遥操系统**：ALOHA、UMI、Gello、SteamVR-based、Apple Vision Pro 遥操。
- **大规模数据集**：Open-X-Embodiment（22 本体、百万 episode）、DROID、RH20T、BridgeData V2。
- **数据增强与仿真**：仿真预训练 + 真机微调（sim-to-real），域随机化。
- **被动数据**：YouTube 视频、人类活动视频（EPIC-Kitchens、Ego4D）作弱监督。

> 综述反复强调：**机器人领域的瓶颈不是模型，是数据**——这与 NLP/CV 形成鲜明对比。

---

## 关键结论（综述提炼）

### 结论 1：通用机器人 = 基础模型 + 跨本体数据

综述的核心论点：要达到 generalist，需要**两条腿**——大模型（提供先验与泛化）+ 跨本体大规模数据（提供具体动作经验）。缺一不可。

### 结论 2：感知/规划基座已成熟，动作基座是瓶颈

- 视觉表征：CLIP/DINOv2 已"够用"，多数工作直接冻结。
- LLM 规划：能做高层分解，但**不能直接执行**。
- **动作基座**：仍在快速演进，没有"赢家通吃"的架构（VLA vs DP vs ACT 各有领地）。

### 结论 3：跨本体泛化（Cross-Embodiment）是关键科学问题

OXE 数据集的开创性工作证明：同一模型能在 22 种本体上泛化。但**本体形态差异大时（臂 vs 人形 vs 四足）泛化显著下降**——这是开放问题。

### 结论 4：评估标准缺失

- 没有统一 benchmark（OXE 是数据集不是 benchmark）。
- 真机评测成本高、复现难。
- 长程任务（>50 步）几乎没有标准化评测。
- 仿真评测（CALVIN、RoboSuite）与真机 gap 仍大。

### 结论 5：Sim-to-Real 仍是工程关键

仿真提供无限数据，但接触丰富任务（插销、装配）的 sim-to-real gap 极大。综述把域随机化、系统辨识、可微仿真列为三大方向。

---

## 三大关键挑战（综述重点章节）

### 挑战 1：数据

- **量级差距**：机器人真机数据 10^5–10^6 episode；NLP/CV 训练数据 10^9–10^12 token/image。
- **质量**：遥操数据噪声、标注不一致、本体差异。
- **采集成本**：人工遥操每小时数十美元；自动采集（self-supervised）难在真机做。

### 挑战 2：泛化

- **新物体**：形状、纹理、质量。
- **新场景**：光照、背景、干扰物。
- **新本体**：臂长、自由度、夹爪形态。
- **新任务**：组合泛化（学了"抓"+"放"，能否"放后抓"）。

综述指出：**当前最好的 VLA（RT-2 档）在新任务上仍只能做到 ~60% 成功率**，离 generalist 还远。

### 挑战 3：评估

- 缺乏统一 benchmark（CALVIN / LIBERO / SimplerEnv 在尝试）。
- 真机评测不可复现（每次 rollout 状态不同）。
- 长程任务评测尤其缺失。
- 没有像 ImageNet/NLP GLUE 那样的"大家都认"的标准。

---

## 对本课题（manipulation 学习）的启发

这是我读这篇综述后，**对自己 manipulation 研究的 5 条具体启发**：

### 1. 优先选择"已有开源基座"的路线

综述清楚显示：**视觉编码器（CLIP/DINOv2/SigLIP）+ 开源 VLA（OpenVLA）/ 开源策略（Diffusion Policy）已足够起步**。不要从零训视觉、从零训 VLM——把工程力放在策略头与数据上。

### 2. 数据策略 > 模型创新

> "The bottleneck is data, not models." —— 综述反复强调。

我的优先级应该是：(1) 搭好遥操 / 采集流水线 → (2) 积累 50–500 条高质量本任务数据 → (3) 用 OpenVLA LoRA 或 DP 微调。模型架构创新是次要的。

### 3. 动作表示是真正的科学问题

综述把 VLA / DP / ACT 三派并列，说明**动作怎么表示**（token vs 扩散 vs chunk）仍是开放问题。我的研究切入点应在**动作头设计**——例如把 ACT 的 chunk 思想 + DP 的扩散头 + VLA 的语义条件融合。

### 4. 评估要刻意构造"分布外"

综述指出多数论文只在训练分布内评测。我设计实验时刻意加入：
- 未见物体（颜色、形状）
- 未见姿态（位置扰动）
- 未见指令（语义泛化）
- 长程组合任务

### 5. 跨本体不要急于求成

OXE 数据虽然跨 22 本体，但综述承认**形态相近时才真正泛化**。做单一本体的强基线，比追求"一个模型吃所有本体"更现实——这是 RT-1 / DP 的经验。

---

## 引用的代表性工作清单（精选）

### VLA / 策略
- **RT-1** (Brohan 2022) — 首个机器人 Transformer
- **RT-2** (Brohan 2023) — VLM + 机器人，涌现泛化 → [`../papers/vla/2023-google-rt2.md`](../papers/vla/2023-google-rt2.md)
- **OpenVLA** (Kim 2024) — 开源 7B VLA → [`../papers/vla/2024-openvla.md`](../papers/vla/2024-openvla.md)
- **Octo** (Team 2024) — 开源多本体 Transformer
- **Diffusion Policy** (Chi 2023) — 扩散动作 → [`../papers/diffusion-policy/2023-chi-diffusion-policy.md`](../papers/diffusion-policy/2023-chi-diffusion-policy.md)
- **ACT / ALOHA** (Zhao 2023) — chunk + CVAE → [`../papers/imitation-learning/2023-zhao-act.md`](../papers/imitation-learning/2023-zhao-act.md)
- **π₀** (Black 2024) — flow matching 动作头
- **RDT-1B** (Liu 2024) — 双臂扩散 Transformer

### 感知基座
- **CLIP** (Radford 2021)
- **SigLIP** (Zhai 2023)
- **DINOv2** (Oquab 2023)
- **SAM** (Kirillov 2023)

### 规划与推理
- **SayCan** (Ahn 2022)
- **Code as Policies** (Liang 2023)
- **VoxPoser** (Huang 2023)
- **PaLM-E** (Driess 2023)

### 世界模型
- **DreamerV3** (Hafner 2023)
- **UniSim** (Yang 2023)
- **Genie** (Google 2024)

### 数据集与采集
- **Open-X-Embodiment** (RT-X Collaboration 2023) → [`../datasets/open-x-embodiment.md`](../datasets/open-x-embodiment.md)
- **DROID** (Khazatsky 2024)
- **BridgeData V2** (Walke 2023)
- **ALOHA** (Fu 2023) / **UMI** (Universal Manipulation Interface, Cheng 2024)

---

## 个人评价

这篇综述是**入门具身智能的最佳单点入口**。它的价值不在提出新方法，而在于把一个**爆炸式增长**的领域（2022–2024 论文量级翻 10 倍）梳理成可学的结构。读完它，你能：

1. 知道**有哪些子领域**（感知/规划/动作/世界模型/数据）。
2. 知道**每个子领域的代表工作**（200+ 引用，精选即可）。
3. 知道**当前共识与开放问题**（数据稀缺、跨本体、评估）。

但它的局限也明显：

- **更新速度跟不上领域**：2024 下半年后的 π₀、RDT-1B、Helix、GR00T 等都未覆盖。
- **偏重 manipulation**：locomotion / navigation 篇幅较少。
- **缺乏定量对比**：是叙事性综述（narrative survey），不是 systematic review，没有 meta-analysis。
- **理论深度有限**：对 scaling law、泛化理论等讨论较浅。

**阅读建议**：把它当**索引**，配合每篇代表论文的笔记（本语料库的 `papers/` 系列）一起读——综述给你地图，单篇笔记给你地形。

后续更聚焦的综述可参考：
- *A Survey on Vision-Language-Action Models*（2024，专门 VLA）
- *Embodied AI: A Survey*（2024，更广的具身智能）
- *Sim-to-Real Transfer in Robotics*（2024，专门迁移）

---

## 相关概念互链

- [`../methods/vision-language-action.md`](../methods/vision-language-action.md) — VLA 总论（综述 §3）
- [`../methods/diffusion-policy.md`](../methods/diffusion-policy.md) — DP 总论（综述 §3）
- [`../methods/imitation-learning.md`](../methods/imitation-learning.md) — IL 总论（综述 §5）
- [`../concepts/foundations/embodied-ai.md`](../concepts/foundations/embodied-ai.md) — 具身智能定义
- [`../concepts/foundations/embodiment.md`](../concepts/foundations/embodiment.md) — 跨本体泛化的概念基础
- [`../datasets/open-x-embodiment.md`](../datasets/open-x-embodiment.md) — 数据基石（综述 §5）
- [`../papers/vla/2023-google-rt2.md`](../papers/vla/2023-google-rt2.md) / [`../papers/vla/2024-openvla.md`](../papers/vla/2024-openvla.md) / [`../papers/diffusion-policy/2023-chi-diffusion-policy.md`](../papers/diffusion-policy/2023-chi-diffusion-policy.md) / [`../papers/imitation-learning/2023-zhao-act.md`](../papers/imitation-learning/2023-zhao-act.md) — 综述引用的代表论文笔记
