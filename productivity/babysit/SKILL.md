---
name: babysit
description: Watch a remote pull/merge request for new review comments and CI failures, and keep resolving them until it settles.
---

Poll the pull or merge request for new review comments and CI or build results. On each pass, fix genuine, actionable review comments: edit the code, commit, push, and reply with what changed. For a failing check, pull its logs, diagnose the cause, fix it, and push. Skip comments that are questions, nits you disagree with, or already answered. Never resolve review threads or approve on the human reviewer's behalf. Keep looping until a full pass turns up nothing new: no unanswered comments, no failing checks. Then report it as settled and stop.
