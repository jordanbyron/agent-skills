# agent-skills

A collection of Claude Code skills for Rails development.

## Skills

- **[domain-modeling](domain-modeling/SKILL.md)** — Guides domain modeling decisions in Rails apps. Favors rich models, vanilla Rails conventions, and pragmatic simplicity over architectural ceremony.
- **[philosophy-of-software-design](philosophy-of-software-design/SKILL.md)** — Applies principles from John Ousterhout's *A Philosophy of Software Design* to code development, review, and refactoring.
- **[rspec-test-prof](rspec-test-prof/SKILL.md)** — Patterns for writing fast, clear, and maintainable RSpec tests using TestProf recipes (`let_it_be`, `before_all`, `factory_default`).

## Installation

Clone into your project's `.claude/skills/` directory, or symlink individual skills:

```bash
git clone https://github.com/jordanbyron/agent-skills.git ~/code/agent-skills
ln -s ~/code/agent-skills/domain-modeling .claude/skills/domain-modeling
```

## Usage

Once installed, Claude Code will auto-discover the skills and invoke them when relevant based on each skill's `description` frontmatter.
