---
name: setup
description: First step, once per host. Grills the user on what they sell and to whom, tests every connection, lays the host wiki down. Always works with the public registry alone, gets significantly better with a CRM and the paid sources wired. Trigger with "set up leadgen", "add a target", "retest the CRM".
argument-hint: "nothing for everything, 'grill' for the interview alone, 'connections' to retest the tools alone"
---

# Setup

Lays `.leadgen/` at the root of the host repo and fills it with facts. The plugin carries the
method; the host carries every fact about the user, their targets and their tools.

`$ARGUMENTS` = `grill` runs Step 0 then Step 1 alone; `connections` runs Step 0 then Step 2 alone;
nothing runs everything.

Each step ends on its own criterion. The next one starts after it.

## Step 0 — The host wiki

The host root is the git root of the working directory, else the working directory. Create
`.leadgen/` there when it is missing:

```
.leadgen/
├── index.md            the map: its first line says this folder is a wiki (short interlinked pages, one fact one page, this file the only entry); the flow of the ten commands; one line per target with the date of its last pass; the link to connections.md; no business fact
├── context.md          the facts: who sells what to whom, the attack gesture, the volume aimed at, the paid cap, the global exclusions
├── connections.md      one block per tool wired: access, the name of the environment variable and whether it is set (never its value), quota, paid cap, tested on
├── .gitignore          targets/*/data/
└── targets/<target>/
    ├── index.md        perimeter, exclusions, personas, signals with window and weight, score threshold, data format, dated decisions, § Passes with one line per pass
    ├── columns.md      optional: blocks added or masked for this target, on the catalogue's shape
    ├── columns/        the code: one script per column, and the prompt file of every ai column
    └── data/           ignored by git: the two tables, and raw/<siren>/pages.md
```

Already there: read `context.md`, `connections.md` and every target page to see which hold facts
rather than skeleton, and run only the steps still empty. Everything filled: say so, print the
flow, stop.

Done when: `.leadgen/` exists at the host root with `index.md`, `context.md`, `connections.md`,
`.gitignore` and `targets/`, and the steps to run are named.

## Step 1 — Grill

Invoke the `grilling` skill when it is installed. Otherwise one question a turn, each with a
recommended answer, facts looked up rather than asked (the register gives the row count of an
activity code: ask which code, not how many rows). No question is written in advance: the facts
below are what the interview must establish, in whatever order the conversation takes.

Write each answer where it lands the moment it is given. A fact the user cannot supply is written
`?` in its place.

Done when the facts are written, not when the questions run out:

- `context.md` carries the offer, the buyers, the attack gesture (call, email, both), the volume
  aimed at, the paid cap per pass, the global exclusions;
- every `targets/<t>/index.md` carries the perimeter (activity codes with their counted noise,
  geography, size), the exclusions with the rule that applies them, the personas ranked (the chief
  executive first unless the user says otherwise), the signals that count with a window and a
  weight each, the score threshold `push` and `run` use, the data format chosen, and the terms
  risk accepted or declined for the sources whose terms forbid extraction;
- no script was written.

## Step 2 — Connections

For every tool the interview named, CRM, MCP, API, CLI or browser: one read call. A CRM is tested
by reading its schema, and the mapping column → field is written under its block. Each block of
`connections.md`: access, the variable's name and set or not, quota, paid cap, tested on today's
date with what was exercised. A tool that fails is struck through with the reason, never left
looking untested.

Done when: every tool named is tested and dated, or struck through with its reason, and no value
of any key appears in the file.

## Step 3 — Close

Print the flow of the ten commands and write it into `.leadgen/index.md`, with one line per target
and the link to `connections.md`. Say what the user still owes: a `?`, a key to set, a tool to
wire.

Done when: `index.md` reads as the map, and the next command for each target is named.
