---
generated: '2026-09-19'
method: generated
name: Verify an action before executing it
description: Ask Kevros for a signed ALLOW/CONSTRAIN/DENY on a proposed agent action, execute only on ALLOW/CONSTRAIN,
  then attest the outcome to the provenance ledger.
api: openapi/taskhawktech-com-openapi.yml
operations:
- signup
- verify-action
- attest-action
source: Grounded in openapi/taskhawktech-com-openapi.yml operationIds and the provider MCP prompts at https://governance.taskhawktech.com/mcp/.
---

# Verify an action before executing it

Mirrors the provider's own MCP prompt `verify-before-act`. Give every consequential action a decision point before it becomes a consequence.

## Auth
- Obtain a trial key once: `signup` (`POST /signup`) with `{"agent_id": "<your-agent-id>"}`; the response carries `api_key` (1,000 calls/month, 10 req/min). Send it as `X-API-Key`. See `authentication/taskhawktech-com-authentication.yml`.
- Without a key the priced endpoints answer **402** with x402 / L402 / MPP challenges (see `errors/taskhawktech-com-problem-types.yml`). Do not pay from stale docs: re-fetch `GET /payment/discovery` and select only a rail marked `enabled: true`.

## Idempotency
- `verify-action` accepts `idempotency_key` in the body — set it to a client UUID so a retried verify returns the cached decision instead of a second paid evaluation. Also set `cmd_id` for replay protection. Coverage is partial (this operation only); see `conventions/taskhawktech-com-conventions.yml`.

## Steps
1. **Verify** — `verify-action` (`POST /governance/verify`, $0.01) with `action_type` (e.g. `send_email`, `deploy`), `action_payload` (the action's parameters), `agent_id`, optional `policy_context` / `template_id`, optional `risk_category` (EU AI Act class MINIMAL/LIMITED/HIGH/UNACCEPTABLE), `idempotency_key`, `cmd_id`.
2. **Branch on `decision`**:
   - `ALLOW` — execute the action as proposed; keep `release_token` and pass it downstream as `X-Kevros-Release-Token`.
   - `CONSTRAIN` (may appear on the wire as legacy `CLAMP`) — execute **`applied_action`**, not your original payload; the gateway has bounded the values.
   - `DENY` — stop. Read `reason`. DENY is still a recorded, paid evaluation. If the gateway is unreachable or errors, treat it as DENY (fail-closed).
3. **Attest** — after executing, `attest-action` (`POST /governance/attest`, $0.02) with `agent_id`, `action_description`, `action_payload`, optional `context`, `prior_attestation_hash` (the previous record's hash to extend your chain) and `risk_category`. Keep the returned provenance hash.

## Errors
- 402 → no credential (get a key or pay a rail); 401/403 with `WWW-Authenticate: Delegation` → governed execution needs an operator-signed Delegation proof; 422 → fix the field named in `detail[].loc`.

## Notes
- Payment buys the evaluation, never an ALLOW.
- Every decision is appended to a hash-chained ledger; a downstream service can verify `release_token` at `POST /governance/verify-token` (live, not in the OpenAPI — use the MCP `verify-token` tool).
