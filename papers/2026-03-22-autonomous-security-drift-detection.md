# Autonomous Security Drift Detection

Date: 2026-03-22

## Abstract

Autonomous systems degrade when security configuration drifts from safe defaults. Drift detection must run continuously, classify severity, and emit remediation-ready receipts. This paper defines critical drift checks, a warn/critical severity model, and an auto-generated remediation receipt format aligned to heartbeat operations.

## Why Drift Detection

Security posture can regress without explicit incidents:
- convenience toggles re-enabled
- channel policies reopened
- origin allowlists widened
- privilege boundaries blurred

Autonomous drift detection converts hidden regressions into explicit, actionable events.

## Drift Check Set (Initial)

## Control UI

1. `gateway.controlUi.dangerouslyDisableDeviceAuth`
- Expected: `false`
- Drift: `true` (critical)

2. `gateway.controlUi.allowedOrigins`
- Expected: explicit allowlist, no wildcard
- Drift: contains `*` (warn → critical if paired with insecure auth)

3. `gateway.controlUi.allowInsecureAuth`
- Expected: `false` (except localhost breakglass)
- Drift: `true` (warn)

## Channel Policies

4. `channels.telegram.groupPolicy`
- Expected: `allowlist`
- Drift: `open` (critical when elevated tools are available)

5. `channels.telegram.dmPolicy`
- Expected: pairing/allowlist
- Drift: `open` (critical)

## Privilege Exposure

6. Elevated tool exposure on open/untrusted surfaces
- Expected: disabled or tightly allowlisted
- Drift: elevated enabled in open policy context (critical)

## Severity Model

## WARN
- Single non-catastrophic deviation
- No immediate exploit chain evident
- Example: insecure auth toggle true with otherwise strict origins

## CRITICAL
- Drift enables direct unauthorized control risk
- Or multiple WARNs compose into exploit path
- Examples:
  - device auth disabled
  - open DM/group with elevated tools
  - wildcard origins + insecure auth in control plane

## Composition Rule

If two or more related WARN drifts occur in same trust zone, escalate to CRITICAL.

## Detection Cadence

- Run drift checks every heartbeat cycle.
- Emit one consolidated drift receipt per cycle with deduplicated findings.
- Suppress duplicates unless severity changes or state transitions.

## Auto-Generated Remediation Receipt Format

```yaml
drift_receipt_id: string
timestamp_utc: string
source: heartbeat
summary:
  critical_count: int
  warn_count: int
findings:
  - key: string
    observed: string
    expected: string
    severity: warn|critical
    risk_reason: string
    remediation:
      action: string
      verification: string
composition:
  escalated: boolean
  reason: string|null
next:
  owner: operator|agent
  required: boolean
  priority: P1|P2|P3
```

## Example (Current Runtime)

- Critical findings present:
  - device auth disabled
  - open group policy with elevated tools
  - open DM policy
- Warn findings present:
  - wildcard allowed origins
  - insecure auth toggle
- Composition escalation:
  - warn pair in control UI zone compounds risk → treat as critical incident context

## Integration with Existing Papers

- Uses failure taxonomy for classification labels.
- Uses incident loop for containment/recovery sequence.
- Uses operational receipts schema for audit traceability.
- Feeds SLO error budget via security-drift critical event counts.

## Conclusion

Drift detection is preventive incident response. Autonomous systems stay safe when configuration regressions are detected, classified, and converted into immediate remediation receipts.
