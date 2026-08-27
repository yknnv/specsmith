# SpecSmith

Spec-driven development for codebases that already exist.

Most SDD tooling assumes a greenfield project: describe what you want, get a spec,
generate code. Real work is rarely that. You have sixty services, consumers you did not
write, and a change that has to ship without breaking any of them. SpecSmith specifies
**a change to a running system** — what it touches, who depends on it, what breaks, and
how you get back.

The premise is small: a spec takes five minutes to fix. Code the agent has already
written does not.

## Three things that make it different

**Brownfield first.** Conventions are read out of your code, not imposed on it.
`/specsmith:init` samples the modules closest to your work and writes down how errors are
actually handled here, how logging is actually done, where the layers actually sit —
including where the codebase contradicts itself. The interview then asks only what the
repository cannot answer. Six questions, not a forty-field form.

**Conventions and policies are separate files.** `conventions.md` is generated and
overwritten freely. `policies.md` is written by people — security, architecture — and
**never touched by the tool.** Mixing them means one regeneration silently deletes a
security control and leaves a plausible-looking document behind.

**Regulatory profiles.** `policies/ru/152-fz-pdn.md` ships with the plugin: seven rules
on personal data under Russian law, each as a trigger the spec phase can match, a
requirement, and the facts a spec must therefore state. Opt-in, nothing enabled by
default. Only public regulations ship here.

SpecSmith **routes, it does not certify.** The output is "this change touches PDN-03,
show it to security" — never "complies with 152-ФЗ". A false sense of coverage is worse
than no check at all, because it stops someone from looking.

## Install

```
/plugin marketplace add yknnv/specsmith
/plugin install specsmith@specsmith
```

## Use

```
/specsmith:init                  # once per repo — infers conventions, stubs policies
/specsmith:spec "<change>"       # interview → spec.md + policies touched
/specsmith:design                # blast radius, contracts, rollback → design.md
/specsmith:tasks                 # ordered, deployable at every step → tasks.md
```

After `init`, edit `.specsmith/policies.md` by hand, enable any profiles you need, and
commit it. That file is the part a tool cannot write for you.

Each phase writes to disk after every answer, so an interview survives an interrupted
session. Run `/specsmith:spec` again on the same change and it resumes from the first
unanswered question.

## Blast radius, and how much to trust it

`/specsmith:design` establishes what depends on the code you are changing: callers,
importers, contract consumers across process boundaries, data, covering tests. It runs
against the points the spec identified — not the whole repository.

If code-graph MCP tools are present, it uses them. If not, it degrades to text search.
**It never requires the graph and never fails because the graph is absent.** Which source
answered is written into the design:

```
Dependency source: graph (complete within static analysis)
Dependency source: text-search (may be incomplete — misses import aliases,
                   re-exports, and dynamic dispatch)
```

Neither is a guarantee. In Python, `getattr`, decorators, registries, and runtime
attribute assignment are invisible to both. A graph gives a fast and nearly complete
answer; it does not give you certainty, and the design says which one you are holding.

## Layout

```
.specsmith/
├── conventions.md              generated — how this codebase is actually written
├── policies.md                 yours — never overwritten
├── project-map.json            cache
└── features/<slug>/
    ├── spec.md                 what changes and why
    ├── design.md               how, what breaks, how to get back
    ├── tasks.md                ordered, deployable at every step
    └── trace.json              which files a task really touched
```

## What the artifacts are for

A spec that restates the code is dead weight — it rots in a sprint and takes the team's
trust with it. SpecSmith writes down what the code cannot tell you:

- **Non-goals** — the adjacent things this change deliberately does not do
- **Failure behavior** — what happens when each dependency is slow, down, or wrong
- **Contract impact** — who breaks, and how they migrate
- **Acceptance criteria** — observable facts, verifiable by someone who did not build it
- **Rejected alternatives** — so the debate does not restart in three months

## Status

v0.1 — `init` through `tasks`, with the 152-ФЗ profile and text-search blast radius.

Next, in order: measurements on real changes ([docs/measurement.md](docs/measurement.md)),
optional code-graph support, more regulatory profiles, and `/specsmith:drift` — comparing
a spec against what the code actually does now, using `trace.json`.

MIT.
