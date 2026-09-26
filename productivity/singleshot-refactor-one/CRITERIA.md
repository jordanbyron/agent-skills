# Criteria

What counts as a refactor worth a request, how to rank candidates, and the order to work in. Generic across languages,
frameworks, and projects.

## Guardrails (hard — a candidate failing any of these is disqualified)

- **Behavior-preserving only.** Refactoring changes structure, never observable behavior. No new features, no bug fixes
  smuggled in, no semantic changes. If behavior should change, it's out of scope.
- **One refactor per request.** A single coherent change with a small, reviewable diff. A second tempting refactor is
  the next run, not this one.
- **Project rules win.** The project's CLAUDE.md, skills, memory, and decision records override the skill. A rule (incl.
  anything in `LESSONS.md`) forbidding a pattern is absolute.
- **Tests are the safety net.** Prefer test-covered areas. In untested code, only do changes the compiler/linter fully
  verifies (rename, extract-with-identical-call, dead-code deletion) — nothing whose correctness rests on judgment
  alone.
- **Must end green.** The project's formatter, linter, and full test suite pass before opening the request. Never
  satisfy the linter by raising limits or disabling rules.
- **Follow the surrounding idiom.** Match the conventions already present in the code being changed.

## Value catalog (what to look for)

A candidate must deliver at least one, ideally several. **Eliminating a dependency is the most valuable kind of refactor
there is** — prefer it over every other kind whenever the confidence and risk are comparable.

- **Minimize dependencies (highest value)** — sever a coupling between two units, drop unused imports, narrow what a
  module reaches for, remove a pass-through layer. The strongest form deletes a collaborator outright: a constructor
  parameter, injected field, or import that no longer has to exist. Hunt for these first and lead with them.
- **DRY** — _true_ duplication (same reason to change) collapsible into one definition. Not incidental similarity.
- **Consistency** — bring an outlier in line with the established pattern: ad-hoc construction → the project's shared
  helper/factory, divergent naming → the convention its siblings use, reinvented behavior → the existing abstraction.
- **Simplicity** — delete dead code, flatten needless nesting, replace a custom implementation with a
  standard-library/framework one, inline a single-use indirection, remove a guard for a state that cannot occur.

### Chase a removed indirection through to the dependency

A pass-through wrapper is worth removing on its own, but its real payoff is usually one step further out. After deleting
one, check every caller for a collaborator that is now unused — the wrapper was often the _only_ reason that
collaborator was held. Follow it through to the field and the constructor parameter and delete those too, in the same
request. That follow-through is what turns a tidy-up into a severed dependency, so it belongs in the same change, not a
later one.

## Scoring — pick exactly one

Rank candidates by `value × confidence ÷ risk`:

- **value** — friction removed. A severed dependency scores highest, then code deleted, then an inconsistency erased.
  Deleting code beats adding it.
- **confidence** — how sure the change is behavior-preserving and correct. Mechanical, compiler- verifiable changes are
  high; judgment-heavy restructures are lower.
- **risk** — blast radius × how untested the area is. Many call sites in untested code is high risk even when
  mechanical.

Favor high-confidence, low-risk, single-file changes. On a tie, take the smaller diff.

A candidate that severs a dependency outranks a purely cosmetic one even when its diff is larger and spans more files —
touching more call sites to delete a collaborator is worth it. Weigh that against risk as usual, but don't drop such a
candidate merely for being the bigger change.

## Tiers (single-file first, ascend only when exhausted)

Evaluated fresh each run — no stored state. "Exhausted" means _this scan found no worthwhile candidate_, which is
exactly when files have become solid enough to branch out.

1. **Within one file.** All of the above, contained to a single file.
2. **Across similar files.** Siblings in the same directory or sharing a naming pattern. Extract a shared helper/base,
   unify a pattern that drifted across the set.
3. **Across related files.** Files connected by use — caller/callee, importer/importee. Move a responsibility to where
   it belongs, sever or invert a dependency, consolidate a leaked seam.

Ascend to the next tier only when the current tier yields nothing across the whole repo.
