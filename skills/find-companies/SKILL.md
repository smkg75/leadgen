---
name: find-companies
description: After setup. Creates the rows of the companies table for one target from the seed cascade, writes their identity, dedups on SIREN. Trigger with "find companies in [sector]", "extend the companies of [target]".
argument-hint: "<target>"
---

# Find companies

The `companies` seed: rows, and the ten or so identity columns one source gives with them.
Nothing else is filled here; `enrich` makes columns.

## Step 0 — Load

Walk up from the working directory to `.leadgen/`. Absent — stop on "Run `/leadgen:setup` first".
`$ARGUMENTS` names the target; no `targets/<t>/index.md` — stop on "Run `/leadgen:setup grill`
for this target".

Read `targets/<t>/index.md` (perimeter, exclusions, data format, the counters of the previous
pass), `.leadgen/context.md` (global exclusions), the `## companies` block of `columns.md` and its
shadow in `targets/<t>/columns.md` when one exists, and the block of each source the cascade names
in `sources.md`.

Done when: the perimeter, the exclusions, the data format and the previous counters are in hand.

## Step 1 — The script

The seed's script is `targets/<t>/columns/companies.<ext>`. Missing: write it on the block's
contract and the source blocks, in the language and the format the target page names, with the
resume witness, the distinct exit codes and the end-of-pass counters every column script holds.
Present: read it before running it.

Done when: the script exists and holds the contract.

## Step 2 — The cascade

Run the cascade in the block's order. Before extracting an activity code, count its rows and
compare with the perimeter's intent: a code that returns ten times the expected volume is noise,
and the count is written in `targets/<t>/index.md` next to the decision to keep or drop it.

Each row lands with its identity columns as dated facts, source and proof. A SIREN already in the
table is skipped: the first source that named it owns the row.

A quota or a closed route stops the script clean; the pass continues on the next rank and names
the stop in the report.

Done when: every rank of the cascade has run or is named in the report, and every row carries its
identity columns.

## Step 3 — Count and record

Rows created, rows per source, rows per code, rows refused by the perimeter. Compare with the
previous pass on the target page: a number that collapses is a bug, not a fact, and the pass says
so before anything else.

Append one line to `targets/<t>/index.md` § Passes: date, `find-companies`, the counters.

Done when: the counters are compared and the line is written.
