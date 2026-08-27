# SpecSmith — spec-driven development for existing codebases

**A Claude Code plugin.** You already write specs. Your tooling assumes the code does not
exist yet.

Spec-driven development has converged on three artifacts — requirements, design, tasks —
and it converged on them for good reasons. What the tooling assumes is a blank
repository: describe what you want, get a spec, generate code. Then you open the actual
repository, and there are sixty services, consumers you did not write, and a change that
has to ship without breaking any of them.

SpecSmith specifies **a change to a running system.** The same three artifacts. What
happens around them is different: conventions are read out of your code instead of
declared, the design carries the blast radius of the change, and your organization's
rules live in a file the tool is forbidden to overwrite.

The premise is small: a spec takes five minutes to fix. Code the agent has already
written does not.

## How it compares

| | Greenfield assumed | Reads your conventions | Blast radius | Regulatory profiles |
| --- | --- | --- | --- | --- |
| GitHub Spec Kit | yes | no | no | no |
| Kiro spec mode | partly | no | no | no |
| SpecSmith | no | yes | yes, with provenance | yes, opt-in |

If you are starting a new project, use one of the others — SpecSmith's entire first phase
is reading code that does not exist yet. It earns its place when the code is already
there, has consumers, and cannot break.

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

## What the extra phases buy you

**Conventions are inferred, so the interview gets shorter.** `/specsmith:init` samples the
modules closest to your work and writes down how errors are actually handled here, how
logging is actually done, where the layers actually sit — including where the codebase
contradicts itself. The interview then asks only what the repository cannot answer. Six
questions, not a forty-field form.

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

## FAQ

**Does this write code?** No. It stops at `tasks.md` and hands the decision back to you.
Implementation is a separate step you take with the agent of your choice.

**Does it work without a code graph?** Yes. Blast radius degrades to text search and says
so in the design. The plugin never requires a graph and never fails because one is absent.

**Is it Python-only?** No. The reading layer is language-agnostic; the caveats about
dynamic dispatch are written with Python in mind because that is where they bite hardest.

**Does the 152-ФЗ profile make my project compliant?** No, and it is built specifically
not to claim that. It routes a spec to the rules it touches so a human reviews it. See
[the profile](policies/ru/152-fz-pdn.md).

**Does it send my code anywhere?** No. It is Markdown instructions for an agent you are
already running. There is no service, no telemetry, and no network call of its own.

## Русское описание

**SpecSmith — плагин для Claude Code, который заставляет агента сначала договориться,
а потом писать код.** Рассчитан на brownfield: не на новый проект с нуля, а на изменение
в системе, у которой уже есть работающее поведение, контракты и потребители.

Отличий от Spec Kit, Kiro и Tessl три:

- **Конвенции выводятся из вашего кода**, а не задаются с нуля — как здесь на самом деле
  обрабатываются ошибки, как логируют, где лежат слои, включая места, где кодовая база
  противоречит сама себе
- **Радиус поражения** — кто вызывает, что импортирует, кто потребляет контракт — с явной
  пометкой, откуда получен ответ: граф кода или текстовый поиск
- **Профили нормативов РФ** — [152-ФЗ о персональных данных](policies/ru/152-fz-pdn.md),
  семь правил в формате «триггер → требование → что должно быть в спеке». Подключается
  явно, по умолчанию ничего не включено

Инструмент **маршрутизирует, а не выносит вердикт**: на выходе «это изменение затрагивает
PDN-03, покажите ИБ», и никогда «соответствует 152-ФЗ». Ложное чувство защищённости
опаснее отсутствия проверки — оно останавливает того, кто иначе пошёл бы проверять.

## Status

v0.1 — `init` through `tasks`, with the 152-ФЗ profile and text-search blast radius.

Next, in order: measurements on real changes ([docs/measurement.md](docs/measurement.md)),
optional code-graph support, more regulatory profiles, and `/specsmith:drift` — comparing
a spec against what the code actually does now, using `trace.json`.

MIT — see [LICENSE](LICENSE).
