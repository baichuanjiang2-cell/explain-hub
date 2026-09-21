# Playbook — field-tested rules

This file is explain-hub's experience library: every rule here comes from a
pit hit during real batch-explainer runs. Consult as needed; anything about
drawing diagrams lives in [diagram-spec.md](diagram-spec.md).

## 1. Getting the stars list — fallback chain

1. The `gh` CLI is often absent — do not assume it is available.
2. Prefer the public API:
   `https://api.github.com/users/<name>/starred?per_page=100&page=N`
   (unauthenticated quota is 60/hour, enough to pull the list; paginate until
   you get an empty array). Verify the account exists with `GET /users/<name>`
   first.
3. Windows note: when parsing JSON with python, keep temp files on
   workspace-relative paths — `/tmp` is invisible to native Windows python.
4. API unreachable → ask the user to export manually (GitHub → Your stars
   page, copy-paste). Do not hang.
5. Delete the temp JSON once parsed; do not leave it in the workspace.

## 2. Sub-agent orchestration & rate limits

- Dispatch sub-agents for large repos (protects the main context). The prompt
  must carry the full template path, output dir, the `file:line` anchor
  requirement, the instruction "say when you are unsure", and a demand that
  the **final reply stays short**.
- Parallel sub-agents will hit model rate limits (symptoms: `[1302]` rate
  errors, silently dead tasks). **Re-dispatch the failures after the others
  finish** — it is not fatal.
- Clone instructions for sub-agents: `--depth 1`, retry twice, then fall back
  to GitHub API analysis (`api.github.com` + `raw.githubusercontent.com`)
  and note it in the README.
- Do **not** delete the clone dir before visual QA fully passes — chart fixes
  need to be checked against the code.

## 3. Facade repos

Stars do not equal content. Spend one minute verifying before you analyze:

- Does `main` have only single-digit commits and a few dozen files? (One 90k★
  repo's `main` turned out to be an empty scaffold rewritten from scratch;
  the real code lived in an archived sibling repo.)
- Does the README advertise features that have no matching implementation
  directory?
- When you find a facade: locate the real codebase (archived branch /
  `-classic` sibling repo / official link), **explain the real code**, and
  state in the README and the index that "the main repo is a scaffold; this
  explainer is based on X".

## 4. Doc-vs-code drift checks

READMEs and SKILL.md files lie (stale, marketing tone). For every key claim
going into an explainer:

- Claimed CLI subcommands → confirm they exist via `grep argparse` / the
  entry file.
- Claimed counts (templates/presets/supported tools) → actually enumerate the
  directory; write both numbers when they differ.
- Claimed versions/star counts → verify via API when you can
  (`api.github.com/repos/<o>/<r>` returns `stargazers_count`); otherwise
  write "as given, not independently verified".
- Drift is itself explainer material: call it out under Caveats.

## 5. Tiering heuristics

Signals for A-tier (full three diagrams) — the more matched, the higher the
priority:

- It is the upstream repo of a locally installed skill/plugin (cross-linking
  the two explainers doubles the value);
- High star count (a clear tooling niche);
- A local copy already exists (saves a clone);
- A core tool the user actually uses (agent clients, editors, productivity
  workbenches).

B-tier (one-pager) covers everything else. Tiering is **provisional**: mark
A/B in the batch list; the user can promote any B at any time. Default cap is
25 A-tier items; beyond that, have the user cut or raise the cap.

## 6. Batch cadence

- First present the full inventory so the user decides **scope and depth**
  once (one question, not per-item nagging).
- During execution, batches of 5–8 items: present the checklist
  (✅ recommended / ⬜ optional + why) → user picks → run them all → next
  batch.
- The two tracks (skills / stars) can be mixed in one pick; if the user says
  "do all / go with your recommendations", execute the whole batch without
  further interruptions.

## 7. Index & cross-references

- One `README.md` master index per output directory: the entry table + a
  status column (⬜ todo / 🖊️ in progress / ✅ done / ❌ broken); flip each
  row as it completes.
- A skill's upstream repo ↔ the installed-skill explainer get **bidirectional
  links**; same-track projects get a one-sentence comparison under Caveats
  (e.g. "X is a local SDK, Y is an HTTP service").
- Record broken skills as a table (name / symptom: empty dir or dead link /
  repair instructions) — one of the most unexpectedly useful outputs.

## 8. Installed-skills mode specifics

- Skill directories are **symlink farms**: `~/.zcode/skills` and
  `~/.agents/skills` point at each other heavily; the real copies live in
  `~/.skills-manager/skills`, `~/.codex/skills` (including `.system`
  built-ins) and plugin caches. Resolve each link with `readlink` and
  deduplicate by real path.
- In `ls -F` output, `@` marks a symlink; the `*/` glob **skips dead links** —
  use `readlink -e` to test whether the target exists.
- Target directory exists but is empty = broken install (empty dir);
  `readlink` returns nothing and the target does not exist = dead link. Both
  go into the broken table.
- Some SKILL.md files are not on the loaded list (missing/disabled
  frontmatter) but are still worth explaining — treat disk contents as the
  source of truth.
- Merge families (same prefix) into one overview covering member division of
  labor and how they cooperate.

## 9. Rendering & visual QA

- The screenshot chain and its common pitfalls (Chromium download failures,
  non-ASCII paths) are in [diagram-spec.md](diagram-spec.md) § Rendering.
- Review-agent prompt essentials: a per-page JSON verdict (pass/fail + a
  positioned issue list), the design-system checklist (orthogonal connectors
  / masked labels / ≤2 coral accents / legend at the bottom), "prefix `minor:`
  = non-blocking", "fail only on user-visible breakage".
- **Re-check only the pages you changed**, and list each fix in the re-check
  prompt so the agent knows where to look.
- Quick reference of common fails: arrow labels sitting on the line (mask
  needs a 6–10px visible gap), dashed line crossing a solid one without a
  half-circle bridge, dangling arrows (endpoints must land on a box edge),
  two lines sharing one attach point, export-format labels swapped, a
  flowchart's end node disconnected.

## 10. Lightweight one-pager template

- Source: `raw.githubusercontent.com/<o>/<r>/main/README.md` (if `main`
  fails, try `master`/`dev`, or `api.github.com/repos/<o>/<r>/readme` with
  `Accept: application/vnd.github.raw`); if everything fails, write from repo
  metadata (description/topics/language) and say so.
- Structure: `# repo — one-line positioning`; `## What it can do` (3–6
  bullets, marketing speak restated as facts); `## How to get started`;
  `## Why it matters to you` (cross-references to other explainers, or the
  use case if none); `## Links`.
- 25–45 lines; filename `<owner>__<repo>.md`; restrained tone; do not
  mention what the source never did.
