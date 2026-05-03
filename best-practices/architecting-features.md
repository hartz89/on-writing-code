# Architecting Features

Rationale: [architecting-features.why.md](./architecting-features.why.md). Strength levels: [../code-style/\_index.md](../code-style/_index.md).

## Prove the Pipeline Early

`strength: strong` · [rationale](./architecting-features.why.md#prove-the-pipeline-early)

- Build a thin end-to-end slice first — UI → state → API → data — using real components, routes, and endpoints. Not a prototype.
- Wire the UI to the API before writing business logic.
- Prefer a working-but-incomplete vertical slice over a complete-but-unconnected horizontal layer.
- The slice is production-bound code. Flesh it out; don't throw it away.
