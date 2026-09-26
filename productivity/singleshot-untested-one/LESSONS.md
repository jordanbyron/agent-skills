# Lessons

Generic testing-judgment lessons learned from rejected untested-file MRs/PRs and assessment verdicts. Stack-agnostic
only — no project names, file paths, language features, or framework APIs (those belong in the project's own context).
Every run reads these as hard constraints. One imperative rule per line, with the why.

<!-- Example:
- **Don't test a file that only declares constants** — there's no behavior to break.
  Why: the "test" just restated the value and broke on every legitimate edit.
-->
