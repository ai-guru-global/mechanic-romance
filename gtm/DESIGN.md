# DESIGN — 机械浪漫 GTM 产品页

<!-- impeccable:design-sidecar 1 · surface: GTM/index.html · mode: introduce -->

记录已构建页面（ground truth），非意图。世界：**工程蓝图 / 研究图纸**——语料库即一套图纸库，笔记是图纸，场景是工序，页面本身就是一张带图签的工程图。与系列页（ResolveAgent 调度室 / FDE Scope）共享 Barlow + Noto Sans SC + JetBrains Mono 字体三角，但视觉世界独立。

## Palette

| Token | Hex | 用途 |
|---|---|---|
| `--bg` | #0a141f | 页面底（蓝图深蓝黑，56px 网格线 rgba(148,196,228,.09)） |
| `--panel/--panel2` | #102133 / #0d1b2b | 表格/卡片面板 |
| `--ink/--ink2/--ink3` | #e9f1f8 / #a9bccd / #7e93a8 | 正文三级（均 ≥4.5:1） |
| `--rose` | #ff5f76 | 「浪漫」强调色：h1 em、主 CTA、Σ合计、M2 版次、可以有。全页唯一暖色，刻意克制 |
| `--cyan` | #5fb8d4 | 图纸标注色：eyebrow、尺寸标注线、OP 工序钢印、代号 |
| `--ok/--warn/--idle` | #3fb950 / #ffc94d / #5c6f84 | 状态：完成 / 进行中+待核实 / 待开始 |

## Components

- **图签 `.tblock`**：工程图纸标题栏（项目/定位/领域/图号 MR-2026-001/版次/比例/绘制/日期/语言）+ foot 条「语料即图纸 · 场景即工序」。hero 右栏，签名组件。
- **十字定位标 `.cross`**：hero 四角青色 registration marks。
- **尺寸标注 `.dim`**：每节 SECTION 编号行，两端带端线竖杠。
- **BOM 明细表 `.bom`**：9 行语料 + Σ合计 67（玫瑰），列 NO./代号/名称/数量/内容。
- **工序卡 `.scard`**：8 场景，OP-01…08 青色钢印 + 类别 + 琥珀虚线「设计文档」章。
- **主题表 `.thtable`**：T1-T8 四列。
- **装配时间线 `.tline`**：年份节点，待核实年份琥珀虚线节点 + tag。
- **路线图 `.rmap`**：M0-M4 状态 chip。

## 内容诚实律（不可退化）

8 场景必须标注**设计文档**（琥珀虚线章 + 底部注）；M3 实验复现=待开始；时间线 2025 部分待核实、2026 待核实（沿用 README 口径）；统计数字同步自 README（2026-09-02 版）。页面不得出现「可运行 demo」等夸大表述。

## Motion / Responsive / A11y

- 仅一次性 `.reveal` 显现（IntersectionObserver），`prefers-reduced-motion` 全关；无 glow 无装饰动效。
- ≤1020 hero 单列、BOM 隐代号列；≤860 场景卡/creed 单列、navlinks 隐藏；≤640 BOM 隐内容列、主题表纵排、路线图隐日期。
- 跳转链接、role=table/row、aria-label、对比 ≥4.5:1。

## 发布

Meoo 项目 **Mechanic Romance GTM**（`ib1uccbep65l`）→ https://ib1uccbep65l.meoo.fun ，静态路径 `meoo deploy --skip-build --force`（dist/ 为 index.html 副本）。CDN 会把 `<title>` 改写为项目名「Mechanic Romance GTM」并注入水印，均为平台行为；若要更优雅的标签页标题需在 Meoo 设置改项目名。
