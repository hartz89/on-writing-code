# Architecting Features — Rationale

Expanded rationale for opinionated rules in [architecting-features.md](./architecting-features.md). Section names mirror the rules file.

---

## File and Directory Structure

**Rule:** Verbose filenames, thin barrels, concise directory names, flat trees.

### Filename verbosity

Developers navigate codebases by jumping to files — fuzzy finders, IDE "go to file," grep output. A filename like `component.tsx` appears in a dozen places and tells you nothing. `FlowerCard.tsx` or `useFlowerSubscription.ts` is unambiguous at a glance. Directory names provide surrounding context, so files don't need to repeat it — but that's an argument for shorter directory segments, not shorter filenames.

### Barrel discipline

Many projects don't need barrel files at all. Direct imports are explicit, greppable, and impose no bundler overhead. A barrel is worth adding only when it delivers a concrete benefit — most commonly a stable public API surface for a shared package, where callers shouldn't care which internal file owns a symbol.

When you do use barrels, keep them to pure re-exports. Logic in `index.ts` is invisible from the outside — a consumer importing from `./flowers` has no idea what else is co-located, and the implementation detail can only be surfaced by static analysis.

Size matters too. When a barrel re-exports an entire feature tree, most bundlers treat every export in the barrel as a single graph node — a consumer importing one symbol pulls in the module graph for all of them, blocking dead-code elimination across that boundary. The fix is scope: a barrel covering a small, cohesive set of exports is fine; one that aggregates a whole domain into a single public surface is a bundler footgun.

### Directory conciseness and flatness

Directory names are read in sequence — `features/flowers/FlowerCard.tsx` — so each segment only needs to add context the previous one didn't. Single-word directory names are fine.

Flat trees reduce the cognitive cost of locating files and lower the friction of moving them. A one-file-per-directory pattern forces you to open several nested folders to see a handful of related files. Nest only when an inner group is genuinely self-contained or meaningfully isolated from its siblings — not as a default organizational reflex.

---

## Prove the Pipeline Early

**Rule:** Build a thin end-to-end slice first before filling out any layer in full.

Adapted from [_The Pragmatic Programmer_, Hunt & Thomas](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/) — "Tracer Bullets" chapter.

### The integration tax is cheapest early

The failure mode this rule guards against: discovering integration mismatches after each layer is complete. Fixing a mismatch between a finished API and a finished UI is far more expensive than catching it when both are thin. The thin slice is the tracer — it traces a path through the real system and shows immediately where something doesn't connect.

### Tracer bullets, not prototypes

Hunt & Thomas draw a sharp distinction. A prototype is written to be thrown away — exploratory, unpolished, expected to be discarded. A tracer slice is production code: it has tests, follows conventions, and ships. The difference is scope (thin) not quality.

### Working horizontally is locally efficient and globally costly

The common alternative: build all API endpoints before touching the UI. Less context-switching per session — but you pay the integration tax at the end, when every layer is already optimized in isolation and the blast radius of a mismatch is largest.
