---
name: tasks
description: Break an approved design.md into an ordered, reviewable task list, and seed trace.json so drift can be detected later. Use after /specsmith:design, when the user asks to break work into tasks or tickets, or says /specsmith:tasks.
---

# Break the design into tasks

Input is `spec.md` and `design.md` under `.specsmith/features/<slug>/`. Output is
`tasks.md` from `${CLAUDE_PLUGIN_ROOT}/templates/tasks.md.tmpl`, plus `trace.json`.

If the design's exit checklist does not pass, stop and say so. A task list built on an
unfinished design encodes its gaps as work items and hides them.

## Sizing

One task is one reviewable unit: half a day to a day for someone who knows the codebase,
producing a diff a reviewer can hold in their head. Split anything larger. If a task
cannot be sized because the approach is unclear, that is a design gap — name it and go
back rather than papering over it with a vague task.

## Ordering

By dependency, not by layer. The system stays deployable at every step:

1. **Expand** — additive, breaks nothing (new columns, new endpoints, flag off)
2. **Implement** — the behavior itself, behind the flag
3. **Migrate** — move consumers and data over
4. **Contract** — remove what is now dead

Mark what can run in parallel, and what must ship in one release together, with why.

Consumer migration from the design's blast radius becomes real tasks here. A breaking
change with three known callers is four tasks, not one.

## Each task carries

- A verb-first title
- The files or modules it touches — you have read the repo, be specific
- Which acceptance criterion from `spec.md` it advances
- How it is verified: the test, the check, the observable outcome
- Its blockers, by task ID

Testing and observability are tasks, not footnotes. If the design says a metric is
emitted, some task emits it.

## trace.json

Write it alongside `tasks.md`, with `files` empty. It is filled in as work happens — the
agent implementing a task records what it actually touched. That record is what
`/specsmith:drift` will later compare against the spec.

```json
{
  "slug": "<slug>",
  "spec": "spec.md",
  "generated": "<date>",
  "tasks": [
    {
      "id": "T1",
      "title": "<verb-first title>",
      "advances": ["A1"],
      "expected_paths": ["app/api/orders.py"],
      "files": [],
      "status": "todo"
    }
  ]
}
```

`expected_paths` is the design's prediction. `files` is what was really touched. Drift
detection lives in the gap between them, so do not pre-fill `files` with the prediction.

## Exit check

- [ ] Every acceptance criterion in `spec.md` is covered by at least one task
- [ ] Every task names its verification
- [ ] Dependency order stated, and the system is deployable between steps
- [ ] No task exceeds one day, or it is split
- [ ] Rollback and migration steps from `design.md` appear as real tasks
- [ ] Consumers named in the blast radius have migration tasks
- [ ] `trace.json` written, with `files` empty

Report coverage against the acceptance criteria explicitly, **including what is not
covered.** Then stop. Implementation is a separate decision the user makes.
