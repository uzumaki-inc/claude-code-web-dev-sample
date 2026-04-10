---
description: Explain what a file or function does in plain English
argument-hint: [file path or function name]
---

# /explain

<!--
  WHAT IS THIS FILE?
  This is a Claude Code SLASH COMMAND — the simplest building block in Claude Code.
  A slash command is just a single .md file that becomes a reusable prompt.
  When you type /explain, the content below is sent to Claude as your prompt.

  WHY A SLASH COMMAND AND NOT A SKILL?
  Slash commands are the right choice when your task is:
  - Simple and focused (one prompt, one output)
  - Always manually triggered — you decide when to run it
  - Self-contained (no need for supporting files or orchestration)

  "Explain this code" is always a one-shot task. There's no complex pipeline,
  no supporting reference files needed, no reason for Claude to auto-trigger it.
  A single .md file is exactly the right amount of structure.

  FRONTMATTER FIELDS USED ABOVE:
  - description: Shown in the / menu so users know what the slash command does.
  - argument-hint: Shown during autocomplete to remind users what to type after /explain.

  COMPARE WITH THE SKILL:
  Open .claude/skills/code-review/SKILL.md to see how a skill differs:
  - It lives in a directory (not a single file)
  - It has a supporting file (general-review-instructions.md)
  - It can be auto-triggered by Claude
  - It orchestrates a multi-step workflow including a subagent

  HOW TO USE THIS SLASH COMMAND:
  /explain <file path or function name>

  Examples:
    /explain README.md
    /explain the getUserById function
    /explain .claude/skills/code-review/SKILL.md

  NOTE ON $ARGUMENTS:
  The placeholder $ARGUMENTS below is replaced with whatever the user types
  after /explain. So `/explain README.md` results in the prompt receiving
  "README.md" in place of $ARGUMENTS.
-->

The user has asked you to explain the following: **$ARGUMENTS**

If they provided a file path:
- Read the file
- Explain its purpose in one or two sentences
- Describe the key sections, functions, or logic
- Explain how it fits into the overall project if that's apparent

If they provided a function or class name:
- Find it in the codebase
- Explain what it does, what it takes as input, and what it returns
- Note any non-obvious behaviors or side effects

If $ARGUMENTS is empty or unclear, ask: "What would you like me to explain? You can give me a file path or a function name."

Keep the explanation clear and accessible. Assume the reader understands programming
but is unfamiliar with this specific code.

Provide all explanations in **English**.
