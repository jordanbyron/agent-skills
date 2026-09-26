# Assessment

How to detect what's already covered, decide whether an untested file _should_ be tested, and rank the gaps. Generic
across languages, frameworks, and projects.

## Detect the test convention first

Before judging coverage, learn how _this_ repo tests. From existing tests and config, determine:

- **Runner & layout** — where tests live and how they're named. Common patterns (use the repo's, not these defaults):
  - sibling: `foo.py`→`test_foo.py`/`foo_test.py`, `foo.go`→`foo_test.go`, `foo.gd`→`foo_test.gd`
  - suffix: `Foo.java`→`FooTest.java`, `Foo.cs`→`FooTests.cs`, `foo.ts`→`foo.spec.ts`/`foo.test.ts`
  - directory: `__tests__/`, `tests/`, `spec/`, `test/` mirroring the source tree
  - inline: `#[cfg(test)] mod tests` (Rust), doctests, same-file test blocks
- **Granularity** — one test file per class, per module, or per public function. Match it; don't impose 1:1 if the repo
  groups differently.
- **Covered ≠ has-a-matching-file.** A class with no `foo_test.x` may still be fully exercised inside a sibling's test
  or an integration/e2e suite. Treat it as covered. Only files whose behavior nothing asserts are gaps.

## Skip — a test usually does NOT make sense

Mark **skip** (with the reason) when the file is any of:

- **Generated / vendored** — codegen output, lockfiles, protobufs, vendored third-party code. Tests would assert the
  generator, not your code.
- **Pure data / configuration** — constants, enums, fixtures, JSON/YAML/TOML, env wiring with no logic.
- **Type-only declarations** — interfaces, protocols, abstract signatures, type aliases with no behavior.
- **Trivial pass-throughs** — plain DTOs/structs, auto getters/setters, barrel/`index` re-export files, one-line
  delegations the compiler already guarantees.
- **Declarative wiring** — route tables, DI registration, dependency manifests, framework bootstrap/`main` whose only
  job is composition (better served by one integration/e2e test than per-file units).
- **Presentation-only** — templates, markup, stylesheet logic, view code with no branching behavior. (Test behavior, not
  UI styling or layout.)
- **Already covered indirectly** — its public behavior is asserted by a higher-level test, even with no 1:1 file.

A skip is a real verdict, not a cop-out: record _why_ so the report stands on its own.

## Needs-test — a test is worth writing

Mark **needs-test** when the file carries behavior that can silently break:

- **Branching / conditional logic** — decisions, state machines, dispatch.
- **Computation & transformation** — calculations, parsing, serialization, mapping, formatting.
- **Domain / business rules** — the logic that encodes how the product is _supposed_ to behave.
- **Error & edge handling** — boundary conditions, failure paths, retries, validation.
- **Bug-prone primitives** — regex, date/time math, money/units, sorting, concurrency, off-by-one-prone loops.
- **A public API surface** with non-trivial behavior that callers rely on.

The strongest candidates combine several and have an observable contract you can assert without reaching into internals.

## Scoring — pick exactly one

Rank needs-test files by `value × confidence ÷ risk`:

- **value** — how much breakage a test would catch: behavior density, blast radius if it broke, how central it is.
- **confidence** — how cleanly the behavior can be asserted through the public API _without_ redesigning the code or
  testing internals. High when the contract is clear; low when you'd have to mock the world.
- **risk** — how likely the test is to be brittle or to encode implementation detail (heavy mocking, timing, private
  state). High risk drops it down even if valuable.

Favor a high-value file with a clean, stable contract. On a tie, take the one whose test is smallest and most
behavioral.

## Guardrails (hard)

- **Test behavior, not implementation.** Assert observable outputs and effects through the public API. Never assert
  private state, call order, UI styling, or things that change when the code is refactored without behavior changing.
- **One file's tests per request.** A single coherent, reviewable MR/PR. The next gap is the next run.
- **Follow the project's convention.** Naming, layout, granularity, and assertion style match what's already there.
- **Don't change production code to make it testable** in this run. If a file needs a design change to be tested, say so
  in the report and pick another candidate — that's a refactor, not an untested-file MR.
- **Must end green.** Formatter, linter, and full suite pass before opening the request. Never weaken lint rules.
- **Project rules win.** CLAUDE.md, skills, memory, ADRs, and `LESSONS.md` override everything here.
