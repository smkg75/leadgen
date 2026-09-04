---
name: push
description: Last step. Writes the selected rows of one target to the CRM. Trigger with "push lot 2 to the CRM", "push the ok rows of [target]".
argument-hint: "<target> [lot <n> | score-min <n> | ok | <keys>]"
---

# Push

Writes the CRM: companies, then people, then the links. The CRM is a destination; its access lives
in the host, never in the catalogue.

`$ARGUMENTS` = the target, then the selection: a lot, a score floor, a list of keys, or `ok` (every
row with verdict `ok` and no `crm_id`).

## Step 0 — Load

Walk up from the working directory to `.leadgen/`. Absent — stop on "Run `/leadgen:setup` first".
Read `.leadgen/connections.md` § CRM: access, schema, the mapping column → field. No such block, or
struck through — stop on "Run `/leadgen:setup connections`". Read `targets/<t>/index.md` and the
latest-value view.

Done when: the CRM's access and mapping are in hand.

## Step 1 — The selection

Resolve the selection to rows: verdict `ok`, the score floor when one is given, no `crm_id` yet,
and every person attached to a selected company. Say the count, companies and people, before the
first write.

Done when: the count is printed.

## Step 2 — Write

At the CRM's rate: create the companies, read their ids back; create the people, read their ids
back; write the links. Every id read back lands in `crm_id`, dated by the push, with the record's
URL as proof. A row the CRM refuses is named and the pass carries on.

Done when: every selected row has its `crm_id` or is named as refused.

## Step 3 — Reconcile and record

Count in the CRM what was just written; the count equals the selection minus the refusals, or the
difference is named row by row. Append one line to `targets/<t>/index.md` § Passes: date, `push`,
companies and people written, refusals.

Done when: the two counts agree or the difference is listed, and the line is written.
