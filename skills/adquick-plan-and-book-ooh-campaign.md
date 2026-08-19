---
generated: '2026-08-13'
method: generated
name: Plan and book an out-of-home campaign
description: Discover OOH inventory in the right markets, assemble a campaign, attach creative, and book it.
api: mcp/adquick-mcp.yml
operations: [search_locations, list_media_types, search_inventory, check_availability, create_campaign, add_units_to_campaign, POST /creatives, POST /creatives/:id/campaign/:campaign_id, POST /campaigns/:id/book]
source: >-
  Tool names verified verbatim in mcp/adquick-mcp.yml (searched from
  https://www.adquick.com/mcp/usage-documentation); REST paths and required Campaign fields verified
  in conventions/adquick-conventions.yml and data-model/adquick-data-model.yml
  (https://docs.adquick.com/campaigns). AdQuick publishes no OpenAPI, so REST steps cite documented
  paths rather than operationIds.
---

# Plan and book an out-of-home campaign

AdQuick's agent surface is split. The **MCP server** (`https://www.adquick.com/mcp`) does discovery,
planning and campaign assembly. The **partner REST API** (`https://api.adquick.com`) does creative
submission and the actual booking. **There is no MCP tool that books a campaign** — see
`mcp/adquick-tool-crosswalk.yml`. Plan with the agent; transact over REST.

## Auth
- MCP: OAuth 2.0 authorization_code + PKCE (S256), dynamic client registration (RFC 7591).
  Scopes needed for this flow: `inventory_read`, `availability_read`, `campaign_read`,
  `campaign_write`. See `authentication/adquick-authentication.yml` and `scopes/adquick-scopes.yml`.
- REST: static `X-PARTNER-TOKEN` header on every request.

## Steps
1. **Locate the market** — `search_locations` (cities, neighborhoods, ZIP codes with inventory), and
   `list_markets` for the DMA list. Geographies accept Nielsen DMA IDs (integer), U.S. ZIP codes
   (string), or Market IDs (integer).
2. **Pick the format** — `list_media_types` for media types and subtypes.
3. **Find inventory** — `search_inventory` filtered by location, media type and screen type, then
   `get_unit_details` for specs, pricing and images on the shortlist.
4. **Confirm it is buyable** — `check_availability` for a single unit and date range, or `find_avails`
   across the campaign's whole flight.
5. **Create the campaign** — `create_campaign` (MCP), or `POST /campaigns` (REST) with the required
   fields: `campaign_type` (`direct` or `rtb`), `partner_id`, `advertiser_id`, `start_date` and
   `end_date` in `MM/DD/YYYY`, and `target_budget_cents` (integer, USD). Optional: `placements`,
   `geographies`, `media_types`. The response carries a campaign `token`.
6. **Attach inventory** — `add_units_to_campaign` for units chosen after creation.
7. **Submit creative** — `POST /creatives`, then schedule it against the campaign with
   `POST /creatives/:id/campaign/:campaign_id`.
8. **Book it** — `POST /campaigns/:id/book`. REST only.

## Errors
- REST returns RFC 9457 `application/problem+json` with `type`/`title`/`status`/`instance` and a
  `trace` object (`timestamp`, `requestId`, `buildId`, `rayId`). Quote `requestId` in support
  tickets. See `errors/adquick-problem-types.yml`.
- MCP returns an OAuth-shaped error instead:
  `{"error":"unauthorized","error_description":"The access token is invalid"}` — re-run the
  authorization code + PKCE flow rather than retrying.

## Notes
- **No idempotency contract is documented.** AdQuick publishes no `Idempotency-Key` header, so
  `POST /campaigns` and `POST /campaigns/:id/book` are NOT safe to blind-retry. On a timeout,
  reconcile with `GET /campaigns` (filters: `current_page`, `start_date`, `end_date`,
  `campaign_type`, `status`) before re-posting.
- **No published rate limits**, but HTTP 429 is in the problem catalogue and no `Retry-After` is
  documented — back off exponentially. See `rate-limits/adquick-rate-limits.yml`.
- `GET /campaigns` is paginated: `current_page`, `next_page`, `prev_page`, `total_pages`,
  `total_entries`.
