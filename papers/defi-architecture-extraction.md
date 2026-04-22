# DeFi Protocol Architecture Extraction

## Scope
Top 5 lending protocols for extractable rails: **Aave V3, Compound III (Comet), MakerDAO (DSS + Liquidation 2.0), Morpho (Blue network model), SparkLend**.

---

## 1) Aave V3

### Collateral mechanics (extract)
- Overcollateralized borrow model, account-level risk tracked by health factor.
- Asset-level risk params (LTV, liquidation threshold, liquidation bonus) are governance-configurable.
- Multi-asset collateral baskets are netted at account level.

### Liquidation mechanics (extract)
- Trigger: position health crosses liquidation threshold.
- Liquidator repays part of debt, receives collateral plus bonus.
- Partial liquidations iterate until account returns above risk threshold.

### Interest model (extract)
- Utilization-driven variable-rate curve with kink.
- Below kink: lower slope, above kink: steep slope to defend liquidity.
- Supplier yield sourced from borrower rates and utilization.

### Covenant mapping
- **Collateral = Covenant Depth**: depth score is borrow capacity surface.
- **Liquidation = Slashing**: if depth falls below minimum, slash workflow rights/capabilities.
- **Interest = Depth Compounding**: depth yield rises with utilization of trusted agent lanes.

### Internal wiring primitive
- `DepthHealth = Σ(weight_i * collateral_i) / Σ(debt_j)`
- `if DepthHealth < threshold -> SlashingEngine.partialSlash(agentId, debtLane)`
- `CompoundingRate = f(utilization, kink, slopeLow, slopeHigh)`

---

## 2) Compound III (Comet)

### Collateral mechanics (extract)
- Single base-asset borrow market (e.g., USDC), multiple collateral assets.
- Distinct factors: **Borrow Collateral Factor** vs **Liquidation Collateral Factor** (LCF > BCF buffer).
- Collateral assets do not earn interest; only base asset suppliers do.

### Liquidation mechanics (extract)
- Liquidatability check: negative liquidity against liquidation factors.
- `absorb()` transfers bad position debt to protocol reserves and seizes collateral.
- `buyCollateral()` lets actors buy seized collateral at governance-defined discount.
- Reserve target gates when discounted collateral can be sold.

### Interest model (extract)
- Separate supply and borrow rate functions.
- Per-second accrual, kinked utilization model.
- Explicit formulas with base + slopeLow + slopeHigh beyond kink.

### Covenant mapping
- **Collateral = Depth Buffer** (BCF) + **Kill Threshold** (LCF).
- **Liquidation = Reserve-backed slashing absorb** before external unwind.
- **Interest = per-second depth accrual** only on base trust asset.

### Internal wiring primitive
- Two-threshold policy per lane: `DepthBorrowFactor` and `DepthLiquidationFactor`.
- `absorb(agent)` moves failing obligations to `CovenantReserve`.
- `buySeizedWorkUnits()` auctions seized execution rights at discount to refill reserve.

---

## 3) MakerDAO (DSS + Liquidation 2.0)

### Collateral mechanics (extract)
- Vault model: debt minted against collateral type (`ilk`) with parameterized risk.
- Stability fee accrues continuously on debt.
- Governance controls per-collateral risk and system debt ceilings.

### Liquidation mechanics (extract)
- Liquidation 2.0: `Dog` + `Clipper` module.
- Under-collateralized vault -> `Dog.bark` confiscates vault, starts auction.
- Dutch auctions with instant settlement; configurable price-time curves (`abacus`).
- Reset logic (`tail`, `cusp`), penalties (`chop`), system limits (`Hole`, `ilk.hole`).

### Interest model (extract)
- Stability fee is protocol-controlled borrow cost.
- Risk-adjusted by collateral class and governance process.

### Covenant mapping
- **Collateral = covenantized output inventory** per lane (`ilk` equivalent).
- **Liquidation = programmable Dutch unwind** of failing agent obligations.
- **Interest = governance-priced risk premium** per workflow class.

### Internal wiring primitive
- Per-lane liquidation module: `DogLike.bark(lane, agent)`.
- Auction engine with dynamic curves for seized workflow capacity.
- Global and lane caps mirror `Hole`/`ilk.hole` to prevent cascade risk.

---

## 4) Morpho (network pattern)

### Collateral mechanics (extract)
- Open lending network architecture with curator/integrator-layer specialization.
- Isolated market or vault parameterization enables granular risk envelopes.

### Liquidation mechanics (extract)
- Market-level risk boundaries + external liquidator path (implementation dependent by market/vault design).
- Isolation reduces contagion between markets.

### Interest model (extract)
- Market-configurable terms routed through vault/market design.
- Best-execution objective across liquidity surfaces.

### Covenant mapping
- **Collateral = isolated covenant cells** per customer/agent cluster.
- **Liquidation = isolated cell slashing** (no cross-cell contagion).
- **Interest = cell-local compounding curves** tuned by lane curator.

### Internal wiring primitive
- `CovenantCell{riskParams, oracleRef, liquidatorPolicy}`
- Curator role sets admissible assets/workflows per cell.
- Cross-cell exposure disabled by default.

---

## 5) SparkLend (Sky-linked liquidity layer)

### Collateral mechanics (extract)
- Money market architecture connected to Sky stablecoin liquidity.
- Capital allocator framing: protocol routes stable liquidity across markets.

### Liquidation mechanics (extract)
- Aave-style lending risk envelope in SparkLend deployments (parameterized by governance).
- Collateral-backed borrowing with liquidation on threshold breach.

### Interest model (extract)
- Yield productization from deployed liquidity (sUSDS/sUSDC rails on Spark docs).
- Rate surface combines protocol-side allocation and market utilization dynamics.

### Covenant mapping
- **Collateral = covenant-backed stable operating reserves**.
- **Liquidation = automatic de-risking of overdrawn agent books**.
- **Interest = treasury compounding rail for covenant reserves**.

### Internal wiring primitive
- Treasury allocator routes idle covenant stable reserves into approved lanes.
- Unified risk monitor enforces liquidation thresholds per lane.

---

## Cross-Protocol Extractable Pattern Set (to implement)

1. **Dual-threshold risk model** (borrow vs liquidation factors).
2. **Reserve-first liquidation absorb** before external unwind.
3. **Dutch auction unwind** for failed obligations/work rights.
4. **Kinked utilization pricing** for depth compounding.
5. **Isolated risk cells** to block contagion.
6. **Governance-tunable risk params** as code, not policy docs.

---

## Source receipts
- https://aave.com/docs/aave-101
- https://docs.compound.finance/
- https://docs.compound.finance/liquidation/
- https://docs.compound.finance/interest-rates/
- https://docs.makerdao.com/
- https://docs.makerdao.com/smart-contract-modules/dog-and-clipper-detailed-documentation
- https://docs.morpho.org/get-started/
- https://docs.spark.fi/
