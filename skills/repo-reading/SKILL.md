---
name: repo-reading
description: How SpecSmith reads an existing codebase — the project map, the conventions sample, and the blast radius. Use when another SpecSmith skill needs repository facts, or when the user asks what a change touches, who calls a symbol, or what this project's conventions are.
---

# Reading an existing repository

Three levels, run **lazily and in order**. Each is more expensive than the last, so no
level runs until something actually needs it. Most of a spec never needs level 3.

Everything SpecSmith writes lives under `.specsmith/` in the target repository:

```
.specsmith/
├── conventions.md          generated, safe to overwrite
├── policies.md             written by people, never overwritten
├── project-map.json        cache
└── features/<slug>/        spec.md · design.md · tasks.md · trace.json
```

---

## Level 1 — Project map

**Cost:** cheap. **Cached:** `.specsmith/project-map.json`.

Establish the shape of the repository, nothing more:

- Language, runtime version, dependency manager, and the manifest file that declares them
- Top-level module or service layout, and what each is for in one line
- Entry points — binaries, HTTP servers, workers, scheduled jobs, CLI
- Where tests live and how they are run
- Where database migrations live, if any
- Build, lint, and CI configuration files

**Invalidation.** Record the dependency manifest's path and modification time in the
cache. If the manifest has changed since, rebuild the map. Otherwise reuse it — do not
re-read the repository to answer a question level 1 already answered.

Write it as JSON so later runs can read it without re-deriving anything:

```json
{
  "generated": "<date>",
  "manifest": { "path": "pyproject.toml", "mtime": "<iso8601>" },
  "language": "python 3.12",
  "package_manager": "uv",
  "modules": [{ "path": "app/api", "role": "HTTP handlers" }],
  "entry_points": [{ "kind": "http", "path": "app/main.py" }],
  "tests": { "path": "tests", "runner": "pytest" },
  "migrations": { "path": "migrations", "tool": "alembic" }
}
```

---

## Level 2 — Conventions by sample

**Cost:** moderate. **Cached:** `.specsmith/conventions.md`.

Do not summarize the whole codebase, and do not describe an ideal. Pick **two or three
modules closest to the work at hand** and extract what they actually do:

- **Errors** — the real shape of raised exceptions and returned error bodies
- **Logging** — logger, format, what carries a correlation ID, what is deliberately not logged
- **Layers** — what calls what, and what is not allowed to call what
- **Naming** — modules, functions, tests, database objects
- **Tests** — levels that exist, what a change is expected to come with, how fixtures work
- **Data** — how migrations are written, whether they are reversible in practice
- **Configuration** — where settings come from, how secrets are handled

Quote a short real example under each rule. A convention with a two-line excerpt from the
repository is usable by an agent; the sentence "errors are handled consistently" is not.

**Where the codebase contradicts itself, say so.** Show both variants and where each
lives. Inconsistency is the most valuable thing this level can find — do not average it
away into a rule that is true nowhere.

Conventions describe **what is**, not what ought to be. Aspirations belong in
`policies.md`, which this level never touches.

---

## Level 3 — Blast radius

**Cost:** expensive. **Never cached** — it is only true for one moment of one codebase.

**Do not run this until there is a hypothesis about where the change lands.** Level 3
answers "what depends on *these* symbols, files, and contracts", not "map the project".
Running it early burns context on modules the change will never reach.

For each identified point of change, establish:

- **Callers** — direct, then transitive one hop out
- **Importers** — modules that would stop compiling or start behaving differently
- **Contract consumers** — HTTP and RPC clients, event subscribers, database readers,
  config keys, CLI invocations, anything crossing a process boundary
- **Data** — tables, columns, and migrations touched
- **Tests** — what currently covers the affected behavior

### Source and degradation

```
MCP code-graph tools available?
├── yes → find_callers / impact query        provenance: "graph"
└── no  → grep the symbol name, then         provenance: "text-search"
          separate definitions from call sites
```

Detect the graph by looking for code-graph MCP tools in the current session. If none are
present, degrade to text search silently — **never require the graph, never fail because
it is absent.**

Text search has known failure modes, and the reader of the design has to know they apply:
it misses calls through import aliases, re-exports, and dynamic dispatch, and it reports
same-named methods on unrelated classes as if they were hits. Sift the results; do not
paste raw grep output into a design.

### Provenance is mandatory

Every blast-radius list carries its source:

```markdown
**Dependency source:** graph (complete within static analysis)
**Dependency source:** text-search (may be incomplete — see limits below)
```

Neither source is a guarantee. In Python, `getattr`, decorators, registries, and runtime
attribute assignment are invisible to both. State this next to the list when the change
touches code that uses them.

---

## What this skill does not do

It does not ask the user questions, and it does not write specs. It is the reading layer
the other skills call. If a fact can be read from the repository, it is read here — no
other skill should be asking the user for it.
