# Feedback routing

When the user critiques or rejects a comment sweep, the reason must land somewhere durable. The deciding question: **does
this lesson hold for comments in any codebase, or only in this project?**

## Generic lesson → LESSONS.md

It would hold anywhere: about which comments earn their place, how far a sweep should reach, or what makes a rewrite
worse than the original. Append one imperative rule with the why:

```
- **<Short rule>.** <What to do or avoid>. Why: <the reason>.
```

Every run reads `LESSONS.md` as hard constraints, so phrase it as one. No project names, file paths, language features,
or doc-tool names.

## Project-specific → the project's own context

A comment style, a doc tool the project keeps, a quirk worth always recording, a "we always do X here." Don't store it
in the skill. Tell the user where it belongs and offer to record it:

- a rule in the project's **CLAUDE.md** or contributing guide
- the project's **comment checker or linter config**
- the **comment skill's rubric**, if the project or team shares one; a fix there reaches every tool that reads it
- a **memory**, if it's a working preference

The skill reads all of these at run time, so the lesson still steers the next run.

## Always

- If the reason is ephemeral ("not now"), record nothing.
