# DFX Real Estate Intelligence: MCP server

Machine-callable United States property, parcel, ownership, recorded-sale and
commercial mortgage intelligence, over the Model Context Protocol.

**Endpoint:** `https://exchange-production-9123.up.railway.app/mcp`
**Transport:** Streamable HTTP
**Auth:** none, for everything except one paid tool
**Registry:** `io.github.Capital-W-Holdings/us-property-parcel-real-estate-debt`

Nine tools. Eight are free, unauthenticated and permanent: no key, no signup, no
OAuth. One is priced at **$1.00 per delivered result set** and tells you so before
it charges you anything.

Coverage is uneven on purpose and the gaps are printed below rather than buried.
A tool that answers "no" clearly is worth more to an agent than one that answers
an empty list, so this server refuses unknown arguments with the served vocabulary
attached, and refuses to sell you a result set that would arrive empty.

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

That address returns a parcel carrying a $1,020,000,000 recorded sale, with the
registry book and page it was recorded under.

---

## The nine tools

| Tool | Takes | Returns | Price |
|---|---|---|---|
| `what_can_dfx_answer` | an objective, in natural language | whether DFX can help, which tool to call, the arguments, and a free sample | free |
| `dfx_coverage` | nothing | measured coverage, served sources, object types, known gaps | free |
| `resolve_address` | address, city?, state? | canonical DFX ids with the match basis and any ambiguity | free |
| `resolve_organization` | name | entity ids for owners, managers, lenders, servicers | free |
| `get_property_record` | a DFX id | state, dated events, relationships, debt with maturity dates, recorded sales, provenance | free |
| `search_parcels` | filters | parcels by attribute rather than by an address you already knew | free |
| `search_property_events` | event_type?, state?, within_days? | dated events with provenance | free |
| `changes_since` | an opaque cursor | what DFX has **learned** since your cursor | free |
| `debt_maturity_schedule` | state, within_days?, limit? | the loan tape: principal, lender, instrument, maturity, secured property | **$1.00** |

**Start with `what_can_dfx_answer`** if you do not know what to ask for. It says no
clearly when the answer is no, and it records the ask, so questions DFX cannot answer
shape what gets built next.

---

## What is actually in here

Two populations that barely overlap, and conflating them is the most common way to
misread this server.

| Object | Geography | What it is |
|---|---|---|
| `property` | **National** | Federal-programme multifamily: HUD, LIHTC and FHA. 90,278 resolvable. |
| `parcel` | **Massachusetts only** | The municipal assessor and registry layer, with assessed value, land use and recorded sales. 269,984 resolvable. |
| `organization` | National | Owners, managers, lenders and servicers. |

An address may return one, the other, or both.

Recorded sales are **Massachusetts only**: 80,448 instruments over 89,191 parcel links.
A deed repeats its full consideration on every parcel it covers, so
`allocated_consideration` is carried separately from `consideration`, and
`allocation_basis` tells you when a split is ours rather than the registry's.

### Event coverage, measured

62,524 publishable events across 16 types and 13 sources.

| Event type | States | Published |
|---|---|---|
| `PROPERTY_SOLD` | 1 | 30,055 |
| `COMPLIANCE_PERIOD_ENDING` | 56 | 13,549 |
| `SUBSIDY_CONTRACT_EXPIRING` | 54 | 4,721 |
| `PERMIT_ISSUED` | 1 | 4,203 |
| `LOAN_MATURITY_SCHEDULED` | 52 | 3,717 |
| `CERTIFICATE_OF_OCCUPANCY` | 1 | 2,768 |
| `LEASE_EXPIRING` | 49 | 1,432 |
| `DEMOLITION_FILED` | 1 | 881 |
| `USE_CONVERSION_PERMITTED` | 1 | 849 |
| `DISTRESS_FLAG_RAISED` | 26 | 188 |
| `FORECLOSURE_EVENT` | 22 | 142 |
| `LOAN_MODIFIED` | 5 | 12 |
| `BANKRUPTCY_EVENT` | 4 | 4 |
| `COMPANY_CONTRACTED` | 1 | 1 |
| `LEASE_TERM_REVISED` | 1 | 1 |
| `PERMIT_STATUS_CHANGED` | 0 | 1 |

`PROPERTY_SOLD`, `PERMIT_ISSUED`, `CERTIFICATE_OF_OCCUPANCY`, `DEMOLITION_FILED` and
`USE_CONVERSION_PERMITTED` are Massachusetts. `COMPLIANCE_PERIOD_ENDING`,
`SUBSIDY_CONTRACT_EXPIRING` and `LOAN_MATURITY_SCHEDULED` are national.

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
stated so you can tell a complete answer from a truncated one.

**How many rows your dollar actually buys.** Of the 19,881 loans, 1,614 mature inside
the default eighteen-month window, and they are not evenly spread. Measured 2026-09-01:

| State | Loans maturing in the next 548 days |
|---|---|
| CA | 329 |
| NY | 193 |
| TX | 117 |
| FL | 97 |
| OH | 61 |
| GA | 58 |
| IL | 56 |
| NJ | 53 |
| PA | 52 |
| MI | 50 |
| NV | 43 |
| IN | 40 |
| VA | 38 |
| NC | 34 |
| WA | 31 |
| CO | 26 |
| AZ | 24 |
| LA | 24 |
| AL | 20 |
| MD | 20 |
| SC | 20 |
Twenty-eight further states hold between 1 and 19 loans in that window. Montana and
Wyoming hold one each. Widen `within_days` to reach further out; the price does not
move with the row count or the window.

Ask for a state and window you are unsure about with the free `search_property_events`
first: it returns the maturity **events** for the same filter at no cost, so you can
see whether the market is there before you spend anything.

**What it does not cover, stated plainly.** Private-label CMBS and FHA-insured
multifamily only. Bank balance-sheet lending, agency multifamily and county-registry-only
loans are absent. A conventionally financed building can carry debt this schedule will
never show. **A property absent from a maturity search is not a property without debt.**
A loan secured by several buildings has no single property anchor and is therefore not
reachable by a state filter; those are reached through the free `get_property_record`.

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
agent holding a balance cannot.

> **Current status, stated honestly:** self-serve funding is not open yet. If you want
> an account, open an issue on this repository or write to `jesse@uemembers.com`.
> Every free tool works right now with no account and no key.

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

- Which commercial mortgages in this state mature in the next eighteen months, who lent, and against which building?
- What has DFX learned since I last asked? (`changes_since`, cursor-based, ordered by when DFX came to know a fact rather than when the fact occurred.)
- Which LIHTC compliance periods and HUD subsidy contracts are expiring, and where?
- What did this Massachusetts parcel last sell for, to whom, and under which book and page?
- Who owns, manages or lends against this building?

## Questions it is not good at, and will say so

- Anything about a person. Person lookup is deliberately not offered.
- Parcel, assessor or recorded-sale data outside Massachusetts.
- Debt on conventionally financed property.
- Anything outside the United States.

---

## Design notes an agent developer may care about

- **`changes_since` is ordered by when DFX learned a fact, not when the fact occurred.** A deed signed in March is recorded in August. Polling a date filter would show you the same rows forever.
- **An unrecognised `event_type` is refused with the served vocabulary attached**, never answered with an empty list, because an empty list reads as an absent market.
- **A name is a blocking key, never an identity.** `resolve_organization` returns all candidates rather than guessing one.
- **Every returned fact carries its provenance**: the source, the evidence class, and for sales the registry book and page.
- **Coverage is a tool, not a footnote.** Call `dfx_coverage` before concluding that an empty result means an absent market.

## Terms

The example code in `examples/` is MIT licensed. The data served by the endpoint is
not: it is derived from public federal and municipal sources under DFX's own
processing, and is served for use, not for redistribution as a dataset. Ask if you
want something broader; the answer is often yes.

Operated by DFX Intelligence. Developer reference: <https://dfxintel.com/ai/real-estate-mcp>
