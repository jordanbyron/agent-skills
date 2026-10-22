# Lessons

Generic comment-sweep lessons learned from rejected requests. Stack-agnostic only: no project names, file paths,
language features, or doc-tool names (those belong in the project's own context). Every run reads these as hard
constraints, except that an entry about which comments stay gives way to the project's or user's comment rules. One
imperative rule per entry, with the why.

- **Leave section-divider comments inside a class alone.** A class that needs headings is two concepts sharing a name,
  and the fix is a split, not a deletion. Why: a reviewer called such dividers "a smell: if we need this it means we
  want an actual domain concept to encapsulate this." Deleting them hides the smell from the refactor that would fix it.
- **Leave a comment that apologizes for the code it sits on** ("only here for the tests", "workaround until X"). It
  points at code that should change or go. Why: one such comment led to deleting two test-only readers outright. Removing
  the comment would have lost that pointer and kept the dead code.
- **Don't mistake repetition for a convention.** A phrase or comment style that recurs across the codebase is a pattern,
  not a rule; only the project's stated rules make it one. Apply the test to it like any other comment. Why: a run kept a
  stock phrase repeated in several files on the guess that it was a convention, and it was noise. It also kept a
  widespread informal doc style the project's conventions don't call for.
- **Survey every comment form, and pick by weight, not count.** Rank candidate files by how many lines all their
  comments take, doc comments included, as a share of the file. Why: a run counted only line comments, missed the
  doc-comment-heavy files where most of the bloat lived, and shipped one request when the user expected three.
- **Sweep the chosen file fully, up to the hunk cap.** Docs that retell another doc comment, walk through a dependency's
  internals, or restate the code fail the test as surely as a stale tag does. Why: a run took only the easy one-liners
  and left a module doc that repeated its class doc nearly line for line.
