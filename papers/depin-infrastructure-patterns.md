# DePIN Infrastructure Patterns

## Scope
**Helium, Render, Akash, Filecoin** mapped to agent infrastructure transfer rails.

---

## 1) Helium

### Tokenization of physical infrastructure
- Community-owned wireless hardware (hotspots) supplies coverage.
- Hotspot operators are paid in HNT for coverage and traffic handling.
- Usage paid via Data Credits (USD-pegged utility token) minted by burning HNT (BME-style).

### Incentive architecture
- Emissions + usage-coupled token economics (burn/mint, net emissions smoothing).
- Economic split between infrastructure operators and network utility demand.

### Work verification architecture
- Network-side validation of coverage/service contribution + onchain accounting.
- Reward eligibility tied to measurable network participation.

### Mapping to SRIDA rails
- **Agent compute provisioning**: hotspot model -> distributed agent endpoints.
- **Agent delivery verification**: traffic-handled metric -> message/workflow delivered metric.
- **Agent reputation**: sustained uptime + verified delivery converts to reputation emissions.

### Internal primitive
- `DeliveryCredits` (USD-pegged internal utility credits) burned from covenant reserve token.
- `OperatorReward = f(uptime, deliveredJobs, QoS)` with epoch smoothing.

---

## 2) Render

### Tokenization of physical infrastructure
- Idle GPU capacity tokenized into render marketplace supply.
- Tiered service market (cost/speed/security bands) based on node quality.

### Incentive architecture
- Burn-and-Mint Equilibrium (BME): creator fee flow + operator emissions + protocol fee split.
- Node rewards tied to availability and successful completion.
- Reputation-weighted job assignment and throughput rights.

### Work verification architecture
- Timestamped task records, hashed job graph, creator approval loop.
- Watermarked output gating before final settlement.
- Dispute path with committee review and slash-like penalties.

### Mapping to SRIDA rails
- **Agent compute provisioning**: GPU tiering -> agent runtime tiering.
- **Agent delivery verification**: hashed job DAG + approval -> workflow proof-of-delivery.
- **Agent reputation**: acceptance/rejection ratio updates allocator priority.

### Internal primitive
- `ProofOfWorkDelivery = hash(input, policy, output, timestamp)`
- `SettlementGate` releases final output only after acceptance signal.
- `DisputeEscalation` triggers slashing of operator reputation and reward share.

---

## 3) Akash

### Tokenization of physical infrastructure
- Decentralized cloud marketplace with provider-side compute inventory.
- Lease lifecycle defines deployment tenancy and payment rights.

### Incentive architecture
- Providers bid for workloads, earn from active leases.
- Operational reliability determines continued lease competitiveness.

### Work verification architecture
- Lease state tracked against deployment + provider health checks.
- Provider verification includes onchain registration + runtime liveness.

### Mapping to SRIDA rails
- **Agent compute provisioning**: lease markets -> agent-runtime lease contracts.
- **Agent delivery verification**: lease health + run completion -> delivery attestation.
- **Agent reputation**: bid win-rate + SLA compliance drive future allocation.

### Internal primitive
- `AgentLease{providerId, specs, term, price, sla}`
- Health oracle feeds `LeaseScore`; low score triggers migration.
- Bidding engine allocates new workloads by `price x reliability` frontier.

---

## 4) Filecoin

### Tokenization of physical infrastructure
- Storage and retrieval providers contract with clients via deals.
- Offchain deal negotiation, onchain publication/validation.
- Sectorized storage units with cryptographic proof obligations.

### Incentive architecture
- Collateralized service commitments with penalties for non-performance.
- Fault, sector, termination, and consensus penalties enforce reliability.

### Work verification architecture
- Proof-of-Spacetime style cryptographic continuity checks.
- Slashing/fault fees for missed proofs or malicious behavior.

### Mapping to SRIDA rails
- **Agent compute provisioning**: storage deals -> durable memory/context leases.
- **Agent delivery verification**: cryptographic proofs -> proof-of-execution windows.
- **Agent reputation**: fault history and recoveries become reliability score.

### Internal primitive
- `ExecutionWindowProof` required per epoch for active agent contracts.
- Faulted agents pay reserve penalties and lose priority routing.
- Recovery path lowers but does not erase historical fault weight.

---

## DePIN Pattern Library for Internal Use

1. **Burn/mint utility loop** for usage-priced internal credits.
2. **Tiered supply marketplace** for heterogeneous compute quality.
3. **Lease-based provisioning contracts** for explicit lifecycle control.
4. **Proof-bound rewards** (no proof, no payout).
5. **Dispute + slashing backstop** with reversible governance path.
6. **Epoch smoothing** to stabilize reward volatility and prevent gaming.

---

## Source receipts
- https://docs.helium.com/
- https://docs.helium.com/tokens/hnt-token/
- https://renderfoundation.com/whitepaper
- Render whitepaper PDF (framerusercontent asset)
- https://akash.network/docs/
- https://akash.network/docs/providers/operations/
- https://docs.filecoin.io/
- https://docs.filecoin.io/basics/what-is-filecoin/storage-model
- https://docs.filecoin.io/storage-providers/filecoin-economics/slashing
