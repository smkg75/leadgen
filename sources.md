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
- **A browser source** runs on a named shortlist only, never on the whole table.
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
limits    7 requests a second per IP on paper, lowered at will by the State: 429s came at 1.3 a second shared by two sessions; one caller per IP at a time, 1 a second at most, honour Retry-After and double the wait on each new 429, stop for 12 hours when the API goes silent (an IP ban); resume at the last code or the last SIREN
bulk      a known list of SIREN never loops on /search: the monthly SIRENE stock in Parquet on data.gouv (StockUniteLegale, StockEtablissement) read in place with DuckDB httpfs, `WHERE siren IN (…)`, answers a few hundred in a second with no limit; status, names, sole-trader name, headcount band, head office, NAF, no officers; the file URL changes each month, read it from the data.gouv API (/api/1/datasets/base-sirene-des-entreprises-et-de-leurs-etablissements-siren-siret/)
pitfalls  the geographic filters match establishments, not the head office; when the boss is who gets called, filter on the head office
          an activity code is not a trade: count a code's rows before extracting it, and write the count next to the decision
          the legal name is not the trade name: never reject a SIREN because the two differ; the brands in parentheses and the trade signs are the best keys to find a domain
          a contract holder or a public buyer is published by SIRET only: resolve the name by SIRET, and an empty name never fails the pass
          a pass that skips every SIREN already seen never backfills a column added later: backfill is its own mode
          a sole trader (legal categories 1000 to 1999) has no mandate: the person named is the business, and reads as a contact
          a corporate president signals a subsidiary: climb to the parent, three levels at most, with a loop guard
tested    2026-08 — search by activity code and postcode, per-SIREN records, officers, the parent climb; 2026-09 — the 429s under 7 a second, the Parquet stock read with DuckDB
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
gives     a verified professional email; a mobile number; and, free, a person's LinkedIn profile with their title, company and city
access    two doors on the same account, and a door is not a price: the session MCP and the API with FULLENRICH_API_KEY both reach the finder. What costs is the operation, never the channel. MCP: the connector of the session, no key posed, no account file to read. API, asynchronous: POST https://app.fullenrich.com/api/v2/contact/enrich/bulk (up to 100 contacts), then GET https://app.fullenrich.com/api/v2/contact/enrich/bulk/<enrichment_id> until done (a webhook is offered instead of polling) · per company, one person or a batch
          balance: GET https://app.fullenrich.com/api/v2/account/credits → {"balance": n}, read before every estimate
limits    paid whichever door it goes through, per contact found, nothing charged on a miss: 1 credit a professional email, 3 a personal email, 10 a mobile (help centre, read 2026-09-04); a credit is about 0.055 € on the public monthly plan, the host's connections.md records the rate actually paid; the batch in flight is kept on disk with its id, so a crash never pays twice; mobile only at the moment of the call, for one named person
search    the people search is free and answers two different questions. Asked a name, it says whether that person has a profile — the narrow door, and it fails whenever the profile carries a middle name or a trade suffix the register ignores. Asked a company name and a list of job titles, it walks the other way: from the trade back to the people, and it finds the owners no name lookup reaches. Run both, they overlap little
          the count of a search is served with it while only ten rows come back, and no cursor is accepted on the way in: the only way to see all of a population is to cut it until every slice holds ten or fewer, then check the slices sum back to the count. Values inside one filter are OR'd, so a dozen owner titles ride in a single call
pitfalls  a mobile bought ahead of the call is a mobile paid for nothing
          filtering owners by job function empties the result on small firms — their current position rarely carries one. Filter on the title text instead
          a place filter matches on substring, so a department name drags in every neighbour whose name contains it: re-read the place on each row returned
          a title filter reads only the position shown as current, so an owner whose main listed job is elsewhere stays invisible
          match a company on single words, never on a phrase: a trade name says the trade its own way, and asking for the two words together loses the ones that say it in one or in a pun. Cast wide and judge the row
          bulk export of a search is charged per contact, unlike the search itself
tested    2026-08 — an email batch submitted and read back; the balance endpoint never
          2026-09-05 — ~700 free people searches, 431 owners of driving schools across 80 departments, nothing charged
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
gives     whether an address is a business or a residence; the establishment record: phone, website, rating, review count, business status, and the trade category it carries; the trade sign, which a register record never holds; the count of establishments found under a name
access    two doors, and a door is not a price. Browser, free: https://www.google.com/maps/search/<name>+<address> lands on a single record when the name matches the address, and the record's site link carries the domain in its href · per company. API, paid: Address Validation and Places (New) text search · GOOGLE_MAPS_API_KEY from a Google Cloud project with both APIs enabled and billing on · per query, a text search by trade and place, to seed rows; per company, the record behind an address; a field mask is mandatory and the bill depends on it
limits    the API is paid per call: named selection, capped. The browser is free and slow, about twenty seconds a record, a handful to a batch; a batch stops on its first failing action, so phrase the read so it always matches something (the site link, or the button offering to add one)
pitfalls  the address is what kills homonymy, the name alone never does: search the two together or the record is a coin toss
          the category on the record is the cheapest proof of trade there is, cheaper than fetching the site and reading it
          no record at the address is a fact worth writing, not a failure: a third of a local trade has neither site nor record
          the link the record shows is often not a site: an Instagram page, a Facebook page, a directory entry, a comparator. Read the host before writing it
          the trade sign on the record is not the legal name, and is often the only name a searchable web knows the company by
tested    2026-09 — the browser door, 114 companies searched by name + address: 73 domains, 41 records genuinely without a site
```

## hellowork

```
gives     job offers from a public search, with the company name of the offer
access    the public search pages, server-rendered · per query, the queries of the target; the company name matched against name and acronym, exact or by a prefix of five characters
tested    never
```

## hiring-cafe

```
gives     job postings gathered from many boards and careers pages, with the employer name, the title, the location and the posting date; one row per posting
access    the site's search, hiring.cafe, by role with date and location filters, in the browser · per query, never per company · the employer name is the only key: matched on name, then confirmed by the SIREN read on the employer's own site, the aggregator gives no identifier; no keyless endpoint recorded yet, the first script to use one writes it here
          mirrors the job-boards/hiring-cafe.md file of the jobhunt plugin, which reads the same site for postings
limits    not measured; a query pass feeds a matching index, never a per-company witness
pitfalls  the legal name is not the trade name: a publisher's postings carry the brand, the registry carries the company; confirm on the SIREN cited on the site, never reject on the name
          a posting reveals the company, not the fit: judge the posting, not the employer, a good company whose posting says nothing useful comes back the day it opens a different role
          the location and the date shown are the board's own enrichment, not the posting's: read them on the employer's page
          the same posting can be listed twice the same day: dedup on employer + title + date
tested    2026-09-01 — one pass by hand in the browser, a few thousand postings reduced to a few hundred independent companies of the target's size band, nearly all with a SIREN established by proof; no script
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
gives     two things on one account: a people database searched by title and seniority, free; an email and phone finder, paid per hit
access    two doors on the same account, and a door is not a price: the session MCP and the API with LEMLIST_API_KEY both reach the database and the finder. What costs is the operation, never the channel. MCP: the database per company, by name or domain, title and seniority from the persona; the finder per person with enrich_lead and bulk_enrich_data, read back with bulk_get_enrichment_results. API: POST https://api.lemlist.com/api/v2/enrichments/bulk (up to 500, enrichmentRequests find_email · verify · find_phone), then GET https://api.lemlist.com/api/enrich/<enrichId> · per person, name and domain
          balance: GET https://api.lemlist.com/api/team/credits → {"credits": n, "details": {"remaining": …}}, read before every estimate
limits    the finder is paid whichever door it goes through, nothing charged on a miss: 5 credits an email found, 1 an email verified, 20 a phone, 1 a LinkedIn profile enriched (help centre, read 2026-09-04); a credit is 1 cent, the host's connections.md records the rate actually paid. Free: the database search and reading the account — free is what the operation is, not what the connector is
pitfalls  the finder matches on the name when the domain does not pin it down, and returns a stranger of the same name on another domain: send a domain, and refuse any address whose host is not the domain sent or its own root under another TLD
          an address host carried by several distinct companies is a supplier's, not theirs — the trade's software vendor, its portal, its franchise head office. The same rule as for domain, on the local part's right side
          a returned phone object can carry notFound: false and no number at all: the batch message states how many rows returned no data, and it is what counts
          an email found is not an email that can be mailed: read the verification status, and keep risky out of a first campaign
tested    2026-09 — find_email + verify on 82 owners of micro-companies, each with a proven domain: 21 addresses returned, 11 of them a stranger's; find_phone on 7 of the survivors, the verified email in the input, returned nothing. Identifying the person better does not conjure a mobile the provider does not hold
```

## linkedin

```
gives     a person's profile URL, headline and city; the title and the current role; the headcount and the People tab of a company page
access    the available browser, on the user's own account, read only · per person, the people search: /search/results/people/?keywords=<first last> · per company, on a named selection
limits    about 150 profiles and 60 company pages a day, 8 to 25 seconds between two, stop at the first captcha and resume the next day
pitfalls  a board mandate is confirmed as a contact only when the profile shows an operational role
tested    2026-09 — 18 people searched in one session on a shortlist, no captcha, no profile opened
```

**The results page is the unit of work, not the profile.** One search per person returns, for every
hit, the name, the headline, the city and the `/in/` URL — everything a match needs. Opening a
profile costs a request against the day's quota and adds nothing when the headline already names
the trade or the company. Reserve it for a hit whose headline is empty.

The pass, per person:

1. navigate to the people search on `"<first name> <last name>"`, exactly as the register spells it;
2. read the results page and judge each hit on three things, in this order — **the company of the
   row named in the headline** (decisive), **the trade** (decisive), **the city or its basin**
   (suggestive only). A hit with none of the three is someone else, whatever the name;
3. take the URL of the one hit that passes. Write the others off by name in the proof, so the next
   pass does not buy them back.

**A name with no hit is an answer**, and a cheap one: "Aucun résultat" on an exact French name is
a dated empty witness worth writing.

**What the search engines cannot replace** (measured 2026-09): DuckDuckGo and Bing **do not index
`/in/` profiles** — zero results on a positive control, and Bing silently drops `site:` and the
quotes, so its answer looks like a result and is not one. Google indexes them but walls the browser
session after about fifteen rapid queries. A people-database connector (fullenrich, lemlist) is the
free first rank, and it is complementary rather than redundant: its index has holes and it returns
homonyms, so it misses people the search finds, and the search disproves candidates it returns.
Run the connector first, the search on what it leaves.

**Read the results page as text, and resolve each link by its label.** That is enough to get the
headline, the city and the `/in/` URL, and it holds whatever drives the browser — scripted reads of
the page's own objects are the first thing to break, and the least portable.

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

## pages-jaunes

```
gives     a phone number and a postal address for an establishment, from its public listing
access    the public directory pages, per company, no key; matched on the trade name and the postcode, never on the legal name alone; the listing URL is the proof
limits    a listing is an establishment, not a company: a company with several branches has several listings, and the number that answers is the branch's, not the head office's; terms not checked here, low volume, last fallback only
pitfalls  a generic trade name matches the wrong listing: require the postcode to agree before keeping the number
          the number shown is often a call-tracking number that stops working: date every reading, and treat one older than six months as stale rather than wrong
tested    2026-08-29 — the last fallback for phone on a table where the registry and the site had given nothing
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
          a portal, a directory or a comparator of the trade speaks the trade better than any of its companies and belongs to none of them: the trade words on a page prove the subject, never the owner
          an exact homonym passes every text check (a short generic domain for a company of the same short name): the audit lists suspects by proof strength, SIREN on the page > strong key cited > domain root derived from the name
          the SIREN is cited as nine digits or as three groups of three
          B2B and B2C counters are read on the home page alone; the thresholds are calibrated on it
tested    2026-08 — the three-page scan, the legal-page SIREN, phone and markers, on a full table
```

## wttj

```
gives     job offers with the organisation name, contract, remote policy and posting date
access    the site's Algolia index, search-only keys served to every visitor in window.env of any public company page (ALGOLIA_APPLICATION_ID, ALGOLIA_API_KEY_CLIENT), set as WTTJ_ALGOLIA_APP and WTTJ_ALGOLIA_KEY, never in the repo; a Referer header of the site's domain is required by the key · per company, the index searched by organisation name, the name settles the match
limits    named shortlist only; the site's pages sit behind a JavaScript challenge and are unreadable to a plain client
pitfalls  the key allows restricting searchable attributes only on a whole group of the same priority
tested    2026-08 — offers matched to a shortlist
```
