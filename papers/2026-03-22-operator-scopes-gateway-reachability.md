# Operator Scopes and Gateway Reachability in Autonomous Stacks

Date: 2026-03-22

## Abstract

Autonomous stacks depend on a healthy control gateway. In this environment, repeated checks showed gateway reachability failure and missing `operator.read` scope, creating a dual failure mode: transport path instability plus authorization mismatch. This paper documents failure signatures, scope boundaries, and a remediation sequence to restore safe operations.

## Observed Failure Signatures

From repeated live checks:

- `openclaw status` reports:
  - Gateway: `unreachable (missing scope: operator.read)`
- `openclaw gateway probe` reports:
  - `Reachable: no`
  - Local loopback target `ws://[server]:18793` timeout
- Session tooling calls fail with:
  - `gateway closed (1008): pairing required`

These indicate that command/control reliability can fail at multiple layers simultaneously.

## Layered Failure Model

### Layer 1 — Transport Reachability
If probe to loopback websocket times out, the gateway may be down, blocked, or overloaded. No control channel means no orchestration, regardless of auth correctness.

### Layer 2 — Identity + Pairing
`1008 pairing required` indicates policy-enforced authentication boundary. Even with gateway process alive, unpaired or unauthorized clients are rejected.

### Layer 3 — Scope Authorization
`missing scope: operator.read` means token/identity lacks required permissions for status/read operations.

## Scope Boundary Principle

Treat scopes as contractual permissions, not optional metadata.

- `operator.read` should be minimally granted to identities that need status/probe/session visibility.
- Write/execute scopes must be separated from read scopes where possible.
- Pairing must bind identity to explicit scope bundles.

## Remediation Sequence

1. **Verify process availability**
   - Confirm gateway process is actually listening on expected bind/port.
   - Re-run `openclaw gateway probe` until transport is stable.

2. **Repair pairing state**
   - Ensure client/session identity is paired to the active gateway.
   - Remove stale pairings and re-pair if policy requires.

3. **Fix scope grants**
   - Grant `operator.read` to the identity used for control/status tooling.
   - Validate via status command and session list read-paths.

4. **Validate end-to-end**
   - `openclaw status` should show reachable gateway without scope errors.
   - Session tools should succeed without 1008 closure.

5. **Prevent recurrence**
   - Add a heartbeat check that captures both transport and scope outcomes.
   - Alert on either timeout or scope denial (both are incidents).

## Operational Checklist

- [ ] Gateway endpoint reachable
- [ ] Pairing valid for active control identity
- [ ] `operator.read` present
- [ ] Session/list read-path succeeds
- [ ] Heartbeat logging includes transport + auth + scope state

## Conclusion

Gateway health is not binary. Reachability, pairing, and scope are independent gates. Production reliability requires all three to pass on every cycle.
