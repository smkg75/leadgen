# 🎯 leadgen

A Claude Code plugin that builds a B2B lead list the way a spreadsheet tool would: one table per
target, one row per company or person, one column per purpose. Each column is filled by a cascade
of sources with a merge rule, a condition and a cost, and every cell carries the value, the date it
was read, the source that gave it and a proof you can check. It works with the public French
company register alone, and gets significantly better with a CRM and the paid sources wired.

## 🔍 What people use it for

- 📞 **Building a call list in a sector.** A founder who sells to one trade picks the activity codes and the geography, and gets a table of companies with an identity, a domain, a phone and a size — before the first call, not after fifty.
- 📰 **Refreshing signals before a batch of emails.** Who is hiring, who just raised, who is running ads, who won a public contract — the perishable columns are re-read the week the emails go out, not the week the list was built.
- 🧭 **Finding who runs each company.** The register's officers first, free and proven, then the databases and the browser, persona by persona, with the mandate that says who actually decides.
- 🧹 **Auditing a list before it reaches the CRM.** A pass that reports what is wrong — a wrong domain, a value with no proof, a duplicate, a company with no person — with the row, the column and the proof, and writes nothing itself.
- 🤖 **Running the whole pass hands-off.** One command, and the list is sourced, enriched, scored, audited and pushed, with a report at the end of what went in and what needs a human.

## 📦 Install

```bash
# from a local checkout
claude plugin marketplace add /path/to/leadgen
# or from the repository
claude plugin marketplace add https://github.com/smkg75/leadgen.git

claude plugin install leadgen@leadgen
```

## 🚀 First run

`/leadgen:setup`, in the repo that will hold the target.

It interviews you: what you sell and to whom, what an attack looks like (a call, an email, both),
the volume you are after, what a pass may spend, who you never want in the list. Then it tests
every tool you named — a CRM by reading its schema, an API by one read call — and writes down what
answered, what failed and why, and which environment variable each one wants (never its value).
Last it lays `.leadgen/` down at the root of the repo and prints the flow, with the next command
for each target.

The public French company register alone is enough to run: identity, officers, headcount bands,
filed accounts, keyless. A CRM and the paid contact sources make it considerably better. Browser
sources need Chrome running with the Claude in Chrome extension active.

## 🧾 Commands

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

## ⚙️ How a pass works

Five steps, each its own command, each relaunchable on its own:

- **find** makes rows. Companies from the register, deduplicated on their SIREN, then the people
  who run them, persona by persona.
- **enrich** makes columns. A domain, a phone, a headcount, a description read off the site, a
  verdict written by a model — free sources first, each column filled by its own cascade.
- **signals** refreshes what perishes. Open roles, funding, live ads, public contracts, and the
  score that ranks the table.
- **verify** reads both tables and reports findings by severity, each with its row, its column and
  its proof. It writes nothing — an audit that lives inside what it audits audits nothing.
- **push** writes the rows you selected into the CRM, and brings the record id back into the table.

Paid columns — an unmasked email, a mobile — run on a named selection only, under a cap you set at
setup. `/leadgen:run` chains the five without you at the keyboard, and pays after verify rather
than before: only the rows the audit did not flag. It ends on four blocks — pushed, to validate,
notable exclusions, what broke — and one line in the target's page.

## 🗂️ The host wiki

```
<repo>/.leadgen/
├── index.md                the map: the flow, one line per target, the link to connections
├── context.md              who sells what to whom, the attack gesture, the volume, the paid cap, the global exclusions
├── connections.md          one block per tool wired: access, variable name, quota, cap, tested on
├── lib/                    optional: code shared by several targets, imported by the column scripts
└── targets/<target>/
    ├── index.md            perimeter, exclusions, personas, signals, threshold, decisions, one line per pass
    ├── columns.md          optional: this target's own column blocks, shadowing the catalogue
    ├── columns/            the code: one script per column, one prompt file per AI column
    └── data/               ignored by git: the two tables, the raw pages
```

## 📚 The catalogues

[`columns.md`](columns.md) is the view by purpose: one block per column, its cascade of sources,
its merge rule, its condition, its cost, and the rules every cell obeys. [`sources.md`](sources.md)
is the view by source: one block per way of reaching data, its access, its limits, its pitfalls,
when it was last exercised. A target adds or masks columns in its own `columns.md`, on the same
shape.

## 🧩 Where the code lives

Not here. The plugin holds the method and the two catalogues; the script of a column and the prompt
of an AI column are written per target, in your own repo, under `targets/<target>/columns/`. A
verdict prompt depends on what you sell, and a script depends on the format the target chose. The
install is a copy replaced on update, so nothing of yours is kept inside it.

## 🛠️ Editing the plugin

The install is a copy, refreshed only when the version changes. `git config core.hooksPath
.githooks` once in the checkout: every commit then bumps the patch version, and `claude plugin
update leadgen@leadgen` picks the new copy up.

[`DECISIONS.md`](DECISIONS.md) holds the arbitrages the method rests on, each with the reason that
settled it, and what is left undone on purpose. Read the entry before changing what it settles.

## ⚖️ Terms of use

Some sources listed in `sources.md` forbid automated extraction in their terms, and the catalogue
says which. The plugin runs those on a named shortlist only, in the user's own browser, and asks
for the risk to be accepted before the first pass. Whether to accept it is the user's decision,
not the plugin's.

## 📄 License

MIT
