# React

> Same principles as [typescript.md](./typescript.md): the simplest thing that works, clear over clever, small units.

Rationale for the opinionated sections lives in [react.why.md](./react.why.md). Strength levels (`non-negotiable` / `strong` / `moderate` / `trivial`) are defined in [\_index.md](./_index.md).

## Component Structure

`strength: strong`

- Components live in `PascalCase.tsx`; colocate styles in `PascalCase.css.ts`.
- Declare the props type at the top of the file. Don't export prop types — consumers use `ComponentProps<typeof Component>`.
- Arrow function assigned to a `const`, typed with `FC<Props>`.
- Always use curly braces around the body (optimize for change).

### Basic

```tsx
// filename: Flower.tsx
import { type FC } from 'react';
import { flowerPhase } from './Flower.css';

interface FlowerProps {
  phase: 'bud' | 'open' | 'seed';
  onAdvance?: () => void;
}

export const Flower: FC<FlowerProps> = ({ phase, onAdvance }) => {
  return (
    <div className={flowerPhase[phase]}>
      <button onClick={onAdvance}>Advance</button>
    </div>
  );
};
```

### With state and handlers

```tsx
// filename: FlowerBed.tsx
import { type FC, useCallback, useState } from 'react';
import { Flower } from './Flower';

type Phase = 'bud' | 'open' | 'seed';

const NEXT_PHASE: Record<Phase, Phase> = {
  bud: 'open',
  open: 'seed',
  seed: 'bud',
};

interface FlowerBedProps {
  initialPhase?: Phase;
}

export const FlowerBed: FC<FlowerBedProps> = ({ initialPhase = 'bud' }) => {
  const [phase, setPhase] = useState<Phase>(initialPhase);

  const handleAdvance = useCallback(() => {
    setPhase((current) => NEXT_PHASE[current]);
  }, []);

  return (
    <div>
      <Flower phase={phase} onAdvance={handleAdvance} />
    </div>
  );
};
```

## Props

`strength: strong`

- Prefer union literals (`'bud' | 'open' | 'seed'`) over `string`.
- Split into separate components when prop combinations are mutually exclusive or complex.

## Hook Order

`strength: trivial`

Declare hooks at the top of the component body, in this order:

1. `useContext`
2. `useReducer` / `useState`
3. `useRef`
4. Derived values / `useMemo`
5. `useCallback`
6. `useEffect` / `useLayoutEffect`

## Custom Hooks

`strength: strong`

- Prefix with `use`. Return a typed object or tuple.
- Colocate with the consumer when used once; promote to a shared `hooks/` directory when reused.

```tsx
// filename: useFlowerPhase.ts
import { useCallback, useState } from 'react';

type Phase = 'bud' | 'open' | 'seed';
const NEXT_PHASE: Record<Phase, Phase> = { bud: 'open', open: 'seed', seed: 'bud' };

export const useFlowerPhase = (initial: Phase = 'bud') => {
  const [phase, setPhase] = useState(initial);
  const advance = useCallback(() => setPhase((p) => NEXT_PHASE[p]), []);
  return { phase, advance } as const;
};
```

## Memoization

`strength: strong` · [rationale](./react.why.md#memoization)

- With the React Compiler enabled: skip manual memoization. Let the compiler handle it.
- Without the Compiler: reach for `memo` / `useCallback` / `useMemo` only when there's a measurable reason — a slow child, a stale effect, a heavy computation.
- When using `memo`, set `displayName` for readable DevTools output.

```tsx
import { type FC, memo } from 'react';

interface PetalProps {
  count: number;
}

export const Petal: FC<PetalProps> = memo(({ count }) => <span>{count} petals</span>);
Petal.displayName = 'Petal';
```

## Event Handlers

`strength: strong` · [rationale](./react.why.md#event-handlers)

- Props: `on*`. Local handlers: `handle*`. Canonical shape: `<Button onClick={handleClick} />`.
- Use a specific verb (`increment`, `saveFlower`) when it describes the action better than `handle*`.
- Inline handlers are fine for trivial one-liners.
- See [Memoization](#memoization) for when to wrap with `useCallback`.

## Conditional Rendering

`strength: moderate`

- `&&` for simple presence.
- Ternary for if/else.
- Prefer inlining conditionals in the return statement over hoisting them into intermediate variables. The JSX reads as a structural mirror of the output; pulling branches out into variables fragments that picture.
- Extract a variable or subcomponent only when the inline form gets genuinely hard to scan (nested ternaries, long branches, repeated subtrees).

```tsx
// good — inline, structure visible
return (
  <Card>
    {isOpen ? <Petals /> : <Bud />}
    {hasLabel && <Label>{label}</Label>}
  </Card>
);

// avoid — nested ternary; extract instead
{
  isOpen ? isBig ? <BigPetals /> : <SmallPetals /> : <Bud />;
}
```

## JSX Colocation

`strength: strong` · [rationale](./react.why.md#jsx-colocation)

- Keep JSX in the component's `return` statement. Don't scatter `<Foo />` expressions into intermediate variables, helper functions, or hooks declared alongside the component.
- A helper that returns JSX is a sub-component. If you're extracting one, name it, type its props, and export (or colocate) it as its own component rather than a `renderX()` function.
- `renderSomething()` helpers, JSX assigned to `const header = <div>...</div>` at the top of a component body, and hooks that return `ReactNode` are all code smells — they usually indicate the component is doing too much and wants to be split.

```tsx
// good — JSX lives in the return; sub-pieces are real components
const FlowerHeader: FC<{ name: string }> = ({ name }) => <h2>{name}</h2>;

export const FlowerCard: FC<FlowerCardProps> = ({ flower }) => {
  return (
    <article>
      <FlowerHeader name={flower.name} />
      <Petals count={flower.petalCount} />
    </article>
  );
};

// avoid — JSX scattered outside the return
export const FlowerCard: FC<FlowerCardProps> = ({ flower }) => {
  const header = <h2>{flower.name}</h2>;
  const renderPetals = () => <Petals count={flower.petalCount} />;

  return (
    <article>
      {header}
      {renderPetals()}
    </article>
  );
};
```

## Lists

`strength: strong`

- Key from data. Index keys are only acceptable for static, never-reordered lists.

```tsx
{
  flowers.map((flower) => <Flower key={flower.id} phase={flower.phase} />);
}
```

## Context

`strength: strong`

- Colocate context, provider, and consumer hook in one file.
- Export a typed `useX` hook that throws outside its provider. Don't export the raw context.

```tsx
// filename: FlowerContext.tsx
import { createContext, useContext, type FC, type ReactNode } from 'react';

interface FlowerContextValue {
  phase: 'bud' | 'open' | 'seed';
}

const FlowerContext = createContext<FlowerContextValue | null>(null);

export const FlowerProvider: FC<{ children: ReactNode; phase: FlowerContextValue['phase'] }> = ({
  children,
  phase,
}) => <FlowerContext.Provider value={{ phase }}>{children}</FlowerContext.Provider>;

export const useFlower = (): FlowerContextValue => {
  const ctx = useContext(FlowerContext);
  if (!ctx) throw new Error('useFlower must be used within FlowerProvider');
  return ctx;
};
```
