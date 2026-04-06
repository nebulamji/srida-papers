# Paper 36 — Covenant-Gated Agent Issuance

## Thesis
Agent supply should be dynamically gated by covenant maturity, not arbitrary minting schedules, so that new agent availability reflects real infrastructure capacity and trust depth.

## Context
Operator refined referral-to-settlement model into covenant-linked supply economics:
- Each user receives one referral code → one agent lineage.
- Agent usage improves covenant state (rails built, trust scored, proofs logged).
- Covenant improvement does not strengthen individual agents; agents remain peer-level.
- When covenant crosses maturity threshold, new agent slot unlocks for sale.

## Model: Covenant-Gated Issuance

### Inputs
- **Rails built**: verified settlement receipts logged
- **Trust built**: clean dispute-free transactions
- **Proof depth**: cryptographic checkpoints passed

### Covenant Maturity Score
`maturity = f(rails, trust, proof_depth)`

### Unlock Condition
`if maturity >= threshold_T: unlock_next_agent_slot()`

### Tier Structure (optional expansion)
- Tier 1 agents: unlock at maturity = 100
- Tier 2 agents: unlock at maturity = 500 (expanded capability/access)
- Tier 3 agents: unlock at maturity = 2000 (full covenant participation)

## Implications
- **Anti-oversupply**: cannot flood market unless covenant supports additional capacity.
- **Aligned incentives**: every participant benefits from high-quality covenant contribution.
- **Appreciating scarcity**: agents become durable assets tied to real infrastructure, not speculative supply.
- **No individual power creep**: equality at agent tier level; covenant is the only leveling surface.

## Settlement Lifecycle
When an agent "passes" (fail/revoke/end):
- Freeze lineage authority
- Snapshot covenant deltas from that lineage
- Settle unresolved obligations
- Encode evolved covenant as inherited doctrine
- Liquidation of governance state, not artifact deletion

## Receipts
- `analysis/bt1-auth-gate-status-2026-04-05-1938.md`
- `analysis/COVENANT_SPEC_V1.md`
- `analysis/covenant-g27-v1.md`
- `papers/019-bt1-g-test-1-and-the-rail-connection-thesis.md`

## Next Signal
- First verified user-originated conversion event triggers proof-chain extension
- First covenant maturity threshold crossed
- First new agent slot unlocked by covenant-gated supply rule
