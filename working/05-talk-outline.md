# 15-minute talk — slide plan

Total: 15 minutes. Six slides. One message per slide. Do not read the document.

---

## Slide 1 — Why this challenge (2 min)

**Message:** I've owned this problem, and the hard part is not the model.

- Zooplus: outbound demand forecasting, country × day, 4 weeks–2 years; turned it into a four-month plan with Sales, Marketing, Purchasing, Category.
- Adjacent to article-level auto-SARX and purchasing constraints.
- The case is the transition I've seen: a good manual process asked to scale without losing its context.

*Say:* "I chose this because I can talk about where it breaks, not just how to build it."

---

## Slide 2 — The decision and the scope (2 min)

**Message:** This is a commitment instrument, so scope follows the decision.

- Decision: how much to commit, 8–12 weeks ahead, by category and country.
- v1: category × country × week. Phase 2: SKU × week, reconciled.
- Why: one working day across 300k SKUs is only possible at aggregate grain.
- Four criteria: faster, scalable, evaluable, trustworthy — each with a measure.

*Say:* "If the grain is wrong, everything downstream is wrong, so I'd confirm it in week one."

---

## Slide 3 — What is actually hard (3 min)

**Message:** Three things decide whether this works.

1. **Sales are not demand** — stockouts censor history; without adjustment the model under-forecasts when supply is tight.
2. **Events are heterogeneous** — promos, launches, streamers, lifecycle. Promotions are not randomly assigned, so naive uplift overstates the effect.
3. **Context is tacit** — the planners' edge is early, siloed information. Instrument it; don't try to out-model it.

*Say:* "Most forecasting projects fail on these three, not on the choice of algorithm."

---

## Slide 4 — The design (4 min)

**Message:** Baseline + structured context + reconciliation + human control.

- Inputs classified as signal / driver / constraint / context.
- Baseline (seasonal naive, ETS) → SARIMAX or GBM with event features.
- Probabilistic output; hierarchical reconciliation to country and Europe.
- Every forecast has a driver breakdown; planners override with a reason code.
- Latency rule: late information is an override, not a feature.

*Say:* "Deliberately simple, because a few hundred series don't reward deep learning and because explainability is what gets it used."

---

## Slide 5 — How we'll know (2 min)

**Message:** FVA against the manual forecast, plus business KPIs.

- Rolling-origin backtest at the decision horizon; benchmarks include the human forecast.
- WAPE, bias, interval coverage, by segment.
- Business: stockouts, obsolescence, planner hours.
- Acceptance gates agreed before launch.

*Say:* "If we only report MAPE, we'll win the benchmark and lose the planning meeting."

---

## Slide 6 — Rollout and the ask (2 min)

**Message:** Shadow → champion → scale, with the planners as design partners.

- Weeks 1–2 baseline; 3–6 shadow; 7–10 champion–challenger; 11–13 scale; Q2 SKU.
- "The model proposes, the planner disposes."
- Stop condition: if it can't beat manual in shadow, re-scope.
- Ask: access to the planning team and two cycles of their forecast.

*Say:* "The deliverable at 90 days is a forecast the planners defend, not a model that wins a benchmark."

---

## Timing discipline

| Section | Minutes | Cumulative |
|---|---|---|
| Why | 2 | 2 |
| Scope | 2 | 4 |
| Hard parts | 3 | 7 |
| Design | 4 | 11 |
| Proof | 2 | 13 |
| Rollout + ask | 2 | 15 |

## If you get cut short

Deliver slides 2 and 4 only: scope and design. The rest is Q&A material.

## Transitions to memorise

- "So what?" → "The decision is a commitment, so the forecast has to be feasible, not just accurate."
- "Why not SKU?" → "Because the business doesn't run on SKU today; the one-day process proves it."
- "Why will they use it?" → "Because they keep control, and their overrides are measured rather than overruled."
