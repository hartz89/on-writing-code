# Architecting Features

Rationale for the opinionated sections lives in [architecting-features.why.md](./architecting-features.why.md). Strength levels (`non-negotiable` / `strong` / `moderate` / `trivial`) are defined in [../code-style/\_index.md](../code-style/_index.md).

## Prove the Pipeline Early

`strength: strong` · [rationale](./architecting-features.why.md#prove-the-pipeline-early)

Build a thin end-to-end slice first — UI → state → API → data — before filling out any layer in full. The slice uses real components, real routes, real endpoints. It is not a prototype.

- Wire the UI to the API before writing business logic. A form that roundtrips to a real endpoint surfaces integration problems while the system is still cheap to change.
- Prefer a working-but-incomplete vertical slice over a complete-but-unconnected horizontal layer.
- The slice is production-bound code. Flesh it out; don't throw it away.
