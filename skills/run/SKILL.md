---
name: run
description: The whole pass - setup if the host is fresh, then find, enrich, signals, verify, push. Always works with the public registry, gets significantly better with a CRM and the paid sources wired. Trigger with "source companies in [sector]", "build me a lead list for [target]", "run the pass on [target]".
argument-hint: "<target>"
---

# Run

One pass end to end, without the user at the keyboard. State the goal on the way in: the target's
name, or what is sold to whom.

## Step 0 — The host

`.leadgen/` absent from the host: `skills/setup/SKILL.md`, interactive, then the pass. Present: the
cap and the threshold come from `context.md` and `targets/<t>/index.md`; no question is asked on
the way.

Done when: `.leadgen/` and `targets/<t>/index.md` exist, and the cap and the threshold are read.

## The flow

Each step is its skill, with `$ARGUMENTS`, in this order:

1. `skills/find/SKILL.md`
2. `skills/enrich/SKILL.md`
3. `skills/signals/SKILL.md`
4. `skills/verify/SKILL.md`
5. `skills/enrich/SKILL.md` with `email <the rows verify did not flag>`, under the cap
6. `skills/push/SKILL.md` with the list of keys of the rows of verdict `ok`, score at or above the
   threshold, that verify did not flag

Paying comes after verify, never before. A step that breaks leaves its rows for the fourth block
and the pass carries on.

## The report

Four blocks: **pushed** · **to validate** (the rows verify flagged, one question a line) ·
**notable exclusions** · **what broke**. Then one line in `targets/<t>/index.md` § Passes for the
whole pass.

Done when: the four blocks are rendered and the line is written.
