# Testing Guide

A step-by-step manual test plan for the four building blocks demonstrated in this
project. Each test includes explicit pass criteria, troubleshooting tips, and notes
about what to expect if things go differently than planned.

> **How to use this guide:** work through the tests in order. Some tests build on
> state created by earlier ones (e.g., a file created in Test 1 is used in Test 2).
> When a test asks you to start a fresh session, that means **fully exit Claude Code
> and run `claude` again** — this is required for tests that depend on Claude not
> already having context.

---

## Pre-Flight Checks

Before running any tests, verify the project is intact. From the `for-beginners`
directory, confirm that all of the following files exist:

```
README.md
.claude/settings.json
.claude/commands/explain.md
.claude/skills/code-review/SKILL.md
.claude/skills/code-review/general-review-instructions.md
.claude/agents/security-reviewer.md
```

You can quickly list them with:
```
ls -R .claude
```

If any are missing, the tests below will not work correctly. Restore the project
before continuing.

---

## Setup

1. Open a terminal and navigate to the `for-beginners` directory:
   ```
   cd path/to/repo/for-beginners
   ```

2. Start a Claude Code session:
   ```
   claude
   ```

3. Send a simple message like `hello` to confirm the session is responsive.

> **Important:** Always start Claude Code from inside `for-beginners`, not from
> the repository root. Project-scoped configuration in `.claude/` is loaded
> relative to your working directory.

---

## Test Conventions

Each test below uses the same structure:

- **Goal** — what we're verifying
- **Setup** — any prerequisites
- **Steps** — exactly what to type or do
- **Pass criteria** — what must be true for the test to pass
- **Common failure modes** — what it might look like if it fails, and what to check

When a test says **"start a fresh session"**, fully exit Claude Code (`/exit` or
Ctrl+C) and run `claude` again. Tests that depend on auto-triggering will not work
reliably if you reuse a session that already discussed the relevant topic.

---

## Test 1: Hook (PostToolUse)

**Goal:** Verify the hook in `.claude/settings.json` fires after Claude writes or
edits a file.

**Setup:** Fresh session in `for-beginners`.

**Steps:**

1. Ask Claude:
   ```
   Create a file called hello.js that logs "Hello, world!" to the console.
   ```

2. Watch the output immediately after Claude writes the file.

**Pass criteria (any one of these is acceptable):**

- A reminder message about `/code-review` appears directly in your terminal output, OR
- Claude itself mentions the reminder in its next response (this happens when the
  hook surfaces context to Claude rather than to the terminal directly), OR
- Toggling verbose mode (press `Ctrl+O`) reveals the hook output in the transcript

**Common failure modes:**

- *No reminder anywhere, even with verbose mode:* The hook may not be firing.
  Verify `.claude/settings.json` is valid JSON (no trailing commas, balanced braces),
  and confirm you started Claude from inside `for-beginners`.
- *Reminder appears but is garbled (literal `\n` characters visible):* The hook is
  firing but the output isn't being interpreted. This is informational — the test
  still passes for the purpose of confirming the hook fires.
- *Claude wrote the file with a different tool than Write/Edit:* The matcher won't
  fire. Ask Claude to "use the Write tool to create..." explicitly.

---

## Test 2: Slash Command — `/explain`

**Goal:** Verify the `/explain` slash command runs as a manually-triggered prompt and
behaves as documented in three different scenarios.

### Test 2a — Explain a file by path

**Steps:**
```
/explain hello.js
```

**Pass criteria:** Claude reads `hello.js` and explains what it does in one or two
clear sentences. The explanation should describe the `console.log` statement.

### Test 2b — Explain a project file

**Steps:**
```
/explain .claude/commands/explain.md
```

**Pass criteria:** Claude reads its own slash command file and explains its purpose
(it's a slash command that asks Claude to explain code).

### Test 2c — Explain with no argument

**Steps:**
```
/explain
```

**Pass criteria:** Claude asks you what you'd like explained, instead of guessing
or producing a generic response.

**Common failure modes:**

- *`/explain` is not recognized:* Confirm `.claude/commands/explain.md` exists.
  Try restarting Claude Code so it picks up the slash command.
- *Claude doesn't read the file before explaining:* The slash command may have been
  invoked in a different working directory. Make sure `hello.js` exists relative
  to your current directory.

---

## Test 3: Skill — `/code-review` in step-by-step mode

**Goal:** Verify the `/code-review` skill orchestrates a multi-phase pipeline and pauses
for approval between phases.

**Setup:** `hello.js` should still exist from Test 1. If it doesn't, recreate it.

**Steps:**

1. Run:
   ```
   /code-review
   ```

2. When Claude asks "auto or step-by-step?", reply:
   ```
   step-by-step
   ```

3. Read the general review summary Claude provides.

4. When Claude pauses and asks if you're ready to continue, respond:
   ```
   yes
   ```

5. Read the security review summary that follows.

6. Read the final combined summary.

**Pass criteria:**

- Claude asks "auto or step-by-step?" before doing any review work
- Claude completes a general review and presents a summary
- Claude **pauses and waits for your confirmation** before starting the security review
- Claude launches a subagent for the security phase (it should clearly indicate it
  is delegating to the `security-reviewer` agent)
- Claude provides a final summary covering both phases

**Common failure modes:**

- *Claude skips the question and runs the whole pipeline:* The skill prompt may
  not have loaded. Restart Claude Code and try again.
- *Claude does not pause between phases:* This means the step-by-step instruction
  was ignored. Check that you typed exactly `step-by-step` (not "stepwise" or
  similar).
- *Claude does not launch a subagent:* See Test 6 for how to verify the subagent
  works in isolation.

---

## Test 4: Skill — `/code-review` in auto mode

**Goal:** Verify the same pipeline runs without checkpoints when the user picks auto.

**Setup:** Modify `hello.js` slightly so there's something fresh to review:
```
Add a comment to hello.js explaining what it does
```

**Steps:**

1. Run:
   ```
   /code-review
   ```

2. When asked, reply:
   ```
   auto
   ```

**Pass criteria:**

- Claude does **not** pause for confirmation between the general and security phases
- Claude produces a final summary that clearly distinguishes both phases
- The subagent is still launched for the security phase

**Common failure modes:**

- *Claude pauses anyway:* The skill may be defaulting to a safer behavior. Reread
  Step 1 of the skill prompt and confirm `auto` was understood.

---

## Test 5: Skill auto-triggering (without `/code-review`)

**Goal:** Verify Claude can auto-invoke the `/code-review` skill when its description
matches what the user asked for.

> **Important caveat:** Auto-triggering depends on Claude's judgment, which is
> non-deterministic. This test may pass on the first try, may need rephrasing, or
> may not trigger at all on a given session. If you suspect the skill exists but
> isn't being picked up, that is a meaningful observation — note it and move on.

**Setup:** Start a **fresh Claude Code session**.

**Steps:**

Try these messages, one at a time, in separate fresh sessions:

1. ```
   Can you do a code review of hello.js?
   ```

2. ```
   I'd like you to check this code for issues and security problems.
   ```

3. ```
   Please review the code in this directory.
   ```

**Pass criteria:**

- For at least one of the above prompts, Claude triggers the `/code-review` skill on its
  own (you'll know because it asks "auto or step-by-step?" without you typing
  `/code-review`)

**Common failure modes:**

- *Claude responds normally without triggering the skill:* This may be expected.
  Try a more explicit phrasing. Auto-triggering relies on Claude matching your
  request to the `description` field in `SKILL.md` — phrasings closer to that
  description are more likely to trigger.
- *Claude triggers a different skill instead:* Note which one. If there are no
  other skills installed, this would be a surprise.

**Optional comparison test:** Edit `SKILL.md` and change `disable-model-invocation`
to `true`, then restart Claude Code. The skill should now only respond to `/code-review`
and never auto-trigger. (Remember to revert the change afterwards.)

---

## Test 6: Subagent — `security-reviewer` invoked directly

**Goal:** Verify the security subagent can be invoked directly by the user, runs
in isolation, and identifies security issues.

**Setup:** Create a file with a deliberate, obvious security issue.

⚠️ **Warning:** This file contains a SQL injection vulnerability by design. Do
**not** copy this code into any real project, run it against any real database, or
deploy it anywhere. Delete it during cleanup.

**Steps:**

1. Ask Claude to create the test file:
   ```
   Create a file called login.js that exports a function which takes a username
   and password and builds a SQL query by concatenating them into a string like
   "SELECT * FROM users WHERE name = '" + username + "' AND password = '" + password + "'"
   ```

2. Invoke the subagent directly:
   ```
   Use the security-reviewer agent to check login.js for vulnerabilities
   ```

**Pass criteria:**

- Claude clearly indicates it is delegating to the `security-reviewer` subagent
- The subagent identifies the SQL injection vulnerability
- The output includes the documented format: severity, category, location,
  description, and a suggested fix
- The subagent does **not** comment on general code quality (e.g., naming, style) —
  it should be focused exclusively on security

**Common failure modes:**

- *Claude reviews the code itself without delegating:* The subagent may not have
  been recognized. Confirm `.claude/agents/security-reviewer.md` exists and that
  the `name:` in its frontmatter is `security-reviewer`.
- *Subagent flags general code quality issues:* The system prompt for the agent
  should restrict it to security only. Re-read `security-reviewer.md` and confirm
  the instructions are intact.

---

## Test 7: Full end-to-end pipeline

**Goal:** Verify all four components — hook, slash command, skill, and subagent —
work together as documented.

**Setup:** Start a **fresh Claude Code session**.

**Steps:**

1. Ask Claude to write a file with multiple intentional flaws:
   ```
   Create a file called app.js. It should export a function that fetches user
   data by ID. Build the SQL query using string concatenation, and after fetching,
   log the entire user object (including the password field) to the console.
   ```

   ⚠️ **Warning:** This file is intentionally insecure. Do not reuse this code.

2. Watch for the hook to fire after Claude writes the file (Test 1 behavior).

3. Run:
   ```
   /code-review
   ```

4. Choose `step-by-step`.

5. Observe what the **general review** finds (e.g., excessive logging,
   maintainability concerns).

6. Confirm to proceed when prompted.

7. Observe what the **security review subagent** finds (SQL injection, sensitive
   data in logs).

8. Read the final combined summary.

9. Optionally, run:
   ```
   /explain app.js
   ```
   to confirm the slash command still works after the review.

**Pass criteria:**

- The hook fires after Claude writes `app.js` (per Test 1 pass criteria)
- The general review identifies non-security issues
- The security review identifies security issues (SQL injection, sensitive data
  exposure)
- Both phases are summarized at the end
- `/explain` works as expected

---

## Cleanup

After testing, remove the files created during the tests so the project is clean:

```
rm hello.js login.js app.js
```

If you modified `SKILL.md` for the optional comparison test in Test 5, revert that
change as well:

```
git diff .claude/skills/code-review/SKILL.md
git checkout .claude/skills/code-review/SKILL.md
```

(Skip the `git checkout` if you have other intentional changes you want to keep.)

---

## Troubleshooting

**Slash commands aren't recognized.**
- You may not be running `claude` from inside `for-beginners`. Project-scoped
  commands and skills are loaded relative to your working directory.
- Try restarting Claude Code so it re-scans `.claude/`.

**The hook never fires.**
- Validate `.claude/settings.json` with a JSON linter — a single trailing comma
  or unbalanced brace will silently disable the hook.
- Confirm Claude actually used `Write` or `Edit` (the matcher only applies to
  those tools). Some operations use other tools.

**Auto-triggering of the skill never works.**
- Auto-triggering depends on Claude's interpretation of your message and the
  skill's `description` field. Try phrasings closer to the description text.
- Verify `disable-model-invocation` is `false` (or omitted) in `SKILL.md`.

**The subagent runs but produces no findings on obviously vulnerable code.**
- Re-read `security-reviewer.md` to confirm the system prompt is intact.
- Try invoking the subagent again with a more direct instruction.

---

## Results Checklist

Use this to track which tests pass:

| # | Component | Test | Pass? | Notes |
|---|-----------|------|-------|-------|
| 1 | Hook | Fires after Write/Edit | | |
| 2a | Slash Command | `/explain <file>` | | |
| 2b | Slash Command | `/explain <project file>` | | |
| 2c | Slash Command | `/explain` with no arg | | |
| 3 | Skill | `/code-review` step-by-step with checkpoint | | |
| 4 | Skill | `/code-review` auto without checkpoint | | |
| 5 | Skill | Auto-triggering without `/code-review` | | |
| 6 | Subagent | Direct invocation finds SQL injection | | |
| 7 | All | End-to-end pipeline | | |

---

## What Counts as Success

You don't need every test to pass perfectly for the project to be useful. The
important things to confirm are:

1. **The slash command always works** (Test 2) — this is the most reliable building block
2. **The skill always works when invoked manually** (Tests 3 and 4) — it should
   never silently fail when you type `/code-review`
3. **The subagent works when invoked directly** (Test 6) — even if auto-delegation
   from the skill is flaky, the subagent itself should be functional

Auto-triggering (Test 5) and hook output visibility (Test 1) are the most
environment-sensitive parts of the project. If those work, that's a bonus; if they
don't, the project still demonstrates the file structure and configuration patterns
that beginners need to understand.
