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

## Avoid `as` Casting

**Rule:** Treat `as X` as a last resort. Prefer narrowing, type guards, or schema validation.

**Strength:** strong

### `as` silences the compiler without proving anything

The type system is only as strong as its weakest link. Every `as` is a claim — "trust me, this is a `Flower`" — that the compiler cannot verify. One unjustified cast upstream poisons every downstream assumption that the type is accurate, and the failures surface at runtime instead of at compile time. That's the thing TypeScript is supposed to prevent.

### Narrowing proves what `as` asserts

`typeof`, `instanceof`, `in`, and user-defined guards prove the type to the compiler. A schema validator (Zod, Valibot) proves the type to the runtime _and_ the compiler. Both give you fully-typed access to the value without the trust gap. The cost of doing it right is usually a few lines; the cost of getting it wrong is a production bug traced back to a five-character cast.

### The acceptable cases are narrow

- **Post-validation handoff:** you've parsed a value with a schema at the boundary and are handing the result across an API that wants a nominal type. The cast is a formality; the proof happened one line up.
- **`as const`:** a compile-time inference hint, not a type assertion. Different mechanism, different risks.
- **Third-party type bugs:** the library's declared type is wrong and you can't fix upstream. Cast with a comment naming the library and the bug.

Everything else — casting to silence an error, casting "because I know better," casting to fit a round peg in a square hole — is papering over a real problem.

---

## Strict Mode

**Rule:** Enable `strict: true`. Don't disable individual strictness flags without justification.

**Strength:** non-negotiable

### Null-pointer errors are the most common avoidable bug

`strictNullChecks` alone catches the largest class of runtime bugs TypeScript can prevent. Without it, every value that's actually `T | null | undefined` is typed as `T`, and the compiler lets you dereference it freely. You'll hit `Cannot read property of undefined` in production that a single tsconfig flag would have caught at compile time.

### Strict is a package deal

`strict: true` bundles `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `useUnknownInCatchVariables`, `alwaysStrict`, and `noImplicitThis`. Each of them closes a real gap. Turning one off usually means a specific codebase pattern is fighting the compiler — fix the pattern, not the flag.

### Test files are the reasonable exception

Test files lean heavily on partials (`{ name: 'test' } as User`) and mocks that don't need full type coverage. A separate `tsconfig.test.json` with `strict: false` or `strictNullChecks: false` relaxes the rules where rigor costs more than it gives — without weakening the production code's guarantees.

---

## Function Signatures

**Rule:** Default to arrows; use `function` when hoisting helps; choose positional vs options by the function's shape.

**Strength:** strong

### Arrow functions are the consistent default

At module scope, arrow-function expressions read the same whether the value is a function, an object, or a primitive: `const x = ...`. No mental toggle between declaration forms. They also compose naturally with `const`, `export const`, and point-free utility patterns.

### `function` declarations earn their keep via hoisting

Hoisting lets you write a file intent-first: the exported function at the top, supporting helpers below in the order they're called. The reader gets the summary before the details, and only parses the helpers when they need to. With `const` arrows, you either put helpers above the thing that uses them (implementation-before-intent) or accept a "`foo` is used before its declaration" error. Allowing `function` for local helpers is the pragmatic middle — defaults to arrow, reaches for `function` when it makes the file read better.

### Shape of the parameter list follows shape of the function

- **Multiple first-class inputs:** positional. `clamp(value, min, max)` reads naturally; `clamp({ value, min, max })` is needless ceremony.
- **Primary + options:** one positional, one object. This is what most functions grow into — a main thing they operate on, and a set of modifiers. `fetchFlower(id, { includeSeeds })` keeps the important argument visible at the call site while letting configuration stay named.
- **Boolean-typed parameters:** always named. `plant(id, true, false)` is unreadable at every call site forever; `plant(id, { fertilize: true, water: false })` tells the reader what it means.

Over-using positional arguments on complex signatures makes call sites brittle and opaque. Over-using options objects on simple ones turns every call into pointless destructuring. Match the shape.

---

## Flatness and Extraction

**Rule:** Keep code flat; extract helpers for complex conditional logic; reach for IIFEs when an expression needs statements.

**Strength:** strong

### Nesting is the best predictor of hard-to-change code

Cyclomatic complexity, cognitive complexity, "how long does it take to understand this function" — they all correlate with indentation depth. Every level of nesting is another piece of state the reader has to carry ("we're in the branch where `x` is true, and also the branch where `y` is defined, and inside the loop over `z`..."). Flat code is scannable; deeply nested code has to be simulated line-by-line.

### Extraction is the conditional-logic escape valve

When a block's decision tree won't flatten — nested conditions, early-return patterns that don't fit the surrounding flow, multi-step validation — pull it into a named helper. The helper's name replaces the comment that would otherwise explain what the block does. The outer function shrinks back to a linear sequence of verbs. This is the highest-leverage refactor available in most codebases.

### IIFEs are the immutability escape valve

Sometimes producing a `const` value takes more than one line: a switch, a chain of guards, a try/catch. The temptation is `let result; if (...) result = ...; else result = ...;` — but a mutable binding with five assignment sites is harder to reason about than an immutable one computed by a local function.

An IIFE — `const x = ((): T => { ... })()` — keeps the binding `const`, keeps the computation scoped to the expression, and doesn't leak temporaries into the surrounding function. It looks a little unusual the first time; it reads cleanly once you've seen the pattern. The alternative is either mutation or an unnecessary top-level helper called exactly once.

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
