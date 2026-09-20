---
generated: '2026-09-19'
method: generated
name: Bind an intent, verify its outcome, export the evidence
description: Declare what an agent intends, bind it to the command it issues, check the outcome matched, and package
  the chain as a compliance bundle.
api: openapi/taskhawktech-com-openapi.yml
operations:
- bind-intent
- verify-outcome
- generate-bundle
source: Grounded in openapi/taskhawktech-com-openapi.yml operationIds and the provider MCP prompts at https://governance.taskhawktech.com/mcp/.
---

# Bind an intent, verify its outcome, export the evidence

Mirrors the provider's MCP prompt `governance-audit`. Closes the loop intent -> command -> action -> outcome -> evidence.

## Auth
- `X-API-Key` from `signup`; see `authentication/taskhawktech-com-authentication.yml`.

## Idempotency
- `bind-intent` carries `cmd_id` (replay protection: duplicate `cmd_id`s inside the replay window are rejected). `verify-outcome` and `generate-bundle` have no idempotency field; `generate-bundle` is read-only in effect (MCP annotation `readOnlyHint: true`) so a retry is safe.

## Steps
1. **Bind** — `bind-intent` (`POST /governance/bind`, $0.02) with `agent_id`, `intent_type` (NAVIGATION | MANIPULATION | SENSING | COMMUNICATION | MAINTENANCE | EMERGENCY | OPERATOR_COMMAND | AI_GENERATED | AUTOMATED), `intent_description`, `command_payload`, optional `intent_source`, `goal_state`, `max_duration_ms`, `parent_intent_id`, `cmd_id`. Record `intent_id` and `binding_id`.
2. **Execute** the command in your own runtime (Kevros does not execute anything).
3. **Verify outcome** — `verify-outcome` (`POST /governance/verify-outcome`, free) with `agent_id`, `intent_id`, `binding_id`, `actual_state`, optional `tolerance`. Read the `OutcomeStatus` (ACHIEVED | PARTIALLY_ACHIEVED | FAILED | BLOCKED | TIMEOUT).
4. **Export** — `generate-bundle` (`POST /governance/bundle`, $0.05) with `agent_id`, optional `time_range_start` / `time_range_end`, `max_records`, `include_intent_chains: true`, `include_pqc_signatures: true`, `include_verification_instructions: true`. The bundle is independently verifiable without Kevros access.

## Errors
- 402 without credential; 422 on a bad enum value; see `errors/taskhawktech-com-problem-types.yml`.

## Notes
- Bindings and attestations are append-only; there is no undo (see `conventions/taskhawktech-com-conventions.yml` reversibility).
