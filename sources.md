# Sources

The catalogue by source. One block per way of reaching data, named by the access and never by a
brand (two funding media on one API are one source), kebab-case, alphabetical. The classification
by purpose lives in [`columns.md`](columns.md): a column names its sources by these headings, to
the letter.

Rules that hold for every source:

- **CLI > API > MCP > browser.** The official endpoint before any scraping; the browser is the
  last route, on the user's own account, read only.
- **A 429 or a captcha stops the pass, clean.** Nothing half-written, the last key written is the
  resume point, and the retry comes hours later, not minutes: on the sources measured here the
  wall stands for hours whatever the wait.
- **Paid last**, on a named selection, under the cap of `.leadgen/context.md`; an in-flight batch
  is kept on disk so nothing is bought twice.
- **A source whose terms forbid extraction** runs on a named shortlist only, and the user accepts
  the risk before the first pass, in `targets/<t>/index.md`.
- **A closed route is struck through and dated** in `.leadgen/connections.md`, otherwise the next
  pass retries it.
- **Keys live in the environment**, never in the repo; a block names the variable, the host's
  `connections.md` says whether it is set, never its value.
- **Every reading is dated and carries its proof.**

Each block: `gives` · `access` (endpoint or tool, the variable and how to obtain it, per company
or per query, the matching key) · `limits` (rate measured, cost, terms, the stop gesture, the
resume key) · `pitfalls` (one a line, with the gesture) · `tested` (a date and what was exercised,
or `never`). A block marked `never` stays short: `gives`, `access`, and under `limits` only what the
provider itself states, nothing measured here.

## ats-public

```
gives     the open roles of a company from its own applicant tracking system, dated by the posting
access    keyless JSON, per company, the <slug> read off the careers link found on the company's site:
          Greenhouse https://boards-api.greenhouse.io/v1/boards/<slug>/jobs · Lever https://api.lever.co/v0/postings/<slug>?mode=json · Ashby https://api.ashbyhq.com/posting-api/job-board/<slug> · Workable https://apply.workable.com/api/v3/accounts/<slug>/jobs (POST, body {}) · Recruitee https://<slug>.recruitee.com/api/offers/
          Teamtailor, Workday, Rippling, Deel, Taleez: the listing page, no keyless API
          this block mirrors the ats/<name>.md files of the jobhunt plugin, the one source of truth for these endpoints; a new ATS is added there first
pitfalls  a slug guessed from the company name reads someone else's roles: the slug comes from a link found in the HTML
          Greenhouse dates only updated_at, and a board-wide update stamps every role the same day; Lever's createdAt is creation, not publication; both are floors
tested    never here; the mirrored files carry their own tested line
```

## bodacc

```
gives     collective proceedings (liquidation, receivership, safeguard, plan, closure) dated by the judgment; accounts filed; officer changes; capital increases read between two consecutive notices, as a date and a percentage, never an amount
access    https://bodacc-datadila.opendatasoft.com/api/explore/v2.1/catalog/datasets/annonces-commerciales/records?where=registre:"<siren>"&order_by=dateparution desc · keyless · per company, or a batch of about twenty SIREN in one where
limits    3.5 M credits a day, measured on 2026-08-23; windows of 24 months for proceedings, 12 for accounts and officer changes, 18 for capital increases; resume at the last SIREN written
pitfalls  the sub-objects of a notice are JSON inside a text field: parse them
          write the "searched, nothing" witness per company, or the next pass pays the sweep again
          a capital increase is read only on a notice whose wording speaks of the capital; on any other the date is that of an officer change
          a threshold separates a round from an option exercise: a fraction of a percent is noise, a double-digit jump is a round; the issue premium is not published, so the amount stays unknown
tested    2026-08-23 — proceedings, accounts, officer changes and capital increases on a full table
```

## company-registry

```
gives     the French company register: SIREN, head-office SIRET and address, legal name, acronym, trade name and trade signs, brands in parentheses in the full name, NAF code, legal form, creation date, administrative status, INSEE headcount band, revenue and net result when accounts are filed, the officers with their mandate, the count of establishments; no website
access    https://recherche-entreprises.api.gouv.fr/search?q=… · keyless · per query with activite_principale=<naf>, code_postal or departement, etat_administratif=A, 25 a page; per company with q=<siren> for the full record
limits    7 requests a second; resume at the last code or the last SIREN
pitfalls  the geographic filters match establishments, not the head office; when the boss is who gets called, filter on the head office
          an activity code is not a trade: count a code's rows before extracting it, and write the count next to the decision
          the legal name is not the trade name: never reject a SIREN because the two differ; the brands in parentheses and the trade signs are the best keys to find a domain
          a contract holder or a public buyer is published by SIRET only: resolve the name by SIRET, and an empty name never fails the pass
          a pass that skips every SIREN already seen never backfills a column added later: backfill is its own mode
          a sole trader (legal categories 1000 to 1999) has no mandate: the person named is the business, and reads as a contact
          a corporate president signals a subsidiary: climb to the parent, three levels at most, with a loop guard
tested    2026-08 — search by activity code and postcode, per-SIREN records, officers, the parent climb
```

## crt-sh

```
gives     the sub-domains a company has published in certificate transparency logs, hence its tools and products online
access    https://crt.sh/?q=%25.<domain>&output=json · keyless · per company; CertSpotter as an alternative, 10 requests an hour anonymous, CERTSPOTTER_KEY raises it
limits    slow and fragile, about 10 requests an hour: named selection only
tested    never
```

## crunchbase

```
gives     funding rounds with date, series, investors and amount when served; headcount band; founders and executives
access    API Basic with CRUNCHBASE_KEY, per company, the organisation looked up by the domain root, the rounds on the raised_funding_rounds card; without a key, the user's own browser on the organisation page, signed out, where rounds, the type of the last round, investors and headcount are readable and amounts and dates blurred
limits    every automated route outside the API is blocked (403 to a plain client, a block page to a headless browser); browser reads follow the LinkedIn rhythm, about 60 pages a day with 8 to 25 seconds between two, stop at the first captcha
pitfalls  investors and amount live on the round, not on the organisation record; a plan that does not serve the card leaves them empty, nothing is reconstituted
          a round without a readable date is not a dated round: kept apart, never as a funding signal
tested    2026-08-21 — a shortlist read in the user's browser, signed out; the API route never
```

## decp

```
gives     public contracts won: object, buyer, amount, notification date, procedure, duration
access    https://data.economie.gouv.fr/api/explore/v2.1/catalog/datasets/decp-2022-marches-valides/records?where=startswith(titulaire_id_1,"<siren>") or startswith(titulaire_id_2,"<siren>") or startswith(titulaire_id_3,"<siren>") · keyless · per query, a batch of SIREN in one where, 100 a page
limits    50 000 requests a day; refreshed daily at D-5; Licence Ouverte; the older data.gouv set is deprecated
pitfalls  the contract holder is published by SIRET over three fields: filter by prefix on the SIREN and keep the identifiers whose type is SIRET (the set also carries VAT, foreign and overseas identifiers)
          the buyer is published by SIRET: resolve its name through company-registry once and cache it, the same buyer returns on dozens of contracts
          date the fact by the notification, never by the reading
          remove the empty witness once a contract is found, or the latest-value view hides contracts notified before it
tested    2026-08 — a full table swept in batches, buyers resolved
```

## france-travail

```
gives     job offers dated by publication, with the employer's name
access    https://api.francetravail.io/partenaire/offresdemploi/v2/offres/search · FT_CLIENT_ID and FT_CLIENT_SECRET from a francetravail.io developer account, exchanged for a client-credentials token · per query, the queries of the target; the employer matched against name and acronym, exact or by a prefix of five characters, never by a counter
          without a key, the candidate site, server-rendered, while it holds; its dates are relative ("published 3 days ago") and are converted to ISO at write time
limits    a per-query sweep: a selection restricts the matching index, not the queries; no resume key, every pass re-sweeps
pitfalls  offers with no named employer are dropped
          no empty witness per company: a company the sweep did not name was not searched
tested    2026-08 — API and site, offers matched to a table
```

## fullenrich

```
gives     a verified professional email; a mobile number
access    API with FULLENRICH_API_KEY, asynchronous: submit a batch, poll for the result · per company, one person or a batch
limits    paid per contact found; the batch in flight is kept on disk with its id, so a crash never pays twice; mobile only at the moment of the call, for one named person
pitfalls  a mobile bought ahead of the call is a mobile paid for nothing
tested    2026-08 — an email batch submitted and read back
```

## google-ads

```
gives     the creatives an advertiser has shown in France, with first and last showing dates and the total in archive
access    the Transparency Center's anonymous endpoints, region FR: SearchSuggestions finds the advertiser (domain root, then the first word of the name), SearchCreatives lists its creatives sorted by last showing, 100 a call · per company · no key
limits    one request a second, global to the process, day and night; the 429 is not a rate counter but the reCAPTCHA "unusual traffic" wall, which closes the whole host and held nine hours without a single request on 2026-08-23: one retry at 60 seconds, then write what is done, say the wall is up, exit with the relaunch code; resume at the last SIREN written
pitfalls  never write "no advertiser" on a 429
          an advertiser already matched spares the search on the next pass: one call instead of two
          the suggestion endpoint matches wide; the advertiser is the one whose name contains the query
tested    2026-08-23 — a full table, the wall met and measured
```

## google-maps

```
gives     whether an address is a business or a residence; the establishment record: phone, website, rating, review count, business status; the count of establishments found under a name
access    Address Validation and Places (New) text search · GOOGLE_MAPS_API_KEY from a Google Cloud project with both APIs enabled and billing on · per query, a text search by trade and place, to seed rows; per company, the record behind an address; a field mask is mandatory and the bill depends on it
limits    paid per call: named selection, capped
tested    never
```

## hellowork

```
gives     job offers from a public search, with the company name of the offer
access    the public search pages, server-rendered · per query, the queries of the target; the company name matched against name and acronym, exact or by a prefix of five characters
tested    never
```

## indeed

```
gives     job offers with the company name and the posting date
access    the Indeed MCP connector of the session, search_jobs, about ten offers a call, no company filter, the company endpoint empty · per query on a named shortlist: the session searches, a script matches the readings by name, equality or a prefix of five characters
limits    shortlist only: outside it, a homonym writes someone else's column
pitfalls  dates come as "August 03, 2026": convert at write time
          no empty witness: a company absent from the reading was not searched
tested    2026-08 — readings matched to a shortlist
```

## lemlist

```
gives     two things on one account: a people database searched by title and seniority, free; an email finder, paid per address
access    MCP on the session for the database, per company, by company name or domain, title and seniority from the persona; API with LEMLIST_API_KEY for the finder, per person, name and domain
limits    the finder is paid: named selection and cap; the database search is free
tested    never
```

## linkedin

```
gives     the headcount and the People tab of a company page; the title and the current role of a person
access    the browser on the user's own account, read only · per company, on a named selection
limits    about 150 profiles and 60 company pages a day, 8 to 25 seconds between two, stop at the first captcha and resume the next day; the terms forbid it and a data vendor was sued over it: the user accepts the risk before the first pass
pitfalls  a board mandate is confirmed as a contact only when the profile shows an operational role
tested    2026-08 — company pages and profiles on a shortlist, quotas held
```

## media-funding

```
gives     funding rounds announced by the trade press: the date of the article, the amount when the title states it, the series, the investors when the body is served
access    the WordPress API frenchweb.fr and maddyness.com expose alike, https://<medium>/wp-json/wp/v2/posts?search=<term>&after=<iso>&per_page=100&page=<n> · keyless · per query: the articles of the last 18 months, the target's companies searched in the titles, never the reverse
limits    frenchweb.fr serves the article body, maddyness.com serves it empty (title only); a page past the last answers HTTP 400 on both, which is how the loop stops, and maddyness.com caps a search at 100 articles, so its page 2 is already past the last
pitfalls  a title that names a round and an acquisition speaks of the operation named first
          an amount alone is not a round: the title must carry a raising expression
          the subject sits just before the raising expression, after the last punctuation; a company cited at the head or the tail of a title is not the subject
          a matching key of one word is six characters at least and never a common word, or it collects the whole feed
          the amount is read in the title or left empty, never estimated
tested    2026-08-23 — both media, 18 months, matched to a table
```

## meta-ads

```
gives     the ads a page runs in France now, their launch dates, the advertiser pages attributed to a company
access    API ads_archive with META_ACCESS_TOKEN; else the public Ad Library pages in the browser, no cookie, no login: a keyword search lists advertisers with page id, page name, advertised link, status and launch date, read from the payload embedded in the page; then each attributed advertiser page, all ads then active ads · per company
limits    a captcha stops the pass; a page with no payload marks the company for a retry
pitfalls  attribution takes two proofs, never a resemblance: the advertised link carries the domain, or the page name equals the domain root or the company name exactly; a prefix match attributed unrelated pages
          launch dates are read on the rendered ads, thirty a page by relevance: exact under about thirty active ads, the most recent known above
tested    2026-08-23 — the browser route on a full table; the API route never
```

## rdap-dns

```
gives     whether a domain exists, its age and its life; the MX for email; the sub-domains that resolve (app, api, my, dashboard, console, status, docs: one is enough to say a product is online); the tools read in MX and TXT records
access    dig, the registry's RDAP (https://rdap.org/domain/<domain>), DNS over HTTPS (https://dns.google/resolve?name=…, cloudflare-dns.com as fallback) · keyless · per company
pitfalls  a third-party whois lies: the registry's RDAP only
          a wildcard zone resolves anything: the probe does not conclude
tested    never
```

## search-engine

```
gives     the domain the register does not carry
access    https://html.duckduckgo.com/html/?q=<name> <city> · keyless · per query, one call at a time
limits    one request every five seconds, capped per pass (about 200); it blocks when pressed, and the pass stops itself when it no longer answers
pitfalls  the first result that does not cite the company is not its site: every candidate passes the same proof as any other, directories excluded
tested    2026-08 — the fallback rank on a table, the cap held
```

## website

```
gives     the SIREN on a legal page, the strongest proof binding a domain to a company; what the company sells and to whom, B2B or B2C markers, from three pages; the phone; the careers link and the ATS behind it; markers such as chat, pixel, booking, pricing, jobs, tag manager
access    direct fetch · per company: the home page, then the first product-or-pricing page and the first about page the home links to, each capped at about 1 500 characters into data/raw/<siren>/pages.md; for the SIREN, the footer links (legal notice, terms, privacy, about), not only /mentions-legales
limits    three pages a company; a site behind a JavaScript challenge is unreadable to a plain client and is written as such
pitfalls  the legal name is not the trade name: never reject a SIREN because the two differ
          the host is cited next to the publisher (a hosting provider, a site builder): retaining it hands the target the host's SIREN
          an exact homonym passes every text check (a short generic domain for a company of the same short name): the audit lists suspects by proof strength, SIREN on the page > strong key cited > domain root derived from the name
          the SIREN is cited as nine digits or as three groups of three
          B2B and B2C counters are read on the home page alone; the thresholds are calibrated on it
tested    2026-08 — the three-page scan, the legal-page SIREN, phone and markers, on a full table
```

## wttj

```
gives     job offers with the organisation name, contract, remote policy and posting date
access    the site's Algolia index, search-only keys served to every visitor in window.env of any public company page (ALGOLIA_APPLICATION_ID, ALGOLIA_API_KEY_CLIENT), set as WTTJ_ALGOLIA_APP and WTTJ_ALGOLIA_KEY, never in the repo; a Referer header of the site's domain is required by the key · per company, the index searched by organisation name, the name settles the match
limits    the terms forbid extraction: named shortlist only, the risk accepted before the first pass; the site's pages sit behind a JavaScript challenge and are unreadable to a plain client
pitfalls  the key allows restricting searchable attributes only on a whole group of the same priority
tested    2026-08 — offers matched to a shortlist
```
