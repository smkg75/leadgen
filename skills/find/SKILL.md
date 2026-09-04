---
name: find
description: After setup. Seeds the companies and people tables for one target. Trigger with "source [target]", "build the list for [target]", "seed both tables for [target]".
argument-hint: "<target>"
---

# Find

`find` makes rows. Two seeds, each its own command, in this order:

1. `skills/find-companies/SKILL.md` with `$ARGUMENTS`.
2. `skills/find-people/SKILL.md` with `$ARGUMENTS`, once the first has returned.

State the goal on the way in: a target name is more precise than a sector.

Done when: both have returned with their counters compared with the previous pass.
