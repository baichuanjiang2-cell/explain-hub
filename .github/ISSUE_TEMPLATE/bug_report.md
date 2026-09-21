---
name: Bug report
about: The skill produced wrong, broken or misleading output
title: "[bug] "
labels: bug
assignees: ""
---

**What happened**

A clear description of what the skill actually produced (wrong facts, a
broken diagram, a skipped step, a hang).

**What you expected**

What the rule set says should have happened, if you know the section —
otherwise describe the reasonable expectation.

**Environment**

- Agent host: Claude Code / Codex / ZCode / other:
- explain-hub mode: A (stars) / B (installed skills) / single-repo:
- OS + shell:

**Evidence**

Paste the relevant excerpt: the offending section of a generated explainer,
a screenshot of a broken diagram, the command that failed. Diagram failures:
say which rule it violates if you can spot it (see
`references/diagram-spec.md` § Acceptance rubric).

**Did a fallback chain fire?**

If you know: no `gh` / clone failure / msedge unavailable / sub-agent rate
limits — note which level you saw.
