---
name: verify
description: Before push. Audits the tables of one target and returns findings, writes nothing in them. Trigger with "verify [target]", "audit the list before I push".
argument-hint: "<target>"
---

# Verify

Reads the tables and the catalogue, comes back with findings ranked by severity, each with its
row, its column, its proof and a suggested decision. Writes nothing in the tables: the human
decides, the session applies. An audit that lives inside what it audits audits nothing.

## Step 0 — Load

Walk up from the working directory to `.leadgen/`. Absent — stop on "Run `/leadgen:setup` first".
Read `targets/<t>/index.md` (windows, thresholds, the counters of the previous pass, and its
§ Verify when one exists: each line there is an extra family for this target, checked like the
seven below), `columns.md` and the shadow, and the latest-value view of both tables.

Done when: the view, the previous counters and the target's own families are in hand.

## Step 1 — The families, in parallel

One sub-agent per family, dispatched in parallel, through the stripped call of `columns.md` § AI
columns, on the model tier of class `judgment` named on the call. Each receives the view, the blocks it needs and its family, and returns
facts: row, column, what was observed, the proof, `null` where the data does not say. The verdict
on each finding is rendered here, not in the sub-agent.

The families:

1. **Domain** — suspect, or proven by the name alone, or without proof.
2. **Verdict** — inconsistent with the description, or with the flags.
3. **Duplicate** — one domain carried by two SIREN, neither marked.
4. **People** — a company with no person; a board mandate counted as a contact.
5. **Cell** — a value without a date, or without a proof.
6. **Window** — a signal outside its window still weighing in the score.
7. **Counter** — a column whose count collapsed against the previous pass.

Done when: the seven families have returned, or the one that broke is named.

## Step 2 — The findings

Rank by severity: what would make a call to the wrong person or the wrong company first, what
mis-orders the attack second, what mis-counts third. Each finding: severity, row, column, the
proof, the decision suggested, and ❓ where the data does not settle it ("unknown, ask in
discovery").

Done when: every finding has its five fields, and the list is rendered most severe first.

## Step 3 — Record

The only write: one line in `targets/<t>/index.md` § Passes, the date, `verify`, the count per
family.

Done when: the line is written and nothing else has changed on disk.
