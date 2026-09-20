# explain-hub

**把 GitHub 收藏（stars）和本机已装的 agent 技能，变成一套带架构图的中文深度讲解。**
Turn your GitHub stars — or the agent skills installed on your machine — into tiered, diagram-backed explainers.

```
你的 GitHub 收藏 · N 个仓库
├── A 级 N 个 → 深度讲解：README（三角度 + file:line 锚点）
│                + 功能全景图 + 模块架构图 + 运行流程图（diagram-design 出品）
└── B 级 N 个 → 一页纸轻量讲解

本机已装技能 · N 个（软链接去重后）
├── 独立技能 → 六段式讲解（定位/何时用/怎么触发/示例/配合/注意）
├── 成系列   → 合并成一篇总览（成员分工 + 配合链）
└── 失效的   → 记录根因 + 重装指引
```

这是把一次真实的批量讲解工程（多批讲解与图表）沉淀成的可复用 agent skill。

## 它做什么 / What it does

| 模式 | 输入 | 产出 |
|---|---|---|
| **A · Stars 讲解** | GitHub 用户名 | `stars讲解/`：A 级项目各得一个文件夹（深度 README + 3 张交互式 SVG 图表 HTML + PNG 预览），B 级一页纸 |
| **B · 技能盘点** | 本机（自动扫描） | `skills讲解/`：按系列合并的技能讲解 + 总索引 + 失效技能记录 |

内置的硬核机制（全部来自实战踩坑）：

- **图表强制 + 视觉验收**：三张图必须通过连线规则（直角肘线、标签蒙版留隙、虚线跨越桥）与复杂度预算（≤9 节点/≤3 分区/≤2 强调色），渲染后逐张截图自检，有视觉验收代理时再过一遍独立评审；
- **分级讲解**：A 级全量深挖、B 级一页纸兜底，分级启发式可调；
- **事实纪律**：每条结论带 `file:line` 锚点；"门面仓库"识别（星数高但 main 是空脚手架）、文档与代码漂移核查、未核实项如实标注；
- **环境降级链**：`gh` 缺失用公开 API、克隆失败用 raw 文件分析、Chromium 下载失败用 msedge 通道截图、子代理限流后重派。

## 安装 / Install

把本目录复制到任意技能发现路径之一：

```bash
# 用户级（所有项目可用）
cp -r explain-hub ~/.agents/skills/explain-hub

# 或项目级（仅当前仓库可用）
cp -r explain-hub .agents/skills/explain-hub
```

要求：能运行 agent（Claude Code / Codex / ZCode 等），`npx playwright` 与系统 Edge（用于图表截图验收）。图表引擎 [diagram-design](third-party/diagram-design/VENDORED.md) 已打包在本仓库内，技能运行时会自动探测/安装。

## 用法 / Usage

对 agent 说人话即可，例如：

- “帮我讲解 GitHub 收藏，用户名 `octocat`”
- “我的 stars 里那些仓库都是干嘛的？挑重点讲透”
- “盘点一下我本机装了哪些技能，各有什么用”
- “讲讲 `vercel/next.js` 这个项目怎么跑起来的”

技能会先给清单和分级方案让你确认，然后分批产出，产物落在新目录里。

## 目录结构 / Layout

```
explain-hub/
├── SKILL.md                     # 主流程（双语，渐进披露入口）
├── references/
│   ├── playbook.md              # 实战规则：降级链/门面仓库/漂移核查/限流重试/批量节奏
│   └── diagram-spec.md          # 三图规范：内容要点/SVG 铁律/渲染链/验收标准
├── assets/templates/
│   ├── readme-template.md       # 三角度深度 README 模板
│   └── *-template.html          # 三张图的页面骨架（皮肤/图例/无障碍已就位）
└── third-party/diagram-design/  # 图表引擎完整副本（MIT，见 VENDORED.md）
```

## 已验证 / Proven on

一次真实运行：多仓库、多图表全部通过独立视觉验收；
本机技能全量讲解。本仓库的 `references/` 就是那次运行踩坑经验的沉淀。

## 许可 / License

MIT。`third-party/diagram-design/` 为独立 MIT 项目的副本，归属见
[VENDORED.md](third-party/diagram-design/VENDORED.md)。
