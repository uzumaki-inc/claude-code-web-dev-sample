# General Review Instructions

<!--
  WHAT IS THIS FILE?
  This is a SUPPORTING FILE for the /code-review skill.

  It exists to demonstrate one of the key advantages of skills over commands:
  skills can include multiple files in their directory. This lets you separate
  concerns — the main SKILL.md handles orchestration, while detailed reference
  content lives here, out of the way.

  Claude reads this file when SKILL.md references it. You can add as many
  supporting files as you need: examples, templates, checklists, scripts, etc.

  A slash command (.claude/commands/explain.md) cannot do this — it's one file only.
-->

When performing a general code review, evaluate the code across these categories:

## Correctness
- Are there any obvious bugs or logic errors?
- Are edge cases handled? (null/undefined values, empty arrays, division by zero, etc.)
- Do loops and conditionals behave as intended?
- Are asynchronous operations handled correctly (missing awaits, unhandled promises)?

## Readability
- Are variable and function names descriptive and consistent?
- Is the code structure easy to follow from top to bottom?
- Are there overly complex expressions that could be simplified?
- Is there dead code or commented-out code that should be removed?

## Maintainability
- Is there duplicated logic that could be consolidated?
- Are there hardcoded values that should be constants or configuration?
- Are functions doing too many things at once?
- Are dependencies between different parts of the code unnecessarily tight?

## Common Issues to Flag
- Unused imports or variables
- Functions longer than ~50 lines (worth noting, not always a problem)
- Non-obvious logic without an explanatory comment
- Inconsistent code style within the same file

## How to Report

For each issue found:
1. Name the file and line number
2. Describe the issue in one sentence
3. Show the fix

Keep fixes minimal — only change what needs to change. Do not refactor beyond
what is necessary to resolve the identified issue.

## Spec Compliance (only when SDD docs are present)

<!--
  This section only applies when /sdd-init was run and planning docs exist.
  If no SDD docs were found, skip this section entirely.
-->

- Does the code implement every user-facing requirement from requirements.md?
- Does the file layout and component structure match design.md?
- Are all tasks in tasks.md completed in the code?
- Flag any gaps between spec and implementation
