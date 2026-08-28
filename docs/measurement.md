# Measurement protocol

The point of v0.1 is not the plugin. It is the answer to one question: **does a written
spec beat a good prompt on real brownfield work, and by how much?**

If the numbers say no, that is the finding, and it gets published as the finding. A
protocol you can only pass is not a measurement.

## Setup

- **Repository:** one real codebase, brownfield, with existing consumers.
- **Changes:** four, chosen from work that was going to happen anyway. **Not invented for
  the experiment** — invented tasks have no consumers to break, which is precisely the
  variable under test.
- **Split:** two through the full Brownspec cycle (`init` → `spec` → `design` → `tasks` →
  implementation), two through an ordinary prompt to the same agent.
- **Pairing:** the four changes should be comparable in size. Note it when they are not.
- **Blast radius:** text-search only in this phase. **Do not enable a code graph** — the
  grep run is the baseline the graph will later be measured against.

## Metrics

| Metric | How to count |
| --- | --- |
| Iterations to a working result | Rounds of "no, redo that" until the change works as intended |
| Unrequested breakage | Times the agent changed something it was not asked to change — count each, including reverted ones |
| Time to merge | Wall-clock hours from first message to merged, excluding time not spent on the task |
| Tokens per change | From session statistics, summed across all sessions for that change |

Count unrequested breakage even when it was caught in review. Catching it is the cost
being measured.

## Recording

One row per change. Fill it in as the work happens, not from memory afterwards.

| # | Change | Track | Iterations | Breakage | Hours | Tokens |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | | brownspec | | | | |
| 2 | | brownspec | | | | |
| 3 | | prompt | | | | |
| 4 | | prompt | | | | |

Alongside the table, keep notes on things the numbers will not show:

- Questions the spec phase asked that turned out to matter
- Questions it asked that were a waste of time
- Anything the spec missed that the blast radius would have caught
- Where text-search gave an incomplete answer, and whether it cost anything

## Honesty rules

- Four changes is an anecdote, not a study. Say so in any writeup, in the first paragraph.
- The same person did both tracks knowing the hypothesis. That is bias, and it is not
  removable at this scale — name it rather than hoping nobody asks.
- Publish the failures. A spec phase that asked six useless questions is a more useful
  result than a clean table.
