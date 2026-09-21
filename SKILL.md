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
  "walk me through this repo", "盘点/讲解 本机已装的技能/installed skills",
  pastes a GitHub username asking for an explainer, or names explain-hub —
  even if they only say "帮我把这些仓库都讲一遍" or "what does this skill do".
---

# Explain Hub — the explainer engine

Two modes, one engine. **Mode A** turns a GitHub account's stars into tiered
explainers. **Mode B** inventories the agent skills installed on this machine
and explains each. Both share the same principles, the same diagram pipeline,
and the same index discipline.

**Every user-facing document is written in the language the user is speaking.**
Keep code identifiers, file paths and CLI terms in their original form.
Artifact names below are the English defaults; when the user speaks another
language, localize them (e.g. Chinese: `stars-explained/` → `stars讲解/`,
`panorama.html` → `功能全景图.html`).

## Non-negotiables

1. **Facts first, anchors verifiable.** Every factual claim in an explainer
   carries a `file:line` anchor (for code) or a source note (for README-derived
   facts). Never present a guess as fact; write down what you could not verify.
2. **Diagrams are mandatory.** Every deep-dive (A-tier) explainer ships three
   diagrams: panorama / module architecture / runtime flow, drawn with the
   bundled diagram-design skill. If diagram-design is missing, install it first
   (§ Diagram engine) — do not degrade to text-only without telling the user.
   Lightweight (B-tier) notes carry no diagrams by design.
3. **Align first, then batch.** Before producing anything, present the
   inventory and the plan as a checklist with per-item recommendations
   (✅ recommended / ⬜ optional) and let the user pick. Batch size 5–8. Never
   re-ask for items the user already decided.
4. **The index is updated the moment work finishes.** The index README is the
   deliverable's front door: every finished item flips its status marker as
   soon as it is done.
5. **Temp artifacts are cleaned up after use.** Clones live in a temp dir,
   survive until QA passes, and are deleted afterwards (unless the user asked
   to keep them).

## Tunables (defaults — override on request)

| Parameter     | Default                                 | Notes                        |
| ------------- | --------------------------------------- | ---------------------------- |
| A-tier cap    | 25                                      | Deep-dive treatment cap      |
| Batch size    | 5–8                                     | Items per confirmation round |
| Output dirs   | `stars-explained/`, `skills-explained/` | Created in cwd               |
| Clone dir     | `_repos/`                               | Temp, deleted after QA       |
| B-tier source | repo README (fetched via API)           | No clone                     |

## Mode A · GitHub stars explainer

1. **Get the stars list.** Prefer
   `https://api.github.com/users/<name>/starred?per_page=100` (public, no
   auth; paginate). `gh` is often absent — do not require it. 403/429 = rate
   limited: wait and retry once, then ask the user to export manually. If the
   list is empty (common for organization accounts), say so and offer to
   explain their public repos (`/users/<name>/repos`) or repos the user names
   — do not continue on guesses. For a single-repo request, skip the list and
   treat that repo as the whole A-tier.
2. **Categorize + propose the tier split.** Categorize the list, then propose
   an A/B split (heuristics in [references/playbook.md](references/playbook.md)
   § Tiering): A-tier = deep dives with diagrams; B-tier = one-pagers. List
   every item when the list is a few dozen or fewer; for hundreds, summarize
   B-tier by category (count + representatives) and list A-tier candidates
   individually. Get confirmation.
3. **Create the master index.** Before any writing, create
   `<output-dir>/README.md`: the full table of entries + a status column
   (⬜ todo / 🖊️ in progress / ✅ done / ❌ broken). From then on, flip each
   entry's status the moment it finishes.
4. **Execute the A-tier in batches.** Output layout: one subdirectory per repo
   `<output-dir>/<repo-name>/` containing `README.md` + three diagram HTML
   files + `_*.png` self-check renders. Per repo: `git clone --depth 1` into
   the temp dir → analyze → write the README from
   [assets/templates/readme-template.md](assets/templates/readme-template.md)
   → draw the three diagrams per [references/diagram-spec.md](references/diagram-spec.md)
   → render PNGs and self-inspect → hand the batch to visual QA when a
   sub-agent runtime is available (with no sub-agent runtime, self-inspect
   strictly and say so at delivery). Dispatch a sub-agent to analyze repos
   larger than ~200 files or monorepos (the prompt must carry the template
   path, output dir, the `file:line` anchor requirement, and "say when you
   are unsure"); read small repos in the main context. Re-dispatch any
   sub-agent that dies of rate limits after the others finish. Re-check a
   fixed page ≤2 rounds; if it still fails, deliver with the issue noted.
5. **Produce the B-tier in bulk.** One page per repo from its README (fetched
   via raw.githubusercontent.com / the GitHub API — no clone), saved as
   `lightweight/<owner>__<repo>.md`. See
   [references/playbook.md](references/playbook.md) § Lightweight one-pager
   for the template.
6. **Wrap up.** Update the index status markers, delete the temp clones,
   report totals and any repos you could not verify.

## Mode B · Installed-skills inventory

1. **Scan and deduplicate.** Scan the common skill roots (`~/.zcode/skills`,
   `~/.agents/skills`, `~/.codex/skills`, `~/.skills-manager/skills`, plus
   plugin caches) and deduplicate — these directories are typically symlink
   farms pointing at the same real copies. Resolve every symlink; record empty
   directories and broken links as broken items instead of skipping them.
2. **Merge families.** Skills from one family (same prefix, e.g. a dozen
   `remotion-*` variants) merge into one series doc; standalone skills get
   their own doc. Propose the grouping for confirmation.
3. **Produce each doc.** Template: one-line positioning / when to use / how
   to trigger / usage example / which skills it pairs with / caveats —
   grounded in each SKILL.md's actual text. No diagrams for this mode.
   Cross-reference Mode A docs when a skill's source repo is also in the
   stars list.

## Diagram engine (diagram-design)

The three diagrams are drawn with the bundled diagram-design skill (vendored
under `third-party/diagram-design/`, MIT, upstream:
<https://github.com/cathrynlavery/diagram-design>).

1. **Probe for an installed copy first** (quality first): check
   `~/.codex/skills/diagram-design/SKILL.md`,
   `~/.agents/skills/diagram-design/SKILL.md`,
   `~/.skills-manager/skills/diagram-design/SKILL.md`, plus the project's own
   skill roots (`<cwd>/.zcode/skills/`, `<cwd>/.agents/skills/`). If found,
   read THAT copy — it may be newer.
2. **If missing, install from the vendor copy**: copy
   `third-party/diagram-design/` to `~/.agents/skills/diagram-design/`, tell
   the user what you installed and from where, then read it from the
   installed path.
3. **Read its rules before drawing.** diagram-design's SKILL.md (§6 connector
   rules, §7 complexity budget) and the relevant `references/type-*.md` are
   the authority. Our distillation in
   [references/diagram-spec.md](references/diagram-spec.md) is a shortcut,
   not a replacement.

## Diagrams & QA

- Read [references/diagram-spec.md](references/diagram-spec.md) before
  drawing anything. It contains the three-diagram content briefs, the render
  chain, and the self-check rubric.
- Render chain (worked on Windows/Git Bash where a plain Chromium download
  usually fails): percent-encode the file URL, then
  `npx playwright screenshot --channel msedge --viewport-size=1280,900 --full-page "<url>" out.png`.
  Only fall back to a full `npx playwright install chromium` if msedge is
  unavailable.
- With a sub-agent runtime: hand the PNGs to a read-only review agent, rubric
  in diagram-spec § Acceptance; **re-check only the pages you changed**, and
  list every fix in the re-check prompt.
- Full fallback chain (ordered; note in the delivery which level you
  reached): diagram-design probe/install → msedge screenshot → chromium
  install → Mermaid-in-Markdown as the last resort. Never silently skip
  diagrams.

## When to read what

| Situation                                    | Read                                                                       |
| -------------------------------------------- | -------------------------------------------------------------------------- |
| Starting a batch, sizing A/B, anything fails | [references/playbook.md](references/playbook.md)                           |
| About to draw or QA diagrams                 | [references/diagram-spec.md](references/diagram-spec.md)                   |
| Writing a deep-dive README                   | [assets/templates/readme-template.md](assets/templates/readme-template.md) |
| Drawing any diagram                          | the matching `assets/templates/*-template.html` chrome                     |
| Toolchain internals                          | `third-party/diagram-design/` (read its own SKILL.md first)                |
