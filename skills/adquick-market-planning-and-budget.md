---
generated: '2026-08-13'
method: generated
name: Size a market and set an OOH budget
description: Recommend markets against an audience, read market-level CPM and budget thresholds, and size spend before committing.
api: mcp/adquick-mcp.yml
operations: [get_audience, recommend_markets, get_location_metrics, get_location_budget, get_supplier_metrics, get_supplier_bookings, generate_market_budget_export]
source: >-
  Tool names verified verbatim in mcp/adquick-mcp.yml (searched from
  https://www.adquick.com/mcp/usage-documentation). Every tool in this skill is MCP-only - none has a
  documented partner REST equivalent (see mcp/adquick-tool-crosswalk.yml).
---

# Size a market and set an OOH budget

Everything in this flow is **MCP-only**. AdQuick's analytics surface is not exposed over the partner
REST API, so an agent connected to `https://www.adquick.com/mcp` can do market planning that a REST
integration cannot.

## Auth
- OAuth 2.0 authorization_code + PKCE (S256). Scopes: `inventory_read`, `supplier_inventory`.
  See `scopes/adquick-scopes.yml`.

## Steps
1. **Resolve the audience** — `get_audience` with keywords to find matching audience segments.
2. **Recommend markets** — `recommend_markets` against audience fit and budget. This is AdQuick's
   AI-assisted recommendation, not a lookup.
3. **Read the market economics** — `get_location_metrics` for market-level CPM, pricing, unit counts
   and supplier counts.
4. **Set the spend tier** — `get_location_budget` returns entry, saturation and domination budgets
   per market. Use these as the bracket for `target_budget_cents` when the campaign is created.
5. **Check the supply side** — `get_supplier_metrics` (unit counts, impressions, CPM, market coverage
   per supplier) and `get_supplier_bookings` (historical booking/revenue by supplier) to see who
   actually has inventory in that market.
6. **Export the plan** — `generate_market_budget_export` produces the market budget data as CSV for
   the media plan.

## Reference data
- `inventory://markets` — all available DMA markets
- `inventory://media-types` — media types and subtypes
- `inventory://suppliers` — supplier directory

## Errors
- An expired or missing token returns
  `{"error":"unauthorized","error_description":"The access token is invalid"}` with HTTP 401.
  Re-authorize; do not retry with the same token.

## Notes
- Budget figures are returned in the tool response; the REST campaign field is
  `target_budget_cents` (integer, USD) — convert before creating the campaign.
- `generate_*_export` tools are write-classed (they produce an artifact), so an agent operating
  read-only should stop at step 5.
