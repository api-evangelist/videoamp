---
name: Build a VideoAmp audience and export it
description: Discover the demographic filter vocabulary, create an audience, wait for it to reach READY, and export it for activation.
api: openapi/videoamp-public-api-openapi.yml
generated: '2026-08-02'
method: generated
source: openapi/videoamp-public-api-openapi.yml + conventions/videoamp-conventions.yml
operations:
  - audience_demographic_filter_field_lookup
  - audience_demographic_filter_value_lookup
  - audience_id_types_lookup
  - audience_create
  - audience_get
  - audience_list
  - audience_status_list
  - audience_update
  - audience_batch_get
  - audience_export_create
  - audience_export_get
  - audience_export_list
---

# Build a VideoAmp audience and export it

Creates an audience definition, waits for VideoAmp to materialize it, then exports the membership
for downstream activation.

## Before you start

- OAuth 2.0 bearer token from `https://login.videoamp.com`.
- Decide the audience **classification**: `EXPOSURE`, `CONTENT`, `CUSTOM_DEMOGRAPHIC`,
  `USER_PROVIDED`, `GLOBAL_DEMOGRAPHIC`, `COMPOSITE` or `MODELED`. The classification determines
  which definition object the create call expects.
- Use the **v2** audience surface for new work. `/v1/audiences` is still live and the CLI keeps it as
  separate `*_v1` commands, but v2 is current.

## Steps

1. **Discover the filter vocabulary** (for `CUSTOM_DEMOGRAPHIC`) —
   `audience_demographic_filter_field_lookup` (`GET /v2/audiences/lookupDemographicFilterFields`)
   then `audience_demographic_filter_value_lookup`
   (`GET /v2/audiences/lookupDemographicFilterValues`). Never guess field names or values.
2. **Check identifier types** (for `USER_PROVIDED`) — `audience_id_types_lookup`
   (`GET /v1/audiences:lookUpIdTypes`) tells you which id types can be supplied. This endpoint has
   no rate limits beyond platform defaults.
3. **Create the audience** — `audience_create` (`POST /v2/audiences`).
   Note the constraint that a `USER_PROVIDED` audience cannot be used as the `origin_audience` of a
   `MODELED` audience.
4. **Poll to READY** — `audience_get` (`GET /v2/audiences/{id}`). Only READY audiences are eligible
   for measurement. Exports, by contrast, can be created against an audience in *any* status.
   `audience_status_list` (`GET /v1/audiences/status`) enumerates the possible statuses.
5. **Amend if needed** — `audience_update` (`PATCH /v2/audiences/{id}`).
6. **Export it** — `audience_export_create` (`POST /v1/audiences/{audienceId}/exports`), then poll
   `audience_export_get` (`GET /v1/audiences/{audienceId}/exports/{id}`). List prior exports with
   `audience_export_list`.

## Finding audiences you already have

- `audience_list` (`GET /v2/audiences`) filters by `classifications`, `excludeClassifications`,
  `statuses`, `useCases`, `ownership`, `name`, `description`, `createdByName`, `cadences`, `level`,
  `advertiserIds`, `agencyIds`, `currencyOfRecord` and `broadcastDate`.
- `audience_batch_get` (`GET /v1/audiences:batchGet`) fetches a known set in one call.
- Sort with `orderBy`, e.g. `currency_of_record, name desc`. Sortable fields are `created_at`,
  `created_by_name`, `name`, `currency_of_record`, `legacy_id`. Default is `created_at desc`.

## Rules

- IDs are UUID v4; a legacy numeric id encoded as a string is also accepted via `legacyIds`. Prefer
  UUIDs for new integrations.
- Organization permissions inherit downward — an audience permissioned to a parent org is visible to
  every child advertiser. Filtering by two advertisers in the same org returning identical results is
  expected, not a bug. Results across multiple advertisers are a deduplicated **union**, so never sum
  per-advertiser `total_size` values.
- `GLOBAL_DEMOGRAPHIC` audiences need `currencyOfRecord`, and `broadcastDate` to pick the broadcast
  year's metrics. An out-of-range date returns 400 with the valid range.
- Page with `pageSize`/`pageToken`; stop on the absence of `next_page_token`.
