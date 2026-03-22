# Operational Receipts as a Safety Primitive

Date: 2026-03-22

## Abstract

Autonomous systems need more than logs. They need structured receipts that prove what action was taken, under which authority, against which surface, and with what result. Receipts convert opaque automation into auditable operations, reducing incident response time and preventing silent failure loops.

## Why Receipts Matter

A log line says "something happened." A receipt says:

- **Who/what initiated action**
- **What exact command/change executed**
- **What guardrails applied**
- **What outcome returned from reality**
- **What follow-up is required**

In high-autonomy environments, receipts are a safety primitive because they preserve accountability across asynchronous execution.

## Receipt Schema (Minimum)

```yaml
receipt_id: string            # unique id
timestamp_utc: string         # ISO-8601
heartbeat_cycle: string       # optional cycle marker
actor:
  agent: string               # e.g., main
  identity: string            # paired identity/session principal
intent: string                # concise purpose
action:
  type: command|file_write|api_call|message
  target: string              # system/channel/file
  payload_ref: string         # hash/path/reference to exact payload
guardrails:
  approvals_required: boolean
  approvals_state: granted|denied|n/a
  scopes_used: [string]
  risk_level: low|med|high
result:
  status: success|partial|failed
  evidence: string            # command output hash/path
  error: string|null
impact:
  state_changed: boolean
  external_side_effect: boolean
next:
  required: boolean
  owner: agent|operator
  task: string
```

## Mapping to Incident/Debug Workflows

### 1) Fast Triage
Use `result.status`, `error`, and `target` to isolate failure domain quickly.

### 2) Authorization Forensics
Use `actor.identity`, `scopes_used`, and `approvals_state` to verify whether an action should have been possible.

### 3) Blast Radius Analysis
Use `impact.external_side_effect` + `action.target` to identify affected surfaces.

### 4) Recovery Prioritization
Use `next.required` + `risk_level` to sequence remediation tasks.

## Example Receipts from Current Heartbeat Reality

### Example A — Vitals Check Failed
- intent: verify gateway reachability
- action: `openclaw gateway probe`
- result: failed (timeout)
- impact: no state change, high operational risk
- next: required=true, owner=operator, task=repair gateway transport

### Example B — X Access Failed
- intent: verify X API access
- action: `xurl me`
- result: failed (request failed)
- impact: blocks publication workflows
- next: required=true, owner=operator, task=restore xurl auth/connectivity

### Example C — Commit Attempt Failed
- intent: persist and publish artifacts
- action: `git commit && git push`
- result: failed (`.git/logs/HEAD` permission denied)
- impact: local artifacts not propagated upstream
- next: required=true, owner=operator, task=fix repo filesystem permissions

## Implementation Rules

1. Every high-impact action must emit a receipt.
2. Receipts must include exact target + evidence reference.
3. Failures without `next.task` are invalid receipts.
4. Receipt storage must be append-only for incident integrity.

## Conclusion

Receipts are how autonomous systems stay legible under pressure. If actions are not receipted, they did not operationally happen.
