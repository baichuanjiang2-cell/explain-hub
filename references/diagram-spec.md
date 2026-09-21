# Diagram Spec — the three diagrams

Every A-tier explainer ships a fixed set of three diagrams. The vendored
diagram-design skill is the drawing authority (read its SKILL.md §6 connector
rules and §7 complexity budget first, then the relevant
`references/type-*.md` for the chart type); this file is the accelerated
digest + this engine's acceptance criteria. When they conflict,
diagram-design's own text wins.

Paths: after install, `~/.agents/skills/explain-hub/third-party/diagram-design/`;
the vendor source directory is `third-party/diagram-design/`.

## What each diagram shows

| Diagram             | Type (diagram-design terms)    | Question answered         | Content brief                                                                                                                                                                 |
| ------------------- | ------------------------------ | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Panorama            | Architecture (zones + modules) | What can it do            | Group by capability into ≤3 zones, ≤9 nodes total; node sub-labels list concrete features with `file:line`; coral focus on the flagship capability                            |
| Module architecture | Architecture                   | How the code divides work | Partition by process/layer/trust boundary (e.g. renderer/main/server or frontend/backend/kernel); arrows labeled with the transport (HTTP/IPC/require)                        |
| Runtime flow        | Flowchart (swimlanes optional) | How a core feature runs   | End-to-end main path ≤9 nodes: oval start, rect steps, diamond decisions (≤3 exits, each exit labeled), merge dots; coral only for the main loop / the most critical decision |

Read diagram-design SKILL.md §3's visual-type table before choosing a type;
when unsure, use Architecture.

## Non-negotiables (details in diagram-design §6)

1. Orthogonal connectors: `r=8` rounded elbows; straight segments only when
   endpoints share x or y; **diagonal lines are an automatic fail**.
2. Arrow labels sit on a paper-colored mask rect with a visible **6–10px gap**
   between mask and line; never on the line, never vertical text.
3. No overlaps, no collinear segments; where a solid line crosses another,
   the **secondary** one gets an `8,8` half-circle bridge; a dashed line
   crossing a solid one must always be bridged.
4. Multiple attach points on the same edge of one box stay ≥12px apart.
5. Connectors must not pass through non-endpoint boxes (unavoidable
   cross-cuts: dashed line + label on the visible end).
6. Complexity budget (§7): ≤9 nodes / ≤12 arrows / ≤3 zones / ≤2 coral
   elements; beyond that, split into "overview + detail" diagrams.
7. Page chrome: eyebrow + Instrument Serif title + subtitle; `<svg
role="img" aria-labelledby>` with **prefixed** `<title>/<desc>`; the
   legend sits horizontally below the bottom hairline (never floating inside
   the chart); load Instrument Serif / Geist / Geist Mono / **Noto Sans SC**
   from Google Fonts (CJK label fallback).
8. If the viewBox has excess top whitespace, crop it (e.g.
   `viewBox="0 76 1180 614"`) — safer than moving coordinates.

## Deliverable

- One **self-contained HTML** per diagram (inline CSS+SVG, Google Fonts the
  only external link), named `panorama.html` / `architecture.html` /
  `flow.html` (localize when the user's language is not English, e.g.
  `功能全景图.html`), in the same directory as the README.
- Palette: the diagram-design default skin — paper `#f5f5f5`, ink `#2d3142`,
  muted `#4f5d75`, accent `#eb6c36` (≤2 focus elements), rule
  `rgba(45,49,66,.10)`.
- The figcaption states the analysis baseline (commit / local copy path);
  the footer states `GENERATED WITH DIAGRAM-DESIGN · date`.

## Rendering (the Windows/Git Bash chain)

1. **Percent-encode the file URL** (non-ASCII paths + spaces break direct
   access):
   `python -c "from urllib.parse import quote; print(quote('panorama.html'))"`
2. **Prefer the msedge channel** (the official Chromium download often fails
   on restricted networks, and local Edge is always there):

   ```bash
   npx playwright screenshot --channel msedge --viewport-size=1280,900 --full-page "<file-url>" _panorama.png
   ```

3. If msedge is unavailable, `npx playwright install chromium` (may need a
   proxy).
4. Name the three self-check renders `_panorama.png` / `_architecture.png` /
   `_flow.png` and keep them in the delivery directory as previews.

## Acceptance rubric (shared by self-check and review agent)

Check every page; a fail means user-visible breakage only:

- Text: no truncation / overlap / tofu (CJK garbage); labels do not overflow
  their boxes
- Connectors: no diagonals; labels do not sit on lines (mask gap visible);
  dashed-over-solid crossings have bridges; no dangling arrows (endpoints
  must land on box edges); no two lines sharing an attach point
- Composition: zones fully contain their nodes; legend horizontal at the
  bottom; ≤2 coral elements; no shadows / purple-cyan neon
- Consistency: header/footer/figcaption complete; the chrome is consistent
  across pages

After fixes, **re-check only the pages you changed** and list every fix in
the re-check prompt.
