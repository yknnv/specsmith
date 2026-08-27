---
name: standards
description: Bootstrap or refresh .specsmith/ for this repository — project-map.json and conventions.md derived from the code, plus a policies.md stub for humans to fill in. Use on first run in a new repo, when conventions have drifted, or when the user says /specsmith:init.
---

# Bootstrap `.specsmith/`

First run in a repository. Produces three things:

| File | Who writes it | On re-run |
| --- | --- | --- |
| `.specsmith/project-map.json` | generated | overwritten |
| `.specsmith/conventions.md` | generated, user corrects | **overwritten** |
| `.specsmith/policies.md` | people — security, architecture | **never touched** |

## The one rule that cannot be broken

**If `.specsmith/policies.md` exists, do not write to it. Do not reformat it, do not
merge into it, do not append a section, do not "helpfully" fill in a blank.** Create it
only when it is absent, and only from
`${CLAUDE_PLUGIN_ROOT}/templates/policies.md.tmpl` — an unmodified stub.

That file holds requirements a security team signed off on. Overwriting it with inferred
content would delete a control and leave a plausible-looking document in its place, and
nobody would notice until an audit. Conventions are cheap to regenerate; policies are not.

Before writing anything, check whether it exists. If it does, say so and leave it alone.

## Phase 1 — Map

Run **level 1** from the `repo-reading` skill. Write `.specsmith/project-map.json`.

## Phase 2 — Conventions

Run **level 2** from the `repo-reading` skill. Sample two or three modules — prefer the
ones with the most recent activity in `git log`, since stale modules teach stale
conventions. Fill in `${CLAUDE_PLUGIN_ROOT}/templates/conventions.md.tmpl` and write
`.specsmith/conventions.md`.

Sources, in order of trust: the code itself where it is consistent, then linter and
formatter configuration, CI pipeline definitions, `CONTRIBUTING.md`, ADRs, PR templates,
and any existing `CLAUDE.md`. **Prefer what the code does over what a document claims.**
Where a document and the code disagree, that disagreement is itself worth recording.

Every rule carries a short real excerpt and the path it came from. A rule with no
example is an opinion, and the next agent to read this file will not be able to apply it.

Where the codebase contradicts itself, record both variants in the Inconsistencies table
rather than picking one. Do not resolve it silently — that is the user's call, and it is
usually the most interesting thing you found.

## Phase 3 — Confirm

Present the conventions draft and ask the user to correct it. **One question per
message**, and only about entries you were genuinely unsure of. Do not walk the document
section by section; the user is correcting a draft, not filling in a form.

Rewrite `conventions.md` after each answer.

## Phase 4 — Hand off the part you cannot do

Create `.specsmith/policies.md` from the stub **only if it does not already exist**.
Then tell the user plainly:

- Policies are not inferred from code, because the code may be what violates them
- Regulatory profiles are opt-in — nothing is enabled until they list one under
  `profiles:` in that file
- `ru/152-fz-pdn` ships with the plugin and applies to any repository touching personal
  data of people in Russia
- The file should be reviewed by whoever owns security or architecture, and committed

## Exit check

- [ ] `project-map.json` written, with the dependency manifest recorded for invalidation
- [ ] `conventions.md` describes observed behavior, every rule with a real excerpt
- [ ] Contradictions in the codebase are recorded as contradictions, not averaged away
- [ ] `policies.md` exists — either created from the untouched stub, or **left exactly
      as it was found**
- [ ] The user has been told which of the two files is theirs to maintain

Everything here is checked into the repository and shared by everyone working in it. Say
so before writing — the user is setting team-wide state, not a personal preference.
