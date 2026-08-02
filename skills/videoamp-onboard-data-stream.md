---
name: Onboard a data stream into VideoAmp
description: Create a data stream, upload files, map the schema, approve delivery, and verify ingestion.
api: openapi/videoamp-public-api-openapi.yml
generated: '2026-08-02'
method: generated
source: openapi/videoamp-public-api-openapi.yml + conventions/videoamp-conventions.yml
operations:
  - data_stream_type_list
  - data_stream_type_get
  - data_stream_create
  - data_stream_get
  - data_stream_list
  - data_stream_update
  - delivery_config_update
  - delivery_config_get
  - delivery_config_upload_url_create
  - delivery_config_uploaded_files_get
  - delivery_config_file_delete
  - delivery_config_access_confirm
  - delivery_config_approve
  - schema_mapping_update
  - schema_mapping_get
  - schema_mapping_approve
  - delivery_inspection_create
  - delivery_inspection_get
  - delivery_inspection_list
  - delivery_inspection_files_list
  - ingestion_list
---

# Onboard a data stream into VideoAmp

Brings first-party data into VideoAmp: define the stream, get files in, bind the columns, approve,
and confirm ingestion.

## Stability warning

The whole data-management surface is **alpha** (`/v1alpha`). Expect breaking change and pin nothing
long-lived to it without a contract with VideoAmp.

## Steps

1. **Choose the stream type** — `data_stream_type_list`
   (`GET /v1alpha/dataStreamTypes`), then `data_stream_type_get`
   (`GET /v1alpha/dataStreamTypes/{dataStreamTypeName}`) for its requirements.
2. **Create the stream** — `data_stream_create` (`POST /v1alpha/dataStreams`).
   Amend later with `data_stream_update` (`PATCH /v1alpha/dataStreams/{dataStreamId}`).
3. **Configure delivery** — `delivery_config_update`
   (`PUT /v1alpha/dataStreams/{dataStreamId}/deliveryConfig`).
   This PUT **is idempotent for `DRAFT` configs**: a second PUT with the same method is a no-op, and
   a PUT with a different method updates it. Read it back with `delivery_config_get`.
4. **Get files in** — `delivery_config_upload_url_create`
   (`POST /v1alpha/dataStreams/{dataStreamId}/deliveryConfig:createUploadUrl`) returns an upload
   URL. List what has landed with `delivery_config_uploaded_files_get`
   (`...:listUploadedFiles`) and remove a bad file with `delivery_config_file_delete`
   (`...:deleteUploadedFile`).
   For pull-based delivery, confirm VideoAmp can reach the source with
   `delivery_config_access_confirm` (`...:confirmAccess`).
5. **Approve delivery** — `delivery_config_approve`
   (`POST /v1alpha/dataStreams/{dataStreamId}/deliveryConfig:approve`).
6. **Map the schema** — `schema_mapping_update`
   (`PUT /v1alpha/dataStreams/{dataStreamId}/schemaMapping`).
   Idempotent, but note the replace semantics: the first PUT creates the mapping with
   `status: DRAFT`, and a subsequent PUT **replaces `column_bindings` wholesale**. Omitting a
   binding removes it — this is not a partial merge. Always read `schema_mapping_get` first and send
   the full set.
7. **Approve the mapping** — `schema_mapping_approve`
   (`POST /v1alpha/dataStreams/{dataStreamId}/schemaMapping:approve`).
   This one is safely retryable: if a previous attempt locked the mapping but did not finish
   advancing the DataStream, calling `:approve` again re-triggers ingestion and completes the
   transition.
8. **Inspect the delivery** — `delivery_inspection_create`
   (`POST /v1alpha/dataStreams/{dataStreamId}/deliveryInspections`), then
   `delivery_inspection_get` and `delivery_inspection_files_list` for per-file detail.
9. **Confirm ingestion** — `ingestion_list`
   (`GET /v1alpha/dataStreams/{dataStreamId}/ingestions`).

## Rules

- `:approve`, `:createUploadUrl`, `:listUploadedFiles`, `:deleteUploadedFile` and `:confirmAccess`
  are AIP-136 custom methods — the colon is part of the path.
- Approval steps are one-way transitions. Verify with the matching GET before approving.
- Errors carry a product-scoped code alongside the HTTP status; there is no `application/problem+json`.
