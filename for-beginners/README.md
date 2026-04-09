# Claude Code — For Beginners

This directory is a hands-on introduction to the four core building blocks of Claude Code:
**slash commands**, **skills**, **hooks**, and **subagents**.

Each one is demonstrated through a code review theme. Try them out, read the comments inside
each file, and you'll have a solid mental model of how Claude Code works.

---

## Prerequisites

- [Claude Code installed](https://code.claude.com/docs)
- Navigate to this directory in your terminal, then run `claude` to start a session
- A small piece of sample code to review and explain. If you don't have anything handy,
  the quickest path is to ask Claude to create one. For example:
  ```
  Create a tictactoe.js file
  ```
  or
  ```
  Create a blackjack.rb file
  ```
  The `/code-review` skill is designed for source code, so pick a real program rather
  than a documentation file (e.g. a README.md file) — that's what makes the four
  building blocks click.

---

## The Four Building Blocks

### 1. Slash Command — `/explain`
**File:** `.claude/commands/explain.md`

A slash command is the simplest building block: a single `.md` file that becomes a reusable prompt.
Slash commands are **always triggered manually** by you — Claude will never invoke one on its own.

**Try it:**
```
/explain README.md
```
or
```
/explain the explain.md file
```

**Key characteristics:**
- Single `.md` file (no directory, no supporting files)
- Always manually triggered
- Best for simple, focused, one-shot tasks

---

### 2. Skill — `/code-review`
**Files:** `.claude/skills/code-review/SKILL.md` + `general-review-instructions.md`

A skill looks similar to a slash command from the outside (you invoke it with `/code-review`), but it's
more powerful under the hood. Skills live in a **directory** and can have supporting files.
They can also be **auto-triggered** by Claude when it determines one is relevant.

**Try it:**
```
/code-review
```

When prompted, try it once with `auto` and once with `step-by-step` to see how the pipeline
behaves differently.

**Key differences from a slash command:**
- Directory structure (not a single file)
- Can include supporting context files (see `general-review-instructions.md`)
- Can be auto-triggered by Claude (controlled by `disable-model-invocation` in the frontmatter)
- Can orchestrate complex multi-step workflows, including launching subagents

---

### 3. Subagent — `security-reviewer`
**File:** `.claude/agents/security-reviewer.md`

A subagent is a separate Claude instance that runs in **isolation** from your main conversation.
It has its own focused instructions and doesn't inherit your conversation history.

The `security-reviewer` agent is launched automatically by the `/code-review` skill during Step 3
of the review pipeline. You can also invoke it directly:

```
Use the security-reviewer agent to review auth.js
```

**Why a subagent for security review?**
- Security review doesn't need your conversation context — it just needs the code
- Keeps verbose security findings out of your main context window
- Specialized agents do one thing well without distraction

---

### 4. Hook — PostToolUse
**File:** `.claude/settings.json`

A hook is a shell command that runs automatically in response to a Claude Code event.
Unlike the other three building blocks, hooks are **not written in a prompt style** —
they're shell commands configured in `settings.json`.

The hook in this project fires after Claude writes or edits any file, reminding you to
run a review.

**You don't trigger hooks manually** — just ask Claude to edit a file and watch what happens:

```
Create a simple hello.js file
```

After Claude writes the file, the hook fires automatically.

**Key characteristics:**
- Configured in `settings.json`, not as a standalone `.md` file
- Triggered by Claude Code events (e.g., after a tool runs), not by you
- Runs a shell command — can be a reminder, a linter, a test runner, anything

---

## The Full Workflow

Here's how the four components work together:

```
1. You ask Claude to write some code
        ↓
2. Hook fires → reminds you to run /code-review
        ↓
3. You run /code-review (skill)
        ↓
4. Skill asks: auto or step-by-step?
        ↓
5. General review runs (using general-review-instructions.md for criteria)
        ↓
6. Security subagent launches and reviews independently
        ↓
7. Both sets of fixes are applied. Review complete.
```

## If you get stuck

At any point, you can run `/explain <file>` to get a plain-English explanation of any file.

---

## File Structure

```
for-beginners/
├── README.md                                   ← you are here
└── .claude/
    ├── settings.json                           ← hook configuration
    ├── commands/
    │   └── explain.md                          ← /explain slash command
    ├── skills/
    │   └── code-review/
    │       ├── SKILL.md                        ← /code-review skill (main entry point)
    │       └── general-review-instructions.md  ← supporting context for the skill
    └── agents/
        └── security-reviewer.md               ← security review subagent
```
