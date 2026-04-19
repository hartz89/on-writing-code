# Testing

> Test for the confidence the tests give you — not for a coverage number. The Testing Trophy (static + integration) tends to give the best return on effort.

Rationale for the opinionated sections lives in [testing.why.md](./testing.why.md). Strength levels (`non-negotiable` / `strong` / `weak`) are defined in [../code-style/\_index.md](../code-style/_index.md).

## Guiding Principles

- Static typing is your first layer of testing. Don't duplicate what the compiler already proves.
- Prefer integration tests. Unit-test pure logic; use e2e sparingly.
- Test the behavior the caller sees; don't assert on implementation details.
- Tests are code. Same naming, same small-units, same readability rules.

## Testing Trophy

`strength: strong` · [rationale](./testing.why.md#testing-trophy)

- Lean on static typing (TypeScript) as the base layer.
- Integration tests for most behavior — real units working together.
- Unit tests for pure, isolable logic (parsers, reducers, utilities).
- E2E for critical user flows only — slower and flakier; spend the budget where it matters.

## Coverage

`strength: strong` · [rationale](./testing.why.md#coverage)

- Don't target a coverage percentage.
- Use coverage reports to find untested code, not to measure quality.
- A high-coverage suite full of implementation-detail tests is worse than a lower-coverage suite that tests behavior.

## Test Behavior, Not Implementation

`strength: strong` · [rationale](./testing.why.md#test-behavior-not-implementation)

- Assert what the caller or user sees.
- Don't assert on internal state, private methods, or instance fields.
- A test that breaks on a behavior-preserving refactor is a broken test.

```tsx
// good — tests behavior
const user = userEvent.setup();
render(<FlowerBed initialPhase="bud" />);
await user.click(screen.getByRole('button', { name: /advance/i }));
expect(screen.getByText('open')).toBeInTheDocument();

// avoid — tests implementation
expect(wrapper.instance().state.phase).toBe('open');
```

## Mock at the Furthest Reasonable Boundary

`strength: strong` · [rationale](./testing.why.md#mock-at-the-furthest-reasonable-boundary)

- Mock as far toward the edge of your code as you reasonably can. Network over function; edge-of-SDK over internal module.
- Prefer MSW (or equivalent) for HTTP — intercept at the network layer, not the `fetch` wrapper.
- Don't mock internal modules. If the only way to test something is to mock an internal module, the test is probably at the wrong level.
- "Reasonable" is a real constraint. Some SDKs don't expose a clean network seam; stop at the last layer that maps to something real.

## Avoid Nesting

`strength: strong` · [rationale](./testing.why.md#avoid-nesting)

- Default to **no** `describe` blocks. Flat `test` at file root.
- Reserve `describe` for two narrow cases: a module exporting several related functions, or a large integration suite with distinct behavior groups. Both are also mild code smells — frequent reaching usually means the subject-under-test is doing too much.

```ts
// good — flat, explicit
test('advances from bud to open', () => {
  const flower = makeFlower({ phase: 'bud' });
  advance(flower);
  expect(flower.phase).toBe('open');
});

test('advances from open to seed', () => {
  const flower = makeFlower({ phase: 'open' });
  advance(flower);
  expect(flower.phase).toBe('seed');
});
```

## Test Structure

`strength: strong` · [rationale](./testing.why.md#test-structure)

- Arrange, act, assert: set up, invoke, check. Separate sections with blank lines, not comments.
- Prefer **fewer, longer tests** that walk a realistic flow over many tiny tests that each verify one thing. A long integration test is still one test if it exercises one user story — even with several act-assert beats.
- AAA is a shape, not a shackle: a single test may iterate arrange → act → assert multiple times as it drives a flow through successive states. Keep beats colocated in one test when they belong to the same story.
- Split when two beats are exercising truly unrelated behaviors.

## Setup Functions

`strength: strong` · [rationale](./testing.why.md#setup-functions)

- Start with setup inline. Duplication is fine until it hurts.
- Extract a setup function (`makeFlower({ phase: 'bud' })`) when repetition drowns the test's intent. Each test calls it explicitly, overriding only what matters.
- Reserve `beforeEach` for fixture isolation (`jest.clearAllMocks()`, fake-timer resets), not building domain objects.

## Naming

`strength: strong`

- Prefer `test()` over `it()`. The `it()` form forces a grammatical frame ("it does X"); `test()` lets you describe what's under test without twisting syntax.
- Describe behavior and condition, not function or variable names.

```ts
// good
test('advances from bud to open');
test('throws when phase is invalid');
test('FlowerBed: initial render shows bud');

// avoid
test('setPhase');
test('works');
```

## F.I.R.S.T.

`strength: strong`

Tests should be:

- **Fast** — optimize for suite speed, not individual test length. Fewer, longer tests often produce a faster overall suite than many tiny ones.
- **Independent** — order-independent and parallel-safe; no shared mutable state.
- **Repeatable** — deterministic. No live network, no wall-clock, no un-seeded randomness.
- **Self-validating** — pass/fail by assertion, without manual inspection at execution time. Debugging tools (`screen.debug()`, `logRoles`) support test authoring, not pass/fail determination.
- **Timely** — written with the code, not months later.

From _Clean Code_, ch. 9.
