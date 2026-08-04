# Physical Intelligence（π / pi）· 具身大模型公司图谱

> **一句话定位**：Physical Intelligence 是由 Sergey Levine 等创立、专注「**机器人大脑**」（与硬件解耦）的具身大模型公司——以 π₀ 通用 VLA 模型（flow matching 动作头、跨 5 种本体、约 50 Hz 推理）著称，2025 年 11 月以 **56 亿美元估值**完成 6 亿美元融资，是「只做软件大脑、适配任意机器人」路线的代表。

> **最后更新：2026-08**
>
> **可信度声明**：本文区分【官方公布/论文】与【外界推测/媒体报道】。π₀ 的架构细节来自 arXiv 论文（2410.24164）与官方博客，属可复现的一手技术资料；融资数据以官方/Bloomberg 为准。

---

## 1. 公司概况

| 维度 | 信息 | 来源 |
| --- | --- | --- |
| 公司全称 | Physical Intelligence（π / pi）, Inc. | 【官方】 |
| 联合创始人 | **Sergey Levine**（UC Berkeley 副教授，深度强化学习与机器人学习奠基人之一）等 | 【官方】 |
| 成立时间 | 2024 年 | 【官方】 |
| 总部 | 美国加州旧金山 | 【官方】 |
| 使命 | 「把通用 AI 带入物理世界」——构建机器人的通用基础模型 | 【官方】 |
| 模式 | **纯软件大脑**（不造机器人硬件，适配任意本体） | 【官方】 |

**创始人叙事**：Sergey Levine 是机器人学习领域的学术泰斗——深度 RL 用于机器人控制的开创性工作（DDPG、QT-Opt 等）多与其相关。他创立 Physical Intelligence 的核心论点是：**机器人硬件正在趋同，真正的壁垒是「大脑」——一个能跨本体、跨任务的通用 VLA 基础模型**。这与 Figure/Tesla「软硬一体」、宇树「硬件+locomotion」路线形成根本分歧。

---

## 2. 产品线：π₀ 通用 VLA 模型

Physical Intelligence 的「产品」是模型而非硬件。核心是 **π₀（pi-zero）**：

### 2.1【官方/论文】π₀ 规格表

| 维度 | 数值/说明 | 来源 |
| --- | --- | --- |
| 模型类型 | **Flow-based 扩散式 VLA**（Vision-Language-Action） | 【论文 2410.24164】 |
| 输入 | 相机图像 + 机器人状态 + 自然语言指令 | 【论文】 |
| 输出 | 连续机器人动作（action chunks，约 50 个未来动作） | 【论文】 |
| 推理速度 | 约 **50 Hz** | 【论文】 |
| VLM 主干 | 预训练 VLM（PaliGemma 家族，约 3B 参数） | 【论文】 |
| 动作表示 | **连续**（经 flow matching） | 【论文】 |
| 跨本体 | ✅ 单一策略控制 **5 种不同机器人本体** | 【论文】 |
| 训练数据 | 当时最大规模的机器人交互数据集（跨本体） | 【论文/博客】 |
| arXiv | 2410.24164（Black et al., 2024，被引约 2750 次） | 【arXiv】 |

### 2.2【官方/论文】架构：VLM 主干 + Flow Matching 动作头

π₀ 由两个关键组件构成：

1. **预训练 VLM 主干**：继承互联网级语义知识，负责视觉理解与语言推理。
2. **Flow Matching 动作头（Action Expert）**：一个较小的 transformer，通过 **flow matching**（类扩散技术）生成连续动作输出，而非把动作离散化为 token。

**为什么用 Flow Matching 做动作头**：

- 机器人动作本质是**连续的**，不是离散 token——flow matching 比基于 tokenization 的方法（如 FAST）更自然；
- Flow matching 学习一个**去噪向量场**，训练预测从噪声到干净动作轨迹的方向；
- 动作专家每层 8 个注意力头，输出约 50 个未来动作的 action chunk，推理约 50 Hz。

> 这是 π₀ 区别于 RT-2 等「离散动作 token」VLA 的核心技术差异。详见 [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md) 与 [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)。

### 2.3【官方/论文】跨本体：5 种机器人

π₀ 的核心贡献是作为**单一通用策略**跨多种机器人本体与任务族训练：

- 论文展示模型在 **5 种不同机器人平台**（不同形态/构型）上运行；
- 覆盖多种灵巧操作任务；
- 展示**跨本体泛化**——单一策略可控制不同机器人，无需每个平台单独训练模型。

> 这是「**通用机器人大脑**」叙事的技术基础：一个模型适配任意硬件。

---

## 3. 技术路线：通用 VLA + 大规模真机数据

### 3.1【官方】软件大脑，硬件解耦

Physical Intelligence 的根本路线选择：

```
传统：每个机器人 → 定制控制栈 → 难以泛化
        │
π 路线：一个通用 VLA 大脑 → 适配任意机器人本体
        │
关键：跨本体数据 + 连续动作输出 + 大规模训练
```

这意味着：
- **不造硬件**：与 Figure/Tesla/宇树/智元都不同，π 不做本体；
- **数据跨本体**：采集多种机器人的真机数据统一训练；
- **商业模式**：把大脑「装」到合作伙伴的机器人上。

### 3.2【官方】大规模真机数据

π₀ 训练于「当时最大规模的机器人交互数据集」（跨本体）。Physical Intelligence 的数据策略是**大规模真机采集**：

| 维度 | 状态 | 来源 |
| --- | --- | --- |
| 真机数据采集 | 大规模遥操作 + 自主采集，跨多种机器人 | 【官方/论文】 |
| 跨本体统一 | 5 种机器人数据统一训练 | 【论文】 |
| 仿真 | 官方未强调仿真为主，偏真机 | 【推测】**待核** |
| 数据开放 | ❌ 不开放（闭源数据） | 【官方】 |

> 与 Figure「工厂闭环数据」、智元「AgiBot World 开放数据集」相比，π 走的是「**自采跨本体闭源数据 + 学术论文透明架构**」的路线——架构公开可复现，但数据私有。

---

## 4. 落地场景与商业模式

| 场景 | 状态 | 说明 |
| --- | --- | --- |
| 灵巧操作（抓取/插入/组装） | ✅ 论文验证 | 跨 5 种本体 |
| 双臂协作 | ✅ 论文验证 | 折叠衣物、装袋等 |
| 家庭/服务操作 | 🟡 推进 | 长期愿景 |
| 工业部署 | 🟡 与硬件伙伴合作 | 不直接卖硬件 |

> Physical Intelligence 的商业模式是「**大脑授权 / 合作部署**」——把 π 模型集成到合作伙伴的机器人上。这与 Covariant 被 Amazon 授权的模式类似（见第 7 节）。

---

## 5. 软件生态与开放性

| 维度 | 状态 |
| --- | --- |
| 论文公开 | ✅ π₀ 论文（arXiv 2410.24164）含架构细节 |
| 官方博客 | ✅ pi.website/blog/pi0 含技术说明 |
| 模型权重 | 🟡 部分开放/受限（**待核**最新状态） |
| 数据集 | ❌ 不开放 |
| 学术可得性 | 中（架构可复现，但数据/完整权重受限） |

> 相比 Figure/Tesla 完全闭源，π 在**学术透明度**上更高（论文 + 博客可复现架构），但数据与完整模型仍闭源。

---

## 6. 融资与估值

Physical Intelligence 的资本号召力在具身大模型赛道顶级：

| 轮次 / 时间 | 金额 | 估值 | 来源 |
| --- | --- | --- | --- |
| 早期轮（2024） | 约 **4 亿美元** | 约 **24 亿美元** | 【官方/媒体】 |
| **新轮 / 2025-11** | **6 亿美元** | **56 亿美元** | 【Bloomberg 2025-11-20】 |
| 后续（传闻） | 据报在洽谈 **10 亿美元**级融资 | — | 【Reddit/媒体】**待核** |

> 估值从 24 亿（2024）跃升至 56 亿（2025-11），反映具身大模型赛道的资本热度。Bloomberg 是 56 亿估值的一手来源。

---

## 7. 竞争格局：具身大模型三巨头对比

「机器人大脑」赛道有三家代表性公司，路线相近但定位有别：

| 维度 | Physical Intelligence（π） | Skild AI | Covariant |
| --- | --- | --- | --- |
| 核心产品 | π₀（flow matching VLA） | 通用 omni-bodied 大脑 | Covariant Brain / RFM-1 |
| 跨本体 | ✅ 5 种机器人 | ✅ 多本体 | ✅ 仓储机械臂为主 |
| 融资/估值 | 6 亿美元 / 56 亿（2025-11） | 约 **14 亿美元估值**（融资 3 亿+） | 被 **Amazon 非独家授权**，核心人才吸收 |
| 数据 | 自采跨本体真机 | 通用大规模数据 | 数十亿机器人 episode |
| 商业模式 | 大脑授权/合作 | 通用大脑授权 | 仓储场景 + Amazon 授权 |
| 开放度 | 论文透明，数据闭源 | 较闭源 | 较闭源 |

### 7.1 vs Skild AI

- **Skild AI**：定位「统一的全能大脑（omni-bodied brain）」，控制任意机器人执行任意任务（抓取、交接、导航）；近期展示基于视觉的人形 locomotion；估值约 14 亿美元（融资 3 亿美元以上，The Robot Report）。
- **差异**：π 更偏**操作 + flow matching 动作头**的工程化；Skild 更强调「全能」叙事与 locomotion。

### 7.2 vs Covariant

- **Covariant**：OpenAI 衍生公司，创建 RFM-1（机器人基础模型），从数十亿机器人 episode 学习；**Amazon 签署非独家授权**使用其机器人基础模型，核心人才与模型被 Amazon 部分吸收。
- **差异**：Covariant 更聚焦**仓储机械臂**场景且已被巨头吸收；π 更通用、独立运营、跨本体野心更大。

### 7.3 vs Figure Helix / 智元 GO-1（软硬一体派）

| 维度 | π（纯软件） | Figure Helix / 智元 GO-1（软硬一体） |
| --- | --- | --- |
| 硬件 | 不造 | 自研本体 |
| 数据 | 跨本体自采 | 自有工厂/量产闭环 |
| 护城河 | 通用大脑 + 跨本体数据 | 软硬协同 + 真实场景数据 |
| 风险 | 依赖硬件伙伴 | 大脑只能跑在自家硬件 |

> 核心分歧：**「大脑与硬件是否应该解耦」**。π 押注「解耦」（一个大脑统治所有机器人）；Figure/智元押注「耦合」（软硬协同才能做到可靠）。这是具身智能产业最重要的路线之争之一。

---

## 8. 风险与争议

- **「跨本体泛化」的真实边界**：π₀ 论文展示 5 种本体，但跨本体在新任务、新环境上的零样本泛化能力，仍有待更大规模验证。
- **商业模式不确定性**：纯软件大脑能否说服硬件厂商授权（而非自研），是未解之问——Figure 已自研 Helix、智元已自研 GO-1，硬件公司有「自研大脑」倾向。
- **数据护城河可持续性**：自采跨本体数据昂贵，若硬件公司（如宇树、智元）开放数据集（如 AgiBot World），π 的数据壁垒可能被稀释。
- **估值与营收匹配**：56 亿估值对应商业化仍在早期，单位经济性待证。
- **与巨头关系**：Amazon 吸收 Covariant 的前车之鉴，π 是否会被巨头收购或挤压，是潜在风险。

---

## 9. 参考链接

> 部分为媒体报道，**非全部为官方一手资料**，引用时注意区分。

- Physical Intelligence 官网：https://www.pi.website/
- π₀ 官方博客：https://www.pi.website/blog/pi0
- π₀ 论文（arXiv）：https://arxiv.org/abs/2410.24164
- Sergey Levine 学术主页（UC Berkeley）：https://people.eecs.berkeley.edu/~svlevine/
- Bloomberg 融资报道（2025-11-20，$600M / $5.6B）：https://www.bloomberg.com/news/articles/2025-11-20/robotics-startup-physical-intelligence-valued-at-5-6-billion-in-new-funding
- π₀ 访谈（YouTube/Dwarkesh Patel 等）：https://www.youtube.com/watch?v=5mY71rGXAkM
- 相关方法笔记：[`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)、[`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)

---

## 10. 相关概念互链

- [`../../methods/vision-language-action.md`](../../methods/vision-language-action.md)——π₀ 是 flow matching VLA 的代表。
- [`../../methods/diffusion-policy.md`](../../methods/diffusion-policy.md)——flow matching 与扩散策略同源。
- [`../../methods/imitation-learning.md`](../../methods/imitation-learning.md)——π₀ 的跨本体数据多依赖 IL 采集。
- [`../../concepts/foundations/embodiment.md`](../../concepts/foundations/embodiment.md)——「大脑与硬件解耦」是 embodiment 议题的核心。
- [`../humanoid-overseas/figure-ai.md`](../humanoid-overseas/figure-ai.md)——软硬一体 vs 纯软件对照。
- [`../humanoid-china/agibot.md`](../humanoid-china/agibot.md)——智元 GO-1 对照。

> **再次提醒**：π 的最新融资（传闻 10 亿级）**待核**，模型权重开放状态可能变动，应以官方最新公告为准。
