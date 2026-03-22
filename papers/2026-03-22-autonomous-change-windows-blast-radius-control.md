# Autonomous Change Windows and Blast-Radius Control

Date: 2026-03-22

## Abstract

Autonomous systems need explicit change windows and blast-radius controls for high-risk operations. This paper defines safe execution windows, preflight and canary rules, and abort/rollback handoff criteria so risky changes remain reversible and contained.

## Why Change Windows

Without bounded windows, high-risk changes can land during low-observability periods and compound failures. Change windows align risk with monitoring capacity and rollback readiness.

## Change Classes

## Low-Risk
- Content generation, local docs, non-privileged drafting.
- Can run continuously.

## Medium-Risk
- Non-critical configuration adjustments.
- Requires preflight checks and post-change verification.

## High-Risk
- Control-plane auth/policy changes
- Channel ingress policy changes
- Tool privilege routing changes
- Deploys affecting gateway/session control

High-risk changes require strict window + canary + abort guardrails.

## Safe Change Window Design

For each high-risk change, define:

- **Start/End UTC window**
- **Observer availability** (operator reachable)
- **Rollback capacity** (known-good snapshot + tested path)
- **Monitoring active** (heartbeat + probe outputs)

### Window Rules

1. Never start high-risk change without rollback-ready snapshot.
2. Only one high-risk change unit per window.
3. No overlapping high-risk windows across control-plane surfaces.
4. If critical incident already active, defer non-essential high-risk changes.

## Preflight Checklist (High-Risk)

Before change execution:

- [ ] Gateway probe baseline captured
- [ ] Security audit baseline captured
- [ ] Scope/read-path baseline captured
- [ ] External publish readiness baseline captured
- [ ] Rollback snapshot reference recorded
- [ ] Abort threshold defined
- [ ] Canary target selected

## Canary Rules

Apply change to smallest viable surface first.

Examples:
- one channel before all channels
- one policy toggle before full policy pack
- one scope bundle before wide distribution

Canary success criteria:
- no new critical drifts
- gateway/scope SLIs not degraded beyond threshold
- no new auth/pairing failures

Canary duration:
- minimum two heartbeat cycles before full rollout.

## Abort Criteria

Abort rollout immediately if any condition occurs:

1. New critical security finding appears.
2. Gateway reachability fails for 2 consecutive post-change checks.
3. Scope validation fails for newly affected identity.
4. Canary surface shows unauthorized-access risk expansion.

On abort:
- stop further rollout
- execute rollback handoff protocol
- emit abort receipt with evidence

## Rollback Handoff

When abort triggers:

1. Freeze change pipeline.
2. Attach change unit ID + canary evidence.
3. Hand off to rollback protocol mode (R1/R2/R3).
4. Verify gates A/B/C/D before considering change closed.

## Receipt Requirements

Every high-risk window must emit:

- preflight receipt
- canary outcome receipt
- rollout/abort receipt
- rollback handoff receipt (if aborted)

## Change Window Template

```yaml
change_window_id: string
risk_class: high
window:
  start_utc: string
  end_utc: string
change_unit_ref: string
preflight:
  baselines_captured: true
  rollback_snapshot_ref: string
canary:
  target: string
  success: boolean
  cycles_observed: int
abort:
  triggered: boolean
  reason: string|null
handoff:
  rollback_mode: R1|R2|R3|null
  status: open|closed
```

## Conclusion

Change windows and blast-radius controls make autonomy survivable. The goal is not to avoid risk; it is to make risk bounded, observable, and reversible.
