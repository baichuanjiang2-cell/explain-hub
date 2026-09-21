# Vendored: diagram-design

This directory is a complete vendored copy of the **diagram-design** skill
(v2.6, MIT-licensed), used as explain-hub's diagram engine.

## Upstream

- Repository: <https://github.com/cathrynlavery/diagram-design>
- Author & copyright holder: Cathryn Lavery
- License: MIT — see [LICENSE](LICENSE) in this directory (verbatim copy of
  the upstream license file)

Provenance note: this copy was originally taken from a local marketplace
install (which carried no git metadata), so the exact upstream commit it
snapshots is unknown. The version stated in its `SKILL.md` frontmatter is
`2.6`, which matched upstream `main` at the time of vendoring. No
modifications were made. To pick up upstream fixes, re-vendor from the
repository linked above.

## Why vendored

explain-hub's deep dives require three diagram-design-style diagrams
(panorama / module architecture / runtime flow). Bundling a copy guarantees:

1. Installation works offline or behind restricted networks (at runtime,
   explain-hub copies this directory into the user's skills directory).
2. The diagram quality bar stays consistent — diagram-design's connector
   rules and complexity budget *are* explain-hub's acceptance criteria.

## Updates

explain-hub prefers a newer copy already installed on your machine: at
runtime it probes the standard skill roots first and uses whatever it finds.
This directory is only the fallback install source.

## License

diagram-design is released under the MIT License, copyright its original
author (Cathryn Lavery). This vendored copy ships under that same license;
any modifications explain-hub makes to this directory are likewise MIT.
