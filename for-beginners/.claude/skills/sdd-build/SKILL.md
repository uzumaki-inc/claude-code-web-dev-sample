---
name: sdd-build
description: Reads the four SDD planning docs (CLAUDE.md, docs/requirements.md, docs/design.md, docs/tasks.md) and implements the app described in them, working through the task list in order. Use this after running /sdd-init.
disable-model-invocation: true
---

<!--
  WHAT IS THIS FILE?
  This is the entry point for the /sdd-build SKILL — the implementation half
  of the lightweight Spec-Driven Development (SDD) workflow shipped with this
  for-beginners project.

  HOW IT FITS WITH /sdd-init
  /sdd-init asks the user 5 questions and generates 4 planning docs.
  /sdd-build reads those 4 docs and turns them into working code.

  Splitting them into two skills means each one stays small and easy to read.
  It also means a user can edit the generated docs by hand between the two
  steps if they want to tweak the plan before any code is written.

  WHY A SKILL AND NOT A SLASH COMMAND?
  Same reasons as /sdd-init and /code-review:
  - Multi-step orchestration (pre-flight check, recap, implement, hand off)
  - Benefits from being structured as numbered Steps with clear pause points

  WHY disable-model-invocation IS true (NOT false LIKE IN /code-review)
  This skill only runs when the user explicitly types /sdd-build. Claude will
  never auto-trigger it, even if the conversation sounds related.

  The reason: a beginner who hasn't generated the four SDD docs yet (or who
  isn't using the SDD workflow at all) might casually say "build the app" or
  "start implementing it". We don't want Claude to silently launch a pipeline
  that depends on planning files the user has never heard of and may not have
  created. Keeping this manual-only makes the SDD workflow an explicit,
  deliberate choice — which matches how the rest of the for-beginners project
  teaches each building block one step at a time.

  /code-review can safely default to auto-trigger because (a) the user has
  clearly asked for a review by the time it kicks in, and (b) it doesn't
  depend on any other files existing first.

  WHAT HAPPENS NEXT?
  After this skill finishes implementation, the user is told to run
  /code-review on the new code — closing the loop with the existing skill.
-->

You are running the SDD implementation pipeline. Your job is to read the four
planning docs the user generated with /sdd-init and turn them into working code.

---

## Step 1: Pre-flight check

Use the Read tool to verify all four of these files exist:

- `CLAUDE.md`
- `docs/requirements.md`
- `docs/design.md`
- `docs/tasks.md`

If any of them is missing, stop and tell the user:

> "I can't find one or more of the SDD planning docs. Please run **/sdd-init**
> first to generate them, then re-run /sdd-build."

List which files are missing so the user knows what's wrong. Do not try to
guess or proceed without the docs.

---

## Step 2: Load context

Read all four docs from top to bottom. Form a clear mental model of:

- What the app is (`CLAUDE.md`, `docs/requirements.md`)
- How it should be structured (`docs/design.md`)
- The order in which to build it (`docs/tasks.md`)

---

## Step 3: Recap and confirm

Summarize in 2 to 3 sentences what you're about to build. Mention the tech
stack and the number of tasks. Then ask:

> "Ready for me to start implementing? Reply **yes** to begin, or tell me what
> to adjust first."

Wait for confirmation before writing any code. If the user wants changes to
the plan, point them at the relevant doc to edit and stop.

---

## Step 4: Implement the tasks in order

Walk through `docs/tasks.md` from the top. For each task:

1. State which task you're starting (one short line).
2. Implement it. Create or modify files as needed. Place new source files
   inside the project root (use a `src/` subfolder if `docs/design.md` calls
   for one — otherwise put files at the project root next to `CLAUDE.md`).
3. State briefly what changed (one short line).
4. Move on to the next task.

Keep each task small. If a task in `tasks.md` looks too big, do the smallest
useful slice of it and note that the rest is still pending.

Do not pause between tasks unless the user interrupts you. Beginners watching
this skill run want to see code appearing.

---

## Step 5: Final summary and hand-off

When all tasks are done, give the user:

- A short list of the files you created or modified
- One sentence on how to run the app (pull this from `CLAUDE.md`)
- A nudge to run **/code-review** on the new code
- A reminder that they can run **/explain <file>** on any file to understand it

Example closing:

> "Done. I created `src/index.js` and `src/todo.js`. Run it with `node src/index.js`.
> Next, try **/code-review** to check the code, or **/explain src/todo.js** if
> you want a walkthrough of how it works."
