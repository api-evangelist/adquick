---
generated: '2026-08-13'
method: generated
name: Diagnose programmatic DSP pacing
description: Find live and pending DOOH programmatic plans, read their pacing, and diagnose under-delivery.
api: mcp/adquick-mcp.yml
operations: [get_recent_programmatic_plans, search_programmatic_campaigns, get_plans_by_date_range, get_plans_not_yet_live, get_programmatic_pacing, diagnose_plan_pacing, get_campaign_delivery_status, schedule_copilot_task_from_template]
source: >-
  Tool names verified verbatim in mcp/adquick-mcp.yml (searched from
  https://www.adquick.com/mcp/usage-documentation). The Programmatic DSP and Scheduling tool groups
  are MCP-only; see mcp/adquick-tool-crosswalk.yml.
---

# Diagnose programmatic DSP pacing

AdQuick's programmatic (DOOH DSP) surface exists **only** on the MCP server. There is no documented
partner REST endpoint for plans or pacing.

## Auth
- OAuth 2.0 authorization_code + PKCE (S256). Scopes: `programmatic_read`, plus `campaign_read` for
  delivery status. See `scopes/adquick-scopes.yml`.

## Steps
1. **Find the plans** — `get_recent_programmatic_plans` for what launched recently, or
   `search_programmatic_campaigns` to find a plan by name, or `get_plans_by_date_range` to scope a
   flight window.
2. **Catch the ones that never started** — `get_plans_not_yet_live` returns plans scheduled but not
   yet activated. This is the cheapest under-delivery to fix.
3. **Read pacing** — `get_programmatic_pacing` for pacing status on active plans.
4. **Diagnose** — `diagnose_plan_pacing` for the detailed diagnosis on an under-delivering plan.
5. **Cross-check guaranteed delivery** — `get_campaign_delivery_status` gives live delivery status
   for booked (non-programmatic) campaign units, so a mixed buy can be read on both sides.
6. **Automate the watch** — `schedule_copilot_task_from_template` schedules a recurring task (for
   example weekly launch-readiness alerts); `list_copilot_scheduled_tasks` and
   `cancel_copilot_scheduled_task` manage them.

## Errors
- HTTP 401 `{"error":"unauthorized","error_description":"The access token is invalid"}` means the
  OAuth token expired — refresh it (`refresh_token` grant is supported) rather than retrying.

## Notes
- Steps 1-5 are read-classed. Step 6 is write-classed: scheduling a Copilot task creates recurring
  server-side work, so it should be behind human confirmation in an autonomous agent.
- No rate limits are published; HTTP 429 exists in the REST problem catalogue and no `Retry-After` is
  documented. Poll pacing on a schedule, not in a tight loop.
