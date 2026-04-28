# claude-spec

A Claude Code skill that turns a plan discussion into a full spec — ready to hand off to an AI agent for implementation.

## What it does

After you discuss a feature with Claude, run `/spec feature-name` and it writes five files:

- `plans/feature-name/product.md` — the why: problem, goal, reasoning, scope
- `plans/feature-name/requirements.md` — user stories, acceptance criteria, dependencies
- `plans/feature-name/design.md` — file changes with line numbers, reuse audit, edge cases
- `plans/feature-name/tasks.md` — JSON task list with tests and verification commands
- `plans/feature-name/kickoff.md` — a ready-to-paste implementation prompt

Then it commits the spec. When you run the kickoff prompt, the agent implements, tests, archives the spec to `plans/completed/`, and commits.

## Install

Copy the skill file into your project:

```
your-project/
  .claude/
    skills/
      spec/
        SKILL.md   ← this file
```

That's it. Claude Code picks it up automatically.

## Usage

1. Discuss a feature with Claude until the approach is clear
2. Run `/spec your-feature-name`
3. Claude writes the five spec files and commits them
4. Paste `plans/your-feature-name/kickoff.md` into a new Claude session to implement

## Requirements

- [Claude Code](https://claude.ai/code)
- A git repo (the skill commits the spec files)
