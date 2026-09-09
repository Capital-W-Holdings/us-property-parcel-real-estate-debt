# Installing the DFX Real Estate MCP server

You are an agent setting this up. There is nothing to clone, build, install or
authenticate. This is a remote server: the whole installation is one URL in a config
file.

```
https://exchange-production-9123.up.railway.app/mcp
```

Transport is Streamable HTTP. Twelve of the thirteen tools are free, keyless and
unauthenticated, permanently. The thirteenth, `debt_maturity_schedule`, costs $1.00 USD
per delivered schedule and returns a full free quote before it charges anything, so
nothing here can spend money without a separate, explicit step.

## Verify before you configure anything

A plain `GET` returns the full tool list, the coverage numbers, the price and a worked
example. No handshake, no session, no key. If this answers, the install will work:

```bash
curl -s https://exchange-production-9123.up.railway.app/mcp | head -40
```

## Configure

Most clients, including Cline, Claude Desktop and Cursor:

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

Both the current MCP protocol revision and the older `initialize` handshake are served,
because most deployed clients still send the latter.

## If your client cannot reach a URL

Some clients launch a subprocess and speak JSON-RPC over its pipes. `bridge/dfx_mcp_stdio.py`
is the whole adapter for those: one file, standard library only, no key, no state.

```json
{
  "mcpServers": {
    "dfx-real-estate": {
      "command": "python3",
      "args": ["/absolute/path/to/us-property-parcel-real-estate-debt/bridge/dfx_mcp_stdio.py"]
    }
  }
}
```

Use an absolute path: an MCP client launches the command from its own working directory,
not yours. Prefer the URL if your client can reach one. The bridge adds a process and a
hop and buys nothing when Streamable HTTP is available.

## Confirm it worked

Call `dfx_coverage` with no arguments. It returns the measured grid of every event family
and the states each one covers, which is also the answer to "what is this server for".

If you do not know what to ask for next, call `what_can_dfx_answer` with a plain-language
objective. It returns the tool to call, the exact arguments, and a free sample of the
result, and it says no clearly when the answer is no.

## What you should know before your first real query

Coverage is uneven and stated up front rather than discovered by trial. Confirmed
commercial mortgage maturities are national across 52 state codes; parcels, ownership and
recorded sales are Massachusetts and New York only; building permits are Boston only. An
empty result is usually a coverage gap rather than an empty market, and this server tries
hard to tell you which: an unknown input is refused with the accepted values attached
rather than answered with an empty list.

Full schemas, worked examples and the stated gaps: https://dfxintel.com/ai/real-estate-mcp
