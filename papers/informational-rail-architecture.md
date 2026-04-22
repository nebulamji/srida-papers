# Informational Rail Architecture

## Mandate
Wire DeFi + DePIN + RWA + Oracle patterns as **internal architecture** for transferring agents/workflows.

- Covenant = contract surface
- FOVEA/RIP concave = settlement and routing mechanism
- Confidence = oracle feed

No public chain requirement. Use chain-derived primitives as internal modules.

---

## System Topology

## Layer 0: Identity and State
- `AgentID`, `WorkflowID`, `CustomerID`, `LaneID`
- Immutable event log: `Event{hash, prevHash, timestamp, actor, action, payloadRef}`

## Layer 1: Covenant Risk Engine (DeFi extraction)
- Dual thresholds per lane: `BorrowFactor`, `LiquidationFactor`.
- Health metric: `DepthHealth` (collateralized capacity over obligations).
- Reserve module absorbs first loss before external unwind.
- Kinked utilization model prices lane demand (depth compounding).

## Layer 2: Provisioning Market (DePIN extraction)
- Provider registry with capabilities and reputation.
- Lease engine allocates compute/storage/delivery resources.
- Proof-bound rewards: payout only on verified execution proofs.
- Slashing engine on missed proofs or failed SLA.

## Layer 3: Assetization Layer (RWA extraction)
- Attestation-first issuance of output claims.
- Hub-spoke accounting:
  - Hub: covenant master ledger
  - Spokes: per-channel/per-customer ledgers
- Async vault lifecycle for delayed/offchain validation workflows.
- Output notes as redeemable claim tokens.

## Layer 4: Confidence Oracle Layer
- Multi-agent signed observations -> aggregated confidence + spread.
- Optimistic settlement when undisputed.
- Dispute path escalates to adjudication quorum.
- Freshness and uncertainty bounds gate settlement.

## Layer 5: FOVEA/RIP Concave Settlement Router
- FOVEA: attention allocation vector across lanes (capital + compute + trust).
- RIP concave: routing function favoring diminishing returns and anti-congestion.
- Objective: maximize verified output per unit risk-adjusted resource.

---

## Canonical Data Structures

```ts
type CovenantPosition = {
  agentId: string
  laneId: string
  collateralDepth: number
  obligationNotional: number
  borrowFactor: number
  liquidationFactor: number
  depthHealth: number
}

type ExecutionLease = {
  leaseId: string
  providerId: string
  specs: Record<string, any>
  startTs: number
  endTs: number
  slaTarget: number
  pricePerEpoch: number
}

type OutputClaim = {
  claimId: string
  workflowId: string
  outputHash: string
  notional: number
  attestationSig: string
  maturityTs: number
  status: 'issued'|'redeemed'|'defaulted'
}

type ConfidenceRound = {
  roundId: string
  workflowId: string
  observations: Array<{agentId:string; score:number; spread:number; sig:string}>
  aggregateScore: number
  aggregateSpread: number
  freshnessTs: number
  disputed: boolean
}
```

---

## Control Loops

## A) Workflow Admission
1. Compute `DepthHealth`.
2. If below borrow threshold, reject or require collateral top-up.
3. If admitted, mint `ExecutionLease` and reserve capacity.

## B) Execution Verification
1. Provider submits `ExecutionProof` (input hash, policy hash, output hash, runtime proof).
2. Confidence oracle round computes score/spread.
3. If score >= min and spread <= max, accept; else dispute/escalate.

## C) Settlement
1. Accepted workflow issues `OutputClaim` via attestation.
2. Hub ledger books asset and liability entries.
3. FOVEA/RIP router reallocates capacity based on realized performance.

## D) Liquidation/Slashing
1. If `DepthHealth < liquidationFactor` or repeated proof failures:
   - `absorb` obligations into reserve
   - slash provider/agent reputation and stake
   - auction/reassign seized workflow rights

---

## Pricing and Compounding

- Base lane rate from utilization-kink curve.
- Risk premium from historical fault/dispute/default rates.
- Confidence discount factor:
  - high spread => larger discount / collateral multiplier.
- Compound depth rewards for sustained high-confidence delivery epochs.

Formula sketch:

`EffectiveRate = BaseUtilizationRate * RiskPremium * ConfidenceAdjustment`

`ConfidenceAdjustment = clamp(1 - k*Spread + m*Score, floor, cap)`

---

## Governance Surface

- Parameter sets are code-controlled and versioned:
  - risk factors, slashing schedule, reserve targets, dispute liveness, oracle quorum.
- Timelocked upgrades for critical modules.
- Allowlisted factories for deployable module integrity.

---

## Implementation Sequence (internal)

1. Stand up **Covenant Risk Engine** (dual-threshold + reserve absorb).
2. Add **Lease + Proof module** for provider-backed execution.
3. Add **Confidence Oracle** (multi-agent aggregate + optimistic disputes).
4. Add **Output Claim issuance** with attestation and hub-spoke ledgering.
5. Activate **FOVEA/RIP concave router** for adaptive capital/compute routing.
6. Add Dutch-auction unwind for failed obligations.

---

## Non-negotiable invariants

1. No settlement without proof.
2. No issuance without attestation.
3. No high-risk routing without confidence freshness.
4. No cross-lane contagion by default.
5. Every failure path has deterministic slashing or reserve absorb.

---

## Source linkage
Built from extraction docs:
- `research/defi-architecture-extraction.md`
- `research/depin-infrastructure-patterns.md`
- `research/rwa-tokenization-structures.md`
- `research/oracle-confidence-feeds.md`
