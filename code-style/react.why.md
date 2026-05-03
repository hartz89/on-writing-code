# React — Rationale

Expanded rationale for opinionated rules in [react.md](./react.md). Section names mirror the rules file.

---

## Memoization

**Rule:** With the React Compiler, skip manual memoization. Without it, use `memo` / `useCallback` / `useMemo` sparingly — only when there's a measurable reason.

### Manual memoization is easy to get wrong

`memo`, `useCallback`, and `useMemo` all take dependency arrays and referential-equality rules that are easy to miscount. A stale dep list silently skips updates; a missed dep causes infinite re-renders.

Most teams offload this to a linter — `eslint-plugin-react-hooks`' `exhaustive-deps` rule catches the vast majority of dep-array mistakes before review. Treat it as table stakes: if it's on, the risk of getting deps wrong drops dramatically; if it's not, add it. Either way, the cost of reasoning about when to memoize is still real, and most of the time the "optimization" saves nothing.

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

---

## Event Handlers

**Rule:** Props are `on*`; local handlers are `handle*`. Canonical shape: `onClick={handleClick}`.

Codified in the React docs, [Responding to Events](https://react.dev/learn/responding-to-events): event-handler props start with `on`, handler functions start with `handle`. Following the convention is free; re-litigating it on every PR isn't.

The two names do different jobs. `onClick` is the event (what the component accepts); `handleClick` is the response (what this component does). Collapsing them to `onClick={onClick}` loses that distinction and reads as a tautology.

When a specific verb fits the action better than `handle*` — `onIncrementClick={increment}`, `onDismiss={close}` — use it. Default to `handle*`; don't force it onto a function whose real name is `increment`.

---

## JSX Colocation

**Rule:** Keep JSX in the component's return statement. Extract real sub-components, not `render*` helpers.

### JSX scattered across a file is a shape problem, not a style problem

When you read a React component, the `return` statement is the summary of what it produces. If half of that output is assembled from JSX constants at the top of the function, JSX returned from inline helpers, or JSX returned from custom hooks, the summary is no longer in one place — you have to trace through the function body to piece the output back together.

### `renderX()` is a sub-component wearing a disguise

A function inside a component that takes no arguments (or closes over the component's state) and returns JSX is, structurally, a sub-component. But it's one that can't be memoized, can't be typed as a component, doesn't show up in DevTools as its own node, and re-creates its JSX on every parent render. The fix is to make it a real component — give it a name, give it props, lift it above (or beside) the parent. You get memoizability, DevTools legibility, testability, and a genuine structural seam.

### Hooks that return JSX are worse

A hook that returns `ReactNode` fuses two concerns: stateful behavior (the thing hooks are for) and rendering (the thing components are for). It's impossible to consume the hook without also rendering its output; it's impossible to render that output without also running the hook. The coupling is hidden behind a `use*` name. Pull the render out into a component that takes the hook's result as props.

### When the pattern appears, it usually signals a split

The impulse to break JSX out of the return statement often comes from "this component is getting long." That's a real signal — but the answer is extracting a sub-component, not extracting a render helper. The sub-component solves the length problem _and_ gives you a reusable, testable, memoizable unit. The helper just hides the length.
