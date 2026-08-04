# LIBERO 基准（Lifelong Robot Learning Benchmark）

> **一句话定位**：2023 年起、2024 年爆火的**操作泛化（manipulation generalization）**评测基准——它把「泛化」这件事拆成三个正交维度（新空间布局 / 新物体 / 新目标），用 130 个任务、5 个评测套件系统化地拷问 VLA「到底能不能迁移」。Liu et al., NeurIPS 2023 Datasets & Benchmarks。

> 最后更新：2026-08

---

## 1. 概览与定位

LIBERO 由 B. Liu 等人（Stanford / MIT 等）发布于 NeurIPS 2023，初衷是研究**终身机器人学习（lifelong robot learning）**与**知识迁移（knowledge transfer）**。但它真正走红，是因为它恰好踩中了 2024 年 VLA 浪潮的评测刚需：VLA 厂商需要一个**设计严谨、覆盖泛化、可复现**的仿真基准来比拼，而 LIBERO 的「三正交泛化维度 + 标准化任务套件」完美契合。

在 LIBERO 之前，操作泛化评测是混乱的——各家自定任务，无可比性；CALVIN 评长程但不评泛化维度，MetaWorld 任务多但泛化设定粗糙。LIBERO 第一次把泛化**结构化**了。

| 属性 | 数值 / 说明 |
| --- | --- |
| 发布年份 | 2023（NeurIPS D&B） |
| 仿真平台 | **robosuite / MuJoCo** |
| 本体 | Franka Panda 7-DoF + Robotiq 夹爪（robosuite 标准） |
| 任务总数 | **130** 个 |
| 任务套件 | **5 个**评测套件（见 §2） |
| 语言指令 | 每个任务一条自然语言指令（PDDL 生成） |
| 观测 | 腕部 + 旁置相机 RGB，关节状态 |
| 动作 | 末端 delta pose（robosuite 标准 OSC） |
| 数据 | 每任务 50 条专家演示（脚本化） |
| 评测指标 | 任务成功率（每任务 20 rollout 平均） |

---

## 2. 五个评测套件（5 Task Suites）

LIBERO 的核心设计是**控制变量**：把「泛化」拆成三个正交维度，每个维度对应一个套件。通过固定两个维度、变化一个，可以精确诊断模型在哪种泛化上强、哪种上弱。

### 2.1 套件一览

| 套件 | 全称 | 任务数 | 训练/测试划分 | 考验的泛化 |
| --- | --- | --- | --- | --- |
| **LIBERO-Spatial** | 空间套件 | 10 | 10 任务全训、留出场景测 | 新空间布局 |
| **LIBERO-Object** | 物体套件 | 10 | 10 任务全训、留出物体测 | 新物体（同任务） |
| **LIBERO-Goal** | 目标套件 | 10 | 10 任务全训、留出目标测 | 新任务目标（同物体同布局） |
| **LIBERO-Long（LIBERO-100）** | 长程/终身套件 | 100 | 90 预训练 + 10 测试（**LIBERO-90 / LIBERO-10**） | 多任务 + 终身迁移 |
| **LIBERO-Direct**（评测变体） | 直评 | 10 | 直接评测（不预训练迁移） | 基线能力下限 |

> 说明：常说的「LIBERO-10」即 LIBERO-Long 的测试子集；「LIBERO-90」是它的预训练子集。社区有时把 Spatial/Object/Goal 各自的「直接评测」也称作 direct 设定。任务提示中提到的「bonus」并非官方套件名，实际对应 LIBERO-Long（多任务终身）这一最复杂的设定。

### 2.2 三正交泛化维度

LIBERO 的精髓——用 PDDL（Planning Domain Definition Language）程序化生成任务，**精确控制哪个变量变、哪些不变**：

```
LIBERO-Spatial：物体不变、目标不变 → 只变物体在桌上的空间布局
LIBERO-Object ：布局不变、目标不变 → 只换物体（如碗换杯）
LIBERO-Goal   ：布局不变、物体不变 → 只换要做的任务（如「放进碗」换「放到架上」）
```

这种「**一次只变一个维度**」的设计，让泛化能力**可归因（attributable）**：模型在 Spatial 上低分，就说明它没学到空间不变性，而非物体或目标的问题。

### 2.3 LIBERO-Long：终身/多任务

LIBERO-Long 把 100 个任务混在一起，考验**多任务共享 + 迁移**：
- **LIBERO-90**：90 个任务用于训练一个统一策略。
- **LIBERO-10**：10 个 held-out 任务测迁移。
- 这模拟「机器人在已学技能基础上，面对新任务能不能迁移」的终身学习场景。

---

## 3. 任务与生成机制

### 3.1 任务族

LIBERO 的任务围绕**桌面物体操作**展开，物体来自 BEHAVIOR 物体库（碗、杯、盘、罐、水果等），动作包括：

- **pick-and-place**：抓起物体放到目标位置。
- **堆叠 / 排列**：把物体放到架子上、按顺序排列。
- **倒 / 推**：把容器内物体倒出、推到目标。
- **开/关**：抽屉、盒子等。

每个任务由一条自然语言指令描述（如 *"pick up the black bowl on the stove and place it on the plate"*），指令由 PDDL 任务模板自动生成，语义精确、无歧义。

### 3.2 为什么用 PDDL

LIBERO 用 **PDDL** 做任务的形式化定义与生成：

- **可组合**：物体、位姿、目标三类谓词可自由组合 → 自动产出海量任务变体。
- **可控**：固定某些谓词、变化另一些，精确制造「只变一个维度」的对照。
- **可判定**：PDDL 的目标条件天然就是成功判定，无需人工写启发式。
- **可复现**：同一个 PDDL 种子生成同一套任务，跨实验室可复现。

这是 LIBERO 比手工设计任务（如 RLBench）在**泛化评测**上更严谨的根本原因。

### 3.3 演示数据

每个任务提供 **50 条脚本化专家演示**（用 robosuite 的脚本规划器采集），用于模仿学习。数据格式与 Open-X-Embodiment 兼容，方便 VLA 直接消费。

---

## 4. 评估指标与协议

### 4.1 主指标：任务成功率

LIBERO 的评测简单清晰：每个任务在**20 条 rollout**上评测，统计成功率（success rate），再在套件内 10 任务上平均，得到**套件平均成功率**。

| 指标 | 定义 |
| --- | --- |
| **Per-task success rate** | 单任务 20 rollout 中成功比例 |
| **Suite success rate** | 套件内所有任务的平均成功率（主榜分） |
| **5-suite average** | 5 个套件成功率的综合（部分工作报） |

成功判定基于**物体状态**（PDDL 目标条件），客观可复现。

### 4.2 评测流程

1. 在指定套件的训练任务（如 LIBERO-90 或 Spatial/Object/Goal 的 10 任务）上训练。
2. 在 held-out 测试任务上，每任务 20 条 rollout。
3. 统计成功率，取套件平均。
4. 跨 5 个套件汇报，形成泛化能力画像。

> 与 CALVIN 不同，LIBERO 的任务**每次 rollout 重置环境**，不评长程连续性；它评的是「单任务能不能泛化到新设定」。

---

## 5. 基线分数参考

LIBERO 上的分数更新极快（VLA 军备竞赛）。下表为**代表性方法**在 5 套件上的成功率（%，不同实现有波动，仅供量级参考）。

| 方法 | 类型 | Spatial | Object | Goal | Long(10) | 备注 |
| --- | --- | --- | --- | --- | --- | --- |
| **BC（LSTM/MLP）** | 经典行为克隆 | ~78 | ~85 | ~70 | ~45 | 基线下限 |
| **BC-Transformer** | Transformer 策略 | ~82 | ~90 | ~80 | ~50 | 加注意力 |
| **Diffusion Policy** | 扩散策略 | ~85 | ~92 | ~85 | ~55 | 多模态动作 |
| **ViNT / RT-1（小）** | 小型 Transformer | ~80 | ~88 | ~78 | ~48 | |
| **OpenVLA（7B）** | 通用 VLA | ~90+ | ~93+ | ~88+ | ~60+ | 微调后 |
| **π₀ / π₀.5** | Flow-matching VLA | ~92+ | ~95+ | ~90+ | ~65+ | 2024 SOTA 量级 |

> 关键观察：
> - **Object 套件通常最容易**（换物体但布局/目标不变，抓取即可）。
> - **Goal 套件最难**（目标语义变了，需理解任务差异）。
> - **Long(10) 是综合考验**，分数普遍低于单维度套件。
> - 精确数字请查 LIBERO 官方 leaderboard 与各方法论文。BC 基线 ~78 在 Spatial 上，说明任务本身不算太难；VLA 把上限推到 90+。

---

## 6. 设计理念与价值

### 6.1 为什么 LIBERO 成为 VLA 评测新标准

1. **泛化维度正交、可归因**。Spatial/Object/Goal 三套件让「模型在哪泛化好/差」一目了然，这是 CALVIN 做不到的。
2. **设计严谨、控制变量**。PDDL 生成 + 固定维度变化，是「实验科学」式的基准设计。
3. **可复现**。robosuite/MuJoCo 开源、任务 PDDL 公开、演示数据可下载，任何实验室能复现。
4. **覆盖全谱**。从简单单任务（Spatial）到终身多任务（Long），梯度合理。
5. **生态友好**。数据格式兼容 OXE，VLA 可直接训；评测脚本标准化。
6. **难度适中**。不像真机那么贵，也不像 MetaWorld 那么简单——正好是「能刷分、能区分」的甜区。

### 6.2 它推动的技术方向

- **VLA 微调范式**：在 OXE 上预训练 → 在 LIBERO-90 上微调 → 在 LIBERO-10 上测。
- **泛化归因分析**：用三套件诊断模型是「视觉泛化弱」还是「语义泛化弱」。
- **终身学习方法**：EWC、回放、模块化等在 LIBERO-Long 上比拼。

---

## 7. 局限

### 7.1 不评长程

LIBERO 每次评测**单任务、重置环境**，不考验「连续 5 任务不重置」的长程能力。这一点 CALVIN 更强。

### 7.2 仿真与真机仍有差距

- MuJoCo 渲染偏干净，无杂乱背景、光照变化、遮挡。
- 物体虽比 CALVIN 多，但仍限于桌面、无真实家庭复杂度。

### 7.3 任务难度上限有限

- BC 基线在 Spatial 上就能到 ~78，说明任务对现代方法**不够难**，区分度在压缩。
- 真正难的精细操作（插销、拧螺丝、双臂）几乎没有。

### 7.4 其他

- **演示来自脚本规划器**，与人类演示分布有差异，可能影响 sim-to-real。
- **物体库有限**（BEHAVIOR 子集），不像 OXE 那样跨本体。
- **LanguageTable 式的语言**仍偏模板化，无复杂指代与推理。

### 7.5 泛化维度仍不完整

Spatial/Object/Goal 三维没有覆盖：**新本体（embodiment）**、**新物理参数（摩擦/质量）**、**新视角**。这些是 Open-X-Embodiment 真机评测才涉及的。

---

## 8. 与同类基准对比

| 维度 | **LIBERO** | CALVIN | RLBench | MetaWorld |
| --- | --- | --- | --- | --- |
| 泛化评测 | **✅ 三正交维度（核心）** | 环境布局 | 任务变体 | 任务 |
| 长程评测 | Long（多任务，非连续） | **✅ 5 链（核心）** | ❌ | ❌ |
| 任务生成 | **PDDL 程序化** | 脚本 | 手工 | 手工 |
| 任务数 | 130 | 34 原语 | 100（PerAct 用 18） | 50 |
| 仿真 | robosuite / MuJoCo | PyBullet | CoppeliaSim | MuJoCo |
| 视觉真实度 | 中 | 低 | 中高 | 低 |
| VLA 适配 | **✅ 强（2024 主流）** | 中（长程向） | 弱（3D 向） | 弱（RL 向） |
| 复现性 | **✅ 强** | 强 | 中 | 中 |

一句话：**CALVIN 评长程，LIBERO 评泛化，RLBench 评精细，MetaWorld 评 RL**。2024 年 VLA 论文常**同时报 CALVIN + LIBERO 双榜**。

---

## 9. 相关概念互链

- [`../README.md`](../README.md)——基准总览，LIBERO 是泛化评测的新标杆。
- [`calvin.md`](calvin.md)——长程操作基准，与 LIBERO 并列为 VLA 双主榜。
- [`rlbench.md`](rlbench.md)——手工精细任务基准，PerAct 系舞台。
- [`../vla-eval/real-world-eval.md`](../vla-eval/real-world-eval.md)——LIBERO 是仿真评测，真机评测仍是终极考验。
- [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)——LIBERO 是 VLA 泛化的主考场。
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)——DP 在 LIBERO 上的强基线。
- [`../../methods/behavioral-cloning.md`](../../methods/behavioral-cloning.md)——LIBERO 的默认训练范式。
- [`../../datasets/open-x-embodiment.md`](../../datasets/open-x-embodiment.md)——VLA 在 OXE 预训练、LIBERO 微调的标准流程。
- [`../../simulation/README.md`](../../simulation/README.md)——robosuite/MuJoCo 是 LIBERO 的底座。

---

## 10. 参考链接

- 项目主页：https://libero-project.github.io/
- GitHub：https://github.com/Lifelong-Robot-Learning/LIBERO
- 论文：Liu et al., *LIBERO: Benchmarking Knowledge Transfer for Lifelong Robot Learning*, NeurIPS 2023 Datasets & Benchamentals. arXiv:2306.03310
- 数据下载：https://libero-project.github.io/datasets
- LeRobot 集成：https://huggingface.co/docs/lerobot/en/libero
- OpenVLA 在 LIBERO：https://openvla.github.io/
