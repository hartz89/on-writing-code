# TypeScript — Rationale

Expanded rationale for opinionated rules in [typescript.md](./typescript.md). Sections here are named to match their counterparts there. Strength levels (`non-negotiable` / `strong` / `weak`) are defined in the rules file.

---

## Types vs. Interfaces

**Rule:** Default to `type`. Use `interface` only for declaration merging or public APIs designed for `extends`.

**Strength:** weak — both work fine for most object shapes. The preference is about defaults, not dogma.

### `type` is a strict superset of `interface` for shapes

Anything an `interface` can describe, a `type` can too. But `type` can also express:

- Unions: `type Phase = 'bud' | 'open' | 'seed'`
- Primitives: `type UserId = string`
- Tuples: `type Pair = [string, number]`
- `typeof` extractions: `type Config = typeof defaultConfig`
- Mapped, conditional, and template literal types

If `FlowerProps` starts as an object and later needs a union variant, an `interface` has to be rewritten as a `type`. Starting with `type` means shape changes are additive.

### Declaration merging is usually a footgun

The defining feature of `interface` is that two declarations with the same name silently merge:

```ts
interface Flower {
  id: string;
}
interface Flower {
  phase: Phase;
} // silently merges — Flower now has both
```

This is essential for augmenting library types (`declare module 'react'`). In application code it's mostly a source of invisible, accidental merges. With `type`, duplicate declarations are a compile error — which is what you want 99% of the time.

### Consistency across a codebase

A real codebase has unions, aliases, and object shapes mixed together. Using `type` for everything means one keyword and no reader wondering "why is `FlowerProps` an interface but `Phase` a type?" Mixed use forces arbitrary rules.

### When `interface` actually wins

- **Intentional declaration merging** — augmenting third-party types, extending `Window`, module augmentation.
- **Public library APIs** where `extends` is part of the advertised contract.
- **Compiler performance at scale** — interfaces are cached references; complex `type` aliases can be re-evaluated. Only matters in very large codebases with deep type graphs.

### The honest take

The [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) itself says "choose based on personal preference" for the object-shape case. Microsoft's older style guide leaned `interface`; the modern community (including Kent C. Dodds) leans `type`. The strongest concrete reason to switch is **avoiding accidental declaration merging**. The rest is consistency and future-proofing.

---

## Immutability

**Rule:** Prefer `const`, `readonly`, and `as const`. Don't mutate arguments or shared state.

**Strength:** strong

### Mutation forces non-local reasoning

When a value can change, understanding any piece of code requires tracking every place it might be modified. Immutable data collapses that to "what is this value right now?" — you can read top-to-bottom without scanning for mutations.

### Referential equality enables optimization

React's `memo`, `useMemo`, and `useCallback` all rely on referential equality to skip work. Mutating in place produces the same reference with different content — the optimization silently breaks. Creating a new reference on change is the contract these APIs expect.

### Shared mutable state breaks concurrency

Async code, Strict Mode double-rendering, Suspense retries, and future concurrent features all assume your code is idempotent. In-place mutation of shared state violates that assumption and produces bugs that only appear under load or in production.

### `readonly` documents intent

`readonly Flower[]` tells the next reader "don't push to this." Even if your current code wouldn't mutate it, the annotation prevents a future change from introducing a bug.

### When mutation is fine

- Hot paths where allocation is measurably a problem (profile first).
- Private, local state inside a function where the mutation can't escape.
- Building up a value that you return once, immutably, at the end.

---

## Discriminated Unions

**Rule:** Model variants with string-literal unions; discriminate with `kind` or `type`; switch on the discriminant.

**Strength:** strong

### The compiler becomes your exhaustiveness checker

When you switch on the discriminant, TypeScript narrows each case to the specific variant. Every property access inside a branch is type-safe without guards or casts.

### Adding a variant surfaces every missed site

If the switch has no default (or uses a `never`-assertion default), adding a new variant to the union produces compile errors at every site that forgot to handle it. This is "make illegal states unrepresentable" in action — you cannot ship code that forgets a case.

```ts
const _exhaustive: never = event; // fails to compile when a new variant is added
```

### Why a `kind` field, not structural tests

Structural tests (`'thorns' in flower`) work but don't scale — every new property could be shared between variants, making the test ambiguous. A dedicated discriminant field is explicit, cheap, and serializes cleanly across network and storage boundaries.

---

## `any` vs `unknown`

**Rule:** Never use `any` without a justifying comment. Use `unknown` for truly unknown values; narrow before use.

**Strength:** non-negotiable

### `any` is viral

`any` doesn't stay where you put it. Every operation on an `any` value returns `any`. Pass it to a function, and the return type collapses to `any`. Destructure it, and every field is `any`. A single `any` at a data boundary can propagate through an entire codebase without the compiler complaining once.

### `unknown` gives you the same semantics with a safety boundary

`unknown` means "I don't know what this is." But unlike `any`, you can't do anything with an `unknown` until you narrow it — a `typeof`, `instanceof`, `in`, or user-defined type guard. This localizes the unsafety: you prove the type at the edge, and everything downstream is fully typed.

### Escape hatches need justification

`any`, non-null assertion (`!`), and type assertion (`as X`) all bypass the compiler. They're sometimes necessary — but each one is a place where you're claiming something the compiler can't verify. A comment explaining _why you know better than the compiler_ makes the claim reviewable. Without one, the assertion is just a wish.

---

## Enums

**Rule:** Prefer string-literal unions or `as const` objects. Don't use `enum`.

**Strength:** strong

### `enum` emits runtime code

Types and `as const` objects compile to nothing (or a plain object literal). `enum` emits a bidirectional lookup object into every file that references it. It's a small cost, but it's a cost for a feature you don't need.

### Numeric enums have surprising behavior

Numeric enums support reverse mapping (`Phase[0] === 'Bud'`), and any number is assignable to a numeric enum by default. That defeats the point of having a named type.

### `const enum` has portability problems

`const enum` avoids runtime emission but breaks under `isolatedModules`, Babel, and `ts-node` in many configurations. You end up special-casing builds to support it.

### String-literal unions and `as const` give you everything enums offer

- Named, typed constants: ✓
- Autocomplete: ✓
- Exhaustiveness in switches: ✓
- Cheap serialization: ✓ (a string is just a string)
- No runtime cost: ✓

The only thing `enum` adds is namespacing (`Phase.Bud`), and `as const` objects give you the same: `PHASE.bud`.

---

## Null vs. Undefined

**Rule:** Use `undefined` for "absent." Reserve `null` for explicit "intentionally empty" or external APIs that require it.

**Strength:** strong

### JavaScript gave you two and the language treats them differently

- `undefined`: missing property, missing argument, uninitialized variable, function with no return.
- `null`: explicitly assigned; means "I know there's nothing here, and I chose that."

Both coerce to falsy and `== null` matches both, but TypeScript treats them as distinct types. Mixing them means every signature becomes `T | null | undefined`, every guard has to handle both, and every reader must figure out which state a given absent value represents.

### Pick one default and go with the grain of the language

`undefined` wins as the default because it's what JavaScript produces automatically. Optional properties (`x?: T`) are `T | undefined`, not `T | null`. Functions without a return statement return `undefined`. Defaulting to `undefined` means fewer signatures to type, fewer guards to write.

### When `null` is appropriate

- An external API returns `null` (JSON payloads, databases, third-party libraries).
- You need to distinguish "not yet loaded" (`undefined`) from "loaded and empty" (`null`).
- Domain semantics demand "intentionally empty" as a distinct state.

In those cases, use `null` deliberately and document the distinction.
