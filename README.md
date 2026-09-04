# leadgen

A Claude Code plugin that builds a B2B lead list the way a spreadsheet tool would: one table per
target, one row per company or person, one column per purpose, each column filled by a cascade of
sources with a merge rule, a condition and a cost. The plugin holds the method and two catalogues,
the columns and the sources; the code and the prompts are written per target, in the host repo.

Always works with the public French company register alone. Gets significantly better with a CRM
and the paid sources wired.

## Commands

| Command | Argument | Produces |
|---|---|---|
| `/leadgen:help` | | the flow, and where this host stands; writes nothing |
| `/leadgen:setup` | nothing, `grill`, `connections` | `.leadgen/` at the host root: the context, the connections tested, one page per target |
| `/leadgen:find` | `<target>` | `find-companies`, then `find-people` |
| `/leadgen:find-companies` | `<target>` | the rows of the companies table, with their identity |
| `/leadgen:find-people` | `<target>` | the rows of the people table, by persona, officers first |
| `/leadgen:enrich` | `<target>`, then `<column>`, `email <selection>`, `mobile <person>` or `new <name>` | the columns, free first; the paid ones on a named selection |
| `/leadgen:signals` | `<target>`, a selection | the signal columns and the score, refreshed |
| `/leadgen:verify` | `<target>` | findings by severity; nothing written in the tables |
| `/leadgen:push` | `<target>`, a selection | the rows in the CRM, `crm_id` written back |
| `/leadgen:run` | `<target>` | the whole pass, and a four-block report |

## Install

```bash
# from a local checkout
claude plugin marketplace add /path/to/leadgen
# or from the repository
claude plugin marketplace add https://github.com/smkg75/leadgen.git

claude plugin install leadgen@leadgen
```

Then `/leadgen:setup` in the repo that will hold the target. It lays `.leadgen/` at the root and
interviews you.

Browser sources need Chrome running with the Claude in Chrome extension active. Paid sources need
their key in the environment; `setup` says which variable.

## The host wiki

```
<repo>/.leadgen/
├── index.md                the map: the flow, one line per target, the link to connections
├── context.md              who sells what to whom, the attack gesture, the volume, the paid cap, the global exclusions
├── connections.md          one block per tool wired: access, variable name, quota, cap, tested on
└── targets/<target>/
    ├── index.md            perimeter, exclusions, personas, signals, threshold, decisions, one line per pass
    ├── columns.md          optional: this target's own column blocks, shadowing the catalogue
    ├── columns/            the code: one script per column, one prompt file per AI column
    └── data/               ignored by git: the two tables, the raw pages
```

## The catalogues

[`columns.md`](columns.md) is the view by purpose: one block per column, its cascade of sources,
its merge rule, its condition, its cost, and the rules every cell obeys. [`sources.md`](sources.md)
is the view by source: one block per way of reaching data, its access, its limits, its pitfalls,
when it was last exercised. A target adds or masks columns in its own `columns.md`, on the same
shape.

## Editing the plugin

The install is a copy, refreshed only when the version changes. `git config core.hooksPath
.githooks` once in the checkout: every commit then bumps the patch version, and `claude plugin
update leadgen@leadgen` picks the new copy up.

[`DECISIONS.md`](DECISIONS.md) holds the arbitrages the method rests on, each with the reason that
settled it, and what is left undone on purpose. Read the entry before changing what it settles.

## Terms of use

Some sources listed in `sources.md` forbid automated extraction in their terms, and the catalogue
says which. The plugin runs those on a named shortlist only, in the user's own browser, and asks
for the risk to be accepted before the first pass. Whether to accept it is the user's decision,
not the plugin's.

## License

MIT
