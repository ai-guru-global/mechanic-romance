# 真机评测协议（Real-World VLA Evaluation）

> **一句话定位**：VLA 模型的**终极试金石**——但也是**最不可复现、最昂贵、最不可比**的评测环节。各家自定任务集、自选机器人、自报成功率，社区至今没有一个公认的统一真机基准。「仿真刷榜易，真机说话难」是 VLA 时代的核心痛点。

> 最后更新：2026-08

---

## 1. 概览与定位

仿真基准（CALVIN / LIBERO / RLBench）提供了可复现的评测沙盘，但它们与真实世界之间隔着 **sim-to-real gap**：渲染不够真实、物理不够准、物体不够杂、语言不够复杂。对于主打「通用泛化」的 VLA（Vision-Language-Action）模型，**真机评测**才是能力的真正度量。

然而，真机评测天生不可复现：每个实验室的机器人、相机、桌面、光照、物体都不一样；评测任务靠人工设计、人工判定；一次评测几十次 rollout 就要数天人工。结果是——**RT-2、OpenVLA、π₀、Octo 等各家报的真机分数，几乎不可横向比较**。这是当前 VLA 评测领域最大的结构性问题。

| 属性 | 数值 / 说明 |
| --- | --- |
| 评测形态 | 真实机器人 rollout + 人工/半自动判定 |
| 主要指标 | **任务成功率**（success rate，二值） |
| 典型 rollout 数 | 每任务 10–20 次（受成本限制） |
| 任务来源 | **各家自定**（无统一标准） |
| 本体 | Google Robot / Franka / UR / xArm 等，各异 |
| 成本 | 单任务评测 ≈ 数小时人工 + 设备折旧 |
| 可复现性 | **极低**（跨实验室几乎不可复现） |
| 标准化尝试 | SimplerEnv / BEHAVIOR-1K / RoboArena（进行中） |

---

## 2. 为什么真机评测如此困难

### 2.1 不可复现（Irreproducibility）

仿真里，同一个种子产生同一个初始状态；真机里，**没有任何两次 rollout 完全相同**：

- **物体位姿**：人工摆放，毫米级随机。
- **光照**：一天内光照变化足以影响视觉策略。
- **相机标定**：每次开机都有微小漂移。
- **机械公差**：同一型号的两台机器人，动力学都有差异。
- **夹爪/物体摩擦**：批差、温湿度敏感。

这意味着：**A 实验室报的 70% 成功率，B 实验室用同样的模型可能只有 50%**——不是模型变了，是环境变了。

### 2.2 昂贵（Cost）

- **设备**：机器人臂（数万美元）+ 夹爪 + 相机 + 力觉 + 安全围栏。
- **人工**：每次 rollout 要人工重置场景、人工判定成功、人工记录。一个任务的 20 次 rollout 可能耗半天。
- **时间**：一个 VLA 模型在 10 个任务上评测，每任务 20 次，就是 200 次 rollout——数天到数周。
- **维护**：机器人会坏、夹爪会磨损、物体会丢。

对比仿真：LIBERO 上跑 130 任务 × 20 rollout = 2600 rollout，一夜搞定。这就是为什么社区默认用仿真预筛、真机定调。

### 2.3 场景各异（Heterogeneity）

每家实验室的「真机评测」都是**自定义场景**：

- **任务集不同**：Google 评 552/201 条指令，OpenVLA 评 29 任务，π₀ 评自定厨房任务。
- **本体不同**：Google Robot（Everyday Robots）、Franka、UR、xArm，动作空间都不同。
- **物体不同**：有的用乐高，有的用厨房用具，有的用工具。
- **难度不同**：有的任务是简单 pick-place，有的是长程多阶段。
- **成功判定不同**：有人工判定、有物体状态判定、有「接近就算」的宽松判定。

结果是：**RT-2 的 62% 和 OpenVLA 的 50% 根本不是一回事**——它们在不同任务、不同本体、不同判定下测出。

### 2.4 统计不可靠

- 真机 rollout 数少（10–20/任务），**置信区间宽**：70% ± 15% 很常见。
- 任务间方差大：一个难任务 10%、一个易任务 90%，平均 50% 掩盖了分布。
- **选择性报告**风险：报擅长的任务、藏失败的任务，是行业潜规则。

---

## 3. 主要玩家的真机评测方式

### 3.1 Google Robot：RT-1 的 552 指令评测 + RT-2 的涌现评测

**RT-1**（Brohan et al., 2022）建立了 Google Robot 的真机评测范式，使用 **Everyday Robots**（单臂 + 夹爪 + 头部相机）：

- **552 条指令**的评测集：覆盖 pick-place、开抽屉、推物等多种技能，每条指令多次 rollout。
- **成功率**为主指标，按指令类别（seen / unseen instruction / unseen skill）分组报。
- RT-1 报告了 **97% seen / 76% unseen instruction** 的成绩，奠定了「大数据 BC 可行」的信心。

**RT-2**（Brohan et al., 2023）在此基础上加入了**语义涌现（emergent）**评测：

- 在数百条真机指令上评测，分成 **seen tasks / unseen tasks / emergent tasks**（符号理解、计数、人类动作、风险理解等）。
- RT-2 在 **emergent 任务上 ~62% 成功率，是 RT-1（~32%）的近两倍**——证明 web-scale VLM 知识能迁移到控制。
- RT-2 还在 Language Table 仿真上达 **90% 成功率**（vs 前SOTA 77%）。

> 注：常被提及的「552 指令评测集」源自 **RT-1** 论文；RT-2 沿用并扩展了 Google Robot 真机协议，但具体指令数和分组在各论文中有细微差异。这是 Google 内部 benchmark，外部实验室无法精确复现。

### 3.2 OpenVLA：29 任务 Google Robot 评测

**OpenVLA**（Kim et al., 2024）为了与 RT-2-X 可比，刻意采用了 **Google Robot** 真机平台：

- **29 个真机任务**，覆盖 seen skills / unseen skills / unseen objects 三类。
- 每任务多次 rollout，报成功率。
- OpenVLA 在 29 任务上**比 RT-2-X（旧版）高 16.5% 绝对成功率**，是唯一在所有任务上都 ≥50% 的方法。
- 配套发布了 **SimplerEnv**（仿真代理），用仿真近似真机评测，提升可复现性。

> OpenVLA 的贡献之一是把「Google Robot 真机评测」对外开放（通过 SimplerEnv），但仍非完全标准。

### 3.3 π₀（Physical Intelligence）：厨房/桌面真机任务

**π₀**（Physical Intelligence, 2024）走的是**更难的真机任务**路线：

- 在折叠衣物、打包杂物、煎蛋、组装等**长程、精细、双臂**任务上评测。
- 任务集**完全自定**，不与 RT-2/OpenVLA 重叠。
- 用 flow-matching head 实现高频连续动作，展示真机精细操作能力。
- 报告的是**演示级成功率**（任务能否完成），无标准化协议。

### 3.4 Octo / RoboFlamingo / GR-1 等

- 多数在**自家真机**或**仿真代理**上报分，任务集各异。
- 部分在 BridgeData 真机评测集、或 RoboArena（新兴真机基准）上报分。
- 普遍缺乏跨实验室可比性。

---

## 4. 标准化真机基准的尝试

### 4.1 SimplerEnv：仿真代理（2024）

**SimplerEnv**（simpler-env.github.io）是 OpenVLA 团队推出的**仿真代理评测**，目标是**可复现地近似 Google Robot 真机评测**：

- 在仿真里重建 Google Robot 任务（如 RT-1 的 pick-place 类）。
- 让策略在仿真里跑，分数作为真机分数的**代理估计**。
- 已支持 OpenVLA / RT-1 / Octo / RT-2-X 等的对比。

价值：**便宜、可复现、跨实验室可比**。局限：仍是仿真，sim-to-real gap 未消除；只能近似，不能替代真机。

### 4.2 BEHAVIOR-1K：家庭场景标准化（2024）

**BEHAVIOR-1K**（Stanford，见 [`../manipulation/behavior-1k.md`](../manipulation/behavior-1k.md)）野心更大：用 **OmniGibson** 仿真 + **1000 个真实家庭活动**，覆盖家务全谱：

- 任务来自**真实人群调查**（「你想让机器人做什么」），生态效度高。
- 仿真物理与渲染接近真实（NVIDIA Omniverse）。
- 配套**真机子集**（BEHAVIOR-Robot），在真实机器人上验证。
- 局限：任务偏家庭/导航，精细操作仍少；真机子集规模有限。

### 4.3 RoboArena / DYOB（Do Your Own Benchmark）

- **RoboArena**：试图建立社区共享的真机评测平台（远程访问机器人）。
- **DYOB**：允许各实验室上传自己的任务集，统一接口评测。
- 局限：覆盖窄、仍处于早期。

### 4.4 Open-X-Embodiment 真机评测协议

OXE 数据集附带**统一的评测协议草案**，鼓励各实验室在**标准化任务子集**上报分。但真机部分仍是「建议」而非「强制」，采纳度有限。

---

## 5. 社区对统一真机基准的呼声

### 5.1 现状：各家不可比

打开任何一篇 VLA 论文的「Real-World Evaluation」章节，你几乎总能看到：

- **自定任务集**（10–50 个任务，互不重叠）。
- **自选机器人**（Google Robot / Franka / xArm / 双臂，各异）。
- **自定成功判定**（人工 / 半自动 / 宽松 / 严格，不一）。
- **自报成功率**（无第三方验证）。

这让**横向比较几乎不可能**。读者只能看「相对提升」（A 比 B 高 X%），但 A 和 B 的任务集可能完全不同。

### 5.2 为什么统一这么难

- **物理门槛**：不是每家都有同型号机器人 + 同款物体。
- **成本**：标准化评测集要大量 rollout，谁出钱、谁出力？
- **利益**：领先者不愿被标准化「拉平」，落后者无力推动。
- **快速演进**：VLA 迭代极快，标准化协议赶不上方法变化。

### 5.3 社区正在做什么

- **仿真代理优先**（SimplerEnv、LIBERO 仿真评测）成为**默认可比层**，真机作为**定性验证**。
- **共享真机平台**（RoboArena、远程机器人租赁）在尝试降低门槛。
- **标准化协议提案**（如 NVIDIA 的「如何评测通用机器人策略」博客、BEHAVIOR-1K 真机子集）在推进。
- **第三方评测机构**（如某些 benchmark 公司）开始出现，但规模仍小。

### 5.4 一个务实的折中

当前社区的**事实共识**是：

1. **仿真刷榜**（LIBERO / CALVIN）作为可复现的定量比较层。
2. **仿真代理**（SimplerEnv）作为真机的近似层。
3. **真机定性验证**（自家任务集，展示能力）作为定性的「能不能用」层。
4. **真机定量比较**仍需等待统一基准成熟。

> 一句话：**仿真比分数，真机看效果**。这是 2024–2025 年 VLA 评测的实情。

---

## 6. 评估指标与协议（真机通用做法）

### 6.1 指标

| 指标 | 定义 | 备注 |
| --- | --- | --- |
| **任务成功率**（task success rate） | rollout 是否完成任务（二值） | **主指标**，但成功判定因任务而异 |
| **部分成功率**（partial） | 完成任务的比例（如多步任务的步数比） | 用于长程任务 |
| **指令遵循率** | 是否按指定指令做（而非随意成功） | 防止「歪打正着」 |
| **效率**（步数/时间） | 完成任务所用步数或时间 | 次要，部分工作报 |
| **泛化分组** | seen / unseen object / unseen skill / emergent | 关键：按泛化维度分组报 |

### 6.2 协议要素

一次严谨的真机评测应明确：

1. **任务定义**：每个任务的语言指令、目标状态、成功判定标准（文字 + 图片）。
2. **物体集**：用了哪些物体、哪些是 seen / unseen。
3. **初始状态**：每次 rollout 如何重置（人工摆放的协议）。
4. **rollout 数**：每任务多少次（至少 10–20 才有统计意义）。
5. **判定方式**：人工判定 / 物体状态自动判定 / 视频回放判定。
6. **本体与硬件**：机器人型号、夹爪、相机配置、控制频率。
7. **失败处理**：超时阈值、卡住时是否人工干预。
8. **置信区间**：报均值 + 标准误或 Wilson 区间。

> 遗憾的是，多数论文只报「任务成功率均值」，省略了 4–8 项，这是不可比的根源。

---

## 7. 设计理念与价值

### 7.1 为什么真机评测不可或缺

1. **sim-to-real gap 真实存在**。仿真 90% 的策略，真机可能 40%。只有真机能定调。
2. **泛化的真正考场**。仿真任务集有限，真机的无限变体才考验泛化。
3. **落地需求**。最终要在真机用，真机评测是最接近「能不能用」的信号。
4. **发现仿真盲点**。真机评测常暴露仿真未建模的失败模式（光照、遮挡、接触）。

### 7.2 真机评测推动的技术

- **sim-to-real 迁移**方法（域随机化、系统辨识）。
- **鲁棒性**设计（对抗光照/位姿扰动）。
- **力觉/触觉融合**（仿真难建模，真机才能体现价值）。
- **长程任务**的规划与错误恢复。

---

## 8. 局限

### 8.1 不可复现

最大硬伤。跨实验室、跨时间的分数不可比，是 VLA 评测的「原罪」。

### 8.2 统计薄弱

rollout 数少、方差大、选择性报告风险高。一篇论文报的「62%」可能在置信区间内与「50%」无显著差异。

### 8.3 成本高昂

限制了评测规模和迭代速度，小实验室被排除在外，加剧了资源不平等。

### 8.4 任务代表性

自定任务集可能**刻意挑选擅长项**，不代表真实部署分布。

### 8.5 判定主观

人工判定「成功」的边界模糊（「差不多到了」算不算？），引入偏见。

---

## 9. 与同类/互补评测对比

| 评测层 | 代表 | 可复现 | 可比性 | 成本 | sim-to-real gap |
| --- | --- | --- | --- | --- | --- |
| **仿真基准** | LIBERO / CALVIN / RLBench | **✅ 强** | **✅ 强** | 低 | 有 |
| **仿真代理** | SimplerEnv | **✅ 强** | 中 | 低 | 较小（逼近真机） |
| **标准化真机** | BEHAVIOR-1K 真机子集 / RoboArena | 中 | 中 | 高 | 无 |
| **自定真机** | RT-2 / OpenVLA / π₀ | **❌ 弱** | **❌ 弱** | **极高** | 无 |

一句话：**仿真比分数，代理估能力，真机看效果**。三者互补，缺一不可。

---

## 10. 相关概念互链

- [`../README.md`](../README.md)——基准总览，真机评测是最难标准化的一环。
- [`../manipulation/calvin.md`](../manipulation/calvin.md)——仿真长程基准，真机评测的对照。
- [`../manipulation/libero.md`](../manipulation/libero.md)——仿真泛化基准，VLA 默认刷分层。
- [`../manipulation/rlbench.md`](../manipulation/rlbench.md)——仿真精细任务基准。
- [`../manipulation/behavior-1k.md`](../manipulation/behavior-1k.md)——野心最大的家庭真机+仿真基准。
- [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)——VLA 模型是真机评测的主角。
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)——真机评测主要评 IL/BC 策略。
- [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md)——OXE 的统一评测协议尝试。
- [`../../simulation/README.md`](../../simulation/README.md)——sim-to-real gap 是真机评测存在的原因。

---

## 11. 参考链接

- RT-1：Brohan et al., *RT-1: Robotics Transformer for Real-World Control at Scale*, 2022. arXiv:2212.06817（552 指令评测集出处）
- RT-2：Brohan et al., *RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control*, 2023. arXiv:2307.15818 — https://robotics-transformer2.github.io/
- OpenVLA：Kim et al., *OpenVLA: An Open-Source Vision-Language-Action Model*, 2024. arXiv:2406.09246 — https://openvla.github.io/
- SimplerEnv：https://simpler-env.github.io/
- π₀：Physical Intelligence, *π₀: A Vision-Language-Action Flow Model*, 2024. — https://www.physicalintelligence.company/
- BEHAVIOR-1K：https://behavior.stanford.edu/
- NVIDIA「如何评测通用机器人策略」：https://developer.nvidia.com/blog/how-to-evaluate-general-purpose-robot-policies-for-real-world-deployment/
- 评测 VLA 经验：*Experiences from Benchmarking Vision-Language-Action Models*, 2025. arXiv:2511.11298
