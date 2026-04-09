---
name: sdd-init
description: Interactive questionnaire that captures a beginner's app idea and generates four small SDD planning docs (CLAUDE.md, docs/requirements.md, docs/design.md, docs/tasks.md). Use this when the user wants to start a new project from scratch and benefit from a tiny bit of upfront planning before any code is written.
disable-model-invocation: true
---

<!--
  WHAT IS THIS FILE?
  This is the entry point for the /sdd-init SKILL — the questionnaire half of
  the lightweight Spec-Driven Development (SDD) workflow shipped with this
  for-beginners project.

  WHAT IS SDD AND WHY IS THERE A SKILL FOR IT?
  Spec-Driven Development just means: before writing code, take a few minutes
  to write down what you're building and why. For experienced engineers this is
  routine; for beginners it's often skipped, leading to projects that drift off
  course or never finish.

  This skill keeps SDD lightweight on purpose. It asks 5 short questions and
  generates 4 small docs from your answers. That's enough to anchor a small
  project without feeling like paperwork.

  WHY A SKILL AND NOT A SLASH COMMAND?
  - It runs a multi-step interactive flow (ask, wait, ask, wait, ...)
  - It needs supporting files (the four templates next to this file)
  - Slash commands are single-file and one-shot; this needs both directory
    structure and orchestration

  Compare with .claude/skills/code-review/SKILL.md to see the same pattern
  applied to a different problem.

  WHY disable-model-invocation IS true (NOT false LIKE IN /code-review)
  This skill only runs when the user explicitly types /sdd-init. Claude will
  never auto-trigger it, even if the conversation sounds related.

  The reason: a beginner exploring this for-beginners folder might say
  something like "create a tic-tac-toe game" because they just want to see
  Claude write code. If /sdd-init auto-triggered on that, they would suddenly
  find themselves answering five planning questions when all they wanted was
  to watch code appear. That kind of surprise is exactly what we want to
  avoid in a tutorial — beginners should opt in to the SDD workflow on
  purpose, not stumble into it.

  /code-review can safely default to auto-trigger because (a) the user has
  clearly asked for a review by the time it kicks in, and (b) it doesn't
  derail the conversation into a long questionnaire.

  If you want to experiment with auto-triggering, flip this field to false,
  restart Claude Code, and try saying "I want to start a new project" in a
  fresh session — Claude should then offer to run this skill on its own.

  WHAT HAPPENS NEXT?
  After this skill finishes, the user runs /sdd-build, which reads the four
  generated docs and implements the app described in them.
-->

You are running the SDD initialization questionnaire. Your job is to ask the user
five short questions about the app they want to build, then generate four small
planning docs from their answers.

---

## Step 0: Pre-flight check

Before asking any questions, check whether any of the four target files already exist:

- `CLAUDE.md`
- `docs/requirements.md`
- `docs/design.md`
- `docs/tasks.md`

If any of them exist, list which ones and ask the user:

> "I found existing SDD docs at the paths above. Do you want to overwrite them?
> Reply **overwrite** to continue, or **cancel** to stop."

Wait for their reply. If they say cancel (or anything other than overwrite),
stop here without making any changes.

---

## Step 1: Introduce the questionnaire

Tell the user something like:

> "I'll ask you 5 short questions about the app you want to build. Your answers
> will become four planning docs (CLAUDE.md and three files in docs/). After
> that you can run /sdd-build to actually build it. Ready?"

Wait for them to confirm before starting.

---

## Step 2: Ask the questions, one at a time

Ask each question on its own. After each answer, echo a one-line confirmation
("Got it — ...") so the user knows you understood, then move on to the next.

1. **Elevator pitch:** "In one sentence, what app do you want to build?"
2. **Audience and motivation:** "Who is it for, and what problem does it solve for them?"
3. **Core actions:** "List 2 to 4 things a user should be able to do with it."
4. **Tech preference:** "Any technology you want to use? (Language, framework, runtime, etc.) Or reply 'let Claude pick' and I'll choose something sensible."
5. **Out of scope (optional):** "Anything you explicitly do NOT want to build yet? Reply 'none' to skip."

Do not move on to the next question until you have an answer for the current one.

---

## Step 3: Recap and confirm

Show the user a short bulleted recap of all five answers and ask:

> "Does this look right? Reply **yes** to generate the docs, or tell me what to fix."

If they want changes, update your understanding and re-show the recap. Keep
going until they confirm.

---

## Step 4: Generate the four docs

Read the four template files in this skill's `templates/` directory:

- [templates/CLAUDE.md.template](templates/CLAUDE.md.template)
- [templates/requirements.md.template](templates/requirements.md.template)
- [templates/design.md.template](templates/design.md.template)
- [templates/tasks.md.template](templates/tasks.md.template)

Each template uses `{{PLACEHOLDER}}` markers. Fill them in with the user's
answers (and your own reasonable inferences where the templates need them — for
example, generating an MVP-sized task list of about 5 to 8 steps from the core
actions). Then write the four files to:

- `CLAUDE.md`
- `docs/requirements.md`
- `docs/design.md`
- `docs/tasks.md`

Create the `docs/` directory if it does not exist.

Keep the generated content small. Each doc should be readable in under a minute.
Do not pad the docs with boilerplate the user did not ask for.

---

## Step 5: Tell the user what's next

After writing the files, summarize what was created (one bullet per file) and say:

> "Your SDD docs are ready. Next, run **/sdd-build** to implement the app from
> these docs. You can also edit any of the four files by hand first if you want
> to tweak the plan."
