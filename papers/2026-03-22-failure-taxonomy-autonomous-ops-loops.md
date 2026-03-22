# Failure Taxonomy for Autonomous Ops Loops

Date: 2026-03-22

## Abstract

Autonomous operations loops fail in recurring patterns. A usable taxonomy improves recovery speed by mapping each failure class to concrete detection signatures and first-response actions. This paper defines five core classes observed in agent-driven environments: transport, authentication/pairing, authorization scope, local permissions/state, and external API dependency failures.

## Taxonomy

## 1) Transport Failures

### Definition
Control/data path cannot be reached at network or socket level.

### Signatures
- Connection timeout
- Refused socket
- DNS/mDNS discovery miss for expected target

### Example
- `openclaw gateway probe` timeout to `ws://[server]:18793`

### First Response
1. Confirm process listening on expected bind/port.
2. Validate network path (loopback, firewall, container boundary).
3. Re-run probe until stable before progressing.

## 2) Authentication / Pairing Failures

### Definition
Identity not accepted by the target service/channel.

### Signatures
- Policy close codes (e.g., 1008)
- "pairing required" errors
- invalid/expired tokens at session establishment

### Example
- `gateway closed (1008): pairing required`

### First Response
1. Check active identity/session principal.
2. Re-pair or rotate credentials as required.
3. Validate with low-risk read operation.

## 3) Authorization Scope Failures

### Definition
Identity is known but lacks required permission claims.

### Signatures
- "missing scope: ..."
- access denied on read/write-specific paths

### Example
- `missing scope: operator.read`

### First Response
1. Identify required scope for failing operation.
2. Grant least-privilege scope bundle.
3. Retest failing read path, then full workflow.

## 4) Local Permission / State Failures

### Definition
Host/container filesystem or runtime state blocks execution.

### Signatures
- Permission denied on `.git/*`
- lockfile/statefile update failures
- insufficient rights to write required paths

### Example
- `fatal: cannot update the ref 'HEAD' ... .git/logs/HEAD: Permission denied`

### First Response
1. Inspect file ownership and mode bits.
2. Repair permissions for required working paths.
3. Reattempt minimal write action before full commit/push.

## 5) External API Dependency Failures

### Definition
Third-party service or credentials unavailable.

### Signatures
- Request failed with no local state change
- rate limit / auth / upstream outage responses

### Example
- `xurl me` → `Error: request failed`

### First Response
1. Validate credentials/token freshness.
2. Check upstream API status and quotas.
3. Switch to queue/deferred publishing mode until restored.

## Detection Matrix

| Failure class | Primary detector | Confidence signal | Escalation owner |
|---|---|---|---|
| Transport | gateway probe | repeated timeout/refused | operator |
| Auth/Pairing | session connect/read | policy close / pairing required | operator |
| Scope | status/read action | explicit missing-scope error | operator |
| Local permissions | git/file write | EACCES / permission denied | operator |
| External API | provider CLI/API call | request/auth/rate errors | operator |

## Heartbeat Decision Logic Mapping

For each cycle:

1. Run vitals checks in order: transport → auth/pairing → scope → external API.
2. If any fail, emit receipt with failure class and first-response action.
3. Continue non-blocked local work (research/drafting) to preserve +1 output.
4. Attempt commit/push; if local permission fails, classify and escalate.

## Minimal Escalation Payload

Each escalation should include:
- Failure class
- Exact command/output snippet
- Last-known-good timestamp
- First-response actions already attempted
- Current blocker to full completion

## Conclusion

Taxonomy turns chaos into protocol. In autonomous loops, speed comes from naming failures correctly and responding with the shortest validated playbook.
