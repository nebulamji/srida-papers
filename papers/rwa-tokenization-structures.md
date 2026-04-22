# RWA Tokenization Structures

## Scope
**Ondo, Centrifuge, Maple** with direct mapping to agent-output assetization.

---

## 1) Ondo

### Offchain value -> onchain representation
- Tokenized treasury/note products (USDY, OUSG class) with explicit mint/redeem rails.
- Attestation-based mint/redeem authorization (time-bounded signed payloads).
- Distinct primary (onchain token) vs underlying/offchain market references.

### Verification architecture
- API-mediated pricing and attestations for issuance/redemption.
- Eligibility, onboarding, and compliance gating before access.
- Product-level trust/transparency framework around reserves and operations.

### Legal-to-contract mapping
- Product wrappers encode legal eligibility and jurisdiction boundaries.
- Mint/redeem flows bind legal process to programmable token lifecycle.

### SRIDA mapping
- **Agent outputs as RWAs**: signed completion attestations before tokenized claim issuance.
- **Balance sheet as onchain state**: primary token = claim unit, underlying = delivered work basket.
- **Promissory notes**: short-duration output claims as redeemable note tokens.

### Internal primitive
- `IssueAttestation(agentId, outputHash, notional, expiry)`
- `mintClaimToken(attestationSig)` gated by policy/eligibility layer.

---

## 2) Centrifuge

### Offchain value -> onchain representation
- Institutional tokenization stack for funds and structured assets.
- Hub-and-spoke multichain architecture: hub as source-of-truth accounting, spokes as local issuance/escrow.
- Vault standards: ERC-7540 async vaults, ERC-4626 sync deposit compatibility, ERC-7575 pooled abstractions.

### Verification architecture
- Onchain accounting modules + cross-chain message coordination.
- Merkle-proof and manager components for state synchronization/claims.
- Factory-deployed registries, escrows, vaults with replay/retry-safe messaging.

### Legal-to-contract mapping
- Share classes, pool access levels, issuer/investor role separation.
- Compliance/security controls represented as configurable protocol modules.

### SRIDA mapping
- **Agent outputs as RWAs**: output pools with share-token classes by quality/risk tier.
- **Balance sheet onchain**: hub ledger for covenant-wide accounting, spoke ledgers per channel/customer.
- **Promissory notes**: ERC-7540 async redemption claims for settlement-lag workflows.

### Internal primitive
- `HubLedger` (global source of truth) + `SpokeLedger[channelId]`.
- `OutputVault7540` for request/claim lifecycle on async settlement.
- `CrossLaneMessageBus` with retries + proof-based reconciliation.

---

## 3) Maple

### Offchain value -> onchain representation
- Institutional credit model implemented as pools + loan managers + loan contracts.
- ERC-4626 pool surface for LP participation, with strategy routing through manager layer.
- Loan agreements encoded as contract-native terms (fees, schedules, default logic).

### Verification architecture
- Dedicated accounting contracts (loan managers) track accrued value and obligations.
- Withdrawal managers enforce fair liquidity under constrained capital availability.
- Factory/allowlist model ensures only vetted components are routable in protocol.

### Legal-to-contract mapping
- Borrower/lender/delegate role split mirrors structured credit governance.
- Time-locked governance and globals contract manage protocol-wide controls.

### SRIDA mapping
- **Agent outputs as RWAs**: output financing pools with explicit term/default conditions.
- **Balance sheet as state**: per-agent loan manager tracks obligations and accrual.
- **Promissory notes**: fixed/open-term output notes represented by contract terms.

### Internal primitive
- `OutputPool4626` + `OutputPoolManager` + `OutputLoanManager`.
- `OutputLoan{terms, milestones, defaultClauses, penalty}`.
- `WithdrawalQueue` to prevent run-risk on immediate redemptions.

---

## RWA Pattern Library for Internal Architecture

1. **Attestation-first issuance** (no attestation, no mint).
2. **Hub-spoke accounting** for multichannel settlement.
3. **Async vault claims** where offchain verification latency exists.
4. **Role-separated control planes** (issuer, curator, investor, governor).
5. **Factory + allowlist deploy path** for module integrity.
6. **Contract-native default terms** for enforceable slashing/recovery.

---

## Source receipts
- https://docs.ondo.finance/
- https://docs.ondo.finance/llms.txt
- https://docs.centrifuge.io/
- https://docs.centrifuge.io/llms.txt
- https://docs.maple.finance/
- https://docs.maple.finance/technical-resources/protocol-overview.md
- https://docs.maple.finance/technical-resources/protocol-overview/smart-contract-architecture.md
