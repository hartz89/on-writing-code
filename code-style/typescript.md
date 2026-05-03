# TypeScript

Rationale: [typescript.why.md](./typescript.why.md). Strength levels: [\_index.md](./_index.md).

## Naming

`strength: moderate`

- PascalCase for types, type aliases, and interfaces.
- camelCase for values, functions, variables.
- SCREAMING_SNAKE_CASE only for module-level literal constants.
- Prefix boolean-returning predicates with `is` / `has` / `can`.

```ts
interface FlowerBed {
  /* ... */
}
type Phase = 'bud' | 'open' | 'seed';

const flowerCount = 3;
const advancePhase = () => {
  /* ... */
};

const MAX_FLOWERS = 12;

const isBlooming = (flower: Flower) => flower.phase === 'open';
```

## Types vs. Interfaces

`strength: moderate` · [rationale](./typescript.why.md#types-vs-interfaces)

- Default to `type` for all shapes, unions, and aliases.
- Use `interface` only for intentional declaration merging or public APIs designed for `extends`.

```ts
// good — type for unions and simple shapes
type Phase = 'bud' | 'open' | 'seed';
type Point = { x: number; y: number };

// good — interface when extension is the point
interface FlowerProps {
  phase: Phase;
}
interface RoseProps extends FlowerProps {
  thorns: number;
}
```

## Immutability

`strength: strong` · [rationale](./typescript.why.md#immutability)

- Prefer `const` over `let`.
- Mark fields `readonly` when they shouldn't mutate.
- Use `as const` for literal-typed immutable data.
- Don't mutate arguments or shared state.

```ts
const phase = 'bud';

interface FlowerBed {
  readonly id: string;
  readonly flowers: readonly Flower[];
}

const PHASES = ['bud', 'open', 'seed'] as const;
type Phase = (typeof PHASES)[number];
```

## Inference

`strength: strong`

- Annotate exported function parameters and return types.
- Let inference handle locals and internal helpers.
- Don't annotate when the initializer tells the full story.

```ts
// exported — annotate the contract
export const advance = (phase: Phase): Phase => NEXT_PHASE[phase];

// locals — inference does the work
const count = flowers.length;
const names = flowers.map((f) => f.name);

// avoid — redundant
const count: number = flowers.length;
```

## Unions and Discriminated Unions

`strength: strong` · [rationale](./typescript.why.md#discriminated-unions)

- Model variants with string-literal unions.
- Discriminate with a `kind` (or `type`) field.
- Switch on the discriminant; the compiler narrows each branch exhaustively.

```ts
type FlowerEvent =
  | { kind: 'bloomed'; flowerId: string; at: Date }
  | { kind: 'withered'; flowerId: string; reason: string };

const describe = (event: FlowerEvent): string => {
  switch (event.kind) {
    case 'bloomed':
      return `flower ${event.flowerId} bloomed at ${event.at.toISOString()}`;
    case 'withered':
      return `flower ${event.flowerId} withered: ${event.reason}`;
  }
};
```

## Avoid `any`; Prefer `unknown`

`strength: non-negotiable` · [rationale](./typescript.why.md#any-vs-unknown)

- Never use `any` without a comment explaining why.
- Use `unknown` for values of truly unknown shape; narrow before use.
- Escape hatches (`any`, `!`, `as X`) require a justifying comment.

```ts
// good
const parseFlower = (raw: unknown): Flower => {
  if (typeof raw !== 'object' || raw === null) throw new Error('not a flower');
  return raw as Flower;
};

// avoid
const parseFlower = (raw: any): Flower => raw;
```

## Avoid `as` Casting

`strength: strong` · [rationale](./typescript.why.md#avoid-as-casting)

- Treat `as X` as a last resort. Prefer narrowing, type guards, schema validation, or fixing the upstream type.
- Acceptable with a justifying comment: post-validation handoffs, `as const`, bridging incorrect third-party types.
- Never use `as` to silence an error.

```ts
// good — narrow, use naturally
if (isFlower(raw)) raw.phase;

// acceptable — schema validated, cast to nominal type
const data = FlowerSchema.parse(raw);
return data as ParsedFlower;

// avoid
const flower = getFlower() as Flower;
```

## Strict Mode

`strength: non-negotiable` · [rationale](./typescript.why.md#strict-mode)

- Enable `strict: true`. Don't disable individual strictness flags without a justifying comment.
- Relaxed strictness is acceptable in test/mock files via a separate `tsconfig.test.json`.

## Type Narrowing

`strength: strong`

- Use user-defined type guards for reusable narrowing.
- Name guards `isX`.

```ts
const isRose = (flower: Flower): flower is Rose => flower.kind === 'rose';

if (isRose(flower)) {
  flower.thorns; // narrowed to Rose
}
```

## Utility Types

`strength: trivial`

- Use built-ins (`Pick`, `Omit`, `Partial`, `Required`, `ReturnType`, `Parameters`) over redefining shapes.
- If a derived type obscures intent, spell the shape out.

```ts
// good
type FlowerSummary = Pick<Flower, 'id' | 'phase'>;

// avoid — the layers hide the actual shape
type WeirdFlower = Omit<Partial<Pick<Flower, keyof Rose>>, 'thorns'>;
```

## Generics

`strength: strong`

- Prefix generic names with `T` in public APIs: `TItem`, `TKey`, `TValue`. It marks them as type parameters at a glance and keeps them from colliding with concrete types sharing the same conceptual name.
- Bare `T` is fine for tight local helpers where there's only one type parameter and the context is obvious.
- Constrain generics — an unconstrained `TItem` is usually too loose.

```ts
export const pluck = <TItem, TKey extends keyof TItem>(
  items: readonly TItem[],
  key: TKey
): TItem[TKey][] => items.map((item) => item[key]);

const first = <T>(items: readonly T[]): T | undefined => items[0];
```

## Enums

`strength: strong` · [rationale](./typescript.why.md#enums)

- Prefer string-literal unions or `as const` objects.
- Don't use `enum`.

```ts
// good
const PHASE = { bud: 'bud', open: 'open', seed: 'seed' } as const;
type Phase = (typeof PHASE)[keyof typeof PHASE];

// avoid
enum Phase {
  Bud = 'bud',
  Open = 'open',
  Seed = 'seed',
}
```

## Null vs. Undefined

`strength: strong` · [rationale](./typescript.why.md#null-vs-undefined)

- Use `undefined` for "absent."
- Reserve `null` for explicit "intentionally empty" or external APIs that demand it.
- Access optional values with `?.` and `??`.

```ts
const find = (id: string): Flower | undefined => flowers.find((f) => f.id === id);
const name = flower?.name ?? 'unknown';
```

## Function Signatures

`strength: strong` · [rationale](./typescript.why.md#function-signatures)

- Default to arrow functions at module scope.
- Use `function` declarations when hoisting helps (intent-first at the top, helpers below).
- Parameter shape:
  - One or two first-class inputs: positional (`clamp(value, min, max)`).
  - Primary + config: positional primary, options object (`fetchFlower(id, { includeSeeds: true })`).
  - 3+ first-class inputs, or any boolean param: single options object.
- Return early; avoid deep nesting.

```ts
interface PlantOptions {
  phase?: Phase;
  waterLevel?: number;
  fertilizer?: boolean;
}

export const plant = (id: string, options: PlantOptions = {}): Flower => {
  if (!id) throw new Error('id is required');
  if (options.waterLevel !== undefined && options.waterLevel < 0) {
    throw new Error('waterLevel must be non-negative');
  }
  // ... happy path
};

export const clamp = (value: number, min: number, max: number): number =>
  Math.min(Math.max(value, min), max);

// function declaration hoisted for top-down readability
export const renderRow = (flower: Flower): Row => ({
  name: formatName(flower),
  phase: formatPhase(flower.phase),
});

function formatName(flower: Flower): string {
  /* ... */
}
function formatPhase(phase: Phase): string {
  /* ... */
}
```

## Flatness and Extraction

`strength: strong` · [rationale](./typescript.why.md#flatness-and-extraction)

- Keep code flat. Extract a helper with a descriptive name when a block's logic gets gnarly.
- For expressions that need statements, reach for an IIFE — keeps the binding `const`, keeps computation local.

```ts
// good — extracted helper
const displayName = pickDisplayName(flower, viewer);

// good — IIFE for a multi-step const
const label = ((): string => {
  if (flower.phase === 'seed') return 'dormant';
  if (flower.waterLevel < 3) return 'thirsty';
  return flower.name;
})();

// avoid — nested conditionals with a mutable accumulator
let label: string;
if (flower.phase === 'seed') {
  label = 'dormant';
} else {
  if (flower.waterLevel < 3) {
    label = 'thirsty';
  } else {
    label = flower.name;
  }
}
```

## Iteration

`strength: moderate`

- Prefer array methods (`map`, `filter`, `reduce`, `forEach`, `find`, `some`, `every`) for straightforward transformations and traversals.
- Reach for an explicit `for` / `for...of` loop when you need fine-grained flow control: early exit (`break`), selective skipping based on coordinated state, or async sequencing where parallelism isn't safe.

```ts
// good — array method for a clean transformation
const openFlowers = flowers.filter((f) => f.phase === 'open');

// good — explicit loop when you need control
for (const flower of flowers) {
  if (flower.phase === 'seed') continue;
  const result = await fertilize(flower);
  if (!result.ok) break;
}
```

## Comments

`strength: moderate`

- Code says _what_; comments say _why_.
- Skip the comment if the why is obvious from naming and structure.
- Never restate the code.

```ts
// good — explains a non-obvious constraint
// Bud phase is intentionally skipped on re-plant to avoid double-advance on Strict Mode remount.
const initialPhase: Phase = isReplant ? 'open' : 'bud';

// avoid — restates the code
// set the phase to bud
const phase = 'bud';
```
