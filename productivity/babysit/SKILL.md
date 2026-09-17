---
name: babysit
description: Watch a remote pull/merge request for new review comments and CI failures, and keep resolving them until it settles.
---

Poll the pull or merge request for new review comments and CI/build results, using whatever host and CI tooling the project already uses — work out which from the git remote and the CLIs or credentials available, rather than assuming a specific provider. On each pass: for a genuine, actionable review comment, fix the code, commit, push, and reply explaining the change; for a failing check, pull its logs, diagnose the cause, fix it, and push. Skip comments that are questions, nits you disagree with, or already answered, and never resolve review threads or approve on the human reviewer's behalf. Keep looping until a full pass turns up nothing new — no unanswered comments, no failing checks — then report it as settled and stop.
