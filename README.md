# agent-skills

A collection of Claude Code skills and rules, organized by domain.

## Layout

```
design/
  domain-modeling/              # MVC domain modeling (Rails-flavored, generally applicable)
  philosophy-of-software-design/ # Ousterhout's APoSD principles
rails/
  rspec-test-prof/              # Fast, maintainable RSpec tests with TestProf
  rules/                        # Path-scoped Rails guidance (controllers, UUIDs, PRs, callbacks)
productivity/
  babysit/                      # Watch a remote PR/MR until review comments and CI settle
  handoff/                      # Compact a conversation into a handoff doc
  singleshot-refactor-one/      # Find one refactor, implement it, open a PR/MR
  unslop/                       # Cut AI tells from any writing
```

## Skills

- **[design/domain-modeling](design/domain-modeling/SKILL.md)** — Guides domain modeling decisions. Favors rich models, vanilla conventions, and pragmatic simplicity over architectural ceremony. Written for Rails but the principles apply to any MVC framework.
- **[design/philosophy-of-software-design](design/philosophy-of-software-design/SKILL.md)** — Applies principles from John Ousterhout's *A Philosophy of Software Design* to code development, review, and refactoring.
- **[rails/rspec-test-prof](rails/rspec-test-prof/SKILL.md)** — Patterns for writing fast, clear, and maintainable RSpec tests using TestProf recipes (`let_it_be`, `before_all`, `factory_default`).
- **[productivity/babysit](productivity/babysit/SKILL.md)** — Watches a remote pull/merge request for new review comments and CI failures, fixing legitimate issues and replying until it settles. Host- and CI-agnostic.
- **[productivity/handoff](productivity/handoff/SKILL.md)** — Compacts the current conversation into a handoff document for another agent to pick up. Originally by [Matt Pocock](https://github.com/mattpocock/skills) (MIT).
- **[productivity/singleshot-refactor-one](productivity/singleshot-refactor-one/SKILL.md)** — Autonomously finds one behavior-preserving refactor, implements it, and opens a pull/merge request. Learns generic lessons from rejected requests. Originally by [Mike Whittemore](https://gitlab.com/mike-whittemore/claude/-/tree/main/skills/singleshot-refactor-one).
- **[productivity/unslop](productivity/unslop/SKILL.md)** — Edits writing to remove common AI patterns (puffery, filler, em dashes, chatbot phrases, vague jargon) and add a human voice. Originally by [Lauren Tan](https://github.com/poteto) in [cursor/plugins](https://github.com/cursor/plugins/blob/main/pstack/skills/unslop/SKILL.md) and [poteto/noodle](https://github.com/poteto/noodle) (MIT).

## Rules

The `rails/rules/` directory contains path-scoped Rails guidance documents. Each rule has a `paths:` frontmatter listing the glob patterns it applies to. Reference them from your project's `CLAUDE.md`, or load them through whatever rules mechanism your setup uses.

## Installation

Skills and rules live in subfolders, so symlink each one individually into your project:

```bash
# Clone once
git clone https://github.com/jordanbyron/agent-skills.git ~/code/agent-skills

# Symlink the skills you want into your project's .claude/skills/
cd ~/your-project
ln -s ~/code/agent-skills/design/domain-modeling              .claude/skills/domain-modeling
ln -s ~/code/agent-skills/design/philosophy-of-software-design .claude/skills/philosophy-of-software-design
ln -s ~/code/agent-skills/rails/rspec-test-prof               .claude/skills/rspec-test-prof
ln -s ~/code/agent-skills/productivity/babysit                .claude/skills/babysit
ln -s ~/code/agent-skills/productivity/handoff                .claude/skills/handoff
ln -s ~/code/agent-skills/productivity/singleshot-refactor-one .claude/skills/singleshot-refactor-one
ln -s ~/code/agent-skills/productivity/unslop                 .claude/skills/unslop

# Optionally symlink rules
ln -s ~/code/agent-skills/rails/rules .claude/rules
```

## License

This repository is licensed under the [MIT License](LICENSE).

The `productivity/handoff/` skill is derived from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, Copyright © 2026 Matt Pocock) — see [`productivity/handoff/LICENSE`](productivity/handoff/LICENSE).

The `productivity/unslop/` skill is derived from Lauren Tan's unslop skill in [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) and [poteto/noodle](https://github.com/poteto/noodle) (MIT, Copyright © 2026 Lauren Tan) — see [`productivity/unslop/LICENSE`](productivity/unslop/LICENSE).

The `productivity/singleshot-refactor-one/` skill is from Mike Whittemore's [mike-whittemore/claude](https://gitlab.com/mike-whittemore/claude) (commit `c900583`), with two extra entries in `LESSONS.md` added here.
