# 灵巧手全景对比 · Dexterous Hands Landscape 2024+

> 5 指灵巧手在 2023-2025 经历"**研究民主化 + 商用爆发**"双重变革：LEAP Hand 把价格从 $1.5 万打到 $3000，OpenAI / Physical Intelligence / 各类人形公司同时涌入。本文横纵对比 12 款主流灵巧手，给出选型决策树。

> 最后更新：2026-08

---

## 1. 灵巧手四代演化

| 世代 | 代表 | 时间 | 关键特征 |
| --- | --- | --- | --- |
| **1. 工业级** | [Shadow Hand](shadow-hand.md) | 2005-2018 | $10-15 万、24-DoF、军工级、闭源 |
| **2. 商用级** | [Allegro](allegro.md) | 2016-2023 | $1.5-2 万、16-DoF、研究主流、SDK 开源 |
| **3. 开源级** | [LEAP Hand](leap-hand.md) | 2023+ | **$3000、16-DoF、全开源、Sim-to-Real 友好** |
| **4. 量产级** | 多家入局 | 2024+ | 配合人形机器人量产，$5k-10k 区间 |

> 2024+ 的趋势是**LEAP 价格 + Shadow 能力 + 开源生态**——这正是 04 阶段。

---

## 2. 12 款主流灵巧手横评

| 手 | 厂家 | DoF | 指 | 触觉 | 价格 | 开源 | 状态 | 定位 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Shadow Hand** | Shadow Robot（英） | **24** | 5 | 20+ | $10-15 万 | 半 | 🟡 | 工业级标杆 |
| **Allegro Hand** | Wonik（韩） | 16 | 4 | 6×6 | $1.5-2 万 | SDK | 🟢 | 商用主流 |
| **LEAP Hand** | Stanford（美） | 16 | 4 | 无 | **$3000** | **全** | 🟢 | 开源新锐 |
| **DLR Hand** | DLR（德） | 12-15 | 4-5 | 12 | $5-8 万 | 半 | ⚪ | 研究向 |
| **HUG Hand** | ENSTA（法） | 16-20 | 5 | 中等 | $1-2 万 | 半 | 🟡 | 学术 |
| **Seed Robotics** | Seed（葡） | 12-17 | 5 | 中等 | $1-3 万 | 半 | 🟡 | 学术 / 工业 |
| **Psychonic** | Psychonic（美） | 6 | 5 | 力 | $10-15k | 闭 | 🟢 | 人形公司用 |
| **Clone Hand** | Clone（美） | 17 | 5 | 中 | 待定 | 半 | 🟡 | 人形公司用 |
| **Figure 灵巧手** | Figure（美） | 16-20 | 5 | 触觉 | N/A | 闭 | 🟢 | Helix 平台 |
| **1X Hand** | 1X（挪/美） | 15-20 | 5 | 触觉 | N/A | 闭 | 🟢 | Neo 平台 |
| **Apptronik Apollo** | Apptronik（美） | 10-15 | 5 | 中等 | N/A | 闭 | 🟢 | Apollo 平台 |
| **Unitree Dex3** | 宇树（中） | 7-9 | 3 | 触觉 | N/A | 闭 | 🟢 | H1 平台 |

> 注：商用人形公司灵巧手大部分未公开销售，配套机器人出货。

---

## 3. 5 维选型决策

### 3.1 价格 vs 能力

```
$3k ──── LEAP Hand（开源民主化）
        ↓
$15k ─── Allegro（商用研究主流）
        ↓
$50k ─── Psychonic / Clone / 单位定制
        ↓
$150k ── Shadow Hand（工业级）
```

### 3.2 DoF vs 控制难度

| DoF 范围 | 难度 | 训练成本 | 适用 |
| --- | --- | --- | --- |
| 6-9 | **低** | 100k RL 步 | 简单抓取 |
| 12-16 | **中** | 1-10M 步 | in-hand rotation（[demo 08](../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)） |
| 17-20 | **高** | 10-100M 步 | 工具使用、乐器演奏 |
| 24+ | **极高** | 100M+ 步 | 拟人操作、神经科学 |

### 3.3 指段数（4 vs 5）

| 维度 | 4 指（Allegro/LEAP） | 5 指（Shadow/Psychonic） |
| --- | --- | --- |
| 抓取稳定 | 中（小指侧弱） | **高** |
| 仿真难度 | 中 | 高 |
| 样本效率 | 高（30-50%） | 低 |
| 工具使用 | 简单工具 | **复杂工具**（如螺丝刀） |
| 拟人 | 72% 任务够用 | 99% |

> **绝大多数研究 4 指就够了**（参考 OpenAI Dactyl 结论）。

### 3.4 触觉

| 触觉配置 | 适用 |
| --- | --- |
| **无触觉** | 仿真/规则任务 |
| **指尖**（DIGIT 1 个/指） | 滑移检测 |
| **指段+指尖**（Allegro 6×6） | 通用灵巧 |
| **全手密集**（Shadow 20+） | 复杂 dexterous |

### 3.5 开源程度

| 等级 | 代表 | 改造能力 |
| --- | --- | --- |
| **完全开源**（CAD+SDK+URDF） | LEAP | 高（换指段、加触觉） |
| **SDK 开源**（URDF+代码） | Allegro | 中（不可改硬件） |
| **半开源**（部分代码） | Shadow / DLR | 低 |
| **完全闭源** | Psychonic / 商用 | 无 |

---

## 4. 2024+ 趋势

### 4.1 价格断崖（$1.5 万 → $3000）

LEAP Hand 出现（2023）打破 5 年价格壁垒，**研究民主化** 趋势确立：
- 学生实验室可承担（$3000 vs $15k）
- Sim-to-Real 友好（论文报告 gap < 5%）
- 全开源 → 改指段 1 天可换

### 4.2 商用灵巧手激增（2024-2025）

伴随人形机器人量产，灵巧手成"**机器人必配**"：

- **Figure 02 + Helix 灵巧手**（2024-09 发布）—— 双 16-DoF + 触觉
- **1X Neo + 灵巧手**（2024-10）—— 15-DoF + 力反馈
- **Apptronik Apollo**（2024-08）—— Mercedes 工厂部署
- **宇树 H1 + Dex3-1**（2024-08）—— 3 指 7-DoF
- **智元 A2 + 灵犀 X1 手**（2024-09）

> 商用灵巧手大部分**不单独销售**（绑定机器人），价格 1-3 万美元配套出货。

### 4.3 触觉传感器普及

**DIGIT（$15/个）** 把高分辨率触觉带进开源：
- Meta 2020 开源
- 用现成相机 + 硅胶做"视触觉"
- 装上 LEAP/Shadow 立刻有触觉
- 2024+ 是"灵巧手 + DIGIT"组合标配

### 4.4 算法与硬件协同

灵巧手进步 = 硬件 + 算法 + 数据 三者协同：
- **数据**：DexCap（2024）、Humanoid Diffusion Policy（2024）
- **算法**：DDEX（2024）、HATO（2024）、CrossFormer（2024）
- **硬件**：LEAP v2、DIGIT Gen 2、Allegro v5

---

## 5. 选型决策树

```
你的预算？
├── < $5000
│   ├── 研究/学生 → LEAP Hand（[leap-hand.md](leap-hand.md)）
│   └── DIY → 3D 打印 + Dynamixel 自己组装
├── $5000 - $30k
│   ├── 学术研究 → Allegro（[allegro.md](allegro.md)）
│   ├── 商业 demo → Psychonic / Clone（联系厂商）
│   └── 工业项目 → DLR / HUG
├── $30k - $100k
│   └── 长期/工业级 → Shadow Hand（[shadow-hand.md](shadow-hand.md)）
└── 配套人形
    ├── Figure 02 → Helix 灵巧手（[demo 05](../../demo/scenarios/05-Figure02-端到端人形/README.md)）
    ├── 1X Neo → 1X 灵巧手
    ├── 宇树 H1 → Dex3-1
    └── 智元 A2 → 灵犀 X1
```

---

## 6. 2025-2026 值得关注

1. **LEAP Hand v2**（2025-2026 计划）：加触觉 + 更高负载
2. **Apptronik Apollo 灵巧手单独发售**？—— 商用灵巧手有可能"**单独销售**"
3. **国内开源灵巧手**：清华 / 浙大 / 中科院 都做了 LEAP 同类
4. **触觉标准化**：DIGIT 之外可能出现"标准触觉接口"
5. **灵巧手 + VLA 融合**：Helix / π0 都把灵巧手当 VLA 落地点

---

## 7. 相关笔记

- 单手笔记：[allegro.md](allegro.md) · [leap-hand.md](leap-hand.md) · [shadow-hand.md](shadow-hand.md)
- 灵巧手专题：[control](dexterous-hands-control.md) · [data-collection](dexterous-hands-data-collection.md) · [sim2real](dexterous-hands-sim2real.md) · [algorithms](dexterous-hands-algorithms.md)
- 人形机器人：[figure-02.md](../humanoids/figure-02.md) · [unitree-h1.md](../humanoids/unitree-h1.md) · [tesla-optimus.md](../humanoids/tesla-optimus.md) · [1x-neo.md](../humanoids/1x-neo.md) · [unitree-g1.md](../humanoids/unitree-g1.md) · [agibot-a2.md](../humanoids/agibot-a2.md)
- 四足：[boston-dynamics-spot.md](../quadrupeds/boston-dynamics-spot.md) · [unitree-go2.md](../quadrupeds/unitree-go2.md)
- 仿真：[nvidia-isaac](../../simulation/platforms/nvidia-isaac.md) · [mujoco](../../simulation/platforms/mujoco.md)
- 方法：[reinforcement-learning](../../methods/reinforcement-learning.md) · [diffusion-policy](../../methods/diffusion-policy.md) · [imitation-learning](../../methods/imitation-learning.md)
- Demo：[08-灵巧手-五指旋转立方体](../../demo/scenarios/08-灵巧手-五指旋转立方体/README.md)

---

## 8. 参考

**官方来源**：

- LEAP Hand 官方站（来源：https://leaphand.com/，访问于 2026-09-02）
- Allegro Hand 官方站（来源：https://www.allegrohand.com/，访问于 2026-09-02）
- Shadow Dexterous Hand 官方产品系列页（来源：https://shadowrobot.com/dexterous-hand-series/，访问于 2026-09-02）
