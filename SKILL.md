---
name: explain-hub
description: >-
  Deep-dive explainer engine for a GitHub account's starred repos and for the
  agent skills installed on this machine. Produces tiered explainers: full
  three-angle deep dives (what it does / how the code divides work / how a core
  feature runs and how to use it) with panorama, architecture and runtime-flow
  diagrams for key repos, plus one-page notes for the long tail — all indexed.
  Use when the user wants to 讲解/解读/搞懂 GitHub 收藏、stars、收藏的仓库,
  asks 一个仓库/技能是干什么的、怎么用、怎么跑起来的, says "explain my stars",
  "盘点/讲解 本机已装的技能/installed skills", pastes a GitHub username asking
  for an explainer, or names explain-hub — even if they only say
  "帮我把这些仓库都讲一遍" or "这个 skill 有什么用".
---

# Explain Hub — 讲解引擎 / The Explainer Engine

Two modes, one engine. **Mode A** turns a GitHub account's stars into tiered
explainers. **Mode B** inventories the agent skills installed on this machine
and explains each. Both share the same principles, the same diagram pipeline,
and the same index discipline.

**产物语言跟随用户的提问语言，默认中文。** All user-facing documents are
written in the language the user is speaking (default: Chinese). Keep code
identifiers, file paths and CLI terms in their original form.

## 通用铁律 / Non-negotiables

1. **事实优先，锚点可核对。** Every factual claim in an explainer carries a
   `file:line` anchor (for code) or a source note (for README-derived facts).
   Never present a guess as fact; write down what you could not verify.
2. **图表强制。** Every deep-dive (A-tier) explainer ships three diagrams:
   功能全景图 / 模块架构图 / 运行流程图, drawn with the bundled
   diagram-design skill. If diagram-design is missing, install it first
   (§ diagram-design 联动) — do not degrade to text-only without telling the
   user. Lightweight (B-tier) notes carry no diagrams by design.
3. **先对齐，再批量干活。** Before producing anything, present the inventory
   and the plan as a checklist with per-item recommendations (✅ 建议 /
   ⬜ 可跳过) and let the user pick. Batch size 5–8. Never re-ask for items
   the user already decided.
4. **索引用完必更新。** The index README is the deliverable's front door:
   every finished item flips its status marker the moment it is done.
5. **临时产物用完即清。** Clones live in a temp dir, survive until QA passes,
   and are deleted afterwards (unless the user asked to keep them).

## 参数 / Tunables (defaults — override on request)

| 参数 | 默认 | 说明 |
|---|---|---|
| A 级数量上限 | 25 | Deep-dive treatment cap |
| 批量大小 | 5–8 | Items per confirmation round |
| 输出目录 | `stars讲解/`、`skills讲解/` | Created in cwd |
| 克隆目录 | `_repos/` | Temp, deleted after QA |
| B 级资料来源 | 仓库 README（API 抓取） | No clone |

## 模式 A · GitHub Stars 讲解 / Stars mode

1. **拿到 stars 列表。** Prefer `https://api.github.com/users/<name>/starred?per_page=100`
   (public, no auth; paginate). `gh` is often absent — do not require it.
   If the API is unreachable, ask the user for a manual export. For a single
   repo request, skip the list and treat that repo as the whole A-tier.
2. **归类 + 提出分级方案。** Categorize the list, then propose an A/B split
   (heuristics in [references/playbook.md](references/playbook.md) §分级):
   A 级 = deep dives with diagrams; B 级 = one-pagers. Show the split as a
   table, get confirmation.
3. **分批执行 A 级。** Per repo: `git clone --depth 1` into the temp dir →
   analyze the real code (dispatch a sub-agent for big repos; the prompt must
   demand `file:line` anchors and explicit "uncertain" flags) → write the
   README from [assets/templates/readme-template.md](assets/templates/readme-template.md)
   → draw the three diagrams per [references/diagram-spec.md](references/diagram-spec.md)
   → render PNGs and self-inspect → hand the batch to visual QA when available.
   Re-dispatch any sub-agent that dies of rate limits after the others finish.
4. **B 级批量产出。** One page per repo from its README (fetched via
   raw.githubusercontent.com / the GitHub API — no clone). See
   [references/playbook.md](references/playbook.md) §B级 for the template.
5. **收尾。** Update the index status markers, delete the temp clones, report
   totals and any repos you could not verify.

## 模式 B · 本机技能盘点 / Installed-skills mode

1. **扫描与去重。** Scan the common skill roots (`~/.zcode/skills`,
   `~/.agents/skills`, `~/.codex/skills`, `~/.skills-manager/skills`, plus
   plugin caches) and deduplicate — these directories are typically symlink
   farms pointing at the same real copies. Resolve every symlink; record
   empty directories and broken links as 失效 items instead of skipping them.
2. **系列合并。** Skills from one family (same prefix, e.g. a dozen
   `remotion-*` variants) merge into one series doc; standalone skills get
   their own doc. Propose the grouping for confirmation.
3. **逐篇产出。** Template: 一句话定位 / 什么时候用 / 怎么触发 / 用法示例 /
   与哪些技能配合 / 注意事项 — grounded in each SKILL.md's actual text.
   No diagrams for this mode. Cross-reference Mode A docs when a skill's
   source repo is also in the stars list.

## diagram-design 联动 / Diagram engine

The three diagrams are drawn with the bundled diagram-design skill (vendored
under `third-party/diagram-design/`, MIT).

1. **探测已装副本**（质量优先）：check `~/.codex/skills/diagram-design/SKILL.md`,
   `~/.agents/skills/diagram-design/SKILL.md`, `~/.skills-manager/skills/…`,
   and the project's skill roots. If found, read THAT copy — it may be newer.
2. **缺失则从 vendor 安装**：copy `third-party/diagram-design/` to
   `~/.agents/skills/diagram-design/`, tell the user what you installed and
   from where, then read it from the installed path.
3. **读它的规则再画图。** diagram-design's SKILL.md (§6 connector rules,
   §7 complexity budget) and the relevant `references/type-*.md` are the
   authority. Our distillation in
   [references/diagram-spec.md](references/diagram-spec.md) is a shortcut,
   not a replacement.

## 图表与质检 / Diagrams & QA

- Read [references/diagram-spec.md](references/diagram-spec.md) before drawing
  anything. It contains the three-diagram content briefs, the render chain,
  and the self-check rubric.
- Render chain (worked on Windows/Git Bash where plain Chromium download
  usually fails): percent-encode the file URL, then
  `npx playwright screenshot --channel msedge --viewport-size=1280,900 --full-page "<url>" out.png`.
  Only fall back to a full `npx playwright install chromium` if msedge is
  unavailable.
- Self-inspect every PNG (Read it) and fix label collisions before calling a
  diagram done. When a read-only judge sub-agent is available, hand it the
  PNGs with the rubric from diagram-spec §验收; re-judge only the pages you
  changed.
- 环境实在不具备时：state plainly that diagrams were skipped and why, and
  ship Mermaid-in-Markdown as the documented fallback. Never silently.

## 参考文件 / When to read what

| Situation | Read |
|---|---|
| Starting a batch, sizing A/B, anything fails | [references/playbook.md](references/playbook.md) |
| About to draw or QA diagrams | [references/diagram-spec.md](references/diagram-spec.md) |
| Writing a deep-dive README | [assets/templates/readme-template.md](assets/templates/readme-template.md) |
| Drawing any diagram | the matching `assets/templates/*-template.html` chrome |
| Toolchain internals | `third-party/diagram-design/` (read its own SKILL.md first) |
