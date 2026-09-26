---
name: singleshot-refactor-one
description:
  Autonomously find one behavior-preserving refactoring opportunity in the current repository, implement it, and open a
  merge/pull request for review. Language-, framework-, and host-agnostic. Favors single-file refactors, ascending to
  similar and related files only when no single-file opportunity remains. Learns generic refactoring lessons from
  rejection feedback. Use when the user wants to auto-refactor, asks for a refactoring MR/PR, says "find a refactor", or
  gives feedback on a previous refactor MR/PR.
---

# Refactor One

A generic refactoring engine. Each run produces one behavior-preserving MR/PR. Each rejection that carries a _generic_
lesson sharpens the skill; project-specific feedback is sent back to the project's own context.

Two modes, picked from the invocation:

- **Run mode** — no args, or `run`. Scan, pick one refactor, implement, open an MR/PR.
- **Feedback mode** — args start with `feedback`, or reference an MR/PR. Fold a generic lesson back into the skill, or
  redirect a project-specific one to where it belongs.

## Detect the environment (every run)

Nothing about the toolchain is hard-coded. Detect from the repository:

- **Verify commands** — formatter, linter, test runner. Infer from config files and CI; **prefer whatever the project's
  own CLAUDE.md, skills, or memory specify**. Run them exactly as the project expects.
- **VCS host** — GitHub (`gh`, "PR") vs GitLab (`glab`, "MR") vs other, from `git remote`.
- **Project rules** — CLAUDE.md, project skills, recalled memory, and any ADR/decision docs are authoritative and
  override the skill's defaults. Read them before proposing anything.

## Run mode

1. **Load constraints.** Read [CRITERIA.md](CRITERIA.md), [LESSONS.md](LESSONS.md), and the project rules above.
   `LESSONS.md` entries are hard constraints — never propose what they forbid.
2. **Scan single-file first.** Use the `Explore` subagent to survey the repo for **tier-1** (within-one-file) candidates
   satisfying the value catalog in [CRITERIA.md](CRITERIA.md). Ask the scan to hunt for dependency-severing candidates
   by name — unused imports, pass-through wrappers, a collaborator only one call site still justifies — since those rank
   highest and are easy to miss when looking only for duplication. Only if the whole repo yields no worthwhile
   single-file candidate, ascend to tier 2 (similar files), then tier 3 (related files). This makes "files become solid,
   then branch out" emerge with no stored state — solid simply means no candidate found this run.
3. **Score and pick one.** Rank by `value × confidence ÷ risk` ([CRITERIA.md](CRITERIA.md)). Take the single best. If
   nothing qualifies at any tier, stop and say so — produce no MR/PR.
4. **Implement.** Call `EnterWorktree` first (this edits repo files). Make the one behavior- preserving change. If it
   can't be done without changing behavior or breaking a guardrail, abandon it and take the next candidate.
5. **Verify.** Run the detected formatter, linter, and full test suite. Drive them green without changing behavior. If
   green is unreachable cleanly, abandon and pick the next candidate.
6. **Open the request.** Commit per the project's git conventions. Push the branch and open the MR/PR with the host CLI
   detected above. Add the labels `ai-generated` and `refactor` (GitHub: `gh pr create --label`; GitLab:
   `glab mr create --label`). If a label doesn't exist and you can't create it, drop just that label and proceed — never
   let labeling block the request. Report the URL and a one-line headline.

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

1. **Find the request.** The MR/PR ref in the args, else the most recent refactor MR/PR from the host, else ask the user
   which one.
2. **Get the reason.** Use the feedback in the args; if absent, ask one question.
3. **Classify and route** (see [FEEDBACK.md](FEEDBACK.md)):
   - **Generic refactoring lesson** (applies to any codebase) → add a rule to [LESSONS.md](LESSONS.md).
   - **Project / framework / language specific** → do **not** store it here. Tell the user to record it in the project's
     own context (CLAUDE.md, a project skill, or a memory), and offer to do it.
4. Confirm what changed and where it landed.

See [CRITERIA.md](CRITERIA.md) for the rubric and tiers, [FEEDBACK.md](FEEDBACK.md) for routing.
