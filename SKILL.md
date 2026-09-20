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
   403/429 = 限流：稍候重试一次，仍失败请用户手动导出。若列表为空（常见于
   组织账号），告知用户并提议改讲其公开仓库（`/users/<name>/repos`）或用户
   点名的仓库——不要凭空继续。For a single repo request, skip the list and
   treat that repo as the whole A-tier.
2. **归类 + 提出分级方案。** Categorize the list, then propose an A/B split
   (heuristics in [references/playbook.md](references/playbook.md) §分级):
   A 级 = deep dives with diagrams; B 级 = one-pagers. 清单数十条以内逐行列出；
   数百条则按类目汇总（A 级候选逐个列出，B 级按类目给数量与代表）。
   Get confirmation.
3. **建总索引。** 开工前创建 `<输出目录>/README.md`：全量篇目 + 状态列
   （⬜待做/🖊️写作中/✅完成/❌失效）。之后每完成一篇立刻翻状态。
4. **分批执行 A 级。** 产物布局：每个仓库一个子目录 `<输出目录>/<repo-名>/`，
   内含 `README.md` + 三图 HTML + `_*.png` 自检图。Per repo:
   `git clone --depth 1` into the temp dir → analyze → write the README from
   [assets/templates/readme-template.md](assets/templates/readme-template.md)
   → draw the three diagrams per [references/diagram-spec.md](references/diagram-spec.md)
   → render PNGs and self-inspect → hand the batch to visual QA when a
   sub-agent runtime is available（无子代理能力则严格自检并在交付时注明）。
   仓库超过约 200 个文件或为 monorepo 时派子代理分析（prompt 必须自带模板
   路径、输出目录、`file:line` 锚点要求和"不确定就明说"）；小仓库主上下文
   直接读。Re-dispatch any sub-agent that dies of rate limits after the
   others finish. 同一页修复重检 ≤2 轮，仍不达标则标注已知问题交付。
5. **B 级批量产出。** One page per repo from its README (fetched via
   raw.githubusercontent.com / the GitHub API — no clone), saved as
   `B级轻量/<owner>__<repo>.md`. See
   [references/playbook.md](references/playbook.md) §B级 for the template.
6. **收尾。** Update the index status markers, delete the temp clones, report
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
   `~/.agents/skills/diagram-design/SKILL.md`, `~/.skills-manager/skills/diagram-design/SKILL.md`,
   plus the project's own skill roots (`<cwd>/.zcode/skills/`, `<cwd>/.agents/skills/`).
   If found, read THAT copy — it may be newer.
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
- 有子代理 runtime 时：把 PNG 交给一个只读评审代理，rubric 见 diagram-spec
  §验收；**只复检改过的页**，并在复检 prompt 里写明每处修复。
- 完整降级链（有序，走到哪级都在交付里注明）：diagram-design 探测/安装 →
  msedge 截图 → chromium 安装 → 全部失败才 Mermaid-in-Markdown 兜底。Never
  silently skip diagrams.

## 参考文件 / When to read what

| Situation | Read |
|---|---|
| Starting a batch, sizing A/B, anything fails | [references/playbook.md](references/playbook.md) |
| About to draw or QA diagrams | [references/diagram-spec.md](references/diagram-spec.md) |
| Writing a deep-dive README | [assets/templates/readme-template.md](assets/templates/readme-template.md) |
| Drawing any diagram | the matching `assets/templates/*-template.html` chrome |
| Toolchain internals | `third-party/diagram-design/` (read its own SKILL.md first) |
