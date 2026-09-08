# DFX Real Estate Intelligence: MCP server

DFX answers dated questions about two things: **United States commercial and
federal-programme real estate debt**, where loan maturities are published across
52 state codes, compliance expiries across 56 and subsidy expiries across
54, and **property records**, where 291,914 Massachusetts parcels carry
ownership and assessed value and 95,494 recorded sale instruments cover Massachusetts and New York.
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

12 tools. 11 are free, unauthenticated and permanent: no key, no signup, no
OAuth. One is priced at **$1.00 per delivered result set** and tells you so before
it charges you anything.

A tool that answers "no" clearly is worth more to an agent than one that answers an
empty list, so this server refuses unknown arguments with the served vocabulary
attached, and refuses to sell you a result set that would arrive empty.

> Every number on this page is measured against production, not typed. Last measured
> **2026-09-08**. Call `dfx_coverage` for the same grid at the moment you read it.

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

---

## The 12 tools

| Tool | Takes | Returns | Price |
|---|---|---|---|
| `resolve_address` | address, city?, state? | canonical DFX ids with the match basis and any ambiguity | free |
| `resolve_organization` | name | entity ids for owners, managers, lenders, servicers | free |
| `get_property_record` | a DFX id | state, dated events, relationships, debt with maturity dates, recorded sales, provenance | free |
| `search_property_events` | event_type?, state?, within_days? | dated events with provenance | free |
| `search_parcels` | filters | parcels by attribute rather than by an address you already knew | free |
| `what_can_dfx_answer` | an objective, in natural language | whether DFX can help, which tool to call, the arguments, and a free sample | free |
| `changes_since` | an opaque cursor | what DFX has **learned** since your cursor | free |
| `debt_maturity_schedule` | state, within_days?, limit? | the loan tape: principal, lender, instrument, maturity, secured property | **$1.00** |
| `open_dfx_account` | an email address | an account key for the one paid tool, issued in the response | free |
| `fund_dfx_account` | an account key and an amount | a funding link a person completes once, after which the agent spends inside the balance | free |
| `dfx_payment_status` | an account key | balance, ceilings and what has been spent | free |
| `dfx_coverage` | nothing | measured coverage, served sources, object types, known gaps | free |

**Start with `what_can_dfx_answer`** if you do not know what to ask for. It says no
clearly when the answer is no, and it records the ask, so questions DFX cannot answer
shape what gets built next.

---

## What is actually in here

Two populations that barely overlap, and conflating them is the most common way to
misread this server.

| Object | What it is | Resolvable |
|---|---|---|
| `parcel` | Massachusetts. The municipal assessor and registry layer, carrying assessed value, land use and recorded sales. | 291,914 |
| `property` | National. Federal programme multifamily: HUD, LIHTC and FHA. | 90,278 |
| `organization` | Owners, managers, lenders and servicers. | not counted separately |

An address may return one, the other, or both.

### Recorded sales

Massachusetts and New York: 95,494 instruments over 118,584 property links.

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

74,409 publishable events across 14 types and 13 sources.

| Event type | States | Published |
|---|---|---|
| `PROPERTY_SOLD` | 2 | 43,857 |
| `COMPLIANCE_PERIOD_ENDING` | 56 | 11,956 |
| `SUBSIDY_CONTRACT_EXPIRING` | 54 | 4,721 |
| `PERMIT_ISSUED` | 1 | 4,203 |
| `LOAN_MATURITY_SCHEDULED` | 52 | 3,422 |
| `CERTIFICATE_OF_OCCUPANCY` | 1 | 2,768 |
| `LEASE_EXPIRING` | 49 | 1,428 |
| `DEMOLITION_FILED` | 1 | 881 |
| `USE_CONVERSION_PERMITTED` | 1 | 849 |
| `DISTRESS_FLAG_RAISED` | 26 | 167 |
| `FORECLOSURE_EVENT` | 22 | 128 |
| `PERMIT_STATUS_CHANGED` | 0 | 13 |
| `LOAN_MODIFIED` | 5 | 12 |
| `BANKRUPTCY_EVENT` | 4 | 4 |

`CERTIFICATE_OF_OCCUPANCY`, `DEMOLITION_FILED`, `PERMIT_ISSUED`,
`USE_CONVERSION_PERMITTED` are Massachusetts only. `COMPLIANCE_PERIOD_ENDING`,
`LEASE_EXPIRING`, `LOAN_MATURITY_SCHEDULED`, `SUBSIDY_CONTRACT_EXPIRING` are national.
Multi-state, with the number of states each reaches: `DISTRESS_FLAG_RAISED` (26),
`FORECLOSURE_EVENT` (22), `LOAN_MODIFIED` (5), `BANKRUPTCY_EVENT` (4), `PROPERTY_SOLD`
(2). `PERMIT_STATUS_CHANGED` carries rows that resolve to no state at all, so a state
filter cannot reach it.

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

**How many rows your dollar actually buys.** Of the 19,881 loans, 1,782 mature inside
the default 548-day window, and they are not evenly spread. Measured 2026-09-08:

| State | Loans maturing in the next 548 days |
|---|---|
| CA | 333 |
| NY | 197 |
| TX | 121 |
| FL | 101 |
| OH | 67 |
| GA | 62 |
| IL | 56 |
| MI | 56 |
| PA | 55 |
| NJ | 51 |
| NV | 43 |
| VA | 41 |
| IN | 37 |
| NC | 34 |
| WA | 34 |
| AZ | 30 |
| CO | 28 |
| LA | 24 |
| MD | 21 |
| SC | 21 |
| AL | 20 |
| MO | 20 |

26 further states hold between 1 and 18 loans in that window; Montana and Wyoming hold
1. Widen `within_days` to reach further out; the price does not move with the row count
or the window.

121 of those 1,782 carry no single state: a loan secured by several
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

- One street address can carry several records. Measured across 5,807 such clusters:
  2,685 agree on unit count and are plausibly one asset registered by more than one
  programme, while 3,122 report DIFFERENT unit counts and are probably genuinely different
  buildings at one address, such as a scattered-site development. DFX has merged none of
  them and resolve() says which case you are looking at rather than choosing.

- Property and parcel are separate populations that barely overlap: 661 clean one-to-one
  pairs out of roughly 100,000 each. An address may resolve to one, the other, or both,
  and they are returned as distinct typed objects rather than merged.

- PROPERTY RECORDS ARE NOT ONE ROW PER BUILDING. 90,278 published property records cover
  83,664 distinct normalised addresses, so a total computed across them overstates by
  roughly 8%. 245 Park Avenue is one tower and thirteen records, because thirteen
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

---

## Questions this server is good at

- Which commercial mortgages in this state mature in the next 548 days, who lent, and against which building?
- What has DFX learned since I last asked? (`changes_since`, cursor-based, ordered by when DFX came to know a fact rather than when the fact occurred.)
- Which LIHTC compliance periods and HUD subsidy contracts are expiring, and where?
- What did this parcel last sell for, to whom, and under which book and page?
- Who owns, manages or lends against this building?

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

`boston_assessing`, `boston_permits`, `fdic_financials`, `ffiec_ubpr`, `fhfa_pudb_mf`, `hmda_lar`, `hud_fha_multifamily`, `hud_lihtc`, `hud_multifamily_arcgis`, `hud_psh`, `massgis_l3`, `nyc_acris`, `sec_abs_ee`

### Required attribution

Some served sources are published under terms that ask to be named. Carry these notices with any republished row:

- **`nyc_acris`**: Source: NYC Department of Finance ACRIS, via NYC Open Data. Include the dataset version and any modifications DFX has made.

## Terms

The example code in `examples/` is MIT licensed. The data served by the endpoint is
not: it is derived from public federal and municipal sources under DFX's own
processing, and is served for use, not for redistribution as a dataset. Ask if you
want something broader; the answer is often yes.

Operated by DFX Intelligence. Developer reference: <https://dfxintel.com/ai/real-estate-mcp>
