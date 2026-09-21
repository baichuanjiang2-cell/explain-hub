# Changelog

All notable changes to this project are documented in this file.

The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-09-21

Initial public release.

### Added

- **Mode A — GitHub stars explainer**: tiered treatment (A-tier deep dives
  with three diagrams + `file:line` anchors, B-tier one-pagers), master
  index with live status markers, batch-confirmation cadence.
- **Mode B — installed-skills inventory**: symlink-farm deduplication,
  family merging, six-section explainers, broken-skills report.
- **Diagram pipeline**: mandatory panorama / module-architecture /
  runtime-flow diagrams via the vendored diagram-design engine, with SVG
  connector rules, complexity budgets, PNG render chain (msedge first) and
  an acceptance rubric shared by self-check and review agents.
- **Field playbook** (`references/playbook.md`): API fallback chains,
  facade-repo detection, doc-vs-code drift checks, sub-agent rate-limit
  re-dispatch, tiering heuristics.
- **Templates**: three-angle deep-dive README template + HTML chrome for
  the three diagrams.
- Vendored copy of
  [diagram-design](https://github.com/cathrynlavery/diagram-design) v2.6
  (MIT) with its license and provenance record.
- Community files (CONTRIBUTING, CHANGELOG, SECURITY, CODE_OF_CONDUCT,
  issue/PR templates) and CI (markdownlint + prettier + template sanity).

### Notes on defaults relative to pre-release working copies

- Repository documentation is now English-canonical, with
  `README.zh-CN.md` as the Chinese entry point. Generated explainers still
  follow the user's language at runtime.
- Default artifact names switched to English
  (`stars-explained/`, `skills-explained/`, `lightweight/`,
  `panorama.html`, `architecture.html`, `flow.html`); the skill localizes
  them when the user speaks another language.
