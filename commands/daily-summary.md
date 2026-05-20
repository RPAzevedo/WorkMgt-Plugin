---
description: Produce a daily summary of the configured Trello board's Doing and Done lists.
argument-hint: "[board name]"
---

Invoke the `daily-trello-summary` skill, passing through the argument
`$ARGUMENTS` (which may be empty — the skill will fall back to the
`default_board` from the user's `~/.claude/work-mgt/config.json`).
