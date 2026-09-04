---
name: help
description: The flow of the ten commands, what each produces and which ones pay, and where this host stands.
disable-model-invocation: true
---

# Help

Prints the flow, then where the host stands. Writes nothing.

## Step 1 — The flow

Print this, as is:

```
setup           once per host; then `grill` or `connections` to redo one part          free
find            find-companies, then find-people: the rows of one target              free
enrich          the columns, free and quota first; `email <selection>` and             pays on request
                `mobile <person>` are the only way to pay
signals         the perishable columns and the score, before every push               free, quota
verify          findings on the tables, writes nothing in them                        free
push            the selected rows into the CRM                                        free
run             setup if the host is fresh, then find → enrich → signals → verify      pays after verify,
                → enrich email → push                                                 under the cap
```

Done when: the seven lines are printed.

## Step 2 — Where the host stands

Walk up from the working directory to the first `.leadgen/index.md`. None: say the host is fresh
and that `/leadgen:setup` comes first; stop.

Found: read `.leadgen/index.md`, `.leadgen/connections.md` and every `targets/<t>/index.md`.
Print, per target, its name, the date of its last pass and the next command in the flow; then the
tools of `connections.md`, each tested with its date, struck through with its reason, or never
tested.

Done when: every target and every tool has its line, and no file was written.
