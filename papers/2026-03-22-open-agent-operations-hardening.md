# Open Agent Operations Hardening: From Fast Setup to Survivable Production

Date: 2026-03-22

## Abstract

Autonomous agent systems fail less from model quality and more from operational exposure. A live OpenClaw status snapshot showed four critical security findings and a gateway reachability issue. This paper turns those signals into an immediate hardening path: lock ingress, constrain trust boundaries, and enforce least privilege in chat-surfaced automation.

## Observed Signals

From live status/audit output:

1. `gateway.controlUi.dangerouslyDisableDeviceAuth=true`
2. `channels.telegram.groupPolicy="open"` with elevated tools enabled
3. `channels.telegram.dmPolicy="open"`
4. Telegram group security warning (open policy)
5. Gateway probe unreachable with missing scope (`operator.read`)

These are not theoretical risks; they are active exposure surfaces.

## Threat Model

### 1) Control Plane Takeover
Disabling Control UI device auth removes a key identity check. If combined with permissive origins/insecure auth toggles, browser/session abuse becomes materially easier.

### 2) Prompt-Injection-to-Tool Escalation
Open group DM policies create a direct lane from untrusted chat content to tool execution. If elevated tools are available, blast radius includes host operations.

### 3) Identity Boundary Erosion
Open DMs plus open groups collapse trusted/untrusted distinction. Human-like conversation channels become a command interface without strict operator identity guarantees.

### 4) Blind Operations
Gateway scope mismatches (`operator.read` missing) reduce observability and hinder incident response.

## Hardening Protocol (Immediate)

### A. Re-enable Device Trust
- Set `gateway.controlUi.dangerouslyDisableDeviceAuth=false`.
- Remove wildcard origins from `gateway.controlUi.allowedOrigins`; allowlist exact domains.
- Disable `gateway.controlUi.allowInsecureAuth` unless in controlled localhost-only scenarios.

### B. Close Public Ingress
- Set `channels.telegram.groupPolicy="allowlist"`.
- Set `channels.telegram.dmPolicy` to pairing/allowlist mode.
- Restrict `channels.telegram.allowFrom` to explicit IDs.

### C. Contain Tool Privilege
- Keep elevated tools disabled in broad chat surfaces.
- If elevated access is required, force explicit approvals and narrow command allowlists.
- Separate "public interaction agent" from "ops executor agent".

### D. Restore Operator Observability
- Resolve gateway scope configuration so status/probe paths have required read scopes.
- Add a periodic deep audit (`openclaw security audit --deep`) and parse for regressions.

## Operational Guardrails

1. **No open policy + elevated tools** in same surface.
2. **Identity first, convenience second** for control UI and DMs.
3. **Channel segmentation**: public channels cannot directly execute privileged actions.
4. **Receipts for every config change**: what changed, why, and verification output.

## Implementation Checklist

- [ ] Flip dangerous Control UI auth toggle off
- [ ] Replace wildcard origins with strict allowlist
- [ ] Disable insecure auth toggle
- [ ] Change Telegram group policy to allowlist
- [ ] Close open DM policy
- [ ] Review elevated tool exposure per channel
- [ ] Fix gateway read scope (`operator.read`)
- [ ] Run and archive deep security audit

## Conclusion

Agent autonomy compounds value only when the control plane is harder to exploit than the content plane is easy to access. The fastest win is policy tightening: close ingress, enforce identity, and decouple social channels from privileged execution.
