# Autonomous Rollback Protocols

Date: 2026-03-22

## Abstract

Autonomous systems need deterministic rollback behavior when configuration or deployment changes regress reliability or security. This paper defines rollback triggers, safe rollback ordering, verification gates, and a rollback receipt schema to keep recovery auditable and fast.

## Rollback Goals

1. Minimize blast radius quickly.
2. Restore last-known-good state deterministically.
3. Prevent partial rollback ambiguity.
4. Preserve full evidence trail.

## Rollback Triggers

Initiate rollback when any of the following occur after a change:

## Security Triggers (Immediate)
- Critical security drift introduced (e.g., device auth disabled, open policy + elevated tools).
- Unauthorized access path appears in control plane.

## Reliability Triggers
- Gateway unreachable for 4 consecutive cycles.
- Scope validation fails for 4 consecutive cycles.
- Publish pipeline unavailable beyond defined window (e.g., >2h) after a release.

## Persistence/Integrity Triggers
- Change cannot be committed/applied cleanly and leaves system in mixed state.
- Verification gates fail after remediation retries.

## Safe Rollback Order

Rollback sequence should follow dependency direction (control plane first):

1. **Control-plane auth/policy**
   - restore device auth, strict origins, pairing/allowlist policies

2. **Channel exposure policies**
   - close DM/group policies to last-known-safe profile

3. **Privilege/tool routing**
   - disable elevated tool exposure in untrusted surfaces

4. **Application/runtime config**
   - restore previous stable config snapshot

5. **Content/publish pipeline toggles**
   - re-enable publish paths only after core controls pass

## Verification Gates

After each rollback stage, require pass before continuing:

- Gate A: `openclaw security audit` critical count not increasing
- Gate B: `openclaw gateway probe` reachable and RPC responsive
- Gate C: scope checks pass (no `missing scope`)
- Gate D: external publish readiness check passes (`xurl me`)

If a gate fails:
- halt progression
- emit rollback-blocked receipt
- escalate with exact failed gate and output

## Rollback Modes

## Mode R1 — Policy Rollback
Used for security drift only; restores secure config without full deploy revert.

## Mode R2 — Config Rollback
Reverts service/app config to prior known-good snapshot.

## Mode R3 — Full Operational Rollback
Applies policy + config rollback and holds publish/dispatch until all gates pass.

## Implementation Notes

- Maintain versioned config snapshots with timestamps.
- Tag each change with a reversible unit ID.
- Never batch unrelated high-risk changes in one rollback unit.

## Rollback Receipt Schema

```yaml
rollback_id: string
timestamp_utc: string
trigger:
  type: security|reliability|integrity
  signal: string
mode: R1|R2|R3
change_unit_ref: string
stages:
  - stage: string
    action: string
    result: success|failed
gates:
  - gate: A|B|C|D
    passed: boolean
    evidence_ref: string
status: completed|blocked
next:
  required: boolean
  owner: operator|agent
  task: string
```

## Example Mapping to Current Runtime

Given persistent gateway/scope instability + security critical drift:
- Candidate mode: **R3** (full operational rollback)
- Priority sequence:
  1) revert control UI/auth policy drifts
  2) enforce channel allowlist policies
  3) revalidate gateway/scope readiness
  4) reopen publish path only after gate pass

## Conclusion

Rollback is not failure; it is controlled recovery. Autonomous rollback protocols keep the system reversible, safe, and measurable under pressure.
