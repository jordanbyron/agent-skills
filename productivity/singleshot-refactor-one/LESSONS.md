# Lessons

Generic refactoring lessons learned from rejected requests. Stack-agnostic only — no project names, file paths, language
features, or framework APIs (those belong in the project's own context). Every run reads these as hard constraints. One
imperative rule per line, with the why.

<!-- Example:
- **Don't extract a single-use helper** — inlining reads better than a one-caller indirection.
  Why: the extraction added a hop without removing duplication.
-->

- **Don't delete an unreachable trailing return after an infinite loop** (e.g. a `while true` whose only exits are early
  returns). Why: a statically-typed compiler's "all code paths return a value" analysis still requires it, so removing
  it breaks the build even though the line never executes at runtime — and a separate linter may pass while the real
  compiler rejects it.
- **Don't collapse self-documenting named functions into one parametric helper when the names carry intent the call site
  would otherwise lose.** Distinct variants like an "attacker" and a "defender" function each read clearly where they're
  called; a single `helper(x)` forces the reader to decode the argument to recover what the name used to state. Why: the
  worth of a small, well-named function is the readability it gives its call site, not its line count, so collapsing
  trivial duplication trades that away for a DRY win that doesn't pay for itself. Judge the collapse by whether the
  merged call site reads as clearly as the separate names did — not by lines removed.
- **Don't delete a liveness/validity guard on a dependency just because that dependency is a required constructor
  parameter that is never reassigned.** Why: constructor-param reasoning proves the dependency was valid at
  construction, not that it is valid at call time. An object that registers itself with a global event bus, singleton
  registry, or observer list at construction can outlive the dependency it was handed, then get re-entered through that
  global after the dependency is destroyed — so the "impossible" state is reachable. Sibling methods dereferencing the
  same field unguarded is evidence of an inconsistency, not evidence the guard is dead. Before removing such a guard,
  find what keeps the object reachable after its owner dies; if a global holds a reference, keep the guard.
- **Fetch the upstream default branch and branch from it immediately before implementing — never from whatever the
  working copy happened to be on.** Why: these runs are often executed concurrently or back-to-back, so another run can
  land the very same cleanup while this one is scanning. Branching from a stale base yields a request that conflicts and
  whose diff is already subsumed, wasting the whole run. If the scan is separated from the implementation by more than a
  moment, re-verify the chosen candidate still exists at the fetched head before editing.
- **When a request's target has already been refactored upstream, abandon the branch and pick a different candidate —
  don't rebase and salvage.** Why: a superseded change rebases to an empty or near-empty diff, and re-applying a
  narrower version on top of the broader one that landed is strictly worse than the existing code. Prefer the upstream
  version even when it arrived second.
- **Deleting a constructor parameter is not severing a dependency when the unit still needs that collaborator and now
  reaches it through another object.** If two parameters happen to hold the same instance, dropping one and reading the
  collaborator off the other is a reach-through, not a severance. Why: the argument count falls while coupling widens —
  the unit still depends on the collaborator, plus on the intermediary, plus on the intermediary continuing to expose
  it. A real severance removes the _need_ for the collaborator. Before claiming one, check that the unit stops using the
  thing entirely; if it doesn't, prefer taking it from the component whose job is to own it over whichever neighbour
  merely has a reference.
- **Don't reroute a dependency through an object whose role is not to supply it, even when that shortens the
  signature.** Where a project is deliberately building modular seams, the component that _owns_ a collaborator is the
  right thing to depend on; a view, node, or model object that merely holds the same reference is not a supply channel.
  Why: routing through it quietly turns an incidental reference into a load-bearing public path, eroding the very
  boundaries the module structure exists to create — a structural regression a green test suite cannot catch. When a
  candidate's payoff is only "fewer constructor arguments," treat it as cosmetic and weigh it against the seam it
  crosses.
- **Name an extracted helper for its role in the domain, never for its mechanism or storage format** — `JsonFile`,
  `AtomicWrite`, `StringUtils` say how, not what. Why: a reviewer asked "what is this json file in the domain?" — the
  name carried no answer, so the reader had to open both callers to find out it was the app's own persisted records.
- **Never leave section-divider comments (`# ----- foo -----`) inside a class you created or reshaped; split it into the
  concepts the dividers name** — a class that needs headings is two concepts sharing a name. Why: a reviewer flagged
  `# ----- pane` / `# ----- fetch` inside one class as "a smell: if we need this it means we want an actual domain
  concept to encapsulate this"; the fix was a second class with the fetch in it and an injected dependency, which also
  removed every pass-through.
