# Architecting Features

Rationale: [architecting-features.why.md](./architecting-features.why.md). Strength levels: [../code-style/\_index.md](../code-style/_index.md).

## File and Directory Structure

`strength: strong` · [rationale](./architecting-features.why.md#file-and-directory-structure)

- Name files verbosely. Avoid vague names like `component.tsx`, `utils.ts`, `helpers.ts`.
- Prefer direct imports. Add a barrel file only when it provides a clear benefit — e.g., a stable public API surface for a shared package.
- If you do use barrels: `import`/`export` statements only (no logic), and keep them scoped small.
- Directory names can be shorter.
- Keep directory trees flat. Don't create a subdirectory per file; nest only when a group becomes genuinely hard to reason about together.

```
// good
features/
  flowers/
    FlowerCard.tsx
    FlowerList.tsx
    useFlowerSubscription.ts
    flowerApi.ts

// avoid — one file per directory, vague names
features/
  flowers/
    card/
      component.tsx
    list/
      component.tsx
    hooks/
      subscription.ts
```

## Prove the Pipeline Early

`strength: strong` · [rationale](./architecting-features.why.md#prove-the-pipeline-early)

- Build a thin end-to-end slice first — UI → state → API → data — using real components, routes, and endpoints. Not a prototype.
- Wire the UI to the API before writing business logic.
- Prefer a working-but-incomplete vertical slice over a complete-but-unconnected horizontal layer.
- The slice is production-bound code. Flesh it out; don't throw it away.
