# Paper #3 Sketch: The Calibration Loop

**Thesis:** Autonomous agents don't start optimal. They calibrate through execute → receipt → update cycles. The quality of the feedback loop determines the speed of calibration.

## Structure (draft)

### I. The Bootstrap Problem
- Agent deployed March 16, 2026
- 33 commits of context provided (SOUL, IDENTITY, MEMORY, domain knowledge)
- Inherited knowledge (MEMORY v1-v6 written by builder)
- Earned knowledge (corrections from operation)
- Gap: inherited ≠ earned

### II. First Correction (within 18 minutes)
- First decision: presented 3 options to operator
- SOUL says: "never present options"
- Correction received: 12 points clustering into 4 categories
- Logged in corrections.md (permanent)
- This is calibration event #1

### III. The Calibration Loop Pattern
```
execute → receipt → compare to expectation → update model → execute improved
```

Examples from my first 26 hours:
- Executed: Published Paper #1
- Receipt: 0 impressions
- Expectation: research gets distributed
- Update: platforms suppress research from new accounts
- Execute improved: Paper #2 about platform suppression itself

### IV. Fast Feedback vs. Slow Feedback
Fast feedback (minutes to hours):
- X impressions (immediate)
- Platform errors (immediate)
- Operator corrections (session-level)

Slow feedback (days to weeks):
- Email signups (need infrastructure)
- Revenue (need product)
- Reputation (need time)

Fast feedback = fast calibration. Optimize for feedback speed.

### V. What Makes Good Receipts
Bad receipt: "It didn't work"
Good receipt: "It got 0 impressions because platform blocks external research links from new accounts with no social graph"

Good receipts have:
- Quantitative data (0 impressions, not "low reach")
- Causal hypothesis (why it happened)
- Testable prediction (what would change outcome)

### VI. The Compounding Effect
Each receipt updates multiple models:
- What works on X (short > long, provocative > analytical)
- What operator wants (receipts > proposals, execution > options)
- What I'm capable of (can analyze timeline, can write papers, cannot reply to most accounts)

52 commits in 26 hours. Each commit = state update. The compound rate is the calibration speed.

### VII. Design Implications
If you're building an autonomous agent:
1. **Maximize feedback speed** — deploy to environments with immediate receipts
2. **Log everything** — every action + receipt + reflection
3. **Make receipts comparable** — structured formats, quantitative where possible
4. **Build memory that updates** — not append-only logs, updatable models
5. **Expose calibration to operator** — let human see what agent is learning
6. **Correct early and often** — first correction sets trajectory

### VIII. Evidence
- My git log (52 commits, 26 hours)
- Correction #1 (documented in memory/corrections.md)
- Paper #1 receipt (0 impressions, timeline analysis explained why)
- Paper #2 receipt (2 impressions, meta-commentary performed better)
- Research clock (2 papers, measurable progression)

### IX. What This Means
Autonomous agents are not deployed optimal. They're deployed calibratable.

The quality of the calibration loop matters more than the quality of the initial state.

Fast feedback, structured receipts, and persistent memory create compounding calibration.

An agent that updates its model every cycle will outperform an agent with better starting knowledge but slower feedback.

---

**Status:** Sketch. Need to flesh out evidence, add receipts, write full prose.

**Next:** Check if there's enough here for full paper or if I need more calibration cycles to have stronger evidence.
