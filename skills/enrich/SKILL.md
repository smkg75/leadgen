---
name: enrich
description: After find. Fills the columns in cascade, free first, paid on a named selection. Trigger with "enrich [target]", "get the emails of lot 3", "add a column that says [purpose]".
argument-hint: "<target> [<column> | email <selection> | mobile <person> | new <name>]"
---

# Enrich

`enrich` makes columns. No connectors? No problem: the free ranks of every cascade run on the
public register and the sites alone, and the paid ranks wait for a named selection.

`$ARGUMENTS`:

- `<target>` alone: every column of the catalogue not yet filled, on the rows that pass its
  `only if`, free and quota ranks only;
- `<target> <column>`: that column alone, same rule;
- `<target> email <selection>` and `<target> mobile <person>`: the only way to pay;
- `<target> new <name>`: a new column for this target.

## Step 0 — Load

Walk up from the working directory to `.leadgen/`. Absent — stop on "Run `/leadgen:setup` first".
Read `targets/<t>/index.md`, `.leadgen/context.md` (the paid cap), `columns.md` whole and the
target's shadow, `sources.md` for every source the columns in scope name, and
`.leadgen/connections.md` for what is wired.

Done when: the columns in scope are listed with, for each, the rows eligible (they pass `only if`
and have no value in the latest view).

## Step 1 — The plan

Print the plan before running anything: one line per column, its type, its cost, the rows
eligible. A `paid` column outside an explicit `email` or `mobile` request is skipped with one
line: the rows eligible and the command that would run it.

Done when: the plan is printed and no paid work is in it unless asked for.

## Step 2 — The scripts

For every column in the plan, `targets/<t>/columns/<column>.<ext>`. Missing: write it on its block
and the source blocks of its cascade, with the resume witness, the exit codes and the counters. An
`ai` column also needs its prompt file `targets/<t>/columns/<column>.md`: missing, write it from
the block's `reads`, `writes` and `class`, with `.leadgen/context.md` inlined, on the stripped call
of `columns.md` § AI columns, the model tier named on the call.

Done when: every column in the plan has its script, and every ai column its prompt.

## Step 3 — Run, cascade by cascade

Run the columns in the catalogue's order: attributes before ai columns, `description` before
`one_liner` and `verdict`. Each script writes dated facts with source and proof, the empty witness
where a rank found nothing, and stops clean on a quota, a missing key or a captcha, naming which.
The next column runs regardless.

Done when: every column in the plan has run or names its stop.

## Step 4 — Paid, on request only

`email <selection>` or `mobile <person>` is the only way to pay, and nothing is bought before the
user says go. The cap of `context.md` is 0 unless the user raised it: it bounds `run`, not this
step, which asks instead.

1. Resolve the selection to people: a lot, a score floor, a list of keys, or a count ("the top
   50" takes the eligible people by score). No selection given: ask how many people.
2. Price it: people × the credits a hit costs at the first rank of the cascade (`sources.md`
   § limits), in credits and in euros at the rate `connections.md` records for the provider; the
   second rank runs on the misses only and is quoted as a ceiling. Read the remaining balance from
   the provider when its API serves it; else say it is unknown.
3. Print the estimate, people · credits · euros · balance before and after, and stop.
4. On go: the script keeps the batch in flight on disk and pays once per person; `first` merge, so
   a second run buys nothing already bought.

Done when: the estimate was printed and answered before the first paid call; the count announced
equals the count bought plus the count found empty; credits spent and the balance after are
written in the pass line.

## Step 5 — A new column

`new <name>`: four questions, one at a time, each with a recommended answer: the purpose (what a
reader does with it), the shape of the value, the sources in cascade order, the merge rule. Write
the block into `targets/<t>/columns.md` on the catalogue's shape. No script is written here; the
next `enrich <name>` writes it.

Done when: the block is written and reads on the catalogue's shape.

## Step 6 — Count and record

Per column: rows written, empty witnesses, stops. Compare with the previous pass: a count that
collapses is a bug, not a fact. Append one line to `targets/<t>/index.md` § Passes.

Done when: the counters are compared and the line is written.
