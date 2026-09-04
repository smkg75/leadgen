# Columns

The catalogue by purpose. One block per column: what fills it, in which order, what settles two
values, when it runs, what it costs. `find` makes rows, `enrich` makes columns, `signals` refreshes
the perishable ones, `push` writes the export ones. Every command reads this file and
[`sources.md`](sources.md) before it writes anything. A target's own `targets/<t>/columns.md`
shadows any block here by name, on the same shape.

A source is named by its `## <source>` heading in `sources.md`, to the letter. A column is
`snake_case`: its name becomes a field name and a script name.

## The cell

A cell is never a bare value. It carries the value, the date of the fact, the date of the reading,
the source that gave it and the proof: the page, the record or the query that lets a reader check
it.

The storage is a **journal of facts**, one line per fact, append only, whatever the format:

```
key · column · value · fact_date · read_date · source · proof
```

Nothing is overwritten. A pass adds lines. A source that finds nothing writes a **dated empty
witness**, so the next pass does not retry and a reader tells "searched, nothing" from "never
searched". A column filled by query rather than by company writes no witness on the companies the
query did not name: "nothing found" on a company that was not searched would extinguish a fact
already acquired.

Reading goes through the **latest-value view**: one row per company or person, each column
reduced to one value by its `merge` rule. The view is recalculated by the scripts and never stored
as a fact. It is what `verify`, `push` and the human read.

A fictional company in the journal:

```
123456789 · domain     · atelier-riviere.fr · 2026-09-01 · 2026-09-01 · website          · https://atelier-riviere.fr/mentions-legales cites 123 456 789
123456789 · headcount  · 10-19              · 2025-12-31 · 2026-09-01 · company-registry · tranche_effectif_salarie 11
123456789 · headcount  · 14                 · 2026-08-28 · 2026-09-01 · linkedin         · company page, "14 employees"
123456789 · job_offers · Chef d'atelier     · 2026-08-20 · 2026-09-01 · ats-public       · https://jobs.example/<slug>/<id>
123456789 · job_offers · Commercial B2B     · 2026-08-27 · 2026-09-01 · ats-public       · https://jobs.example/<slug>/<id>
123456789 · funding    · ∅                  ·            · 2026-09-01 · media-funding    · no title matched in 18 months
123456789:marie-durand · job_title · Présidente · 2026-09-01 · 2026-09-01 · company-registry · mandate "Président"
```

Two `headcount` facts kept, the view shows the LinkedIn one; one line per offer; a dated empty
witness on `funding`; a person keyed `siren:slug`.

## Two tables

`companies`, keyed by `siren`. `people`, keyed by `siren` + `slug`; a person points at their
company by the SIREN, never by name. Two tables, never one joined table, even in an export.

## The data format is free

SQLite, CSV, anything the project chooses. `targets/<t>/index.md` says which, and the journal
shape above holds whatever it is.

## What a column script holds

One column = one script, `targets/<t>/columns/<column>.<ext>`, written in the host repo on the
block below and on the source blocks of its cascade. Every script:

- resumes: a witness per row lets a rerun skip what is done and start at the last key written;
- stops clean: distinct exit codes for a quota reached, a missing key, and a captcha or 429;
  nothing half-written, so the caller knows whether to relaunch later or fix something first;
- counts: rows written, empty witnesses, failures, printed at the end of the pass and compared
  with the previous pass in `targets/<t>/index.md`. A number that collapses is a bug, not a fact;
- keeps keys in the environment, never in the repo, and names the variable it wants when it is
  missing.

## Types, costs, merges, classes

`type` names the nature of the value:

- `seed` creates rows: the work of `find`;
- `attribute` is a stable property of the company or the person, refreshed by `latest`: `enrich`;
- `ai` is written by a model, batch by batch: `enrich`;
- `reveal` is a paid contact unmasked, email and mobile: `enrich`, on an explicit request, a named
  person or selection;
- `signal` is a dated, perishable event, appended: `signals`;
- `derived` is recalculated entirely on every read, never stored as a fact: whoever reads;
- `export` is what comes back from a write to a third party: `push`.

`cost`: `free` · `quota` (a browser or a rate-limited endpoint, the rate measured in `sources.md`)
· `paid` (per row; a named selection is mandatory; the cap of `context.md` defaults to 0, so `run`
never pays unless the host raised it, and a paid request by hand is gated by an estimate, people ·
credits · euros · balance, the user answers go to).

`merge`, how the view reduces several facts to one value: `first` (the earliest proven value wins,
later ones stay facts) · `latest` (the most recent reading wins) · `append` (every fact stays a
line).

`class`, on `ai` columns, dictates the model tier: `summary` (the cheapest tier) · `judgment` (the
intermediate tier) · `writing` (the intermediate tier).

## AI columns

An `ai` column is a script like any other. It prepares batches of ten rows, runs the model call
itself, ingests the JSON that comes back and writes one fact per row. Ten rows a call is a rule of
thumb, not a measurement: past a few dozen the model mixes rows, and a batch of two pays the prompt
for nothing.

The prompt is a file of the target, `targets/<t>/columns/<column>.md`, long and autonomous: the
template of the value, the fields of the JSON expected, and the whole of `.leadgen/context.md`
inlined, because a verdict depends on what the user sells. The block gives the contract (`reads`,
`writes`, `class`); the prompt is code, written per target.

The model tier is chosen by `class` and **named on every call**, never inherited from the session
that launches the script: a hundred sub-agents that inherit the session's model spend the top tier
on summaries.

The prompt states and the script enforces: nothing invented, `null` where the input does not say,
the proof cited for every value, and the raw text of a page **never enters the table**. The pages
live in `data/raw/<siren>/pages.md`, the proof a reader opens.

### The stripped call

The call runs through the Claude Code CLI, on the user's subscription, with nothing of the machine
loaded. Measured on 2026-09-04 with the cheapest tier: with settings, built-in tools and skills
already stripped, a one-word answer still cost 45 757 input tokens, because the MCP servers of the
machine send their tool definitions on every call. With the flags below, the same answer cost 414
tokens with a one-line system prompt and 631 with the 200-word prompt of a description column. A
real batch of ten companies with three pages each, about 45 000 characters, came to 12 500 input
tokens and 1 100 output tokens in eleven seconds.

```bash
claude -p --model <tier> --max-turns 1 --output-format json --no-session-persistence \
  --setting-sources "" --tools "" --disable-slash-commands \
  --strict-mcp-config --mcp-config '{"mcpServers":{}}' \
  --system-prompt-file targets/<t>/columns/<column>.md \
  -- "<the batch, as JSON>" < /dev/null
```

Then read `.result` from the JSON, strip the code fence the model sometimes wraps it in, and parse
the array; `.usage` carries the token counts for the pass line.

What the flags buy: `--tools ""` and `--disable-slash-commands` remove the built-in tools and the
skills; `--setting-sources ""` ignores every settings file; `--strict-mcp-config` with an empty
config keeps every MCP server of the machine out, which is where the 45 757 tokens came from;
`--` closes the options, because `--mcp-config` takes several values and a batch placed right after
it, or a batch that starts with a dash, would be read as an option; `< /dev/null` skips the wait
for a piped input. `--exclude-dynamic-system-prompt-sections` is not in the recipe: it applies to
the default system prompt only and does nothing once `--system-prompt-file` is set. Two consequences: the model reads no file and reaches no tool, so **the
script inlines the batch in the prompt**; and `--bare` stays out, because it requires an API key
and bills the API instead of the subscription.

The recipe holds for any question a script wants answered by a model, not only for columns.

## Block shape

Key-value lines, in this order: `type`, `cost`, `cascade`, `stop`, `merge`, `only if`, `writes`;
`ai` columns add `reads` and `class`. `cascade` is numbered: rank, source, scope, and the
condition that passes to the next rank. `scope` is `per company` (one call a row, resumable at the
last SIREN) or `per query` (one sweep a pass, feeding a matching index the companies are
recognised against). `stop` is what is written when the cascade is exhausted. `only if` is the
condition a row must pass. `writes` is the shape of the value and what counts as proof.

A block normally owns one column. It owns several when one gesture on one source yields them all
and none has a cascade of its own: the seed writes the identity columns, a site scan writes its
markers. Such a block is named after the gesture, lists every column it writes under `writes`
with the shape of each, and a column that needs a cascade or a signal of its own keeps its own
block and reads the gesture's proof file rather than fetching again.

# Companies

## companies

```
type      seed
cost      free
cascade   1 company-registry   per query     activity codes × perimeter × active, paginated; next when the codes are done
          2 google-maps        per query     establishments by trade and place, when a key is set; next when the queries are done
          3 a file the user hands over, one SIREN a row (no block in sources.md: the file is the user's)
stop      the rows are what they are; a code counted and refused is written in targets/<t>/index.md with its count
merge     append · one row per SIREN; the first source that names a SIREN owns the row
only if   the SIREN passes the perimeter and the global exclusions of context.md
writes    siren · siret (head office) · name · naf (code + INSEE label) · legal_form · city · created · active · headcount (band) · revenue (when accounts are filed) — columns of the table, no block of their own · proof = the record URL or the query
```

## proceedings

```
type      attribute
cost      free
cascade   1 bodacc   per company   collective proceedings on 24 months; accounts filed and officer changes on 12
stop      none, dated: searched on the window, nothing found; the last accounts filed and the last officer change dated in the proof
merge     latest · the last judgment known is the state of the company
only if   active
writes    LJ · RJ · SAUVEGARDE · PLAN · CLOTURE · none, dated by the judgment · proof = the notice URL
```

## domain

```
type      attribute
cost      free
cascade   1 website         per company   candidates built from the name, the acronym, the brands in parentheses and the trade signs of the register record, on .fr .com .io .eu .net; kept when a legal page cites the SIREN or the page cites a strong key; next when no candidate is proven
          2 search-engine   per query     the same proof on the first results outside directories; next when the pass cap is reached
stop      empty, dated witness written, so the next pass does not retry
merge     first · a domain derived from the name alone is SUSPECT, settled before any signal runs
only if   verdict is not excluded
writes    the bare domain · proof = the page or the query that gave it, and its strength: SIREN on the page > strong key cited > name root alone
```

## phone

```
type      attribute
cost      free
cascade   1 website       per company   the contact and legal pages; next when none is found
          2 google-maps   per company   the establishment record, when a key is set
stop      empty, dated witness
merge     latest
only if   domain is set
writes    E.164 · proof = the page or the record
```

## headcount

```
type      attribute
cost      quota
cascade   1 linkedin           per company   the company page, browser, on a named selection; next when the page is not found or the day's quota is reached
          2 company-registry   per company   the INSEE band, already written by the seed
stop      the seed's band stands
merge     latest when the reading is under 90 days; older, the band stands
only if   verdict ok
writes    a number or a band · proof = the page, or the record field
```

## description

```
type      ai
cost      free
cascade   1 website   per company   three pages, home, product or pricing, about, fetched by the script and capped at about 1 500 characters each into data/raw/<siren>/pages.md; next when the site answers nothing readable
stop      null with the reason, dated
merge     latest
only if   domain is set and not SUSPECT
writes    400 to 600 characters on a fixed template, in this order: what it sells, to whom, B2B or B2C, where, size, one distinctive trait · JSON fields siren, description, reason, proof · proof = the page that gave the main claim
reads     data/raw/<siren>/pages.md, once per batch of ten
class     summary
```

## one_liner

```
type      ai
cost      free
cascade   none, the batch is built from the view
stop      null when description is null
merge     latest
only if   description is set
writes    one sentence · proof = description
reads     description and nothing else
class     summary
```

## verdict

```
type      ai
cost      free
cascade   none, the batch is built from the view
stop      doubt with the reason, when the inputs do not settle it
merge     latest
only if   description is set
writes    ok · doubt · excluded, with one line of reason · proof = the sentence of the description or the flag that settled it
reads     description, context.md, flags
class     judgment
```

## flags

```
type      derived
cost      free
cascade   none, recalculated from the view on every read
stop      no flag
merge     recalculated entirely, never stored as a fact
only if   always
writes    the exclusions the row trips, each with its rule and the value that tripped it; reversible by editing the rule in targets/<t>/index.md; a rule is validated on the real data, every field (legal name, acronym, trade sign), before it is trusted
```

## duplicate

```
type      derived
cost      free
cascade   none, recalculated from the view
stop      no duplicate
merge     recalculated entirely
only if   domain is set
writes    the SIREN kept, on the other rows sharing the domain: the largest INSEE headcount, then the score of the previous pass, then the oldest creation, then the smallest SIREN; marked, never deleted; a row without domain is nobody's duplicate
```

## job_offers

```
type      signal
cost      free, quota from rank 5
cascade   1 ats-public       per company   the ATS found on the careers link of the site; next when the site names no ATS
          2 website          per company   the careers page, titles matched against the roles of the target, undated; next when there is no careers page
          3 france-travail   per query     the queries of the target, the employer matched by name
          4 hellowork        per query     same matching
          5 wttj             per company   named selection only
          6 indeed           per query     MCP, on a named shortlist only
stop      witness on the per-company ranks only; a per-query rank writes nothing on a company it did not name
merge     append · one line per offer, dated by the posting
only if   verdict ok
writes    the title · proof = source, matched role, URL · hiring signals: open roles as a growth indicator, hiring velocity read across passes, never from one
```

## funding

```
type      signal
cost      free, quota on rank 3
cascade   1 media-funding   per query     articles of the last 18 months, the target's companies matched in the titles
          2 bodacc          per company   capital increases above the threshold on 18 months
          3 crunchbase      per company   named selection only, API when a key is set, else the user's browser
stop      dated empty witness on ranks 2 and 3
merge     append · one line per round or increase
only if   verdict ok
writes    amount when read, series, or "round" · dated by the article or the notice · proof = title and URL, or capital before → after · recent news: funding, leadership changes
```

## active_ads

```
type      signal
cost      quota
cascade   1 google-ads   per company   creatives shown in the last 12 months
          2 meta-ads     per company   ads active now
stop      dated empty witness: no advertiser matched, with the queries tried
merge     append · one line per reading
only if   verdict ok
writes    a count per network, with the advertiser matched, first and last showing · proof = the transparency page URL
```

## contracts

```
type      signal
cost      free
cascade   1 decp   per query   a batch of SIREN a request, contract holders matched by SIRET prefix
stop      dated empty witness, removed once a contract is found
merge     append · one line per contract, dated by the notification
only if   verdict ok
writes    object · buyer · amount · notification date · proof = the contract URL
```

## score

```
type      derived
cost      free
cascade   none, recalculated from the view
stop      0 · a company with no signal sits at the bottom of the list, never outside it
merge     recalculated entirely
only if   verdict ok
writes    a sort counter: each signal in its window weighs what targets/<t>/index.md says; the distribution is compared with the previous pass, a collapse is a bug
```

## crm_id

```
type      export
cost      free
cascade   none, written by push
stop      empty until pushed
merge     latest
only if   pushed
writes    the CRM's own id, dated by the push · proof = the record URL in the CRM
```

# People

## people

```
type      seed
cost      free, quota on rank 3
cascade   1 company-registry   per company   the officers: a direction mandate is a contact, a board mandate is a lead, a corporate president is a subsidiary to climb (three levels at most), a sole trader is the person; next when no persona is matched
          2 lemlist            per company   the people database by title and seniority, MCP, the search is free; next when the persona is still not matched
          3 linkedin           per company   the People tab, browser, named selection only
stop      the company keeps its officers; a dated witness for the persona search
merge     append · dedup on name + siren · a cap per company, set in targets/<t>/index.md · personas ranked, the chief executive first by default
only if   verdict is not excluded, and a persona is defined for the target
writes    name · slug · mandate or title · the persona matched · best entry point: person and why, when the personas rank them · proof = the record, the search or the page
```

## job_title

```
type      attribute
cost      quota
cascade   1 linkedin   per company   the profile, browser, named selection; confirms the mandate and gives the operational title
stop      the mandate stands, dated witness
merge     latest
only if   the person has a mandate or a title to confirm
writes    the title as displayed · proof = the profile URL
```

## email

```
type      reveal
cost      paid
cascade   1 lemlist      per company   the finder by name and domain, the cheaper of the two; next when not found or not verified
          2 fullenrich   per company   asynchronous, the batch in flight kept on disk
stop      empty, dated witness, paid once
merge     first · paid once per person, never bought twice
only if   verdict ok AND the person is in the named selection AND, under run, the cap of the pass is not reached, or, by hand, the user said go to the estimate
writes    the verified address · proof = the provider, the batch id, the verification status
```

## mobile

```
type      reveal
cost      paid
cascade   1 fullenrich   per company   one person at a time
stop      empty, dated witness
merge     first
only if   one person named, at the moment of the call, after go to the estimate
writes    E.164 · proof = the provider and the batch id
```

## icebreaker

```
type      ai
cost      free
cascade   none, the batch is built from the view
stop      null when no fact under 180 days exists, with the reason
merge     latest
only if   verdict ok and at least one signal under 180 days
writes    one sentence that cites one dated fact under 180 days and its proof · proof = the fact's line
reads     description, the signal columns of the company
class     writing
```

## crm_id

```
type      export
cost      free
cascade   none, written by push
stop      empty until pushed
merge     latest
only if   pushed
writes    the CRM's own id, dated by the push · proof = the record URL in the CRM
```
