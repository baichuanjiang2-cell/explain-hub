# Contributing to explain-hub

Thanks for your interest in improving explain-hub. This repo is an agent
skill — the "source code" is the rule set: `SKILL.md`, `references/`, and the
templates in `assets/templates/`. Treat them as code.

## Ground rules

1. **Bring field evidence.** Every rule in this repo was earned by a real
   failure during a batch run (`references/playbook.md` is literally that
   log). When you propose changing a rule or adding one, describe the run
   where the current behavior failed or the new one proved itself. "Seems
   cleaner" is not evidence.
2. **Fact discipline applies here too.** Claims in our docs carry anchors or
   links — including yours. If a statement cannot be verified, label it.
3. **Don't fork diagram-design's rules.** The diagram rules in
   `references/diagram-spec.md` are a digest; the authority is the vendored
   [diagram-design](third-party/diagram-design/) skill. If a rule conflicts,
   fix the digest (or re-vendor upstream), don't invent a third version.
4. **The vendored directory stays unmodified.** `third-party/diagram-design/`
   is an unmodified snapshot of
   [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design).
   Propose substantive changes upstream, then re-vendor here.

## Language policy

English is the canonical language of this repository (`SKILL.md`,
`references/`, templates). `README.zh-CN.md` mirrors the English README —
when you change one, change the other in the same PR. Explain what the
_skill generates_ stays in the user's language; that is runtime behavior,
not repo policy.

## Testing your change

There is no unit suite — the test is a real run:

1. Install your modified copy into a scratch skill path
   (`cp -r . /tmp/skills-test/.agents/skills/explain-hub` or similar).
2. Run one small, known repo through Mode A (for example
   `tj/commander.js`): confirm the README comes out with `file:line`
   anchors and the three diagrams pass the acceptance rubric in
   [references/diagram-spec.md](references/diagram-spec.md).
3. If you touched Mode B, point it at your skills directory and check the
   deduplication and the broken-skills report.

Paste a short excerpt of the run (not 40 files of output) into the PR.

## Before you open a PR

- [ ] Docs changed in English; `README.zh-CN.md` kept in sync if user-facing
- [ ] `npx markdownlint-cli2` and `npx prettier --check .` pass locally
- [ ] CHANGELOG.md updated for user-visible changes
- [ ] Field evidence described in the PR body

## Reporting problems that are not contributions

Use the issue templates (bug / feature). Include your agent host
(Claude Code / Codex / ZCode / …), the mode, and what actually came out —
screenshots of diagram failures are especially welcome.
