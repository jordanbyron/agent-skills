---
name: singleshot-untested-one
description:
  Scan the whole repository for source files that have no corresponding tests, assess which ones genuinely warrant a
  test (and which legitimately should not), write tests for the single best candidate, and open a merge/pull request.
  Language-, framework-, and host-agnostic. Learns generic testing-judgment lessons from rejection feedback. Use when
  the user wants to find untested files, write missing tests, ask "what needs tests", auto-test a file, or gives
  feedback on a previous untested-file MR/PR.
---

# Untested One

A generic untested-file engine. Each run does two things: produce a **repo-wide assessment** of every untested source
file (needs-test vs. skip, with reasoning), and turn the single highest-value gap into **one reviewable MR/PR**.
Re-running picks up the next gap. Each rejection that carries a _generic_ lesson sharpens the skill; project-specific
feedback is routed back to the project's own context.

Two modes, picked from the invocation:

- **Run mode** — no args, or `run`, or a path to narrow the scan. Assess gaps, pick one, write its tests, open an MR/PR.
- **Feedback mode** — args start with `feedback`, or reference an MR/PR. Fold a generic lesson back into the skill, or
  redirect a project-specific one to where it belongs.

## Detect the environment (every run)

Nothing about the toolchain is hard-coded. Detect from the repository:

- **Test convention** — the runner, the file-naming pattern, and where tests live (sibling `foo_test.x`, `__tests__/`,
  `tests/`, inline test modules, one-test-file-per-class vs. per-module). Infer from existing tests and config; **prefer
  whatever the project's own CLAUDE.md, skills, or memory specify**. See [ASSESSMENT.md](ASSESSMENT.md).
- **Verify commands** — formatter, linter, test runner. Run them exactly as the project expects.
- **VCS host** — GitHub (`gh`, "PR") vs GitLab (`glab`, "MR") vs other, from `git remote`.
- **Project rules** — CLAUDE.md, project skills, recalled memory, and any ADR/decision docs are authoritative and
  override the skill's defaults. Read them before proposing anything.

## Run mode

1. **Load constraints.** Read [ASSESSMENT.md](ASSESSMENT.md), [LESSONS.md](LESSONS.md), and the project rules above.
   `LESSONS.md` entries are hard constraints.
2. **Map coverage.** Use the `Explore` subagent to list source files and match each against the detected test
   convention. A file is _covered_ if a corresponding test exists **or** its behavior is exercised by a higher-level
   test elsewhere — not only by a 1:1 filename match. Output the set of untested files.
3. **Assess each gap.** For every untested file, decide **needs-test** or **skip**, with a one-line reason, per the
   rubric in [ASSESSMENT.md](ASSESSMENT.md). Present this as the repo-wide report — this is a deliverable on its own,
   even before any MR.
4. **Score and pick one.** Rank the needs-test files by `value × confidence ÷ risk` ([ASSESSMENT.md](ASSESSMENT.md)).
   Take the single best. If nothing qualifies (everything is legitimately skip), report that and produce no MR/PR.
5. **Write the tests.** Call `EnterWorktree` first (this edits repo files). Author tests that follow the project's
   existing convention and test **behavior through the public API**, not implementation details. If the file can't be
   tested without first changing its design, note that in the report and take the next candidate.
6. **Verify.** Run the detected formatter, linter, and full test suite. The new tests must pass and the suite must stay
   green. If green is unreachable cleanly, abandon and pick the next candidate.
7. **Open the request.** Commit per the project's git conventions. Push the branch and open the MR/PR with the host CLI
   detected above. Add the label `ai-generated` (GitHub: `gh pr create --label`; GitLab: `glab mr create --label`). If
   the label doesn't exist and you can't create it, drop it and proceed — never let labeling block the request. Report
   the URL, the full assessment, and a one-line headline.

## Checking whether the request merged

If a run waits for its MR/PR to merge before continuing, ask the **host** for the request's state — never infer merge
from git history. A squash or rebase merge rewrites your commit to a new SHA and may add a separate merge commit, so
`git merge-base --is-ancestor <your-sha> main` returns false forever even after the request is merged. The authoritative
signal is the host:

- **GitHub:** `gh pr view <num> --json state,mergedAt` → merged when `state` is `MERGED`.
- **GitLab:** `glab mr view <num>` (or `glab api projects/:id/merge_requests/<num>`) → merged when `state` is `merged`.

Treat the host state as truth. Source-branch deletion and the commit message appearing on the main branch are weaker
hints, not proof — a branch can be deleted without merging, or kept after one.

## Feedback mode

1. **Find the request.** The MR/PR ref in the args, else the most recent untested-file MR/PR from the host, else ask
   which.
2. **Get the reason.** Use the feedback in the args; if absent, ask one question.
3. **Classify and route** (see [FEEDBACK.md](FEEDBACK.md)):
   - **Generic testing-judgment lesson** (applies to any codebase) → add a rule to [LESSONS.md](LESSONS.md).
   - **Project / framework / language specific** → do **not** store it here. Tell the user it belongs in the project's
     own context (CLAUDE.md, a project skill, or a memory), and offer to record it.
4. Confirm what changed and where it landed.

See [ASSESSMENT.md](ASSESSMENT.md) for the skip/needs-test rubric and scoring, [FEEDBACK.md](FEEDBACK.md) for routing.
