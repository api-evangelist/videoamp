---
name: Plan and optimize a VideoAmp media buy
description: Create a campaign, assemble inventory with rate cards and reach curves, run a plan optimization, and approve the draft.
api: openapi/videoamp-public-api-openapi.yml
generated: '2026-08-02'
method: generated
source: openapi/videoamp-public-api-openapi.yml + conventions/videoamp-conventions.yml
operations:
  - campaign_create
  - campaign_get
  - campaign_list
  - campaign_update
  - inventory_list
  - inventory_get
  - inventory_update
  - rate_card_create
  - rate_card_list
  - reach_curve_create
  - reach_curve_list
  - plan_create
  - plan_get
  - plan_list
  - plan_draft_create
  - draft_plan_update
  - draft_plan_approve
---

# Plan and optimize a VideoAmp media buy

Runs reach-and-frequency optimization across an inventory set to produce an optimized budget
allocation for a campaign.

## Stability warning

The planning surface is **beta**: campaigns, inventories, rate cards, reach curves and plans are all
on `/v1beta`, and the draft/approve plan lifecycle is on `/v2beta`. Pin the channel you build
against and expect change.

## Steps

1. **Create the campaign** — `campaign_create` (`POST /v1beta/campaigns`). The campaign must have
   valid audiences before a plan can be optimized against it. Amend with `campaign_update`
   (`PATCH /v1beta/campaigns/{campaignId}`).
2. **Pick the inventory set** — `inventory_list` (`GET /v1beta/inventories`) and `inventory_get`
   (`GET /v1beta/inventories/{inventoryId}`). `inventory_update`
   (`PATCH /v1beta/inventories/{inventoryId}`) renames inventory dimensions.
3. **Attach a rate card** — `rate_card_create`
   (`POST /v1beta/inventories/{inventoryId}/rateCards`).
   Send `validateOnly=true` first: it validates that Kantar rates fall within the accepted
   $1.00–$10,000.00 range and returns 200 when all are valid, or 400 with per-title violation
   detail. List existing cards with `rate_card_list`.
4. **Add custom reach curves** (optional) — `reach_curve_create`
   (`POST /v1beta/inventories/{inventoryId}/reachCurves`); list with `reach_curve_list`.
5. **Run the optimization** — `plan_create` (`POST /v1beta/plans`).
   The plan defines budget allocation across inventory titles plus constraints, optimizing for
   reach, impressions or audience concentration. Use the `OBJECTIVE_FORECAST_ONLY` objective to skip
   optimization and forecast reach/impressions/frequency from a fixed unit allocation — the right
   choice for a secondary read on a buy already committed in another currency.
6. **Poll the plan** — `plan_get` (`GET /v1beta/plans/{planId}`). If a plan fails, fix the
   configuration and create a **new** plan; there is no retry-in-place.

## Draft workflow (v2beta)

7. `plan_draft_create` (`POST /v2beta/plans`) creates a draft.
8. `draft_plan_update` (`PATCH /v2beta/plans/{planId}`) revises it.
9. `draft_plan_approve` (`POST /v2beta/plans/{planId}:approve`) promotes it.

## Rules

- `:approve` is an AIP-136 custom method — the colon is part of the path, not a separator.
- Plan creation is not documented as idempotent. Check `plan_list` (`GET /v1beta/plans`) before
  re-issuing a create after a timeout.
- Page with `pageSize`/`pageToken`; stop on the absence of `next_page_token`.
