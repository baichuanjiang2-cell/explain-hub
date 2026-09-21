# explain-hub

**把 GitHub 收藏（stars）和本机已装的 agent 技能，变成一套带架构图的深度讲解。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/baichuanjiang2-cell/explain-hub/actions/workflows/ci.yml/badge.svg)](https://github.com/baichuanjiang2-cell/explain-hub/actions/workflows/ci.yml)

[English](README.md) | 简体中文

```text
你的 GitHub 收藏，自动分级
├── 重点仓库 → 深度讲解：README（三角度 + file:line 锚点）
│             + 功能全景图 + 模块架构图 + 运行流程图
└── 长尾仓库 → 一页纸轻量讲解

你本机已装的 agent 技能，自动盘点
├── 独立技能 → 六段式讲解（定位/何时用/怎么触发/示例/配合/注意）
├── 成系列   → 合并成一篇总览（成员分工 + 配合链）
└── 失效的   → 记录根因 + 重装指引
```

explain-hub 是一个可复用的 [agent 技能](https://github.com/topics/agent-skills)，
从真实的批量讲解工程中沉淀而来，每张图表都经过独立视觉验收。装进
Claude Code、Codex、ZCode 或任何兼容 Agent Skills 的宿主，然后对你的
agent 说话即可。

## 产出示例

对 [`tj/commander.js`](https://github.com/tj/commander.js) 的一次真实运行——
完整深度讲解见 [`examples/tj__commander.js/`](examples/tj__commander.js/)
（带 `file:line` 锚点的 README + 三张图表 HTML + 渲染 PNG）：

<p align="center">
  <img src="docs/images/panorama.png" alt="功能全景图示例 — commander.js 的能力分区与模块" width="720">
</p>
<p align="center"><em>功能全景图（每个 A 级讲解固定三图之一）。</em></p>

## 它做什么

| 模式               | 输入             | 产出                                                                                                        |
| ------------------ | ---------------- | ----------------------------------------------------------------------------------------------------------- |
| **A · Stars 讲解** | GitHub 用户名    | `stars-explained/`：A 级项目各得一个文件夹（深度 README + 3 张交互式 SVG 图表 HTML + PNG 预览），B 级一页纸 |
| **B · 技能盘点**   | 本机（自动扫描） | `skills-explained/`：按系列合并的技能讲解 + 总索引 + 失效技能记录                                           |

内置的硬核机制（全部来自实战踩坑）：

- **图表强制 + 视觉验收**：三张图必须通过连线规则（直角肘线、标签蒙版留隙、虚线跨越桥）与复杂度预算（≤9 节点/≤3 分区/≤2 强调色），渲染后逐张截图自检，有子代理运行时再过一遍独立评审；
- **分级讲解**：A 级全量深挖、B 级一页纸兜底，分级启发式可调；
- **事实纪律**：每条结论带 `file:line` 锚点；“门面仓库”识别（星数高但 main 是空脚手架）、文档与代码漂移核查、未核实项如实标注；
- **环境降级链**：`gh` 缺失用公开 API、克隆失败用 raw 文件分析、Chromium 下载失败用 msedge 通道截图、子代理限流后重派。

## 安装

把本目录复制到任意技能发现路径之一：

```bash
# 用户级（所有项目可用）
cp -r explain-hub ~/.agents/skills/explain-hub

# 或项目级（仅当前仓库可用）
cp -r explain-hub .agents/skills/explain-hub
```

要求：能运行 agent（Claude Code / Codex / ZCode 等），`npx playwright` 与系统 Edge（用于图表截图验收）。图表引擎 [diagram-design](third-party/diagram-design/VENDORED.md) 已打包在本仓库内，技能运行时会自动探测/安装。

## 用法

对 agent 说人话即可，例如：

- “帮我讲解 GitHub 收藏，用户名 `octocat`”
- “我的 stars 里那些仓库都是干嘛的？挑重点讲透”
- “盘点一下我本机装了哪些技能，各有什么用”
- “讲讲 `vercel/next.js` 这个项目怎么跑起来的”

技能会先给清单和分级方案让你确认，然后分批产出，产物落在新目录里。所有文档用你提问的语言书写。

## 目录结构

```text
explain-hub/
├── SKILL.md                     # 主流程（渐进披露入口）
├── references/
│   ├── playbook.md              # 实战规则：降级链/门面仓库/漂移核查/限流重试/批量节奏
│   └── diagram-spec.md          # 三图规范：内容要点/SVG 铁律/渲染链/验收标准
├── assets/templates/
│   ├── readme-template.md       # 三角度深度 README 模板
│   └── *-template.html          # 三张图的页面骨架（皮肤/图例/无障碍已就位）
└── third-party/diagram-design/  # 图表引擎完整副本（MIT，见 VENDORED.md）
```

## 实战沉淀

`references/` 里的每条规则都来自真实批量运行的踩坑：API 降级链、门面仓库识别、
限流恢复、文档与代码漂移核查，以及三张图背后的视觉验收闭环。完整实战样例见
[`examples/`](examples/)。

## 参与贡献

规则本身欢迎迭代——带上你的实战证据来。请阅读 [CONTRIBUTING.md](CONTRIBUTING.md)（英文）。

## 许可

MIT，见 [LICENSE](LICENSE)。内置第三方软件：
[diagram-design](third-party/diagram-design/)（作者 Cathryn Lavery，MIT，
未修改的 vendored 副本，见其 [VENDORED.md](third-party/diagram-design/VENDORED.md)
与[许可文件](third-party/diagram-design/LICENSE)）。
