# Decisions

Why the method is shaped the way it is. Each entry is an arbitrage that was settled once, with the
reason that settled it, so it is not re-litigated or silently undone.

What is deliberately left undone sits at the end, under [Deferred](#deferred).

**Writing an entry.** The reason is what was observed, never where it was observed. No client, no
target company, no person, no record URL: the repository is public, and a line that says whose
list a pass ran on tells a reader something about the author rather than something about the
method. What is worth keeping from a real exercise is the date and what was exercised.

## The plugin

**The plugin holds the method only** · 2026-09-04
The repository carries the two catalogues and ten commands. The code of a column and the prompt of
an AI column are written per target, in the host's `targets/<t>/columns/`: a verdict prompt depends
on what the user sells, and a script depends on the format the target chose. The install is a copy
replaced on update, so anything of the user's kept inside would be destroyed by the next one.

**Two flat catalogues, no `shared/`, no `agents/`, no per-source files** · 2026-09-04
`columns.md` and `sources.md` at the root, each with its transverse rules at its head, next to what
they govern. What a command reads is capped per block, not per file, so splitting the sources into
a folder bought nothing and cost an index. `shared/` scattered rules away from their subject;
`agents/` would have frozen a prompt that belongs to the target; `scripts/` contradicted "no code
in the plugin"; a `SKILL.md` at the plugin root is not discovered by the harness.

**English throughout; kebab-case sources, snake_case columns** · 2026-09-04
Same line as the author's two other public plugins. A source name is a heading a column cites to
the letter; a column name becomes a field name and a script name, which is what settles the case.

**Column types are named by the nature of the value** · 2026-09-04
`seed · attribute · ai · reveal · signal · derived · export`. Three sets were compared. A type named
after a means reads twice, once for the means and once for what it produces; `ai` is the exception
because it is the market's word. `fact` was refused because it is already the journal's unit and a
signal is a fact too. `reveal` is the verb the contact vendors use for unmasking a paid email or
mobile.

**`help` is the only command not model-invocable** · 2026-09-04
A model reads the frontmatters; a human does not. `help` prints the flow and where the host
stands, for the human, and exists on the author's request against a first reading that held the
descriptions already did the job.

## The commands

**`find` makes rows, `enrich` makes columns** · 2026-09-04
The only rule that fits one sentence, and `find` is measurable by it: about ten identity columns
from one source, nothing else. A command exists when it can be relaunched alone, when it costs, or
when it waits for a human; it names an intention, never a source. Seventeen commands of an earlier
draft folded into ten on that rule: import into `find`; waterfall, formula, verify-as-a-column and
dedupe into `enrich`, because those are types of column; watch into `signals`; write into `push`;
notify, schedule, template and audit out.

**`enrich` keeps the vendor meaning** · 2026-09-04
The market says enrich for everything, from a free attribute to a paid mobile. Paid work is set
apart by the named selection and the cap, not by a verb: `reveal`, `qualify`, `fill` and `research`
were tried as command names and dropped.

**`run` pays after `verify`, never before** · 2026-09-04
The cap and the threshold are set at setup; `run` applies a decision taken earlier and pays only
for the rows `verify` did not flag. This reverses the skill it replaces, which forbade chaining
anything paid: there the selection was named by hand every time, here it is named once, at setup,
and audited before the money goes.

**The paid cap defaults to zero, and a paid request by hand is gated by an estimate** · 2026-09-04
`run` applies the cap without asking, so the only safe default is that it never pays: the host
raises the cap on purpose or `run` skips the paid step and reports the count. When the user asks
for emails or mobiles by hand, the command resolves the people, prices them in credits and euros,
reads the remaining balance when the provider serves it, prints the estimate and waits for go.
What a person needs before paying per contact is the count, the price and what is left.

**`verify` reports, never writes** · 2026-09-04
An audit that lives inside what it audits audits nothing. `verify` returns findings with row,
column, proof and a suggested decision, ranked by severity; the human decides, the session applies.
Its brief lives in its own skill, dispatched to parallel sub-agents by family, not in `find` (there
is nothing to audit after a seed) and not in `enrich`. The name is `verify`, on the author's choice
over `review`; a blind-verdict review with an agreement measure is deferred.

**`find-people` is a persona cascade, officers are rank 1** · 2026-09-04
In the tools of the market, find people searches by title and seniority. The register's mandates
come first because they are free and proven; the databases and the browser follow, per persona. A
persona is a fact of the target, grilled at setup: absent, the command stops.

**No `grill.md`: setup's Done when lists the facts, the interview is free** · 2026-09-04
Fixed questions are no longer a grilling. `setup` names the facts that must be written when it
ends and delegates the interview to the `grilling` skill when it is installed, one question at a
time with a recommended answer otherwise.

## The data

**Data format is free, the cell is not** · 2026-09-04
SQLite, CSV, anything the target chooses. Whatever it is, a cell carries the value, the date of the
fact, the date of the reading, the source and the proof; nothing is overwritten, a pass appends,
and a source that finds nothing writes a dated empty witness. The latest-value view is
recalculated, never stored.

**Raw page text never enters the table** · 2026-09-04
Two site-text columns were dropped: 4 800 characters per row, dragged by every command that
re-reads the table, for a thousand rows. The pages live in `data/raw/<siren>/pages.md`, the
`description` script reads them once per batch, and the proof points at the file.

**`.leadgen/` at the host root, one folder per target, `targets/<t>/columns.md` shadows the catalogue** · 2026-09-04
No external data directory with a marker line: a lead list belongs to the project that attacks it.
One folder per target holds its page, its blocks, its code and its data; a folder by nature or a
flat root was refused because the target is what a reader looks for. The catalogue is a copy
replaced on update, so a target's own blocks live in the target and shadow the catalogue's by
name.

**AI columns run through a stripped model call from the column script** · 2026-09-04
Measured on the author's subscription: with settings, tools and skills already stripped, a
one-word answer still sent 45 757 input tokens, the tool definitions of the machine's MCP servers;
with the MCP servers kept out, 414. The script prepares batches
of ten, runs the call itself and ingests the JSON; the model tier is chosen by the column's class
and named on every call, never inherited from the session, because a hundred sub-agents that
inherit spend the top tier on summaries. `--bare` was refused: it requires an API key and bills the
API instead of the subscription.

**`ats-public` mirrors the ATS files of the job-search plugin** · 2026-09-04
One source of truth for the public ATS endpoints: the `ats/<name>.md` files of `jobhunt`, which
exercise them against live forms. The block here copies the endpoints and points at the mirror; a
new ATS is added there first.

## Deferred

Known holes, kept visible on purpose.

**The `domain` example was corrected while writing.** The block validated before writing had the
register at rank 1, "next when the record carries no website". Read on 2026-09-04, the register
carries no website field at all; the block was written with the site's legal pages at rank 1,
proven on the keys the register does give (name, acronym, brands, trade signs), and the search
engine at rank 2.

**A `stack` column**, the tools seen on the site, in the DNS and in the certificate logs: the
probes are known, the column is not written.

**`notify`, `schedule`, a credits `audit`**: out of scope; a loop skill covers the cadence, the
catalogue and the pass lines cover the spend.

**The top tier for `verify`**: the class `judgment` names the intermediate tier; whether the audit
needs more is unmeasured.

**A standard CSV export at `push`**: two tables, companies and people, for a CRM with no access; the
mapping exists in the host, the export does not.

**A blind-verdict review with an agreement measure**, human and model judging the same rows without
seeing each other, if `verify` proves insufficient.

**`tested` lines dated to a month.** Several source blocks were exercised in August 2026 on a table
whose journal did not keep the day; the month is what is known. `crunchbase` carries the date of a
browser reading where the validated draft said `never`.

**`setup` takes no host path.** The host is the git root of the working directory. A repository that
wants several hosts, one per business, has no way to say so: a `setup <path>` argument is the
obvious shape, unwritten until a second such repository asks for it.

**A target cannot shadow `sources.md`.** A source that exists for one target only (a public file
of one trade's ministry, say) is described inline in the cascade of its block, like the file the
user hands over to the seed; a `targets/<t>/sources.md` on the catalogue's shape would be the
symmetric answer, unwritten until a target needs more than one such source.
