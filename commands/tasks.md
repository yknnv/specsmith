---
description: Break an approved design into an ordered task list that stays deployable at every step.
argument-hint: [feature slug]
---

Use the `tasks` skill from the SpecSmith plugin.

Feature: $ARGUMENTS — if empty, use the most recently modified directory under
`.specsmith/features/`, and say which one you picked.

Write `trace.json` alongside `tasks.md`, with `files` empty. Report acceptance-criteria
coverage explicitly, including gaps.
