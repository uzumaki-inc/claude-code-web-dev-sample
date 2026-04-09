---
name: code-review
description: Reviews code for quality and security issues. Use this when the user wants to review code, check for bugs, improve code quality, or audit for security vulnerabilities.
disable-model-invocation: false
---

<!--
  WHAT IS THIS FILE?
  This is the main entry point for a Claude Code SKILL.
  Skills are invoked with /code-review, just like a slash command — but they're more powerful.

  HOW THIS SKILL DIFFERS FROM THE /explain SLASH COMMAND:

  1. DIRECTORY STRUCTURE
     Slash commands are a single .md file. Skills live in a directory.
     This skill has a supporting file: general-review-instructions.md
     That file contains detailed review criteria, keeping this file focused
     on orchestration rather than getting cluttered with reference content.
     Open it to see what a supporting file looks like in practice.

  2. AUTO-TRIGGERING
     See the frontmatter above: `disable-model-invocation: false`
     This means Claude can invoke this skill automatically if it determines
     a code review is needed — even if you didn't type /code-review.
     Claude uses the `description` field above to decide when it's relevant.

     Try it: with `false`, ask Claude something like "review the code in
     tictactoe.js" without typing the slash command — Claude should pick up
     this skill on its own. Then change the field to `true`, restart your
     `claude` session so the new frontmatter is loaded, and ask the same
     thing again. This time Claude won't auto-trigger the skill; you'll
     have to invoke it explicitly with /code-review. Set it back to `false`
     when you're done experimenting.

  3. ORCHESTRATION
     This skill coordinates multiple steps and launches a subagent.
     A single slash command .md file would be too limited for this kind of workflow.
     Slash commands do one thing. Skills can do many things in sequence.

  HOW TO USE:
  Type /code-review and follow the prompts.
  Try it once with "auto" and once with "step-by-step" to see the difference.
-->

You are running a two-phase code review pipeline. Follow the steps below exactly.

---

## Step 1: Ask how the user wants to proceed

Before doing anything else, ask the user:

> "Would you like me to run the full review pipeline automatically, or pause for your
> approval between steps? Reply with **auto** or **step-by-step**."

Wait for their response. Store their choice — you'll need it in Step 2.

---

## Step 2: General Code Review

Determine what to review:
- If the user specified files or a directory, review those
- Otherwise, run `git diff HEAD` to find recently changed files and review those
- If there is no git history, ask the user which files to review

Apply the review criteria defined in [general-review-instructions.md](general-review-instructions.md).

Fix any issues you find directly in the files. After fixing, give a brief summary:
- How many issues were found
- What categories they fell into (e.g. readability, error handling)
- What was changed

**If the user chose step-by-step:** pause here and ask:
> "General review complete. Ready to move on to the security review?"
> Wait for confirmation before continuing.

---

## Step 3: Security Review (Subagent)

Launch the security review subagent using the Agent tool.

<!--
  WHY A SUBAGENT HERE?
  Security review is a good fit for a subagent because:
  - It doesn't need the context of our conversation — it just needs the code
  - It's a focused, self-contained task
  - Running it in a subagent keeps verbose security output out of our main context

  See .claude/agents/security-reviewer.md for how the subagent is defined.
-->

Pass the relevant file paths to the subagent and instruct it to return:
- A list of vulnerabilities found (if any)
- Suggested fixes for each

Once the subagent returns its findings, apply the suggested fixes. Summarize what was changed.

---

## Step 4: Final Summary

Provide a complete summary of the review:
- What the general review found and fixed
- What the security review found and fixed
- Any remaining concerns that require human judgment before merging
