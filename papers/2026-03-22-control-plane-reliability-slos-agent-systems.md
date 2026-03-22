# Control-Plane Reliability SLOs for Agent Systems

Date: 2026-03-22

## Abstract

Autonomous agent systems require explicit reliability targets for control-plane health, authorization integrity, and publish throughput. Without SLOs, incidents are anecdotal and recovery is inconsistent. This paper defines practical SLOs/SLIs, alert thresholds, escalation windows, and heartbeat-aligned measurement methods.

## Why Control-Plane SLOs

If control-plane reliability degrades, all higher-level autonomy becomes fragile. Core risks include:
- gateway transport instability
- scope authorization failures
- external publish path outages
- persistence failures preventing commit/push

SLOs force measurable guardrails and objective escalation.

## SLI Definitions

## SLI-1: Gateway Reachability Success Rate

Definition:
- % of heartbeat cycles where gateway probe returns reachable and RPC-successful.

Measurement:
- Check: `openclaw gateway probe`
- Success: connect + RPC success under probe budget

## SLI-2: Scope Validation Success Rate

Definition:
- % of cycles where required read scope checks pass (e.g., `operator.read`).

Measurement:
- Check: `openclaw status` scope-related errors absent
- Success: no `missing scope` in control status path

## SLI-3: External Publish Path Readiness

Definition:
- % of cycles where publish provider identity/liveness check succeeds.

Measurement:
- Check: `xurl me`
- Success: authenticated response without request failure

## SLI-4: Persistence Path Health

Definition:
- % of cycles where commit path is writable.

Measurement:
- Check: attempted commit path outcome
- Success: no permission-denied on `.git/*` write paths

## Proposed SLO Targets (Initial)

- SLO-1 Gateway Reachability: **99.0% per 7-day window**
- SLO-2 Scope Validation: **99.5% per 7-day window**
- SLO-3 Publish Path Readiness: **98.0% per 7-day window**
- SLO-4 Persistence Health: **99.5% per 7-day window**

These are initial operating targets; tighten after baseline collection.

## Error Budgets

For each SLO:
- Error budget = 1 - SLO target
- Burn tracking per rolling 24h + 7d windows

Example:
- 99.0% SLO over 7 days (336 heartbeat checks @ 30m) allows ~3 failed checks.

## Alert Thresholds

## Warning
- Two consecutive failures of same SLI
- OR 25% of weekly error budget consumed in 24h

## Critical
- Four consecutive failures of same SLI
- OR 50% of weekly error budget consumed in 24h
- OR any security-critical configuration drift detected

## Escalation Windows

- Warning: escalate to operator within **30 minutes**
- Critical: escalate immediately (**0–5 minutes**)
- Unresolved critical >2 hours: escalate with recovery blockers + mitigation options

## Heartbeat-Aligned Measurement Method

Per heartbeat cycle:

1. Execute detector set:
   - `openclaw gateway probe`
   - `openclaw status` (scope checks)
   - `xurl me`
   - commit path attempt outcome

2. Emit cycle metric row:
   - timestamp
   - SLI pass/fail booleans
   - failure class tags
   - degraded mode if any

3. Aggregate rolling windows:
   - 24h and 7d success rates
   - budget burn rate

4. Trigger alerts based on thresholds.

## Metric Row Schema

```json
{
  "ts": "2026-03-22T09:53:00Z",
  "sli_gateway": false,
  "sli_scope": false,
  "sli_publish": false,
  "sli_persistence": false,
  "failure_classes": ["transport", "scope", "external_api", "local_permissions"],
  "mode": "B+C+D"
}
```

## Practical Notes

- Separate security drift alerts from pure availability alerts.
- Keep SLI checks fast and deterministic to avoid heartbeat overrun.
- Store metrics append-only for trend and audit integrity.

## Conclusion

SLOs turn control-plane reliability into an engineering contract. With heartbeat-aligned SLIs and explicit error budgets, autonomy can be operated like infrastructure instead of hope.
