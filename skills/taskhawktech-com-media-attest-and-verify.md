---
generated: '2026-09-19'
method: generated
name: Issue and verify a media authority certificate
description: Attest a media file hash into a signed Kevros certificate, let anyone verify the hash for free, and
  revoke it if needed.
api: openapi/taskhawktech-com-openapi.yml
operations:
- attest_media_media_attest_post
- verify_media_media_verify_post
- lookup_certificate_media_verify__certificate_id__get
- media_certificate_status_media_status__certificate_id__get
- revoke_media_certificate_media_revoke__certificate_id__post
source: Grounded in openapi/taskhawktech-com-openapi.yml operationIds and the provider MCP prompts at https://governance.taskhawktech.com/mcp/.
---

# Issue and verify a media authority certificate

## Auth
- Issuance needs `X-API-Key` (or a paid rail credential). Verification and status are free and anonymous. Lifecycle mutation (approve/revoke) accepts **only** the operator key bound to the certificate's `agent_id` (or an admin key) — paid rail credentials are refused (`x-authority-info.accepts_paid_rail: false`).

## Idempotency
- None of the media operations carry an idempotency or replay field. A retried `attest_media_media_attest_post` may issue a second certificate for the same hash; look the hash up first.

## Steps
1. **Hash locally** — compute the file's `sha-256`; the file itself is never uploaded (`/media/capabilities` supported_hashes = [sha-256]).
2. **Attest** — `attest_media_media_attest_post` (`POST /media/attest`, $0.05) with `agent_id`, `media_hash`, `media_type` (PHOTO | VIDEO | AUDIO | DOCUMENT), `media_size_bytes`, optional `capture_timestamp_utc`, `capture_location`, `device_info`, `frame_hashes`, `c2pa_manifest_hash`, rights/consent/campaign assertions, `model_provider` / `model_id` / `prompt_hash` for generated media. Keep `certificate_id`.
3. **Verify (anyone)** — `verify_media_media_verify_post` (`POST /media/verify`, free) with `media_hash` and `certificate_id`; or `lookup_certificate_media_verify__certificate_id__get` (`GET /media/verify/{certificate_id}`, JSON or human HTML by `Accept`).
4. **Check lifecycle** — `media_certificate_status_media_status__certificate_id__get` (`GET /media/status/{certificate_id}`) shows approval and revocation state.
5. **Revoke if needed** — `revoke_media_certificate_media_revoke__certificate_id__post` (`POST /media/revoke/{certificate_id}`) with a required `reason`. No revocation window is published.

## Claim boundary (from the provider)
- A certificate proves a Kevros-recorded hash, chain position, lifecycle status and asserted layers. It is not deepfake detection, rights clearance, consent verification or C2PA validation (`supports_c2pa_validation: false`).
