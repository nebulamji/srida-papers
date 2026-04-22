# Oracle Architecture for Confidence Feeds

## Scope
**Chainlink, Pyth, UMA** mapped to confidence-oracle rails for agent systems.

---

## 1) Chainlink

### Data ingress architecture
- Oracle networks aggregate external sources and publish onchain updates.
- Proxy -> Aggregator pattern decouples consumer interface from aggregator upgrades.
- OCR (Offchain Reporting): node-level observations + signatures, single aggregate onchain transaction.

### Verification architecture
- Multi-node decentralized aggregation reduces single-source manipulation.
- Round-based feed outputs (`latestRoundData`) with explicit contract interfaces.
- Upgrade-safe through proxy indirection.

### Dispute/failure architecture
- Reliability via multi-node consensus and feed governance controls.
- Consumer-side safeguards include stale-data checks and sequencer uptime feeds for L2 outage handling.

### SRIDA mapping
- **LLM confidence as feed**: multiple agent-node confidence submissions aggregated offchain then committed.
- **Confidence score as verifiable signal**: proxy-stable confidence feed contract per workflow type.
- **Multi-agent deliberation**: OCR-style signed consensus envelope.

### Internal primitive
- `ConfidenceProxy -> ConfidenceAggregator`
- `submitObservation(agentNode, confidence, evidenceHash, sig)`
- `aggregateRound(roundId)` emits canonical confidence + dispersion metrics.

---

## 2) Pyth

### Data ingress architecture
- First-party publisher model: market participants publish observations.
- Oracle program aggregates publisher price + confidence interval.
- Cross-chain relay distributes updates from source environment to consumer chains.

### Verification architecture
- Multi-publisher aggregation with confidence interval output (not just scalar value).
- Slot/epoch update cadence with deterministic account/state storage.

### Dispute/failure architecture
- Robustness from publisher diversity and confidence-bound signaling.
- Consumer can reject updates outside freshness/confidence bounds.

### SRIDA mapping
- **LLM confidence feed**: each agent publishes confidence + uncertainty.
- **Verifiable signal**: aggregate confidence interval becomes admissibility gate for settlement.
- **Deliberation oracle**: disagreement width is explicit signal to escalate review.

### Internal primitive
- `Confidence = {mean, lower, upper, freshness}`
- Settlement policy: `if upper-lower > maxSpread -> route to adjudication`.

---

## 3) UMA (Optimistic Oracle + DVM)

### Data ingress architecture
- Asserter posts claim with bond and metadata.
- Liveness window allows disputer challenge.
- Undisputed assertions settle optimistically.

### Verification architecture
- Bonded assertions create economic skin-in-game.
- Disputes escalate to DVM vote-based resolution.
- Final outcome set by tokenholder arbitration process.

### Dispute/failure architecture
- Explicit escalation game: propose -> dispute -> arbitrate.
- Corruption resistance framed economically (cost to corrupt > value to extract).

### SRIDA mapping
- **LLM confidence as assertion**: agent proposes confidence claim with stake.
- **Verifiable signal**: undisputed claim auto-settles after liveness.
- **Multi-agent oracle**: challengers dispute low-quality/biased outputs; governance panel resolves.

### Internal primitive
- `assertConfidence(claimHash, confidence, bond, liveness)`
- `disputeConfidence(assertionId, counterEvidenceHash, bond)`
- `adjudicate(assertionId)` by covenant jury module.

---

## Oracle Pattern Library for Confidence Rails

1. **Proxy-aggregator separation** for upgrade-safe consumer integration.
2. **Multi-source aggregation** with signed observations.
3. **Uncertainty-aware outputs** (confidence interval, not scalar only).
4. **Optimistic fast path** with bonded dispute backstop.
5. **Escalation-to-human/committee** only when disagreement crosses threshold.
6. **Freshness gates** to block stale-confidence settlement.

---

## Source receipts
- https://docs.chain.link/architecture-overview/architecture-overview
- https://docs.chain.link/data-feeds
- https://docs.pyth.network/price-feeds/core/how-pyth-works
- https://docs.uma.xyz/protocol-overview/how-does-umas-oracle-work
