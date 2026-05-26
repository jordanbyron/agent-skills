# Pull Request Size

Keep PRs small enough to review in one sitting. Target under 300 lines of
diff; treat 500 as a hard ceiling. A 1000+ line PR is a process failure,
not a feature.

## How to size work

- **Plan before branching.** When executing a multi-task plan, each logical
  slice should land as its own PR before the next slice starts. Don't
  accumulate an entire plan on one branch.
- **Natural slices to look for:** model/domain layer first, controller
  layer second, UI/presets third. Security fixes, CI hygiene, and style
  changes belong in separate PRs from feature work.
- **Test changes count.** Spec code is part of the diff. A "small" PR that
  adds 400 lines of feature code plus 600 lines of tests is not small.

## When a PR starts growing

If you're midway through a plan and the branch is approaching the ceiling,
stop, open a PR for what's already done, and start a new branch from that
merge for the next slice. Don't "finish the plan first and split later" —
splitting a merged branch is painful and loses review history.

## Exceptions

Mechanical refactors (rename, codemod, formatting sweeps) can be larger
because reviewers scan them differently. Call out in the PR description
when a change is mechanical so reviewers know what kind of attention it
needs.
