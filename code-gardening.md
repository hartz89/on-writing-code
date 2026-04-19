# Code Gardening

Software isn't a static structure; it's a living ecosystem. The "building" metaphor is a trap—it implies that once the last brick is laid, you're done. In reality, large-scale engineering is about continuous maintenance. We don't just build systems; we tend to them.

### Infrastructure as Soil Health

A plant is only as healthy as the soil it grows in. In a codebase, your "soil" is your infrastructure, CI/CD pipelines, and core architectural patterns. If the environment is depleted—meaning brittle tests, inconsistent DX, or poor documentation—even the best features will eventually rot. Invest in the soil so that everything else has a chance to thrive.

### Weeding is Mandatory

A garden left alone quickly becomes a thicket. In software, "weeds" are the things that actively harm the codebase: technical debt, dead code, outdated dependencies, and broken windows. Weeding isn’t a sign that something went wrong; it’s a requirement for the system to survive.

The harder problem is compounding decay. A broken window left unfixed signals that decay is acceptable — and that signal spreads. One ignored `TODO`, one commented-out block, one dependency three major versions behind, and the team stops noticing. If you can’t fix a broken window immediately, ticket it or quarantine it so the decay doesn’t normalize.

### Pruning is Important

Weeding is reactive — it removes what’s broken or rotting. Pruning is proactive — it cuts back what’s healthy but overgrown: unused feature flags, kept-just-in-case abstractions, safety nets no one is tripping anymore.

Left to accumulate, overgrowth costs the same as weeds: slower reads, higher cognitive load, more code to change when the system shifts. Prune before it hardens into load-bearing complexity.

### Intentional Cultivation

Healthy growth isn't an accident. When adding to a system, don't just "plug and play" a fix. Consider the landscape. New features should be integrated naturally, with clear boundaries and maintainable patterns. If you just tack things on without a plan, you disrupt the balance of the whole ecosystem.

### From Builder to Steward

This is a mindset shift from "builder" to "steward." The goal isn't just to ship a feature and move on; it's to nurture a system that remains resilient, scalable, and actually pleasant to work in two years from now. A codebase is never "finished"—it's either growing, being tended to, or dying.
