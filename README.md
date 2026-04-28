# claude-spec

A Claude Code skill that turns a plan discussion into a full spec, ready to hand off to an AI agent for implementation.

If you've used spec-driven development workflows before, this is that: a portable, self-contained Claude Code skill you can drop into any repo.

## What it does

After you discuss a feature with Claude, run `/spec feature-name` and it writes five files:

- `plans/feature-name/product.md`: the why, problem, goal, reasoning, scope
- `plans/feature-name/requirements.md`: user stories, acceptance criteria, dependencies
- `plans/feature-name/design.md`: file changes with line numbers, reuse audit, edge cases
- `plans/feature-name/tasks.md`: JSON task list with tests and verification commands
- `plans/feature-name/kickoff.md`: a ready-to-paste implementation prompt

Then it commits the spec. When you run the kickoff prompt, the agent implements, tests, archives the spec to `plans/completed/`, and commits.

## Example output

Here's what a `kickoff.md` looks like after running `/spec rate-limiting`:

```
Add per-user rate limiting to the API so abusive clients can't degrade service
for everyone else.

Read these files before starting:
- plans/rate-limiting/product.md
- plans/rate-limiting/requirements.md
- plans/rate-limiting/design.md
- plans/rate-limiting/tasks.md

Then implement each task in order. Rules:
- Complete tasks in the order listed in tasks.md
- Do not change any file not listed in the tasks
- After each task, run the task's verification command and confirm the expected outcome
- After all tasks, run the full relevant test suite and confirm it passes
- After the full test suite passes, move the entire plans/rate-limiting/ directory
  into plans/completed/rate-limiting/
- Commit all changes with a descriptive message reflecting what was implemented.
  This is the final step.
```

Clean enough to paste directly. The five spec files it references contain all the detail: file paths, line numbers, edge cases, and test scaffolds.

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

1. Use the **Plan agent** (`/plan` in Claude Code) to think through the approach first. It's better at architectural reasoning and will surface trade-offs before you commit to a design.
2. Once the plan looks right, run `/spec your-feature-name` in the same conversation.
3. Claude writes the five spec files and commits them.
4. Paste `plans/your-feature-name/kickoff.md` into a new Claude session to implement.

## Requirements

- [Claude Code](https://claude.ai/code)
- A git repo (the skill commits the spec files)
