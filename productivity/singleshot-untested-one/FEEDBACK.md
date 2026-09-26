# Feedback routing

When the user critiques or rejects an untested-file MR/PR — or an assessment verdict — the reason must land somewhere
durable. The deciding question: **is this lesson generic to testing judgment, or specific to this project's
language/framework/conventions?**

## Generic testing-judgment lesson → LESSONS.md

The lesson would hold in _any_ codebase — about what deserves a test, what legitimately doesn't, or what makes a test
good regardless of stack. Examples: don't write a test that only restates a constant; a barrel/re-export file isn't
worth a test; don't assert call order to "cover" a delegation; prefer one behavioral test over many that pin internals.

Append to [LESSONS.md](LESSONS.md) one imperative rule with the why:

```
- **<Short rule>** — <what to do / avoid>. Why: <the reason>.
```

Phrase it as a constraint — every run reads `LESSONS.md` as hard constraints. Keep it general enough to fire on the next
similar candidate, specific enough to act on. No project names, file paths, language features, or framework APIs.

## Project / framework / language specific → the project's own context

The lesson is about _this_ codebase — its test runner, naming/granularity convention, a "we don't unit-test X here"
decision, a framework idiom, a domain term. This must **not** go in the skill. Tell the user it belongs in the project's
own context and offer to record it:

- a rule or note in the project's **CLAUDE.md**
- a dedicated **project skill** for the testing convention
- a **memory** (if it's a working preference)
- a **decision record / ADR** (if a file is intentionally left untested and a future run would otherwise re-suggest it)

The skill reads all of these at run time, so a project-specific lesson recorded there still steers future runs — it just
lives where it belongs instead of polluting the generic engine.

## Always

- A rejected candidate isn't "solved" — it simply won't be re-picked the same way once the lesson (generic or
  project-side) is in place. If a file is deliberately untestable-by-design, record that as a skip reason in the
  project's context so it stops surfacing.
- If the reason is ephemeral ("not now"), record nothing.
