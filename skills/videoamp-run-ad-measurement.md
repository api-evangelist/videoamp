---
name: Run a VideoAmp ad measurement report
description: Find a READY audience, create an ad measurement report against it, poll until results land in S3, and optionally add a lift post-process.
api: openapi/videoamp-public-api-openapi.yml
generated: '2026-08-02'
method: generated
source: openapi/videoamp-public-api-openapi.yml + conventions/videoamp-conventions.yml
operations:
  - audience_list
  - audience_get
  - measurement_create
  - measurement_get
  - measurement_list
  - measurement_get_post_process_requirements
  - measurement_create_post_process
  - measurement_get_post_process
---

# Run a VideoAmp ad measurement report

Measures how an ad campaign performed against a VideoAmp audience. Results are computed
asynchronously and delivered as CSV/Parquet to a bucket you own.

## Before you start

- Authenticate with OAuth 2.0 against `https://login.videoamp.com` and send
  `Authorization: Bearer <token>`. The CLI does this with `videoamp login`.
- You need: a READY audience, access to the data sources, an S3 bucket, a Currency of Record, and
  a measurement request type.
- All audiences in one report must share the same Currency of Record.

## Steps

1. **Find an eligible audience** — `audience_list` (`GET /v2/audiences`).
   Filter with `statuses=READY` and `useCases=AD_MEASUREMENT`. Only READY audiences are eligible for
   measurement. Page with `pageSize` + `pageToken`; stop when `next_page_token` is absent — do not
   compare against `total_size`, which is a point-in-time snapshot.
2. **Confirm the audience** — `audience_get` (`GET /v2/audiences/{id}`).
   Verify `status`, `use_cases` and `currency_of_record` before committing. Not every
   `GLOBAL_DEMOGRAPHIC` audience supports `AD_MEASUREMENT` — some only support `CONTENT_MEASUREMENT`.
3. **Dry-run then create the report** — `measurement_create` (`POST /v2/adMeasurements`).
   Set the audience ids, the time period, `streams`, `type`, and the data sources.
   - Send `validate_only=true` first to validate without persisting.
   - **This operation is explicitly NOT idempotent.** Set a unique `external_id` and call
     `measurement_list` filtered by that `external_id` before creating, so a retry cannot duplicate
     a report.
   - For a recurring report, add `delivery_schedule` and an active `status`. A suspended recurring
     report cannot be reactivated.
   - If the report uses a constrained content-measurement datasource type, a 400 with
     `MRC_0152`–`MRC_0157` means one of five content gates failed (audience, time shift, Currency of
     Record, dimension set, metric type). The supported values are resolved at request time and are
     not enumerated by the API — read the 400 body and the `DataSource.type` field.
4. **Poll for completion** — `measurement_get` (`GET /v2/adMeasurements/{id}`).
   Creation returns immediately with metadata; computation takes roughly 30 minutes to 4 hours and
   the report then becomes `ready` or `failed`. Poll on a backoff, not a tight loop — HTTP 429 means
   you have exceeded the platform rate limit.
5. **Collect results** from the S3 bucket configured on the report. Note that S3 path provisioning
   is a VideoAmp Professional Services step.

## Optional — add a lift post-process

6. **Check what is possible** — `measurement_get_post_process_requirements`
   (`GET /v2/adMeasurements/{adMeasurement_uuid}/postProcesses:requirements`) returns the valid
   post-process configuration options for that report.
7. **Create it** — `measurement_create_post_process`
   (`POST /v2/adMeasurements/{adMeasurement_uuid}/postProcesses`). Use `type: LIFT`;
   `LINEAR_LIFT` is deprecated.
8. **Poll it** — `measurement_get_post_process`
   (`GET /v2/adMeasurements/{adMeasurement_uuid}/postProcesses/{postProcessUuid}`).

## Rules

- Errors carry a product-scoped code (`MRC_*`, `DGR_*`, `DSO_*`) alongside the HTTP status. There is
  no `application/problem+json`.
- 404 means "does not exist **or** is not visible to your organization" — the two are deliberately
  conflated. Do not infer existence from a 404.
- Do not change filter parameters midway through a paginated traversal.
