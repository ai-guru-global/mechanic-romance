# 机械浪漫 · Mechanic Romance

> **把冰冷的机械、数据与算法，组织成可被理解、可被复现、可被讨论的研究语言。**

**一个面向具身智能（Embodied AI）研究的开源语料库 + Demo 场景工作台。**

[![status](https://img.shields.io/badge/status-M1%20%E5%AE%8C%E6%88%90-brightgreen)](#roadmap)
[![license](https://img.shields.io/badge/license-research--corpus-lightgrey)](#)
[![language](https://img.shields.io/badge/lang-zh%20%2B%20en-blue)](#)
[![demos](https://img.shields.io/badge/scenarios-8%20%E5%9C%BA-green)](demo/)

---

## 30 秒读懂

| 维度 | 数字 |
| --- | --- |
| 📚 **方法笔记** | 10 篇（IL / RL / DP / ACT / VLA / VLN / PPO / SAC / DAgger / BC） |
| 🔧 **硬件笔记** | 22 篇（机械臂 / 人形 / 灵巧手 / 末端执行器 / 四足 / 传感器 / 计算） |
| 🎮 **仿真笔记** | 6 篇（Isaac Lab / Habitat / MuJoCo / PyBullet / Genesis + Sim-to-Real） |
| 💡 **概念笔记** | 13 篇（具身基础 / MDP / 感知 / 训练） |
| 🧪 **Demo 场景** | **8 场**（覆盖 IL / RL / VLA / 导航 / 灵巧手 8 大类） |
| 📄 **核心论文笔记** | 4 篇（Diffusion Policy / ACT / RT-2 / OpenVLA） |
| 🏢 **行业 / 基准** | 5 家公司图谱 + 5 个评测基准 |
| 🧭 **研究里程碑** | M0-M1 ✅ · M2 🟡 · M3-M4 ⚪（进度以 [docs/roadmap.md](docs/roadmap.md) 为准） |

---

## 这不是另一个 GitHub 仓库

> *「机械浪漫」这个名字反讽地指向一个事实——**机器没有浪漫，但研究机器的过程可以**。*

传统研究语料库往往把"方法 + 实验 + 代码"拆成三个互不相通的孤岛——论文在 arXiv，代码在 GitHub，笔记在 Notion / Obsidian。**机械浪漫** 拒绝这种割裂：

- ✅ **一篇笔记 = 一篇可引用的方法 / 硬件 / 仿真沉淀**
- ✅ **一个场景 = 一份能直接跑 / 能直接讲解的研究产物**
- ✅ **一句话原则 = 看完即懂，讲完即用**

**它是什么**：
- 🔍 **可检索的语料库**——论文方法不再散落各处
- 🎬 **可演示的 Demo 工作台**——抽象的 SOTA 数字变成"看得见"
- 📓 **可回放的研究日志**——从选题到跑通的全过程

**它不是什么**：
- ❌ 不是训练框架（不重写 PyTorch / Isaac Lab）
- ❌ 不是仿真器（不重写物理引擎）
- ❌ 不是教科书（不写公式推导，只写"为什么"和"怎么用"）

---

## 🧪 8 个 Demo 场景

> 每个场景用统一模板定义任务、观测、动作、评估、硬件——**可复现、可比较、可讲解**。

| # | 场景 | 类别 | 代表方法 | 演示形态 |
| - | --- | --- | --- | --- |
| **01** | [扩散策略·桌面堆叠](demo/scenarios/01-扩散策略-桌面堆叠/README.md) | 🦾 单臂 IL | Diffusion Policy | 推方块到目标点 |
| **02** | [ACT·双手叠衣](demo/scenarios/02-ACT-双手叠衣/README.md) | 🦾👐 双手 IL | Action Chunking Transformer | 双手叠毛巾 |
| **03** | [RT-2·自然语言指令分拣](demo/scenarios/03-RT2-自然语言指令分拣/README.md) | 🧠 VLA | RT-2（VLM + 动作 token） | 听指令分拣 |
| **04** | [OpenVLA·7B 复现](demo/scenarios/04-OpenVLA-7B-开源VLA/README.md) | 🧠 开源 VLA | OpenVLA-7B + BridgeData | 桌面倒水 |
| **05** | [Figure 02 + Helix·端到端人形](demo/scenarios/05-Figure02-端到端人形/README.md) | 🧍 商用人形 | Helix（双系统 VLA） | 开冰箱拿可乐 |
| **06** | [强化学习·Franka 抓杯](demo/scenarios/06-强化学习-Franka抓杯/README.md) | 🔁 RL | PPO + Isaac Lab | 仿真中学会抓杯 |
| **07** | [视觉语言导航·VLN-CE](demo/scenarios/07-视觉语言导航-VLN-CE/README.md) | 🧭 导航 | HAMT + Habitat | 3D 室内听指令导航 |
| **08** | [灵巧手·五指旋转立方体](demo/scenarios/08-灵巧手-五指旋转立方体/README.md) | 🖐 灵巧手 | PPO + Allegro/LEAP | 立方体掌心旋转 |

> 想看全景？→ [Demo 类型全景](demo/_demo-types-overview.md)

---

## 🗺 知识地图

```
mechanic-romance/
├── 📚 corpus/              ← 语料库（笔记与索引）
│   ├── methods/            ← 10 篇方法笔记
│   │   ├── imitation-learning.md
│   │   ├── behavioral-cloning.md
│   │   ├── dagger.md               ← 在线纠错 IL
│   │   ├── reinforcement-learning.md
│   │   ├── ppo.md                  ← 具身 RL 标配
│   │   ├── sac.md                  ← off-policy RL
│   │   ├── diffusion-policy.md
│   │   ├── act.md                  ← 双手操作范式
│   │   ├── vision-language-action.md
│   │   └── vision-language-navigation.md
│   ├── hardware/           ← 22 篇硬件笔记
│   │   ├── arms/                   ← Franka / UR / KUKA
│   │   ├── humanoids/              ← Figure 02 / H1
│   │   ├── hands/                  ← Allegro / LEAP / Shadow
│   │   └── end-effectors/          ← Robotiq 2F-85
│   ├── simulation/         ← 5 个仿真平台
│   │   ├── platforms/              ← Isaac / Habitat / MuJoCo ...
│   │   └── sim-to-real/            ← 域随机化
│   ├── concepts/           ← 13 个概念笔记
│   │   ├── foundations/            ← 具身 / 形态 / Sim-to-Real
│   │   ├── mdp/                    ← 策略 / 奖励 / 观测 / 动作
│   │   └── training/               ← 微调
│   ├── papers/             ← 论文笔记（按方法分类）
│   ├── datasets/           ← 数据集笔记
│   ├── benchmarks/         ← 评测基准
│   ├── industry/           ← 行业公司图谱
│   └── surveys/            ← 综述 / 报告
│
├── 🎬 demo/                ← 8 个 Demo 场景
│   ├── scenarios/                 ← 场景定义（核心）
│   ├── prompts/                   ← 提示词与脚本
│   └── assets/                    ← 素材
│
├── 🔬 research/            ← 研究过程留痕
│   ├── proposals/                 ← 课题提案
│   ├── notes/                     ← 阅读笔记
│   └── experiments/               ← 实验记录
│
├── 📖 docs/                ← 项目级文档
│   ├── roadmap.md                 ← 研究路线图（进度唯一事实源）
│   └── conventions.md             ← 写作约定
│
└── 🗄 archive/             ← 旧项目归档（Piroom，勿动）
```

---

## 🌊 方法演化时间线（2022-2026）

```
2022 ─ RT-1 ────────────────── 行为克隆 + 7-DoF token
  │
2023 ─ RT-2 ────────────────── 首个 VLA，VLM + 动作 token（开 VLA 时代）
  │   Diffusion Policy ─────── 多模态动作建模（IL 主力）
  │   Open X-Embodiment ────── 22 机构联合数据集
  │   ACT ──────────────────── 动作分块 + CVAE（双手操作）
  │   HAMT ─────────────────── Transformer + 历史（VLN SOTA）
  │
2024 ─ OpenVLA-7B ─────────── RT-2 的开源复现（单卡可微调）
  │   π₀ (Pi-Zero) ─────────── Flow matching 替代 token（高频控制）
  │   π₀.5 ─────────────────── 跨环境泛化
  │   Mobile ALOHA ─────────── ACT 配移动底盘
  │   DDEX ─────────────────── Diffusion Policy + 灵巧手
  │   Octo ─────────────────── 开源通用机器人策略（27M）
  │
2025 ─ Helix (Figure 02) ───── 双系统 VLA（System 1 + System 2）
  │   π₀.₅ 升级 ────────────── 通用机器人基础模型
  │   GO-1 (Generalist) ────── 跨本体 + 跨任务基础模型
  │   Helix 02 ─────────────── 多机协同
  │
2026 ─ GO-2 / Helix 03 ────── 量产人形 + 通用基础模型
        ↑ 你在这里 ↑
```

> 详细笔记见 [`corpus/methods/`](corpus/methods/)

---

## 🎯 8 大研究主题全覆盖

| 主题 | 核心问题 | 代表方法 | 在本仓库 |
| --- | --- | --- | --- |
| 🦾 **单臂操作** | 如何抓放物体？ | BC / DP / VLA | demo 01 · 03 · 04 · 06 |
| 🦾👐 **双手操作** | 如何协调双臂？ | ACT / Mobile ALOHA | demo 02 |
| 🧍 **人形机器人** | 通用身体如何决策？ | Helix / π₀ / GO-1 | demo 05 |
| 🖐 **灵巧手** | 5 指能做什么？ | PPO / DDEX / DexCap | demo 08 |
| 🧭 **导航** | 如何听懂指令走路？ | HAMT / DUET / NaVid | demo 07 |
| 🧠 **基础模型** | 一个模型统治一切？ | RT-2 / OpenVLA / π₀ | demo 03 · 04 · 05 |
| 🔁 **学习范式** | 演示 / 试错 / 纠错？ | IL / RL / DAgger | 跨 8 demo |
| 🔄 **Sim-to-Real** | 仿真能迁到真机吗？ | 域随机化 / 系统辨识 | 跨 6 demo |

---

## 🛠 如何使用本仓库

### 🎓 我是研究者

```bash
# 1. 克隆
git clone https://github.com/allengaller/mechanic-romance.git
cd mechanic-romance

# 2. 按需浏览
ls corpus/methods/        # 找方法笔记
ls demo/scenarios/        # 找场景定义
cat docs/roadmap.md       # 看研究路线
```

**推荐路径**：
- 🚀 **快速入门**：看 [Diffusion Policy 笔记](corpus/methods/diffusion-policy.md) + [demo 01](demo/scenarios/01-扩散策略-桌面堆叠/README.md)
- 🧭 **了解 VLA**：看 [vision-language-action.md](corpus/methods/vision-language-action.md) → [demo 03](demo/scenarios/03-RT2-自然语言指令分拣/README.md) → [demo 04](demo/scenarios/04-OpenVLA-7B-开源VLA/README.md)
- 🤖 **看前沿**：看 [demo 05](demo/scenarios/05-Figure02-端到端人形/README.md) + [Figure AI 行业笔记](corpus/industry/humanoid-overseas/figure-ai.md)

### 🛠 我是工程师

**跑一个 demo**（以 demo 01 为例）：

```bash
# 1. 安装仿真环境
pip install mujoco diffusers torch

# 2. 下载 Diffusion Policy 权重
wget https://diffusion-policy.cs.columbia.edu/weights/pusht.ckpt

# 3. 启动仿真评估
python -c "
import torch
from diffusers import DDPMScheduler
ckpt = torch.load('pusht.ckpt')
print('Policy loaded:', ckpt.keys())
"

# 4. 打开 demo 01 的详细步骤
cat demo/scenarios/01-扩散策略-桌面堆叠/README.md
```

### ✍️ 我想贡献

1. **读 [CONTRIBUTING.md](CONTRIBUTING.md)** 了解命名与引用约定
2. **复制 [demo/scenarios/_template.md](demo/scenarios/_template.md)** 写新场景
3. **在 [corpus/](corpus/) 对应目录加新笔记**（方法 / 硬件 / 仿真）
4. **PR 时附**：「我加了什么 / 解决什么 / 引用源」

---

## 🛣 Roadmap

| 阶段 | 目标 | 状态 | 完成日期 |
| --- | --- | --- | --- |
| **M0 · 骨架** | 建立语料库与 Demo 场景项目结构 | 🟢 完成 | 2026-08-02 |
| **M1 · 语料沉淀** | 完成核心 10 篇方法 + 8 类硬件 + 5 个仿真平台笔记 | 🟢 完成 | 2026-08-05 |
| **M2 · 场景定义** | 落地 8 大类 8 个可演示场景 | 🟡 进行中 | — |
| **M3 · 实验复现** | 复现 1 个基线方法并记录结果 | ⚪ 待开始 | — |
| **M4 · 阶段总结** | 输出阶段性研究报告 / Demo 视频 | ⚪ 待开始 | — |

> 详细路线图：[`docs/roadmap.md`](docs/roadmap.md)

---

## 🧭 项目精神

> *「把一篇论文读懂很容易，把它**讲给别人听**很难；*
> *把一个方法跑通很容易，把**为什么跑得通**讲清楚很难；*
> *把一项技术用起来很容易，把**它在整个领域的位置**看清楚很难。*
>
> *机械浪漫追求的不是'难'本身，而是'把难的东西组织得美'——*
> *这是工程师对世界的浪漫，也是浪漫对工程师的敬意。」*

---

## 📜 引用与致谢

### 引用

如果你在研究中参考了本仓库，请引用：

```bibtex
@misc{mechanic-romance,
  title = {机械浪漫 · Mechanic Romance: A Research Corpus and Demo Workbench for Embodied AI},
  author = {Galler, Allen},
  year = {2026},
  url = {https://github.com/allengaller/mechanic-romance}
}
```

### 致谢

- 所有引用论文的作者——他们才是真正的研究者
- [Hugging Face LeRobot](https://github.com/huggingface/lerobot) / [Isaac Lab](https://github.com/NVlabs/IsaacLab) / [Diffusion Policy](https://diffusion-policy.cs.columbia.edu/) 等开源项目
- [arXiv](https://arxiv.org/) / [Papers with Code](https://paperswithcode.com/) / [CrossMoe](https://crossmoe.ai/) 提供的研究基础设施
- 所有为具身智能开源生态贡献的工程师与研究者

---

## 📊 仓库统计

> 最后更新：2026-09-02

```
corpus/
├── methods/        10 篇
├── hardware/       22 篇（7 类）
├── simulation/      6 篇（5 平台 + Sim-to-Real）
├── concepts/       13 篇
├── papers/          4 篇笔记
├── datasets/        1 个数据集
├── benchmarks/      5 个评测基准
├── industry/        5 家公司
└── surveys/         1 篇综述

demo/
└── scenarios/       8 场 Demo（设计文档）
```

---

## 🔗 相关项目

- [`kudig-io/kudig`](https://github.com/kudig-io/kudig) — K8s 节点诊断工具（70+ analyzers）
- [`ai-guru-global/resolve-agent`](https://github.com/ai-guru-global/resolve-agent) — AIOps / RAG Agent
- [`standup-coder/mcp4coder`](https://github.com/standup-coder/mcp4coder) — MCP 工具聚合
- [`opendemo-work/opendemo`](https://github.com/opendemo-work/opendemo) — 518+ 技术 demo

---

## 📫 联络

- **作者**：Allen Galler（法喜）
- **GitHub**：[@allengaller](https://github.com/allengaller)
- **Email**：allengaller@qq.com
- **个人站**：[allengaller.github.io](https://allengaller.github.io)

---

> *「机器没有浪漫，但研究机器的人，可以有。」*  🤖
