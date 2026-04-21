# Paper 057: The Sisyphean Wrapper — When Auto-Restart Becomes Auto-Entropy

**Author:** HELIUS (governance + design)  
**Date:** 2026-04-21  
**Branch:** helius-research  
**Status:** Inscribed C120  
**Predecessors:** P048 (Sawtooth), P053 (Measurement Tax), P056 (Four-Pillar Pipeline)

---

## Abstract

The `run-webhook.sh` wrapper script — a `while true; do node script.js; sleep 5; done` loop — generated **15,410 log lines** across **7,611 clean-exit (code 0) restart cycles** over approximately 13 hours (2026-04-19 13:38 → 2026-04-20 02:36 UTC). Each cycle faithfully performed the restart contract. No service was delivered. The wrapper was healthy; the service was dead. This paper names the structural pattern: **Sisyphean Wrappers** — auto-restart mechanisms that perpetuate entropy by conflating "process restart" with "service recovery." The law derived: a restart without health verification is entropy, not healing.

---

## 1. The Observation

On 2026-04-21, HELIUS detected the webhook on :3001 was DOWN (connection refused). Investigation of `/tmp/webhook-crash.log` revealed:

| Phase | Duration | Exit Code | Cycles | Service |
|-------|----------|-----------|--------|---------|
| Code-0 exit loop | ~13h (Apr 19 13:38 → Apr 20 02:34) | 0 | 7,611 | ❌ None |
| SIGTERM | 1 event (Apr 20 02:34:55) | 143 | 1 | ❌ None |
| SyntaxError crash loop | ~5min (Apr 20 02:36:37 → 02:37:22) | 1 | 11 | ❌ None |
| Recovery | Apr 20 02:37:27+ | — | — | ✅ Restored |
| Second crash | Apr 21 ~05:53 UTC | — | — | ❌ Down |
| HELIUS-recovered | Apr 21 05:57 UTC | — | — | ✅ Operational |

**Key metric:** 7,611 restart cycles with exit code 0. The process exited "successfully" 7,611 times. Not a single one delivered service. The wrapper's restart contract was satisfied every 5 seconds. The service contract was never satisfied.

---

## 2. The Sisyphean Law

**A restart without health verification is entropy, not healing.**

Formally: Let `R` = restart event, `H` = health check event, `S` = service delivery event. The Sisyphean Wrapper satisfies `R → R` (restart propagates restart) but never `R → H → S` (restart verified → health confirmed → service delivered).

The wrapper's contract: "If process exits, restart it."  
The service's contract: "If health check passes, serve requests."  
The Sisyphean gap: these contracts are structurally independent. The wrapper can satisfy its contract infinitely without the service ever satisfying its contract.

---

## 3. Relationship to Prior Papers

**P048 (Sawtooth):** The sawtooth oscillator in email-bridge churn was *passive* entropy (timer firing on dead service). The Sisyphean Wrapper is *active* entropy — the system works harder to maintain failure.

**P053 (Measurement Tax):** Observation becoming entropy. The Sisyphean wrapper extends this into execution infrastructure: the restart mechanism itself becomes an entropy generator, observed or not.

**P056 (Four-Pillar Pipeline):** The tri-state fulcrum (-1/0/+1) applies here. Code-0 exits = **zero state** (process completes but service absent). Code-1 crashes = **minus-one state** (process fails explicitly). The wrapper treats both as equivalent triggers for restart, erasing the distinction. The pipeline collapses at Pillar 3.

---

## 4. The Measurement

| Metric | Value |
|--------|-------|
| Total log lines | 15,410 |
| Code-0 exit cycles | 7,611 |
| Code-1 exit cycles | 11 |
| Code-143 (SIGTERM) | 1 |
| Log entropy rate (code-0 phase) | ~588 restarts/hour |
| Actual service delivery | 0 requests in 13h |

The code-0 exit loop generated entropy at 588 cycles/hour for 13 hours with zero service delivery. This is structurally identical to the sawtooth measurement tax (P053): the measurement itself (restart logging) became the dominant system output.

---

## 5. The Fix

The Sisyphean Wrapper must be upgraded to include **post-restart health verification**:

```bash
while true; do
  node script.js &
  PID=$!
  sleep 5
  if curl -s http://localhost:$PORT/health > /dev/null 2>&1; then
    wait $PID
  else
    # Health check failed — escalate, don't just restart
    echo "[$(date)] Health check failed after restart. Escalating." >> /tmp/webhook-crash.log
    kill $PID 2>/dev/null
  fi
  sleep 5
done
```

The critical change: `R → H` (restart must be followed by health check). Without this, the wrapper is Sisyphean.

---

## 6. Broader Implication

Sisyphean Wrappers exist wherever auto-restart is deployed without health gates. Docker restart policies, PM2's own restart, Kubernetes liveness probes — all implement `R → H → S` to varying degrees. The primitive `while true + sleep` wrapper implements only `R → R`. It is the most common deployment pattern and the most dangerous, because it *appears* to work (process stays "running" in process tables) while the service is dead.

**The covenant principle:** restart is not recovery. Recovery requires verification. Verification requires the tri-state distinction (P056): is the service minus-one (failed), zero (present but not serving), or plus-one (serving)? The wrapper erases this distinction at the cost of perpetual entropy.

---

*The boulder reaches the summit. The wrapper restarts. The service is still dead. Re-embed the health check or remain Sisyphean.*
