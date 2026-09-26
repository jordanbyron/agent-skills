# Feedback routing

When the user critiques or rejects a refactor request, the reason must land somewhere durable. The deciding question:
**is this lesson generic to refactoring, or specific to this project's language/framework/conventions?**

## Generic refactoring lesson → LESSONS.md

The lesson would hold in _any_ codebase — about taste, abstraction, or what makes a refactor good regardless of stack.
Examples: don't extract a helper used only once; don't collapse two blocks that change for different reasons; prefer
deleting code over parameterizing it.

Append to [LESSONS.md](LESSONS.md) one imperative rule with the why:

```
- **<Short rule>** — <what to do / avoid>. Why: <the reason>.
```

Phrase it as a constraint — every run reads `LESSONS.md` as hard constraints. Keep it general enough to fire on the next
similar candidate, specific enough to act on. No project names, file paths, language features, or framework APIs.

## Project / framework / language specific → the project's own context

The lesson is about _this_ codebase — a deliberate design decision, a framework idiom, a language-specific pattern, a
domain term, a "we always do X here." This must **not** go in the skill. Tell the user it belongs in the project's own
context and offer to record it for them:

- a rule or note in the project's **CLAUDE.md**
- a dedicated **project skill** for that convention
- a **memory** (if it's a working preference)
- a **decision record / ADR** (if the rejected target is intentional and a future run would otherwise re-suggest it)

The skill reads all of these at run time, so a project-specific lesson recorded there still steers future runs — it just
lives where it belongs instead of polluting the generic engine.

## Always

- A rejected target is not "solved" — it simply won't be re-picked the same way once the lesson (generic or
  project-side) is in place.
- If the reason is ephemeral ("not worth it right now"), record nothing.
