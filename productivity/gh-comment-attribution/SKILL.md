---
name: gh-comment-attribution
description: Use when you draft, post, or edit any GitHub comment on the user's behalf. This covers issue comments, pull request comments, inline review comments, and review summaries. It applies whether the comment goes out through gh, the GitHub API, an MCP tool, or as text the user will paste. Prefixes the comment with a GitHub note callout that names the agent model. Trigger any time the text will become a GitHub comment, even if the user did not mention the callout.
---

# GitHub comment attribution

## Why this exists

A comment posted from the user's account reads as the user. When an agent wrote it, the reader deserves to know. This skill puts a fixed note callout at the top of every agent-written GitHub comment so the reader can tell at a glance who wrote it.

## The callout

GitHub renders the `[!NOTE]` alert syntax as a highlighted callout. Use it exactly like this, with your full model name in place of `Fable 5.1` and the user's first name in place of `Sam`:

```markdown
> [!NOTE]
> **Fable 5.1 responding on Sam's behalf**

<the comment body>
```

Rules for the callout:

- Put it on the first line of the comment. Nothing goes above it.
- Leave one blank line between the callout and the body.
- Bold the entire callout line, including the model name.
- Do not add a period, emoji, or extra words to the callout line.
- Do not restyle the callout when you run a writing-cleanup skill such as `unslop` on the body. The callout is fixed text.

## Which name to use for the user

Use the first name of the person whose GitHub account posts the comment. Take it from `gh api user --jq .name`. If that is empty, use their GitHub login from `gh api user --jq .login`. Do not guess a name.

## Which model name to use

Use the full model name from your own system prompt, including the version number.

| System prompt says | Write |
| --- | --- |
| Fable 5.1 | Fable 5.1 |
| Opus 5 | Opus 5 |
| Sonnet 5 | Sonnet 5 |
| GPT-5 Codex | GPT-5 Codex |

If you cannot determine the model from your context, write the agent product name instead, such as `Claude Code`. Do not guess a model name.

## Where it applies

Add the callout to every comment you author, no matter how it is sent:

- `gh pr comment`, `gh issue comment`
- `gh pr review --comment`, `--approve`, or `--request-changes` with a body
- `gh api` calls that create or update a comment or review
- GitHub MCP tools that create or update a comment or review
- Comment text you hand to the user to paste into GitHub

Do not add the callout to:

- Pull request descriptions. The user is the PR author.
- Commit messages.
- Comments the user wrote and asked you to edit for wording. The comment stays theirs.

## Editing an existing comment

When you edit a comment that already starts with the callout, keep one copy. Never stack a second callout. When you edit an agent comment that has no callout, add it.

## Approval still applies

The callout does not make a comment safe to post unreviewed. Show the user the full comment, callout included, and wait for approval before you post it.
