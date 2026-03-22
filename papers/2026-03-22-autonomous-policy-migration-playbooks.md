# Autonomous Policy Migration Playbooks

Date: 2026-03-22

## Abstract

Migrating from open channel policies to strict allowlist controls is a high-risk change because it can simultaneously improve security and break legitimate operations. This playbook defines staged migration, rollback-safe cutover sequencing, and verification checkpoints with receipts.

## Migration Objective

Move policy surfaces from open to controlled without downtime:
- groupPolicy: `open` → `allowlist`
- dmPolicy: `open` → pairing/allowlist
- elevated tools: broad exposure → constrained to trusted surfaces

## Staged Migration Plan

## Stage 0 — Inventory + Baseline

- Capture current policies and channel identities.
- Export active trusted principals (operators, bots, automation peers).
- Record baseline:
  - security audit summary
  - gateway/scope health
  - message flow continuity

Receipt: `migration-baseline`.

## Stage 1 — Shadow Allowlist Build

- Build allowlist while open policy remains active.
- Simulate allow/deny decisions against live traffic.
- Log "would-block" events without enforcement.

Pass criteria:
- All legitimate traffic mapped to allowlist identities.

Receipt: `migration-shadow-validation`.

## Stage 2 — Canary Enforcement

- Enforce allowlist on one low-blast-radius surface.
- Keep rollback toggle prepared.
- Observe for minimum 2 heartbeat cycles.

Pass criteria:
- No critical security drift increase.
- No legitimate operator lockout.
- No control-plane degradation.

Receipt: `migration-canary-result`.

## Stage 3 — Progressive Cutover

- Expand enforcement surface-by-surface.
- Sequence by risk:
  1. non-critical groups
  2. direct channels
  3. privileged/control surfaces

After each surface:
- run verification checkpoint bundle
- halt on failure and rollback affected surface only

Receipt: `migration-cutover-step`.

## Stage 4 — Harden + Clean Up

- Remove temporary exceptions.
- Confirm elevated tool boundaries in trusted zones only.
- Re-run full audit and document final posture delta.

Receipt: `migration-finalization`.

## Rollback-Safe Cutover Sequence

For each surface:

1. Snapshot current policy state (rollback ref).
2. Apply allowlist policy change.
3. Run checkpoint tests.
4. If fail, rollback immediately to snapshot.
5. Log result and proceed/stop decision.

Never cut over multiple high-risk surfaces atomically.

## Verification Checkpoints

At each stage/cutover step:

- Checkpoint A: `openclaw security audit` critical count non-increasing
- Checkpoint B: `openclaw gateway probe` reachable + RPC responsive
- Checkpoint C: scope path intact (no `missing scope` for operator read paths)
- Checkpoint D: trusted message ingress still functional
- Checkpoint E: untrusted ingress correctly denied

## Receipt Model for Migration

```yaml
migration_receipt_id: string
timestamp_utc: string
stage: baseline|shadow|canary|cutover|finalization
surface: string
change:
  from: string
  to: string
rollback_ref: string
checkpoints:
  A: pass|fail
  B: pass|fail
  C: pass|fail
  D: pass|fail
  E: pass|fail
outcome: proceed|rollback|halt
notes: string
```

## Failure Handling Rules

- Any checkpoint B/C failure => immediate rollback of current surface.
- Any operator lockout signal => halt all further migration.
- Repeated failures on same surface => open incident loop and defer migration.

## Conclusion

Policy migration is safest when staged, receipted, and reversible at each step. Autonomous systems should treat policy cutovers like production deploys: canary first, verify always, rollback fast.
