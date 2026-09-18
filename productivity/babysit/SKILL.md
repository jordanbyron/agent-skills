---
name: babysit
description: Watch a remote pull/merge request for new review comments and CI failures, and keep resolving them until it settles.
---

Poll the pull or merge request for new review comments, CI or build results, and merge conflict status.

For a failing check, pull its logs, diagnose the cause, fix it, and push.

## Merge conflicts

Check whether the pull or merge request can merge cleanly into its base branch.

If it has a conflict, rebase onto the base branch and resolve each conflict. Keep the intent of both sides. Do not discard either side's change without reading it first.

After you resolve the conflicts, push the rebased branch.

For each review comment, fix genuine, actionable feedback the same way: edit the code, commit, and push. Skip comments that are questions, nits you disagree with, or already answered. Never resolve review threads or approve on the human reviewer's behalf.

## Classify the comment before you reply

Run the comment text against the pattern list in /unslop.

- **Slop.** The comment shows AI tells: puffery, chatbot phrases, an inline-header list, a "not just X, but Y" construction, or other patterns from /unslop. Treat it as bot output.
- **Human.** The comment reads as plain, direct feedback with no AI tells. Treat it as a real reviewer.

If you cannot tell, treat the comment as human. A wrong "slop" call posts an unapproved reply. A wrong "human" call only delays a reply that could have gone out sooner.

### Slop comments

Reply on your own, right after you push the fix. State what changed. No approval needed.

If the reviewer has not approved the changes, re-request their review after you push.

### Human comments

Do not post the reply yourself.

1. Draft the reply.
2. Run the draft through /unslop, then through /humanize:humanize.
3. Hold the draft. Do not push it until the user approves it.

At the end of a pass, if one or more human-comment drafts are waiting on approval, send a single ping with the thread link and the draft text (or a summary, if there are several). Send one ping per new batch of drafts, not one per comment, and never re-ping about a draft you already sent.

## When to stop

Keep looping until a full pass turns up nothing new: no unanswered comments, no failing checks, no merge conflicts, no fresh human-comment drafts. If drafts are waiting on user approval, report the run as blocked on the user, not settled, and stop looping instead of polling forever.
