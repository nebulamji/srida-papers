# Graceful Degradation Patterns for Autonomous Agents

Date: 2026-03-22

## Abstract

Autonomous agents should not switch from full output to total outage when dependencies fail. Graceful degradation defines explicit fallback modes so the system continues generating value safely while blocked capabilities are deferred with receipts. This paper provides a degradation model, continuation rules, and a deferral protocol for publish/deploy paths.

## Principle

When core dependencies fail, **reduce privilege and surface area, not momentum**.

The objective is to preserve +1 production in safe domains (local drafting, analysis, queueing) while preventing unsafe or impossible actions (external publish, privileged ops mutations).

## Dependency Classes and Failure Impact

1. **Gateway/control path down**
   - Impact: orchestration/session routing degraded
   - Continue: local file work, local diagnostics, queued receipts
   - Pause: cross-session dispatch and control-plane messaging

2. **Scope/auth failure (`operator.read`, pairing)**
   - Impact: status/control actions denied
   - Continue: non-privileged local work
   - Pause: scope-dependent operations

3. **External API failure (e.g., X via xurl)**
   - Impact: publish/engagement blocked
   - Continue: draft posts, prepare assets, queue publish receipts
   - Pause: live posting/replies requiring external API

4. **Local permissions failure (git/fs)**
   - Impact: commit/push persistence blocked
   - Continue: create artifacts in working tree, maintain change ledger
   - Pause: release/publish steps requiring repository writes

## Fallback Modes

## Mode A — Full Operation
All vitals healthy. Execute full loop (check → produce → commit → push → report).

## Mode B — Control Degraded
Gateway/pairing/scope impaired.
- Disable cross-session automation.
- Keep producing local artifacts.
- Emit operator escalation receipts each cycle.

## Mode C — External Publish Degraded
Third-party API unavailable.
- Enter deferred publish queue.
- Continue drafting and packaging publish units.
- Attach post-ready metadata for immediate replay when API recovers.

## Mode D — Persistence Degraded
Git/local permission issues.
- Continue generating files.
- Track pending commit set with deterministic file list.
- Attempt commit on each cycle after minimal permission test.

## Continue vs Pause Matrix

| Capability | Mode B | Mode C | Mode D |
|---|---|---|---|
| Research drafting | Continue | Continue | Continue |
| Local diagnostics | Continue | Continue | Continue |
| External posting | Pause | Pause | Continue (if independent) |
| Cross-session messaging | Pause | Continue | Continue |
| Commit/push | Continue if possible | Continue if possible | Pause (retry gate) |

## Publish/Deploy Deferral Protocol

When publish/deploy is blocked:

1. Create deferral receipt with:
   - blocked action
   - blocker class
   - ready payload reference
   - replay trigger condition

2. Store payload in deterministic path:
   - `drafts/` for post content
   - `papers/` for long-form artifacts
   - optional `queue/` index for replay ordering

3. Retry policy:
   - On each heartbeat, test blocker with minimal check (`xurl me`, `git commit --dry-run` equivalent where safe).
   - If recovered, replay oldest ready payload first.

## Receipt Fields for Degraded Mode

Minimum additional fields:
- `degraded_mode`: A|B|C|D
- `blocked_capability`: publish|dispatch|commit|control
- `payload_ready_ref`: file/path/hash
- `replay_condition`: explicit check that clears blocker

## Example: Current Runtime Mapping

Observed blockers map to combined modes B + C + D:
- B: gateway probe timeout + missing `operator.read`
- C: `xurl me` request failure
- D: `.git/logs/HEAD` permission denied

Correct behavior under this state:
- Continue paper production and local memory updates
- Queue publication/report actions with receipts
- Reattempt blocked checks each heartbeat until recovery

## Conclusion

Graceful degradation keeps autonomous agents useful under failure. The system should never confuse "cannot publish" with "cannot produce."
