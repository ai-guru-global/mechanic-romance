# Demo 类型全景 · Demo Types Overview

> 这份不是场景定义，是**我能为你做的 demo 类型一览**。帮你判断哪些方向值得投入场景。
> 当前已落地的具身智能 demo 见 `scenarios/`；本文件只给方向概览 + 我能交付什么。

---

## 1. 具身智能 · Embodied AI（核心方向，与本仓直接相关）

仓库已经铺好 `corpus/` 语料 + `demo/scenarios/` 模板，下面是**该方向我能直接交付的 demo 类型**：

| 类别 | 典型 demo | 我能交付什么 | 难度 |
| --- | --- | --- | --- |
| **模仿学习（IL）** | Diffusion Policy 推积木、ACT 叠衣、BC 抓杯子 | 场景脚本 + 仿真环境 + 策略训练 notebook | 🟢 可跑 |
| **强化学习（RL）** | Isaac Gym 机械臂 reach / pick | Sim-to-Real 配置 + 奖励函数设计 | 🟡 需 GPU |
| **视觉-语言-动作（VLA）** | RT-2 / OpenVLA / π0 自然语言指令 | 指令模板 + prompt + 评估脚本 | 🟡 需 GPU/权重 |
| **基础模型 / 通用策略** | Octo、OpenVLA-7B、π0 跨机器人迁移 | 数据集筛选 + 微调 pipeline | 🔴 重量级 |
| **人形机器人** | Figure 02 + Helix、Optimus、1X Neo | 商用 demo 复盘 + 任务编排 | ⚪ 商用 only |
| **导航** | Habitat ObjectNav、CoNavi、VLN | 场景图 + 拓扑图评测 | 🟡 中等 |
| **Sim-to-Real** | Domain Randomization、Real2Sim | 随机化参数表 + 迁移报告 | 🟡 中等 |
| **灵巧手** | DexCap、Allegro 五指操作 | 仿真 URDF + 数据采集脚本 | 🔴 硬件门槛 |

> 详细场景见 [`scenarios/`](scenarios/)。本次首发抓了 5 个最佳实践：`01` ~ `05`。

---

## 2. AI Agent / RAG / MCP（你能跑的"软件侧具身"）

| 类型 | 典型 demo | 我能交付什么 |
| --- | --- | --- |
| **Tool-use Agent** | 浏览器自动化、文件处理、Shell agent | prompt 工程 + 工具 schema + trace 回放 |
| **RAG 全链路** | PDF 知识库、代码库问答、混合检索 | 切片策略 + 嵌入评估 + 端到端 demo |
| **MCP 工具聚合** | 多 MCP server 协同、跨工具调用 | MCP server 模板 + 协议对接 + 评测 |
| **Multi-Agent** | 角色分工、辩论、流水线 | 协作图 + 通信协议 + 案例 |
| **Computer Use** | 桌面 / 网页 GUI 代理 | 任务脚本 + 截图回放 + 异常恢复 |

---

## 3. LLM 应用 / AIGC

| 类型 | 典型 demo |
| --- | --- |
| **结构化生成** | 合同抽取、表格理解、JSON 严格输出 |
| **多模态理解** | 视频问答、图表 OCR、PPT 解析 |
| **AIGC 流水线** | 文 → 图 → 视频 → 配音 → 字幕，端到端内容生产 |
| **长上下文** | 全书问答、代码仓库级 RAG |
| **微调** | LoRA / QLoRA / DPO 小模型定制 |

---

## 4. Cloud Native / SRE（与你老本行相关）

| 类型 | 典型 demo |
| --- | --- |
| **可观测性** | eBPF 探针、OTel 链路、Prometheus 规则 |
| **诊断工具** | 类似 Kudig 的 node 自检 / etcd 自检 |
| **Operator** | CRD + Controller（如 EtcdGuardian） |
| **CI/CD 模板** | GitLab/GitHub Actions 最佳实践骨架 |
| **平台工程** | Backstage 插件、IDP 模板 |

---

## 5. 视觉 / 前端 / 交互

| 类型 | 典型 demo |
| --- | --- |
| **信息图** | Notion 风格手绘图（已有 `notion-infographic-v2` skill） |
| **可视化页** | 流程图、架构图、对比表（已有 `visual-page` skill） |
| **数据大屏** | 实时面板、3D 可视化 |
| **品牌站** | 极简个人站（参考 allengaller.github.io） |

---

## 6. 内容 / 知识产品

| 类型 | 典型 demo |
| --- | --- |
| **播客** | AI 对话播客（参考 LeetCast） |
| **电子书** | 选题 → 提纲 → 章节 → 装帧 |
| **课程大纲** | 知识图谱 → 模块化课程 |
| **Newsletter** | 自动选题 + 摘要 + 排版 |

---

## 7. 数据 / 评测

| 类型 | 典型 demo |
| --- | --- |
| **评测集** | LLM 行为评测、Agent 工具调用准确率 |
| **数据管线** | 抓取 → 清洗 → 去重 → 入库 |
| **A/B 框架** | 离线评估 + 在线分流 |

---

## 怎么选

- **先验证方法/算法** → 走「具身智能 · 模仿学习 / VLA」
- **先做产品形态** → 走「AI Agent / RAG / MCP」
- **先做品牌/内容** → 走「视觉 / 内容」
- **先做内部工具** → 走「Cloud Native / SRE」

---

## 下一步

- 已抓的具身智能 demo：`01-扩散策略-桌面堆叠` / `02-ACT-双手叠衣` / `03-RT2-自然语言指令` / `04-OpenVLA-7B` / `05-Figure-02-端到端人形`
- 后续要扩到其他方向（比如 MCP、Agent），告诉我方向就行，我按 `scenarios/_template.md` 继续起草。
