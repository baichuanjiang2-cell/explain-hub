# <project> — <one-line positioning> (A-tier deep dive)

> Repo: [<owner>/<repo>](https://github.com/<owner>/<repo>) (<stars>★) | Version <x.y.z> | License <license>
> Local copy: <path> (or "analyzed from a shallow clone of commit <sha>")
> Diagrams: [panorama](panorama.html) · [module architecture](architecture.html) · [runtime flow](flow.html)

In one sentence: <explain this project to a layperson — what it is, who it is for, what problem it solves.>

---

## Angle 1 · What it can do (panorama)

Grouped by capability domain (mirrors the panorama diagram):

**<domain 1>**

- <feature> (`<file>:<line>`)
- <feature> (`<file>:<line>`)

**<domain 2>**

- …

## Angle 2 · How the code divides work (module architecture)

Mirrors the architecture diagram; table: layer / key files / responsibility.

| Layer   | File/dir | Responsibility                                              |
| ------- | -------- | ----------------------------------------------------------- |
| <layer> | `<path>` | <responsibility, incl. entry points and key function names> |

**Transport**: <how the layers/processes talk and over what channel>.

## Angle 3 · How a core feature runs: <pick the most representative path>

Mirrors the flow diagram; numbered steps, each with `file:line`.

1. **<start>**: <what happens> (`<file>:<line>`)
2. …

## How to use it

| Audience   | Action             | Notes                                    |
| ---------- | ------------------ | ---------------------------------------- |
| End users  | <install/download> | <platform requirements / known pitfalls> |
| Developers | `<command>`        | <prerequisites>                          |

**<Key config/account requirements>** (if applicable: <what happens without login / tier differences>)

## Caveats

- <License & compliance, API freshness, known defects, doc-vs-code drift — every item sourced>
- <One-sentence comparison within the same track (if applicable): this repo takes the X approach; see <> for the related explainer>

<!--
Writing discipline:
1. Every fact carries a file:line anchor or a source note; write "unverified"
   when you could not check it — never fabricate.
2. Restate marketing claims as facts ("supports X", not "powerful X").
3. Confirm README-claimed capabilities in the code first; record drift
   under Caveats.
4. Write the document in the user's language; keep code identifiers,
   paths and commands in their original form.
-->
