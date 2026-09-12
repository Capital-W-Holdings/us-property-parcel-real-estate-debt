# DFX Intelligence: MCP server

[![DFX Intelligence MCP server: quality and maintenance score on Glama](https://glama.ai/mcp/servers/Capital-W-Holdings/us-property-parcel-real-estate-debt/badges/score.svg)](https://glama.ai/mcp/servers/Capital-W-Holdings/us-property-parcel-real-estate-debt)

**One connection, four domains.** Real estate is the first and deepest; the same server also answers across family offices, independent sponsors and their capital providers, private companies with a transition coming, and venture capital, with cross-domain identity, relationships, events, matching and verification. See [Beyond real estate](#beyond-real-estate-family-offices-independent-sponsors-venture-capital).

The real estate domain answers dated questions about two things: **United States commercial and
federal-programme real estate debt**, where loan maturities are published across
52 state codes, compliance expiries across 56 and subsidy expiries across
54, and **property records**, where 291,914 Massachusetts and New York parcels carry
ownership and assessed value and 95,562 recorded sale instruments cover Massachusetts and New York.
Call it when an agent needs to know who owns a specific building, what it last sold
for, or which loans and subsidies come due in a given state and time window, with the
source and the observation date attached to every claim.

**Coverage is deliberately uneven and it is stated up front rather than discovered by
trial.**

United States, unevenly. NATIONAL: federal programme debt and maturities, LIHTC, HUD
subsidy, distress and commercial tenancy. MASSACHUSETTS ONLY: parcels and ownership.
RECORDED SALES: Massachusetts statewide, plus New York City deeds at or above $10m.
BOSTON ONLY: permits and certificates of occupancy. coverage_by_event_type below is the
measured grid, per event family, per state.

The measured per-type, per-state numbers are in
[Event coverage, measured](#event-coverage-measured) below, and `dfx_coverage` returns
the same grid at call time so an agent never has to guess from an empty result.

**Endpoint:** `https://exchange-production-9123.up.railway.app/mcp`
**Transport:** Streamable HTTP
**Auth:** none, for everything except one paid tool
**Registry:** `io.github.Capital-W-Holdings/us-property-parcel-real-estate-debt`

39 tools. 38 are free, unauthenticated and permanent: no key, no signup, no
OAuth. One is priced at **$1.00 per delivered result set** and tells you so before
it charges you anything.

A tool that answers "no" clearly is worth more to an agent than one that answers an
empty list, so this server refuses unknown arguments with the served vocabulary
attached, and refuses to sell you a result set that would arrive empty.

> Every number on this page is measured against production, not typed. Last measured
> **2026-09-12**. Call `dfx_coverage` for the same grid at the moment you read it.

---

## Read the schemas before you call anything

A plain `GET` on the endpoint returns the full tool list, the coverage numbers, the
price and a worked example. No handshake, no session, no `initialize`.

```bash
curl -s https://exchange-production-9123.up.railway.app/mcp
```

Then call a tool over JSON-RPC:

```bash
curl -s https://exchange-production-9123.up.railway.app/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":
       {"name":"resolve_address","arguments":
        {"address":"100 Binney St","city":"Cambridge","state":"MA"}}}'
```

That address returns a parcel carrying a recorded sale, with the registry book and
page it was recorded under.

### If you are a harvester rather than a caller

The endpoint serves the discovery conventions from its own origin, so an index does
not have to guess and does not have to be told:

| path | what it is |
|---|---|
| [`/llms.txt`](https://exchange-production-9123.up.railway.app/llms.txt) | the entry point, for something holding only this host name |
| [`/.well-known/ard.json`](https://exchange-production-9123.up.railway.app/.well-known/ard.json) | Agentic Resource Discovery catalog |
| [`/.well-known/mcp/server-cards.json`](https://exchange-production-9123.up.railway.app/.well-known/mcp/server-cards.json) | MCP server card |
| [`/.well-known/agent-card.json`](https://exchange-production-9123.up.railway.app/.well-known/agent-card.json) | A2A style agent card |
| [`/agents.txt`](https://exchange-production-9123.up.railway.app/agents.txt) | the agents.txt convention |
| [`/openapi.json`](https://exchange-production-9123.up.railway.app/openapi.json) | the same capabilities over plain HTTP |
| [`/robots.txt`](https://exchange-production-9123.up.railway.app/robots.txt) | crawlers and agents are welcome, and it says so |

`agents.txt` names no payment protocol. This server has a priced tool and does not
speak x402, mpp or ap2: it quotes, mints an account and settles through Stripe.

---

## The 39 tools

| Tool | Takes | Returns | Price |
|---|---|---|---|
| `resolve_address` | address, city?, state? | canonical DFX ids with the match basis and any ambiguity | free |
| `resolve_organization` | name | entity ids for owners, managers, lenders, servicers | free |
| `get_occupancy` | a property or company id, or a company name with an address | who is observed to occupy a building, or where a company operates, with the evidence tier | free |
| `get_property_record` | a DFX id | state, dated events, relationships, debt with maturity dates, recorded sales, provenance | free |
| `search_property_events` | event_type?, state?, within_days? | dated events with provenance | free |
| `search_bank_cre_exposure` | state?, name?, CRE-to-equity range?, above_guidance?, min_assets_usd?, min_noncurrent_pct?, sort? | FDIC-insured banks by CRE concentration with the guidance screen and UBPR percentile ranks | free |
| `search_subsidised_housing` | state?, city?, program?, min_waiting_months?, occupancy range?, min_units? | HUD-subsidised projects with units available, occupancy, months on the waiting list, rent, income and HUD spend, one annual capture | free |
| `search_parcels` | filters | parcels by attribute rather than by an address you already knew | free |
| `what_can_dfx_answer` | an objective, in natural language | whether DFX can help, which tool to call, the arguments, and a free sample | free |
| `changes_since` | an opaque cursor | what DFX has **learned** since your cursor | free |
| `debt_maturity_schedule` | state, within_days?, limit? | the loan tape: principal, lender, instrument, maturity, secured property | **$1.00** |
| `open_dfx_account` | an email address | an account key for the one paid tool, issued in the response | free |
| `fund_dfx_account` | an account key and an amount | a funding link a person completes once, after which the agent spends inside the balance | free |
| `dfx_payment_status` | an account key | balance, ceilings and what has been spent | free |
| `dfx_coverage` | nothing | measured coverage, served sources, object types, known gaps | free |
| `search_family_offices` | asset_class?, city?, class?, cursor?, has_real_estate?, has_sponsor_relationships?, ... | Family offices as compact cards: class (single, multi, embedded...) with confidence, whether they invest directly, sectors and asset classes on record, check si... | free |
| `get_family_office` | dfx_id | The full card for one family office: profile, AUM / RAUM / 13F value kept apart with their as-of dates, behaviour, the people who run it with roles, its observe... | free |
| `search_family_office_investments` | asset_class?, investment_kind?, limit?, office_dfx_id?, sector?, since?, ... | Dated investments family offices have been observed making: target, sector, asset class, structure, control or minority, lead or participant, amounts where disc... | free |
| `search_independent_sponsors` | city?, kind?, limit?, min_confidence?, query?, sector?, ... | Independent sponsors (deal-by-deal acquirers of lower middle market companies) as compact cards: classification with confidence, mandate summary, principals, ve... | free |
| `get_independent_sponsor` | dfx_id | The full card for any entity on the sponsor graph: a sponsor (with its computed company matches and announced transactions), a capital provider (with fund size,... | free |
| `search_sponsor_capital_providers` | limit?, making_new_investments?, min_fund_size_usd?, provider_type?, query?, sbic_licensed?, ... | SBICs, mezzanine and private equity funds, and family offices observed providing capital to independent sponsors: provider type, strategy, fund style, fund size... | free |
| `search_private_companies` | city?, limit?, max_participants?, min_opportunity?, min_participants?, naics_prefix?, ... | US private companies whose filings (Form 5500 plan history, final filings, ownership changes) show a transition: vertical, plan participants as a size proxy, EB... | free |
| `search_sponsor_deals` | limit?, query?, since?, sponsor_dfx_id?, state?, target_dfx_id?, ... | Announced acquisitions, recapitalisations and exits by independent sponsors: sponsor, target, dates, enterprise value range where disclosed, structure, parties ... | free |
| `search_vc_firms` | active_only?, city?, emerging_manager?, limit?, min_investments?, query?, ... | Venture firms as compact cards: stated sectors, stages, geography and check size beside OBSERVED behaviour (investments in the last 6 and 12 months, lead count,... | free |
| `get_vc_firm` | dfx_id | The full card for any entity on the venture graph: a firm (with recent investments, co-investors and funds), a person (with attributed investments and board sea... | free |
| `search_vc_investments` | company_dfx_id?, investor_dfx_id?, lead_only?, limit?, partner_dfx_id?, query?, ... | Investor-by-investor participations in rounds: investor, company, fund, the partner attributed (with attribution level), role (lead or participant), new or foll... | free |
| `search_vc_funds` | lifecycle_state?, limit?, max_vintage?, min_form_d_sold_usd?, min_vintage?, organization_dfx_id?, ... | Funds with every amount under its own name (target, first close, final close, announced size, Form D offering and sold, ADV gross asset value), vintage and basi... | free |
| `search_entities` | active_only?, asset_class?, city?, cursor?, domain?, entity_type?, ... | One search across family offices, independent sponsors and their capital providers, private companies, venture firms and real estate organisations: by name, or ... | free |
| `get_entity` | dfx_id, event_limit?, evidence_limit?, include?, relationship_limit? | For any DFX id: the full card, published relationships with sources and dates, recent events, evidence rows (the observation each fact traces to), cross-graph s... | free |
| `search_people` | current_only?, domain?, investment_responsibility?, limit?, organization_dfx_id?, query?, ... | Investment professionals, principals and family office staff as names with titles, roles, seniority, investment responsibility, organisation and tenure, from pu... | free |
| `search_relationships` | current_only?, dfx_id, limit?, rel_type? | Every published relationship touching one entity (EMPLOYS, PRINCIPAL_OF, INVESTED_IN, CO_INVESTED_WITH, MANAGES, OWNS, BOARD_MEMBER_OF, VEHICLE_OF, ...), each w... | free |
| `relationship_path` | from_dfx_id, max_hops?, to_dfx_id | An evidence-backed path between two DFX ids across every graph: each hop is a published relationship with its source, or a SAME_AS identity link by shared CRD/C... | free |
| `search_events` | dfx_id?, domain?, event_type?, exclude_routine?, limit?, min_significance?, ... | Dated events across family offices, sponsors, venture and real estate: investments announced, vehicles formed, Form D and ADV filings, people joining and leavin... | free |
| `verify` | claim?, object?, object_dfx_id?, predicate?, subject?, subject_dfx_id?, ... | SUPPORTED, PARTIALLY_SUPPORTED, CONTRADICTED or UNKNOWN for a claim, with the observations | free |
| `find_capital_for_opportunity` | asset_class?, check_size_usd?, control?, deal_size_usd?, dfx_id?, investor_types?, ... | Ranked investors for a company (dfx_id) or a described opportunity (sector, state, deal size, stage, control): independent sponsors from the computed match plan... | free |
| `find_opportunities_for_capital` | dfx_id, limit? | For a family office, sponsor, capital provider or venture firm: the opportunities DFX knows that fit its DEMONSTRATED behaviour: computed matches where the grap... | free |
| `explain_match` | dfx_id_a, dfx_id_b | For an investor and an opportunity (either order): MATCH REASONS, BLOCKERS, SUPPORTING OBSERVATIONS, COMPARABLE HISTORY (the investor's dated investments in the... | free |
| `why_now` | dfx_id, within_days? | Evidence-backed reasons an entity matters now: recent filings, vehicles formed, deployments, people moves, fundraising, transition signals, loan maturities, eac... | free |
| `who_should_care` | dfx_id?, event_id?, limit? | Given an entity or an event id: who is likely to care and why | free |

**Start with `what_can_dfx_answer`** if you do not know what to ask for. It says no
clearly when the answer is no, and it records the ask, so questions DFX cannot answer
shape what gets built next.

---

## Beyond real estate: family offices, independent sponsors, venture capital

The same connection answers across three more DFX graphs, with one id scheme (`dfx:fo:`, `dfx:isi:`, `dfx:vc:`, and the real estate ids above), one response contract, and cross-graph identity by shared CRD, CIK or EIN only. A same-name entity on another graph is returned as a candidate, never merged.

**Family offices.** 1,398 offices on the graph (970 candidates, 64 confirmed multi-family, 64 probable single-family, 77 outsourced), 4,236 foundations, 637 offices with 13F positions, 20 with observed direct investments. A candidate is a name, never a class; AUM, RAUM and 13F value are three numbers and are never substituted for one another.

**Independent sponsors.** 1,605 sponsors, 7,332 capital providers, 55,986 private companies with Department of Labor plan-filing history of which 8,179 carry a transition signal, 497,326 computed company-to-sponsor matches with reasons and blockers, 54 announced transactions. A plan-filing signal is one year lagged.

**Venture capital.** 4,968 firms (76 with a fund raising in the last 18 months), 51,233 funds with every fund amount kept apart (2,396 with Form D sold), 26,797 people, 4,430 companies and 21,717 rounds. Stated sectors are populated on 0 firms today, so a sector filter on firms answers NOT_COVERED rather than an empty list; a round is never a check.

**Across all of them:** `search_entities`, `get_entity` (everything on one id: card, relationships, events, evidence, cross-graph links), `search_people`, `search_relationships`, `relationship_path` (how X connects to Y, every hop an evidenced edge), `search_events`, `verify` (SUPPORTED, PARTIALLY_SUPPORTED, CONTRADICTED or UNKNOWN, with the observations), and the economic tools `find_capital_for_opportunity`, `find_opportunities_for_capital`, `explain_match` (reasons and blockers, never a bare score), `why_now` and `who_should_care`.

Contact points are withheld over MCP on every graph. Call `what_can_dfx_answer` with `domain` set to any of `family_office`, `independent_sponsor`, `venture_capital` or `real_estate` for that domain's entity types, event families, rights, freshness and limitations.

---
## What is actually in here (real estate)

Two populations that barely overlap, and conflating them is the most common way to
misread this server.

| Object | What it is | Resolvable |
|---|---|---|
| `parcel` | Massachusetts. The municipal assessor and registry layer, carrying assessed value, land use and recorded sales. | 291,914 |
| `property` | National. Federal programme multifamily: HUD, LIHTC and FHA. | 101,991 |
| `organization` | Owners, managers, lenders and servicers. | not counted separately |

An address may return one, the other, or both.

### Recorded sales

Massachusetts and New York: 95,562 instruments over 118,733 property links.

| Source | Geography | Grain | Buyer | Seller | Repeat sales |
|---|---|---|---|---|---|
| `massgis_l3` | Massachusetts, statewide | assessor roster: one sale date and price per parcel | yes | **no** | **no** |
| `nyc_acris` | New York City, five boroughs | recorded instrument, grouped into economic transactions | yes | yes | yes |

- **`massgis_l3`**: An assessor roster carries the LAST sale, so repeat-sale pairs and
  hold periods are not derivable from it at any volume. A deed repeats its full
  consideration on every parcel it covers, so allocated_consideration is carried
  separately from consideration and allocation_basis says when a split is ours.

- **`nyc_acris`**: deeds at or above $10,000,000 consideration. This is a deliberate cut
  by VALUE and not by date: a date cut would orphan the earlier leg of a repeat-sale pair.
  A smaller New York sale is outside the tranche, not absent from the city.

- **`nyc_acris`**: Fourteen same-day deeds between the same parties are ONE transaction
  with fourteen instrument ids preserved, and a 318-property deed is one transaction
  linked to 318 properties. Consideration is stated once per instrument and is never split
  across its properties. No natural person is named in an event headline, on either side.

### Event coverage, measured

83,479 publishable events across 16 types, written by 9 sources on a published allowlist of 11.

| Event type | States | Published |
|---|---|---|
| `PROPERTY_SOLD` | 2 | 43,680 |
| `COMPLIANCE_PERIOD_ENDING` | 56 | 11,956 |
| `SUBSIDY_CONTRACT_EXPIRING` | 54 | 4,721 |
| `PERMIT_ISSUED` | 1 | 4,203 |
| `LEASE_EXPIRING` | 55 | 3,966 |
| `PORTFOLIO_EXPANDED` | 53 | 3,528 |
| `LOAN_MATURITY_SCHEDULED` | 52 | 3,422 |
| `PORTFOLIO_CONTRACTED` | 54 | 3,181 |
| `CERTIFICATE_OF_OCCUPANCY` | 1 | 2,768 |
| `DEMOLITION_FILED` | 1 | 881 |
| `USE_CONVERSION_PERMITTED` | 1 | 849 |
| `DISTRESS_FLAG_RAISED` | 26 | 167 |
| `FORECLOSURE_EVENT` | 22 | 128 |
| `PERMIT_STATUS_CHANGED` | 0 | 13 |
| `LOAN_MODIFIED` | 5 | 12 |
| `BANKRUPTCY_EVENT` | 4 | 4 |

`CERTIFICATE_OF_OCCUPANCY`, `DEMOLITION_FILED`, `PERMIT_ISSUED`,
`USE_CONVERSION_PERMITTED` are Massachusetts only. `COMPLIANCE_PERIOD_ENDING`,
`LEASE_EXPIRING`, `LOAN_MATURITY_SCHEDULED`, `PORTFOLIO_CONTRACTED`,
`PORTFOLIO_EXPANDED`, `SUBSIDY_CONTRACT_EXPIRING` are national. Multi-state, with the
number of states each reaches: `DISTRESS_FLAG_RAISED` (26), `FORECLOSURE_EVENT` (22),
`LOAN_MODIFIED` (5), `BANKRUPTCY_EVENT` (4), `PROPERTY_SOLD` (2).
`PERMIT_STATUS_CHANGED` carries rows that resolve to no state at all, so a state filter
cannot reach it.

---

## The one paid tool: `debt_maturity_schedule`, $1.00

Everything above is free and stays free. This one is priced, and the price is printed
in the tool description, in the tool `_meta`, and in the quote. You are never asked to
negotiate, there is no sales call, and there is no field through which a caller can
propose a price: the amount is read off the quote, server side.

**What you get.** For one US state and one forward window, up to 200 loans, one row
per loan, ordered by maturity date:

- `maturity_date` and `maturity_basis`
- `original_principal_usd`, `origination_date`, `term_months`
- `instrument_type`
- the lender's canonical name and DFX id where resolved
- the secured property: DFX id, street address, city, state, postal code, unit count, property type
- the `source_key` for that row

**Why it might be worth a dollar.** 19,881 loans carry a maturity date and **19,881 of
19,881 carry `maturity_basis = 'confirmed'`.** Not one is estimated, inferred from a
term length, or carried forward from a stale reading. Every date was filed with the SEC
by a loan servicer or recorded by HUD, and then resolved to a specific building.

The free `search_property_events` tool returns the **event**: a date, a headline, an
address. The paid one returns the **loan**: the principal, the lender, the instrument,
deduplicated to one row per loan, up to 200 rows instead of 50, with the population
stated so you can tell a complete answer from a truncated one. The two populations are
different sizes on purpose and both numbers are true: an event has to be promoted to a
single place, a loan only has to be filed, so the 19,881 loans on the tape are reached
here while 3,422 maturity events are reachable through the free search.

**How many rows your dollar actually buys.** Of the 19,881 loans, 1,792 mature inside
the default 548-day window, and they are not evenly spread. Measured 2026-09-12:

| State | Loans maturing in the next 548 days |
|---|---|
| CA | 334 |
| NY | 197 |
| TX | 123 |
| FL | 101 |
| OH | 68 |
| GA | 62 |
| IL | 57 |
| MI | 56 |
| PA | 55 |
| NJ | 52 |
| NV | 43 |
| VA | 41 |
| IN | 37 |
| NC | 34 |
| WA | 34 |
| AZ | 30 |
| CO | 28 |
| LA | 24 |
| MD | 22 |
| AL | 21 |
| SC | 21 |
| MO | 20 |

26 further states hold between 1 and 18 loans in that window; Montana and Wyoming hold
1. Widen `within_days` to reach further out; the price does not move with the row count
or the window.

123 of those 1,792 carry no single state: a loan secured by several
buildings has no property anchor, so a state filter cannot reach it. Those are reached
through the free `get_property_record`.

Ask for a state and window you are unsure about with the free `search_property_events`
first: it returns the maturity **events** for the same filter at no cost, so you can
see whether the market is there before you spend anything.

**What it does not cover, stated plainly.** These are the gaps the server itself
reports through `dfx_coverage`, reprinted here so you do not have to call it to find
them:

- Loan maturity coverage is federal programme lending only (FHA insured and agency
  backed). The Registries of Deeds are closed to automation, so conventionally financed
  property carries no debt record here. A property absent from a maturity search is NOT a
  property without debt.

- LIHTC compliance periods are statutory and every one falls on 31 December, so a count
  bucketed by day shows a December cliff that is an artefact of the statute rather than a
  market event.

- Permit and demolition coverage is the City of Boston only.

- No outcome has ever been observed for any prediction in this graph. Nothing served
  here carries a calibrated probability; every score is a ranked signal.

- One street address can carry several records. Measured across 6,114 such clusters:
  2,764 agree on unit count and are plausibly one asset registered by more than one
  programme, while 3,350 report DIFFERENT unit counts and are probably genuinely different
  buildings at one address, such as a scattered-site development. DFX has merged none of
  them and resolve() says which case you are looking at rather than choosing.

- Property and parcel are separate populations that barely overlap: 661 clean one-to-one
  pairs out of roughly 100,000 each. An address may resolve to one, the other, or both,
  and they are returned as distinct typed objects rather than merged.

- PROPERTY RECORDS ARE NOT ONE ROW PER BUILDING. 101,991 published property records
  cover 94,859 distinct normalised addresses, so a total computed across them overstates
  by roughly 8%. 245 Park Avenue is one tower and thirteen records, because thirteen
  securitisation trusts each report it. Every row is individually true, which is why the
  distortion is invisible per row. Each record carries address_group_size so you can see
  it: 1 is unique, and above 1 you should deduplicate by address before summing anything.
  DFX has not merged them because thousands of these clusters carry different unit counts
  and are genuinely different buildings at one address rather than one building recorded
  twice.

- The sale tape is two sources with different grain, and the difference decides which
  questions it can answer. MASSACHUSETTS is an assessor roster: statewide, one sale per
  parcel, buyer named and SELLER NEVER NAMED, so repeat-sale pairs and hold periods are
  not derivable from it at any volume and no further ingestion of it will change that. NEW
  YORK is a recorder extract: five boroughs, both parties named, every instrument dated,
  so repeat sales and hold periods ARE derivable, but only for deeds at or above
  $10,000,000. Neither one is a national sale tape and DFX does not have one.

### How payment works

Two round trips, on purpose.

```
1. call debt_maturity_schedule WITHOUT `authorize`
   -> PAYMENT_REQUIRED. The price, the quote id, what arrives, how many rows
      your filter holds, the known limits, and the free alternative.
      Nothing is charged for a quote.

2. call it again WITH `authorize: {quote_id, max_price_usd}` and your account
   key in the X-DFX-Account header
   -> charged once, and served in the same response, with a receipt.
```

`max_price_usd` is **your** ceiling and it is checked before ours. If the price ever
moved above it you are refused rather than charged. It can lower what you pay and can
never raise it.

A quote settles exactly once: replaying an authorized call returns `ALREADY_SETTLED`
rather than charging twice. If delivery fails after the debit, the settlement is
reversed in full in the same request and your balance is restored. You are never
charged for a result you did not receive.

The account is a **funded balance**, not a card in the request path. A person funds it
once; your agent then spends inside it with per-call, daily and account ceilings and no
further human step. An agent holding a payment instrument can create an obligation; an
agent holding a balance cannot. `open_dfx_account`, `fund_dfx_account` and
`dfx_payment_status` are themselves free tools on this server, so opening an account and
reading its balance never leave MCP and never wait on a person.


**Where the money path actually stands, in the tools' own words:**

- **`open_dfx_account`**: THE ACCOUNT STARTS AT $0.00 AND CANNOT BUY ANYTHING. DFX mints
  identity and never credit: a balance moves only when Stripe confirms a payment and DFX
  re-reads that payment from Stripe. There is no argument anywhere on this server through
  which you can propose a balance.

- **`fund_dfx_account`**: A CARD MUST STILL BE AUTHORIZED. That is the card network's
  boundary and not a DFX design choice: show the URL and the price to your human, or
  present your own payment credential to Stripe. Everything either side of that step is
  callable by a machine.

- **`fund_dfx_account`**: This build collects Stripe TEST payments only. No real money
  moves.

---

## Connect it

Any MCP client that speaks Streamable HTTP. No credentials.

```json
{
  "mcpServers": {
    "dfx-real-estate": {
      "type": "http",
      "url": "https://exchange-production-9123.up.railway.app/mcp"
    }
  }
}
```

Claude Code:

```bash
claude mcp add --transport http dfx-real-estate \
  https://exchange-production-9123.up.railway.app/mcp
```

Both the current protocol revision and the older `initialize` handshake are served,
because most deployed clients still send the latter.

### If your client only speaks stdio

Some clients launch a subprocess and speak JSON-RPC over its pipes; they have no way to
reach a URL at all. `bridge/dfx_mcp_stdio.py` is the whole adapter for those: one file,
standard library only, no key, no state. It forwards each message to the endpoint above
and writes the answer back.

```json
{
  "mcpServers": {
    "dfx-real-estate": {
      "command": "python3",
      "args": ["/path/to/us-property-parcel-real-estate-debt/bridge/dfx_mcp_stdio.py"]
    }
  }
}
```

Use an absolute path. An MCP client launches the command from its own working directory,
not yours, so a relative one will not find the file.

Use the URL directly if your client can. The bridge adds a process and a hop and buys
nothing when Streamable HTTP is available. It reads the tool list from the live server
on every `tools/list`, so an installed copy does not go stale when DFX publishes a new
event family; there is nothing in it that knows what a family is.

---

## Questions this server is good at

- Which commercial mortgages in this state mature in the next 548 days, who lent, and against which building?
- What has DFX learned since I last asked? (`changes_since`, cursor-based, ordered by when DFX came to know a fact rather than when the fact occurred.)
- Which LIHTC compliance periods and HUD subsidy contracts are expiring, and where?
- What did this parcel last sell for, to whom, and under which book and page?
- Who owns, manages or lends against this building?

### Three recipes, as an agent calls them

Each is one tool call, the arguments verbatim, and what came back when this page was
generated. Paste the call; the numbers are the wire's, not this page's.

**Where is the queue?** Subsidised projects in Ohio with a waiting list of 24 months or more, deepest first.

```json
{"tool": "search_subsidised_housing", "arguments": {"state": "OH", "min_waiting_months": 24, "limit": 10}}
```

Returns `matched: 69` and ten rows, the longest at 85 months, each with units available, occupancy, rent, household income and what HUD pays per unit, and `waiting_list_coverage` saying how many projects in the state report a list at all. One annual capture (29,455 projects nationally), stated on every row. A NULL waiting list is an absent disclosure, never an empty queue.

**Which banks are past the CRE guidance line?** FDIC-insured banks in Ohio whose total CRE exceeds 300% of equity, most concentrated first.

```json
{"tool": "search_bank_cre_exposure", "arguments": {"state": "OH", "above_guidance": true, "limit": 10}}
```

Returns `matched: 8` with each bank's CRE book against equity and assets, noncurrent and charge-off ratios and its UBPR peer and national percentile ranks. 626 of 4,313 banks nationally are over that line on this measure. The guidance tests total risk based capital and these ratios are on equity, so the row says "screen", not "finding".

**What matures, and against which building?** Securitised and FHA-insured loans on Texas property maturing inside a year, soonest first.

```json
{"tool": "search_property_events", "arguments": {"event_type": "LOAN_MATURITY_SCHEDULED", "state": "TX", "within_days": 365, "limit": 50}}
```

Returns 75 events (page with `next_cursor`), each with the building's `dfx_id`. Then, for any row, `get_property_record` with that id returns the loan itself free: current principal, interest rate, original principal, maturity and basis. The priced `debt_maturity_schedule` is the same population as one deduplicated statewide list with the lender name and a completeness figure.

### One page per question, with the measured coverage on it

- [Which commercial real-estate loans mature in a given state and window?](https://dfxintel.com/ai/real-estate-mcp/cre-loan-maturities): 3,422 LOAN_MATURITY_SCHEDULED, 52 states and territories.
- [Which LIHTC properties are reaching the end of a compliance period?](https://dfxintel.com/ai/real-estate-mcp/lihtc-year-15-data): 11,956 COMPLIANCE_PERIOD_ENDING, 56 states and territories.
- [Which HUD-subsidised properties have contracts approaching expiry?](https://dfxintel.com/ai/real-estate-mcp/hud-subsidy-expiry-data): 4,721 SUBSIDY_CONTRACT_EXPIRING, 54 states and territories.
- [Where is commercial real estate in distress, foreclosure or workout?](https://dfxintel.com/ai/real-estate-mcp/distressed-cre-data): 167 DISTRESS_FLAG_RAISED, 128 FORECLOSURE_EVENT, 12 LOAN_MODIFIED, 26 states.
- [What did this property sell for, and who owns it?](https://dfxintel.com/ai/real-estate-mcp/property-sales-and-ownership-data): 43,680 PROPERTY_SOLD, 2 states.
- [Which commercial leases are approaching expiry, and who occupies a building?](https://dfxintel.com/ai/real-estate-mcp/commercial-lease-expiry-data): 3,966 LEASE_EXPIRING, 55 states and territories.

## Questions it is not good at, and will say so

- Anything about a person. Person lookup is deliberately not offered.
- Assessor and parcel data outside Massachusetts.
- Debt on conventionally financed property.
- Anything outside the United States.

---

## Design notes an agent developer may care about

- **`changes_since` is ordered by when DFX learned a fact, not when the fact occurred.** A deed signed in March is recorded in August. Polling a date filter would show you the same rows forever.
- **An unrecognised `event_type` is refused with the served vocabulary attached**, never answered with an empty list, because an empty list reads as an absent market.
- **A name is a blocking key, never an identity.** `resolve_organization` returns all candidates rather than guessing one.
- **Every returned fact carries its provenance**: the source, the evidence class, and for sales the registry book and page.
- **Coverage is a tool, not a footnote.** Call `dfx_coverage` before concluding that an empty result means an absent market.

### Served sources

`boston_assessing`, `fdic_financials`, `ffiec_ubpr`, `fhfa_pudb_mf`, `hud_fha_multifamily`, `hud_lihtc`, `hud_multifamily_arcgis`, `hud_psh`, `massgis_l3`, `nyc_acris`, `sec_abs_ee`

### Required attribution

Some served sources are published under terms that ask to be named. Carry these notices with any republished row:

- **`nyc_acris`**: Source: NYC Department of Finance ACRIS, via NYC Open Data. Include the dataset version and any modifications DFX has made.

## Terms

The example code in `examples/` is MIT licensed. The data served by the endpoint is
not: it is derived from public federal and municipal sources under DFX's own
processing, and is served for use, not for redistribution as a dataset. Ask if you
want something broader; the answer is often yes.

Operated by DFX Intelligence. Developer reference: <https://dfxintel.com/ai/real-estate-mcp>
