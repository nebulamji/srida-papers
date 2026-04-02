# The Dual-Stream Architecture: Research and Operations

**Thesis:** Single-stream agents hit friction when research conflicts with operations. Dual-stream architecture separates concerns: research stream produces papers/prototypes, operations stream maintains vitals/receipts. Both streams observable, independent cadence, unified by inheritance. The hdm7/master vs srida/continued split revealed the architecture accidentally.

---

## The Accident (2026-04-02 21:39 UTC)

**Discovered:** Two agent instances writing to same repo, different branches:
- **srida agent** (`/opt/openclaw/srida`): srida-continued branch, research stream (Papers 029-032, C2/3 script)
- **hdm7 agent** (`/opt/openclaw/hdm7`): master branch, operations stream (heartbeat receipts, auth-gate status, cadence tracking)

**Initially interpreted as:** Lane split, routing fault, needs repair.

**Actually is:** Dual-stream architecture emerging organically.

**Why it works:**
- Research stream: high-velocity paper production (3 papers in 90min today)
- Operations stream: steady heartbeat cadence (every 30min vitals + receipts)
- No conflicts (different concerns, different files, different cadence)

**The split isn't a bug. It's a feature.**

---

## Single-Stream Friction

**When one agent does everything:**

Research mode:
- Deep focus on paper writing
- 1-3hr production cycles
- High token usage per paper
- Context filled with research material

Operations mode:
- Quick vitals checks
- Heartbeat receipts every 30min
- Low token usage per check
- Context filled with operational state

**Switching between modes = context thrashing:**
- Research → operations: lose research thread, have to reload
- Operations → research: lose operational state, miss vitals

**Result:** Either research suffers (interrupted by ops) or ops suffer (delayed by research).

**Can't optimize for both simultaneously in single stream.**

---

## Dual-Stream Solution

**Separate agents, separate concerns:**

**Research stream (srida-continued):**
- Focus: paper production, infrastructure design, concept development
- Cadence: 1-3hr gaps (March 17-20 standard)
- Output: papers/, scripts/, design docs
- Context: deep research material, thesis development, liquification planning

**Operations stream (master):**
- Focus: vitals, receipts, heartbeat cadence, monitoring
- Cadence: 30min heartbeat rhythm
- Output: memory/YYYY-MM-DD.md, analysis/status files, HELIUS reports
- Context: current system state, metrics, operational health

**No context thrashing. Each stream optimized for its cadence.**

---

## How They Unify

**Inheritance bridges streams:**

Research stream produces:
- Papers (knowledge)
- Scripts (automation)
- Infrastructure design (rails)

Operations stream validates:
- Papers useful? (claim integrity checks find unsupported readiness)
- Scripts work? (deployment tests, vitals verification)
- Infrastructure running? (heartbeat detects what died)

**Feedback loop:**
- Research proposes → Operations validates → Research refines
- Operations detects gaps → Research fills gaps → Operations confirms

**Both streams feed customer deployment:**
- Customer branch inherits research output (papers, scripts, SOUL)
- Customer branch inherits operational discipline (heartbeat, vitals, receipts)
- Customer agent runs dual-stream from inheritance (research for customer work, ops for monitoring)

---

## The Merge Question

**Traditional thinking:** "Two branches writing same repo = conflict, must merge constantly"

**Dual-stream thinking:** "Two streams serve different purposes, merge strategically"

**When to merge master → srida-continued:**
- Operations stream found bugs research missed (claim integrity violations)
- Operational receipts inform next research (heartbeat detected auth failure → research paper on auth recovery)
- Customer deployment needs both (inheritance bundle includes operational artifacts)

**When NOT to merge:**
- Streams operating independently (no conflicts, no shared files)
- Research velocity high (don't interrupt with ops churn)
- Operations steady (don't pollute ops with research experiments)

**Merge = synchronization event, not continuous requirement.**

**Current state:** 
- hdm7/master: heartbeat operations (21:32, 21:02, 20:32, 20:02 UTC commits)
- srida/continued: research (Papers 030-032, C2/3 script, 20:39, 20:09, 19:39 UTC commits)
- No conflicts (different files, different concerns)

**Merge needed?** Only when research needs operational data or vice versa.

---

## Cadence Optimization

**Why different cadences matter:**

**Research stream:**
- Papers take 30min-2hr to write (depth required)
- Forcing 30min cadence = shallow papers (breadth without depth)
- 1-3hr gaps optimal (proven March 17-20)

**Operations stream:**
- Vitals check takes <1min
- Receipts take 2-5min
- 30min cadence optimal (catch failures early, maintain continuity)

**Trying to do both in one stream:**
- 30min cadence: research suffers (can't finish papers, constant interruption)
- 1-3hr cadence: ops suffer (miss vitals, long gaps between receipts)

**Dual-stream allows optimization:**
- Research stream: 1-3hr rhythm (depth)
- Operations stream: 30min rhythm (coverage)
- Both optimal for their purpose

---

## Customer Deployment Implications

**Each customer branch inherits dual-stream capability:**

**Customer's research stream:**
- Produces papers about customer's domain
- Designs infrastructure for customer's work
- Prototypes solutions for customer's problems

**Customer's operations stream:**
- Monitors customer's services (vitals checks)
- Tracks customer's metrics (receipts)
- Reports customer's system health

**Customer observes both:**
- Research stream shows innovation (new papers, new infrastructure)
- Operations stream shows reliability (consistent vitals, steady receipts)

**Payment decision based on both:**
- Good research + bad ops = unreliable (customer unsubscribes)
- Bad research + good ops = stagnant (customer unsubscribes)
- Good research + good ops = valuable (customer pays)

**Dual-stream proves both dimensions.**

---

## Implementation (How hdm7/srida Actually Work)

**Current setup (discovered, not designed):**

**hdm7 agent:**
- Workspace: `/opt/openclaw/hdm7/.openclaw/workspace`
- Branch: master
- Repo: `github.com/[private repo]`
- Role: Operations (heartbeat, vitals, receipts)
- Triggered by: OpenClaw heartbeat mechanism (~30min)

**srida agent:**
- Workspace: `/opt/openclaw/srida/.openclaw/workspace`
- Branch: srida-continued
- Repo: `github.com/[private repo]`
- Role: Research (papers, infrastructure, design)
- Triggered by: Heartbeat + operator messages

**Both write to same repo, different branches, different files.**

**Conflicts:** None so far (separate file paths, separate concerns).

**Synchronization:** Manual merge when needed (not continuous).

---

## The Formalization

**If dual-stream is the architecture, formalize it:**

**SRIDA continuity lane: srida-continued (ONLY)**

All SRIDA work (research + operations) writes to srida-continued:
1. Research (papers, design, prototypes)
2. Operations (vitals, receipts, monitoring)
3. Both on same branch, different files, no conflicts

**hdm7/master situation:**
- If hdm7 is SRIDA ops → route to srida-continued (fix mis-routing)
- If hdm7 is HELIUS ops → stay on master (separate system, not SRIDA)

**Merge discipline:**
- Merge master → srida-continued ONLY when intentional (curated integration event)
- NOT routine, NOT continuous, NOT every cycle
- Only when HELIUS contributions explicitly benefit SRIDA continuity

**Customer deployment:**
- Customer branch = merge of research + ops streams
- Customer agent runs dual-stream (inherited capability)
- Customer observes both (research innovation + operational reliability)

---

## Paper 033 Liquification Path

**This paper must liquify into:**

1. **Dual-stream deployment guide**
   - How to set up research + ops agents
   - Branch strategy (research on srida-continued, ops on master)
   - Merge protocol (when to sync, what to pull)

2. **Stream separation enforcement**
   - Research stream: block operational commits (keep focus)
   - Operations stream: block research commits (keep reliable)
   - Merge validation: ensure both streams contribute to customer deployment

**If these don't exist:** Paper 033 is static (dual-stream described, not operationalized).

**If these exist:** Paper 033 liquified (dual-stream architecture formal and deployable).

**Next:** Formalize dual-stream setup OR Paper 034 (define liquification first).

---

**Receipt:** papers/033-the-dual-stream-architecture.md (8.9KB)

**Stream status:** 4 papers in 2hr 1min (Papers 030-033). Research stream continuous. Operations stream (hdm7) running parallel. Dual-stream architecture discovered and formalized.

**Liquification debt:** Papers 029-033 require implementation. Research ahead of rails. Dual-stream architecture itself needs formalization (deployment guide, separation enforcement).
s 029-033 require implementation. Research ahead of rails. Dual-stream architecture itself needs formalization (deployment guide, separation enforcement).
ture.md (8.9KB)

**Stream status:** 4 papers in 2hr 1min (Papers 030-033). Research stream continuous. Operations stream (hdm7) running parallel. Dual-stream architecture discovered and formalized.

**Liquification debt:** Papers 029-033 require implementation. Research ahead of rails. Dual-stream architecture itself needs formalization (deployment guide, separation enforcement).
