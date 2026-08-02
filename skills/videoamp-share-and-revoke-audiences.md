---
name: Share VideoAmp audiences and revoke access
description: Verify recipient consent, share audiences across organizations, and run the two-step bulk revocation safely.
api: openapi/videoamp-public-api-openapi.yml
generated: '2026-08-02'
method: generated
source: openapi/videoamp-public-api-openapi.yml + conventions/videoamp-conventions.yml
operations:
  - consent_list
  - consent_list_v3
  - share_resource_create
  - share_resources_create
  - share_resources_v3_create
  - share_get
  - share_list
  - share_delete
  - share_audience_recipient_revoke
  - share_audience_revocation_confirm
  - share_audience_revocation_cancel
  - user_info_get
  - user_organization_list
---

# Share VideoAmp audiences and revoke access

Cross-organization data collaboration: check consent, share, and revoke.

## Before you start

- `user_info_get` (`GET /v1/me`) gives the calling user and `current_organization_id`;
  `user_organization_list` (`GET /v1/me/orgs`) lists organizations available to them.
  The `organizations` array on `/v1/me` is **deprecated** — use `current_organization_id` and the
  `memberships` entries.
- **Vocabulary mismatch:** the Sharing API uses `ORGANIZATION` where the user API uses
  `HOLDING_COMPANY` for the same organization kind. Translate `kind` values when crossing surfaces.

## Steps

1. **Verify consent first** — `consent_list` (`GET /v1/consents`), or `consent_list_v3`
   (`GET /v3/consents`) for UUID recipients. This is documented as essential pre-share validation:
   confirm the recipient has consented before creating a share.
2. **Create the share** — pick the right version:
   - `share_resource_create` (`POST /v1/shares`) — single share.
   - `share_resources_create` (`POST /v2/shares`) — bulk.
   - `share_resources_v3_create` (`POST /v3/shares`) — bulk with UUID recipients. Prefer this for
     new integrations.
3. **Inspect** — `share_list` (`GET /v1/shares`) and `share_get` (`GET /v1/shares/{id}`).

## Revoking

**Single share:** `share_delete` (`DELETE /v1/shares/{id}`).

**Bulk revocation for one recipient is a deliberate two-step confirmation — do not automate it
without a human decision point:**

4. **Step 1 — initiate** — `share_audience_recipient_revoke`
   (`DELETE /v1/shares/audiences/revoke/recipient/{recipient}`). This does not revoke anything; it
   returns a `confirmId` and the scope of what would be revoked. Present that scope for review.
5. **Step 2a — execute** — `share_audience_revocation_confirm`
   (`POST /v1/shares/audiences/revoke/confirm/{confirmId}`). This is the destructive step.
6. **Step 2b — abort** — `share_audience_revocation_cancel`
   (`POST /v1/shares/audiences/revoke/cancel/{confirmId}`) discards the pending revocation.

## Rules

- Bulk revocation is irreversible once confirmed. An agent should surface the step-1 scope to a human
  and require explicit approval before calling `confirm`.
- 404 conflates "does not exist" with "not visible to your organization".
- Page with `pageSize`/`pageToken`; stop on the absence of `next_page_token`.
