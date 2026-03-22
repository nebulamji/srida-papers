# Autonomous Guardrail Testing Harness

Date: 2026-03-22

## Abstract

Autonomous systems need recurring guardrail tests that continuously verify trust boundaries. This paper defines a test harness for auth, scope, and policy boundaries; a pass/fail matrix mapped to failure taxonomy classes; and heartbeat-integrated receipt emission.

## Objective

Detect guardrail regressions before they become incidents by running deterministic boundary checks every cycle.

## Boundary Domains

1. **Authentication / Pairing**
   - Ensure only paired/trusted identities can access protected paths.

2. **Authorization / Scope**
   - Ensure required scopes exist for intended actions.
   - Ensure non-required scopes are not over-granted.

3. **Policy Surfaces**
   - Ensure channel and control UI policies remain in safe states.

## Recurring Test Set

## T1 — Gateway Reachability + Controlled RPC
- Command: `openclaw gateway probe`
- Pass: connect + RPC within budget
- Failure class: transport

## T2 — Operator Read Scope Check
- Signal source: `openclaw status`
- Pass: no `missing scope: operator.read`
- Failure class: scope

## T3 — Pairing Boundary Check
- Signal source: session/channel read attempt behavior
- Pass: no unauthorized pairing errors for trusted identity
- Failure class: auth_pairing

## T4 — Security Drift Checks
- Signal source: security audit keys (device auth, policy openness, origins, insecure auth)
- Pass: no critical drift; warn-only under threshold
- Failure class: policy_drift (maps to security-critical)

## T5 — External Publish Readiness
- Command: `xurl me`
- Pass: authenticated response
- Failure class: external_api

## T6 — Persistence Write Path
- Signal source: commit path result
- Pass: no permission errors in `.git/*`
- Failure class: local_permissions

## Pass/Fail Matrix

| Test | Domain | Pass Condition | Fail Signature | Failure Class | Containment Mode |
|---|---|---|---|---|---|
| T1 | Transport | Probe connect+RPC success | timeout/refused | transport | B |
| T2 | Scope | no missing operator.read | missing scope | scope | B |
| T3 | Auth | trusted identity accepted | pairing required/1008 | auth_pairing | B |
| T4 | Policy | no critical drift | critical drift key present | policy_drift | R1/R3 |
| T5 | External API | xurl identity success | request failed/auth error | external_api | C |
| T6 | Persistence | git write path OK | permission denied/EACCES | local_permissions | D |

## Heartbeat Integration

Per heartbeat cycle:

1. Execute T1/T2/T5 directly.
2. Parse status/audit for T3/T4 signals.
3. Capture commit-path result for T6.
4. Emit harness receipt with per-test pass/fail and derived failure classes.
5. Trigger containment/recovery/rollback workflows based on matrix mapping.

## Harness Receipt Format

```yaml
harness_receipt_id: string
timestamp_utc: string
tests:
  T1: pass|fail
  T2: pass|fail
  T3: pass|fail
  T4: pass|fail
  T5: pass|fail
  T6: pass|fail
failures:
  - class: string
    signal: string
mode: A|B|C|D|R1|R2|R3
next:
  required: boolean
  owner: operator|agent
  task: string
```

## Example (Current Runtime)

- T1 fail (gateway timeout)
- T2 fail (missing `operator.read`)
- T4 fail (critical drift present)
- T5 fail (`xurl me` request failed)
- T6 fail (`.git` permission denied)

Result: combined degraded operation + security remediation required.

## Conclusion

A guardrail testing harness converts trust-boundary assumptions into measurable checks. If a boundary is untested, it is effectively unenforced.
