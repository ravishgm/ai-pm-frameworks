# AI PM Frameworks · `ai-pm-frameworks`

> Decision frameworks for product managers evaluating, scoping, and shipping AI features in enterprise industrial software.

---

## Why I Built This

The hardest part of being a PM at a supply chain software company in the AI era isn't knowing what AI can do. It's knowing how to make a defensible decision about whether to build an AI feature, how to phase it safely, and what "good enough" looks like before you ship it to customers who run 24/7 distribution centers.

Most AI PM frameworks I found were built for consumer apps or SaaS products — where a bad recommendation is annoying but recoverable. In warehouse software, the stakes are different. A misclassified alarm, a bad routing decision, or an AI feature that operators don't trust can stop a sorter, miss an SLA, or get the feature blamed for a downtime event it didn't cause.

I built these frameworks to answer the specific questions I kept hitting in my own work: *Should we actually add AI to this feature, or are we doing it because we feel pressure to? How do we sequence an AI feature without blowing up customer trust? What does acceptance criteria look like for a model, not a UI?*

These are my working answers. They'll evolve as I use them.

---

## Framework Index

| Framework | When to Use |
|-----------|------------|
| [AI Feature Decision Matrix](#1-ai-feature-decision-matrix) | Should this feature be AI-powered at all? |
| [AI Sequencing Model](#2-ai-sequencing-model) | How do you phase AI from shadow mode to autonomous? |
| [Operator Trust Ladder](#3-operator-trust-ladder) | What level of AI authority is appropriate for this use case? |
| [AI Acceptance Criteria Template](#4-ai-acceptance-criteria-template) | What does "good enough to ship" look like for an AI feature? |
| [Build vs Buy vs Partner for AI](#5-build-vs-buy-vs-partner-for-ai) | Which path makes sense for this capability? |

---

## 1. AI Feature Decision Matrix

**When to use**: You have an idea for an AI feature and need to decide whether to build it.

Score each dimension 1–3:

| Dimension | 1 (Low) | 2 (Medium) | 3 (High) |
|-----------|---------|-----------|---------|
| **Rule-based ceiling**: How badly does the current rule-based approach fail? | Rules work well | Rules work but are brittle | Rules fundamentally can't solve this |
| **Data availability**: Is the required training data already captured? | No data exists | Partial data | Clean, labeled data available |
| **Error tolerance**: How bad is it when the AI is wrong? | Catastrophic / hard to reverse | Recoverable with effort | Easily detected and corrected |
| **Volume**: Does this operate at scale where AI efficiency matters? | Low volume, manual OK | Medium volume | High volume, automation critical |
| **Differentiation**: Is AI here a competitive differentiator? | Table stakes / parity | Nice to have | Meaningful moat |

**Scoring**:
- 12–15: Strong candidate. Prioritize for roadmap.
- 8–11: Conditional. Resolve data or error tolerance gaps first.
- 5–7: Wait. Address foundational gaps before committing to AI.
- <5: No-go for now. The problem may not be AI-appropriate.

---

## 2. AI Sequencing Model

**When to use**: You've decided to build an AI feature. This is how you phase it.

```
Phase 0 · INSTRUMENT
────────────────────
Goal: Ensure the data pipeline exists.
Ship: Logging, event capture, labeling hooks.
Don't call this AI yet.

Phase 1 · SHADOW MODE
──────────────────────
Goal: Run AI in parallel without customer impact.
Ship: AI makes predictions; humans still decide.
Measure: Accuracy vs. human decisions (or rule-based baseline).
Gate: >X% accuracy on held-out data before advancing.

Phase 2 · AI RECOMMENDATIONS
──────────────────────────────
Goal: AI advises; human confirms.
Ship: "AI suggests: Route to Lane 4 [Accept / Override]"
Measure: Override rate, outcome quality when AI is accepted vs. overridden.
Gate: Override rate <Y% AND outcome quality meets threshold.

Phase 3 · AI WITH HUMAN OVERRIDE
──────────────────────────────────
Goal: AI decides; human can intervene.
Ship: AI acts autonomously by default; override always available.
Measure: Override rate, exception escalation rate.
Gate: Appropriate for use cases where speed > perfect accuracy.

Phase 4 · FULL AUTONOMY
────────────────────────
Goal: AI decides with no human in the loop.
Ship: Only when error tolerance is very high.
Appropriate for: Low-stakes, high-frequency, easily-reversible decisions.
```

**Key principle**: Never skip phases. The data from each phase is required to safely advance to the next.

---

## 3. Operator Trust Ladder

**When to use**: Designing the UX for an AI feature in an operational environment.

```
Level 1 · INFORM
─────────────────
AI provides information. No recommendation.
Example: "Sorter efficiency is 12% below baseline this shift."
Use when: Operators are experts; surfacing data is enough.

Level 2 · SUGGEST
──────────────────
AI makes a recommendation. Human decides.
Example: "Consider rerouting high-priority orders to Lane 3."
Use when: AI accuracy is moderate; stakes are medium.

Level 3 · RECOMMEND WITH CONFIDENCE
─────────────────────────────────────
AI makes a specific recommendation with confidence signal.
Example: "Route to Lane 3 (High confidence). Tap to apply."
Use when: AI accuracy is high; operators need fast decisions.

Level 4 · ACT WITH NOTIFICATION
─────────────────────────────────
AI takes action; operator is notified and can undo.
Example: "AI rerouted 847 units to Lane 3. [Undo]"
Use when: Speed is critical; error is reversible; trust is established.

Level 5 · FULLY AUTONOMOUS
───────────────────────────
AI acts; no human notification unless exception.
Use when: Decision frequency is too high for human involvement; error impact is minimal.
```

**Design rule**: Start every AI feature at Level 1 or 2. Earn the right to advance levels through production data.

---

## 4. AI Acceptance Criteria Template

**When to use**: Writing the definition of done for an AI feature.

```markdown
## AI Feature Acceptance Criteria

### Accuracy Thresholds
- [ ] Precision on held-out test set: ≥ [X]%
- [ ] Recall on held-out test set: ≥ [X]%
- [ ] Performance does not degrade >5% on data from a new customer site

### Latency
- [ ] P95 inference time: ≤ [X] ms (must not block the real-time control loop)

### Failure Modes
- [ ] System falls back to rule-based behavior gracefully when model confidence < [threshold]
- [ ] Failure mode is logged and observable in monitoring
- [ ] No silent failures — all AI decisions are auditable

### Human Override
- [ ] Operator can override any AI decision within [X] seconds
- [ ] Override is logged with timestamp and operator ID
- [ ] Override rate is tracked in product analytics

### Explainability (if applicable)
- [ ] For each AI decision, a human-readable reason is surfaced in the UI
- [ ] Reason is accurate (not post-hoc rationalization)

### Monitoring
- [ ] Model performance dashboard exists with: accuracy over time, override rate, confidence distribution
- [ ] Alerting configured for: accuracy drop >10%, inference latency spike, override rate spike
```

---

## 5. Build vs. Buy vs. Partner for AI

**When to use**: Deciding how to acquire an AI capability.

| Dimension | Build | Buy (vendor/platform) | Partner |
|-----------|-------|----------------------|---------|
| **Core to your product?** | Yes | No | Depends |
| **Proprietary data advantage?** | Yes | No | Sometimes |
| **Time to production?** | Long (6–18 mo) | Fast (weeks) | Medium (3–9 mo) |
| **Customization needed?** | High | Low | Medium |
| **Internal ML capability?** | Required | Not required | Helpful |
| **IP ownership?** | Full | None | Negotiated |
| **Cost structure?** | CapEx heavy | OpEx/per-seat | Hybrid |

**Heuristics**:
- If the AI capability is your primary differentiator: **Build**
- If it's infrastructure/commodity: **Buy**
- If you need speed + customization + don't have ML talent: **Partner**
- If you don't know yet: Start with Buy or Partner, migrate to Build once you have data and confidence

---

## Tradeoffs and What I'd Do Differently

**What I'd add next**: A framework for AI feature sunset decisions — when to pull an AI feature that isn't performing. Most teams have no process for this, and it's a real gap in the industrial software context where customers have long memories of bad AI experiences.

**What I deliberately left out**: A framework for model selection (which LLM/ML approach to use). That decision is too context-dependent to generalize, and the landscape changes too fast. The frameworks here are designed to be stable regardless of what model you're using.

**Where I'm not sure I got it right**: The scoring weights in Framework 1 treat all dimensions equally. In practice, error tolerance probably deserves a higher weight for industrial software than the matrix implies — a score of 1 on error tolerance should arguably be a hard veto, not just a low score. I'm still working out whether to make that explicit.

**The hardest part to calibrate**: The sequencing model (Framework 2) assumes you have the organizational patience to run shadow mode until the accuracy gate is met. In practice, there's almost always pressure to skip to Phase 2 once the model "looks good." The gate thresholds are deliberately unspecified here because they depend on the use case — but that also makes them easy to fudge under pressure.

---

## Contributing

These frameworks are opinionated — they reflect experience in enterprise, industrial, and supply chain software. PRs with additions, counter-examples, or calibration data welcome.

---

*Maintained by [@ravishgm](https://github.com/ravishgm)*
