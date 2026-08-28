---
name: design
description: Turn an approved spec.md into design.md — the technical approach, the blast radius of the change, contract changes, failure handling, and migration path. Use after /brownspec:spec, or when the user asks how to build something already specified, or says /brownspec:design.
---

# Design the change

Input is `.brownspec/features/<slug>/spec.md`. Output is `design.md` beside it. If the
spec is missing or its exit checklist does not pass, say so and run `/brownspec:spec`
instead — designing against an unfinished spec produces a document nobody trusts.

Read `.brownspec/conventions.md` first. A design that departs from how this codebase is
written needs a written reason, not silence.

## Phase 1 — Blast radius

**This is the phase that earns the document.** The spec named where the change lands;
now find everything that depends on those points. Run **level 3** of the `repo-reading`
skill against them — and only them.

For each point of change: callers (direct, then one hop out), importers, contract
consumers across process boundaries, data touched, tests currently covering it.

**Record the provenance at the top of the dependency section, verbatim:**

```markdown
**Dependency source:** graph (complete within static analysis)
```
```markdown
**Dependency source:** text-search (may be incomplete — misses import aliases,
re-exports, and dynamic dispatch)
```

If the change touches code using `getattr`, decorators, registries, or runtime attribute
assignment, say so next to the list. Neither a graph nor a grep sees those, and a reader
deciding how carefully to review needs to know which kind of list they are holding.

An unmarked dependency list reads as complete. Never leave one unmarked.

## Phase 2 — Propose, don't ask

Unlike the spec phase, most design questions have a defensible default once the spec is
settled and the blast radius is known. Read the affected code, pick an approach, and
**present it as a recommendation with the alternatives you rejected and why.** Fill in
`${CLAUDE_PLUGIN_ROOT}/templates/design.md.tmpl` and write it to disk before discussing.

Ask only where the choice is genuinely the user's: cost, risk appetite, migration window,
or an org constraint invisible from the repo. One question per message, and rewrite
`design.md` after each answer.

## Phase 3 — The parts that carry the weight

**Contract changes.** Every API, event, schema, and config key that changes, each marked
additive or breaking. For each breaking change: who consumes it — from the blast radius,
not from memory — and what they have to do.

**Failure handling.** Per dependency: slow, unavailable, or returning something invalid.
Timeouts, retries, fallback, and what the caller sees. "It will retry" is not a design —
how many times, what backoff, what happens after the last attempt.

**Migration and rollback.** How the system reaches the target state without downtime, and
how to get back. Data migrations get their own sequence: expand, backfill, contract. State
explicitly whether the change is backward-compatible during the rollout window, and the
point past which revert is no longer clean.

**Rejected alternatives.** Two or three, each with its reason. This is what stops the same
debate restarting in three months.

## Phase 4 — Policies carried forward

Copy the `Policies touched` list from the spec and, for each entry, name where the design
answers it — or record that it is still open. Same rule as the spec phase: **route, do not
certify.** A design does not make anything compliant; it makes the open questions visible
to the person who decides.

## Exit check

- [ ] Every requirement in `spec.md` maps to something in the design
- [ ] Blast radius established, **with its provenance stated**
- [ ] All contract changes listed, breaking ones with a consumer migration path
- [ ] Failure behavior per dependency, with concrete numbers
- [ ] Rollback path described, including where it stops being clean
- [ ] Observability named: what is logged, what is measured, what alerts
- [ ] At least two rejected alternatives with reasons
- [ ] Every policy the spec touched is either answered here or listed as still open
- [ ] Departures from `conventions.md` are written down with justification

Then hand off to `/brownspec:tasks`. Do not write implementation code here.
