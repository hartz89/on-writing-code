# Architecting Features — Rationale

Expanded rationale for opinionated rules in [architecting-features.md](./architecting-features.md). Section names mirror the rules file.

---

## Prove the Pipeline Early

**Rule:** Build a thin end-to-end slice first before filling out any layer in full.

Adapted from [_The Pragmatic Programmer_, Hunt & Thomas](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/) — "Tracer Bullets" chapter.

### The integration tax is cheapest early

The failure mode this rule guards against: discovering integration mismatches after each layer is complete. Fixing a mismatch between a finished API and a finished UI is far more expensive than catching it when both are thin. The thin slice is the tracer — it traces a path through the real system and shows immediately where something doesn't connect.

### Tracer bullets, not prototypes

Hunt & Thomas draw a sharp distinction. A prototype is written to be thrown away — exploratory, unpolished, expected to be discarded. A tracer slice is production code: it has tests, follows conventions, and ships. The difference is scope (thin) not quality.

### Working horizontally is locally efficient and globally costly

The common alternative: build all API endpoints before touching the UI. Less context-switching per session — but you pay the integration tax at the end, when every layer is already optimized in isolation and the blast radius of a mismatch is largest.
