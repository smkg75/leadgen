---
name: signals
description: After enrich, before every push. Refreshes the perishable columns and the score for one target. Trigger with "refresh the signals on [target]", "who is hiring in [target]", "rescore [target]".
argument-hint: "<target> [lot <n> | score-min <n> | all]"
---

# Signals

The `signal` columns and the `score`, relaunched before every push: a signal orders the attack, it
never qualifies, and it is perishable.

`$ARGUMENTS` = the target, then the selection: a lot, a score floor, or `all`. No selection: the
rows with verdict `ok`.

## Step 0 — Load

Walk up from the working directory to `.leadgen/`. Absent — stop on "Run `/leadgen:setup` first".
Read `targets/<t>/index.md`. No signals defined there: grill, one question a turn with a
recommended answer (which event counts, on which window, with which weight), and write them into
the page before going on. Read the `signal` and `score` blocks of `columns.md` and the shadow, the
source blocks they name, `.leadgen/connections.md`.

Done when: every signal has a window and a weight on the target page, and the selection is
resolved to a list of keys.

## Step 1 — The signal columns

Each signal column has its script, `targets/<t>/columns/<column>.<ext>`, written on its block when
missing. Run them in the catalogue's order on the selection. A source whose terms forbid
extraction, or a browser source, runs on the named selection only, never on the whole table. Each
fact is appended, dated by the event, with its proof; a per-company rank writes its empty witness,
a per-query rank writes nothing on a company it did not name.

Done when: every signal column has run on the selection or names its stop.

## Step 2 — The score

Recalculate `score` entirely from the view: each signal in its window weighs what the target page
says. Score 0 sits at the bottom of the list, never outside it.

Done when: every row in the selection has a score, and the distribution is printed.

## Step 3 — Count and record

Per signal column: facts appended, companies with at least one fact, witnesses. The score
distribution against the previous pass: a collapse is a bug, not a fact. Append one line to
`targets/<t>/index.md` § Passes.

Done when: the counters are compared and the line is written.
