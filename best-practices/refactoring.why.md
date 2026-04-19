# Refactoring — Rationale

Expanded rationale for opinionated rules in [refactoring.md](./refactoring.md). Section names mirror the rules file.

---

## Workflows of Refactoring

**Rule:** Refactoring happens in distinct rhythms, each suited to a different moment.

**Strength:** strong

Adapted from [Martin Fowler, _Workflows of Refactoring_](https://martinfowler.com/articles/workflowsOfRefactoring/).

### Refactoring is maintenance, not an event

The "clean up later" model fails: _later_ rarely comes, and when it does, the accumulated cost dwarfs what many small improvements would have cost. Fowler's rhythms describe how refactoring actually fits into a working day — embedded in other activities, not scheduled as its own phase.

### Preparatory refactoring is the highest-leverage moment

Before adding a feature, ask: what's making this hard? Often the current structure doesn't quite fit. _Make the change easy, then make the easy change._ A 10-minute preparatory rename can turn a messy 500-line PR into a clean 50-line PR.

### Comprehension refactoring turns reading into lasting value

Reading unfamiliar code is mandatory and expensive. If you do the work of understanding and walk away, the cost repeats for every future reader. If you apply your understanding back to the code — a clearer name, a better-extracted function, a comment explaining the _why_ — you've converted a one-time cost into a durable improvement.

### Planned refactoring is a smell

Needing scheduled, carved-out time for refactoring usually means the other rhythms aren't happening. Sometimes architectural debt genuinely requires dedicated time, but a frequent need for planned refactoring is a signal to fix the culture, not just the code.

---

## Leave the Code Better Than You Found It

**Rule:** Improve code opportunistically (the campsite rule), balanced against review cost and coordination cost in large codebases.

**Strength:** strong

### The campsite rule compounds

"Leave the campground cleaner than you found it" — the formulation in _Clean Code_ (Robert C. Martin, ch. 1), adapted from the Boy Scout rule. A small improvement takes five minutes; it saves the next reader five seconds. With enough readers and enough small improvements, the time-saved overtakes the time-spent by orders of magnitude. Teams that quietly do this have codebases that stay healthy; teams that don't end up with "planned refactoring" sprints that could have been avoided.

### It isn't free in an enterprise codebase

In a codebase with many contributors and mandatory review, opportunistic refactors have real costs:

- **Reviewer load.** A PR mixing a feature with unrelated improvements is harder to review. Reviewers context-switch between "is this feature correct?" and "is this refactor correct?" — and often rubber-stamp one to get through the other.
- **Conflict with in-flight work.** Another team may have an open PR touching the same code. A drive-by rewrite can force them to rebase through nontrivial semantic changes — a tax on trust as much as time.
- **Review bandwidth isn't infinite.** Every diff costs someone's attention. The campsite rule presumes that attention is abundant; in a busy codebase, it isn't.

### How to balance

- If the cleanup is genuinely trivial (a typo, a rename, an obviously dead import), do it.
- If it's larger, ask whether the PR it's in actually needs it. If not, extract it to its own commit or PR.
- When you notice something that wants refactoring but you can't afford to do it now, leave a comment or file a tracking issue. Visibility is almost as good as action.
- Err on the side of deferring in shared, high-churn code; err on the side of fixing in code you own.
