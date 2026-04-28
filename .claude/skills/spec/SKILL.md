---
name: spec
description: Formalize the current plan discussion into spec files (product, requirements, design, tasks, kickoff prompt)
---

Take the plan we just discussed and formalize it into spec files. The goal is a spec precise enough that implementation produces clean, compilable code on the first pass — so err on the side of concrete file paths, line numbers, and verification commands over prose.

1. Create the directory `plans/$ARGUMENTS/` (use a short kebab-case name if no argument given, derived from the feature name).

2. Write `plans/$ARGUMENTS/product.md` containing:
   - **Problem Statement** — what's broken or missing today, in concrete terms. What triggered this work.
   - **Goal** — what this change achieves; the desired end state. One or two sentences.
   - **Success Criteria** — how you know it worked (measurable where possible: "tests pass", "phase runs in under N seconds", "no regressions in X").
   - **Product Reasoning** — why this approach over alternatives; what trade-offs were considered in the plan discussion. Capture the reasoning that would be lost when the conversation scrolls away.
   - **Scope Boundary** — what this feature deliberately does NOT try to solve. Feeds into requirements "Out of Scope".

   This should be short — a half-page document, not a manifesto. It captures the *reasoning* from the plan discussion so that requirements, design, and tasks all trace back to a clear "why."

3. Write `plans/$ARGUMENTS/requirements.md` containing:
   - **Context / Problem** — one short paragraph consistent with product.md's Problem Statement and Goal. Convert any relative dates from the plan discussion into absolute dates.
   - Overview of the feature
   - User stories (As a... I want... So that...) — the "so that" clauses should connect to the Goal in product.md
   - Acceptance criteria (Given/When/Then)
   - Out of scope (consistent with product.md's Scope Boundary)
   - Dependencies (files, modules, external systems involved)

4. Write `plans/$ARGUMENTS/design.md` containing:
   - Architecture overview (where does this fit, what's the approach)
   - Data model changes (new classes, fields, schema changes — or "none")
   - **File Changes** — list every file that will be created or modified. For each modified file, include the function name or line range where the edit goes (e.g. `planner.py::load_backlog (~line 61)`). Skip line numbers only when the change is a rename/global refactor that touches the whole file. If any change alters user-facing behavior, identify the relevant documentation file and list it here alongside the code files. If no user-facing behavior changes, note "No docs update needed" explicitly.
   - **Reuse / Existing Code Audit** — list at least 3 existing functions, classes, utilities, or patterns in the codebase that this change will reuse, with file path and symbol. If nothing applies, state "No applicable reuse" and explain in one sentence. Do not fabricate reuse — verify each entry exists before listing it.
   - **Technical Detail** — data flow, prompt structures, algorithms, signal wiring
   - **Edge Cases** — at least 3 non-happy-path scenarios and how the design handles each (e.g. "spec file exists but has no criteria", "file write fails mid-task", "retry on already-partial state")

5. Write `plans/$ARGUMENTS/tasks.md` containing a JSON array of implementation tasks, ordered by dependency. Each task must have:
   - `title` — short descriptor
   - `description` — detailed enough to stand alone as a kickoff prompt; reference exact functions and line numbers from design.md
   - `files` — array of file paths touched
   - `tests` — array of test file paths to create or update. Each entry must include a short descriptor of the new assertions being added, in the form `"tests/test_x.py — covers criterion N: <short phrase>"`. Use `["none"]` only for doc-only or trivial tasks; a task that changes observable behavior must list at least one test. If a behavior is genuinely untestable (e.g. visual output requiring manual verification), write `["none: <reason>"]` so the justification is co-located with the task.
   - `complexity` — S / M / L
   - `is_visual` — bool
   - `verification` — a concrete command and expected outcome (e.g. `"pytest tests/test_models.py -v — all tests pass"`). Manual checks are allowed when automation doesn't fit ("Manual: run X, confirm Y in output").

   Tasks should be independently implementable and independently verifiable. Large tasks should be broken into sequential S/M parts.

   When documentation files appear in the File Changes list: fold them into the relevant task's `files` array (preferred — keeps the doc update co-located with the code it documents). If a single doc update spans multiple tasks, add a final dedicated documentation task instead: `files: ["docs/<file>.md"]`, `tests: ["none"]`, `complexity: S`, `is_visual: false`, `verification: "Manual: read docs/<file>.md, confirm <behavior> is accurately documented"`. Do not add a documentation task if no docs files appear in File Changes.

6. Write `plans/$ARGUMENTS/kickoff.md` — a ready-to-paste implementation prompt. Strict template:
   - One sentence stating what is being built and why (the why comes from product.md).
   - `Read these files before starting:` followed by all 4 spec file paths (product.md, requirements.md, design.md, tasks.md).
   - `Then implement each task in order. Rules:` followed by exactly these 6 rules:
     - Complete tasks in the order listed in tasks.md
     - Do not change any file not listed in the tasks
     - After each task, run the task's `verification` command and confirm the expected outcome
     - After all tasks, run the full relevant test suite and confirm it passes
     - After the full test suite passes, move the entire `plans/$ARGUMENTS/` directory into `plans/completed/$ARGUMENTS/` (create `plans/completed/` if it does not exist)
     - Commit all changes with a descriptive message reflecting what was implemented. This is the final step.
   - **Optional:** up to 3 additional rules if the design has non-obvious constraints that would otherwise get lost (e.g. `"use_tools=False is non-negotiable — context isolation depends on it"`). Do not add rules that merely restate the design doc.
   - Nothing else. This file is meant to be copied and pasted directly as a prompt.

7. After all five files are written, commit them:
   ```
   git add plans/$ARGUMENTS/
   git commit -m "docs(spec): add spec for $ARGUMENTS"
   ```

8. **Pre-write checklist** — before writing any of the five files, confirm:
   - [ ] Product reasoning captured (not just what, but why this approach)
   - [ ] Context/Problem consistent with product.md's Problem Statement and Goal
   - [ ] At least 3 reuse candidates identified and verified to exist, or "No applicable reuse" justified
   - [ ] Every modified file in design.md has a function name or line range
   - [ ] Every task in tasks.md has a concrete `verification` command
   - [ ] Every acceptance criterion in `requirements.md` maps to at least one `tests` entry across the task list, or is explicitly marked untestable with a one-line reason. No criterion silently has zero coverage.
   - [ ] Edge Cases section lists ≥3 scenarios
   - [ ] kickoff.md contains only the 6 required rules plus (if needed) ≤3 justified extras
   - [ ] Every user-facing behavior change has its documentation file in a task's `files` array — or "No docs update needed" is stated in design.md File Changes

   If any box can't be checked, go back to the plan discussion and ask the user rather than guessing.

Use the plan discussion above as the source of truth. If anything was left ambiguous, make a reasonable choice and note it in the design doc.

---

## Example — expected format and level of detail

### product.md

**Problem Statement**
Manual JSON parsing across the codebase is fragile — three bugs in the last month traced back to missing keys or wrong types in LLM output. Each call site has its own ad-hoc parsing.

**Goal**
All LLM JSON envelopes validated through Pydantic models, so structural bugs are caught at parse time with clear error messages instead of surfacing as KeyError deep in business logic.

**Success Criteria**
- All existing tests pass after migration
- New validation tests cover every LLM output shape
- No raw `json.loads` calls remain on LLM output paths

**Product Reasoning**
Considered runtime type checking with beartype, but Pydantic is already a dependency and gives us schema generation for free. Also considered TypedDict, but it doesn't validate at runtime — only at type-check time. Pydantic catches malformed LLM output at the boundary, which is the actual failure mode.

**Scope Boundary**
- Does not change prompt templates or LLM behavior — only how responses are parsed
- Does not add new LLM calls or change retry logic
- Does not refactor non-LLM JSON (config files, state files)

---

### requirements.md

**Context / Problem**
One short paragraph consistent with product.md's Problem Statement and Goal. Convert relative dates to absolute (e.g. "by 2026-04-20"). Everything below assumes the reader has read this.

**Overview**
One paragraph describing what this feature does at a functional level.

**User Stories**
- As a developer, I want X so that Y.

**Acceptance Criteria**
- [ ] Given precondition, when action, then expected result.

**Out of Scope**
- What this change does NOT touch.

**Dependencies**
- `path/to/file.py` — why it's involved.

---

### design.md

**Architecture Overview**
Where this fits in the system and the approach at a high level.

**Data Model**
New classes, fields, schema changes — or "No changes."

**File Changes**
- Modified: `src/parsing/models.py::load_backlog (~line 61)` — swap manual json.loads for `backlog_adapter.validate_json`.
- Modified: `src/execution/runner.py::_extract_result_text (~line 118)` — use `ClaudeOutput.model_validate_json`.
- Created: `src/models.py` — Pydantic models for all LLM JSON envelopes.
- Modified: `docs/how-it-works.md` — update confidence tracking section to reflect new ClaudeOutput shape. *(If no user-facing behavior changes, write "No docs update needed" here instead.)*

**Reuse / Existing Code Audit**
- `src/utils.py::slugify` — reuse for deriving the feature slug, do not reimplement.
- `src/runner.py::invoke` — existing retry/timeout wrapper; all new LLM calls go through it.
- `src/logging.py::get_logger` — standard logger factory; do not create new logging infrastructure.

**Technical Detail**
Data flow, prompt structures, algorithms. Be specific about what calls what.

**Edge Cases**
- Partial LLM output: Pydantic validation fails — catch `ValidationError`, log details, fall back to raw string.
- Retry on partial state: Task rerun after previous crash — check for existing artifact and skip regeneration if valid.
- Empty input: Caller passes empty list — return empty result without hitting the LLM.

---

### tasks.md

```json
[
  {
    "title": "Feature — Part 1: specific piece",
    "description": "Detailed implementation instructions referencing exact functions and line numbers from design.md. Enough context to be a kickoff prompt on its own.",
    "files": ["src/models.py", "docs/how-it-works.md"],
    "tests": ["tests/test_models.py — covers criteria 1,2: validates well-formed JSON and rejects partial output"],
    "complexity": "S",
    "is_visual": false,
    "verification": "pytest tests/test_models.py -v — all tests pass"
  }
]
```
