# React — Rationale

Expanded rationale for opinionated rules in [react.md](./react.md). Section names mirror the rules file. Strength levels are defined in [\_index.md](./_index.md).

---

## Memoization

**Rule:** With the React Compiler, skip manual memoization. Without it, use `memo` / `useCallback` / `useMemo` sparingly — only when there's a measurable reason.

**Strength:** strong

### Manual memoization is easy to get wrong

`memo`, `useCallback`, and `useMemo` all take dependency arrays and referential-equality rules that are easy to miscount. A stale dep list silently skips updates; a missed dep causes infinite re-renders. The cost of getting it wrong is real, and most of the time the "optimization" saves nothing.

### Most components don't need it

React's render pass is already fast. Memoizing every handler and child creates complexity — new hook calls, dependency arrays to maintain — for a cost that doesn't show up in profiling. Reach for memoization when you have a specific slow path (deep trees, heavy computations, a flame graph showing waste), not prophylactically.

### When you _do_ need it

- A child is wrapped in `memo` and a parent passes it a handler — wrap that handler in `useCallback`, otherwise the `memo` is defeated every render.
- A `useEffect` depends on a value recomputed every render — `useMemo` it so the effect doesn't re-run.
- A component is genuinely expensive (large lists, complex layouts) and receives stable props — wrap in `memo`.

### The React Compiler changes the default

When enabled, the Compiler auto-memoizes components, values, and handlers. Manual `memo` / `useCallback` / `useMemo` become redundant and can interfere with the Compiler's output. The default flips: _don't_ reach for these; trust the Compiler.

### `displayName` on memo components

`memo(Component)` returns an anonymous component. DevTools shows it as `Memo(Unknown)` unless you assign `displayName`. One line, real debugging payoff.
