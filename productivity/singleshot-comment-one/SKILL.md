---
name: singleshot-comment-one
description:
  Autonomously trim, rewrite, or delete the comments in one file of the current repository and open a merge/pull
  request for review. Use when the user wants a comment cleanup MR/PR, says "sweep comments" or "trim comments", or
  gives feedback on a previous comment-sweep MR/PR.
---

# Comment One

Each run opens one small MR/PR that leaves one file's comments shorter and truer. It runs unattended, so the request has
to be something a reviewer can read and merge in a minute. The project decides what a good comment is. This skill
decides how much one run touches.

Two modes, picked from the invocation:

- **Run mode**: no args, or `run`. Pick one file, sweep its comments, open an MR/PR.
- **Feedback mode**: args start with `feedback`, or reference an MR/PR. Fold a generic lesson back into the skill, or
  redirect a project-specific one to where it belongs.

## Detect the environment (every run)

- **Verify commands.** Formatter, linter, test runner, and CI script, as the project's CLAUDE.md, skills, or memory
  specify, else inferred from config.
- **VCS host.** GitHub (`gh`, "PR") or GitLab (`glab`, "MR"), from `git remote`.
- **Comment rules.** CLAUDE.md, contributing docs, linter config, and any comment hook in the project's or user's Claude
  settings (a `PostToolUse` comment checker, for example). These rules decide what a comment here may say, and they
  override this skill. A checker's findings are candidates, not verdicts: one that flags every comment can't tell a good
  comment from a bad one, and a lower count afterwards proves nothing.
- **Writing skill.** If a humanize, unslop, or writing-style skill is available, load it. Every comment you rewrite goes
  through it.

## The test

One question per comment: does it say something the code can't? (Ousterhout, _A Philosophy of Software Design_.) A
comment that repeats the code goes. A comment that records why it's that way (a decision, a constraint, a quirk of
something outside the code) stays, cut down to what that takes. Doc comments in the style the project's conventions
call for are the interface; tighten them, don't strip them. A comment that retells a project doc can shrink to a pointer
to it, once you've checked the doc carries the detail.

## Depth

- **One file, about half a dozen hunks.** If it has more failing comments than that, fix the clearest ones and leave the
  rest for the next run.
- **Comments only.** The diff changes no code. When a comment could only go after a rename or an extraction, leave it,
  because that work belongs to a refactor.
- **Subtract, don't add.** Delete, shorten, or correct. Don't write new comments.
- **Nothing worth doing means no request.** Say so and stop.

## Run mode

1. **Load constraints.** Read [LESSONS.md](LESSONS.md) and the project rules above. `LESSONS.md` entries are hard
   constraints.
2. **Pick one file.** Survey the repo (an `Explore` subagent works) for the file whose comments fail the test most
   clearly. Dense comment blocks, class comments that say how instead of what for, and comments that retell a project
   doc are good places to look. Skip files that an open MR/PR touches, so the request can't conflict. Asked for more
   than one request, pick that many different files and give each its own branch off the fresh default branch.
3. **Sweep it.** Fetch the default branch and branch from it in a worktree (`EnterWorktree` when the session is in this
   repo). Apply the test, the writing skill, and the project's rules.
4. **Verify.** Run the detected formatter, linter, tests, and comment checker. Read the diff back: every changed line is a
   comment. If either check fails, fix it or abandon the run.
5. **Open the request.** Commit per the project's conventions. Push and open the MR/PR with the labels `ai-generated` and
   `documentation`; drop a label that doesn't exist and can't be created. If the project gates readiness on a local CI
   script, run it on the pushed HEAD. Report the URL and a one-line headline.

## Feedback mode

1. **Find the request.** The MR/PR ref in the args, else the most recent comment-sweep MR/PR from the host, else ask.
2. **Get the reason.** Use the feedback in the args; if absent, ask one question.
3. **Classify and route** per [FEEDBACK.md](FEEDBACK.md): a generic lesson goes in [LESSONS.md](LESSONS.md), a
   project-specific one goes to the project's own context.
4. Confirm what changed and where it landed.
