---
name: find-people
description: After find-companies. Creates the rows of the people table by persona cascade, officers first, for one target. Trigger with "who runs [company]", "find the decision makers of [target]".
argument-hint: "<target>"
---

# Find people

The `people` seed: one cascade per persona, the officers at rank 1, the databases and the browser
after. Assumes the companies table is seeded.

## Step 0 — Load

Walk up from the working directory to `.leadgen/`. Absent — stop on "Run `/leadgen:setup` first".
Read `targets/<t>/index.md`: no persona there — stop on "Run `/leadgen:setup grill` for this
target: a persona is a fact of the target". An empty companies table — stop on "Run
`/leadgen:find-companies <t>` first".

Read the `## people` block of `columns.md` and its shadow, the blocks of the sources it names, the
cap per company and the counters of the previous pass.

Done when: the personas, ranked, and the cap are in hand.

## Step 1 — The script

`targets/<t>/columns/people.<ext>`. Missing: write it on the block and the source blocks. Present:
read it first.

Done when: the script exists and holds the contract.

## Step 2 — The cascade, per persona

For every company whose verdict is not excluded, for every persona in rank order, run the cascade
of the `people` block until the persona is matched or the cascade is exhausted. The block says what
each rank reads, what is written when nobody is found, and how rows are deduped and capped.

Done when: every company in scope has been through every persona, or the stop is named.

## Step 3 — Count and record

People created, per persona and per rank; companies with no person at all, as a share of the
table; board mandates counted as leads. Compare with the previous pass: a share of companies
without a person that grows, or a count that collapses, is a bug.

Append one line to `targets/<t>/index.md` § Passes.

Done when: the counters are compared and the line is written.
