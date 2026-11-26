# Demand forecasting for a large European digital-entertainment retailer

**HSE Data Science Technical Case — Challenge 2**
Prepared by Rajinder Singh · Pre-read for technical interview

---

## Executive summary

The planning team produces a demand forecast in spreadsheets over a full working day, combining sales history with context that lives in people's heads: promotions, launches, stock, supplier constraints, streamer events. It works. The problem is that it does not scale, cannot be evaluated, and depends on a handful of people.

I propose to keep the judgement and replace the mechanics: a statistical baseline at category × country × week over an 8–12 week commitment window, with the planners' tacit context captured as structured inputs, hierarchical reconciliation, probabilistic output, and a human override that is measured rather than suppressed.

The design is deliberately conservative. At this grain there are a few hundred series, so accuracy comes from clean features, correct treatment of stockout-censored history and honest backtesting — not model capacity. Machine learning is deferred to phase 2, where hundreds of thousands of SKU-level series and cold-start launches make it necessary.

Success is not a better MAPE. It is a forecast the planning team trusts enough to change their decisions: fewer stockouts, less obsolete stock, and planner time redirected from assembling numbers to judging them.

I chose this challenge because I have owned it. At Zooplus I was responsible for outbound demand forecasting — order volume by country and day across horizons from four weeks to two years — and for turning that forecast into a planning decision with Sales, Marketing, Purchasing and Category Management.

---

## 1. Why this challenge

I'm choosing Challenge 2 because I've owned this problem from both sides.

At Zooplus I was responsible for outbound demand forecasting and for turning the forecast into a four-month plan, aligning Sales, Marketing, Purchasing and Category Management around it. I also worked alongside the article-level forecasting that fed purchasing — per-article models with horizons set by lead time, and purchase orders constrained by MOQ and supplier allocation — though my ownership was at the aggregate layer.

That is why I'm confident about this challenge and specific about where it is hard. The modelling is the part everyone discusses. The part that decides whether it works is whether the people who own the forecast trust it enough to change their decisions. This case describes exactly that: a manual process that works because it carries context a model cannot see, being asked to scale. I've been in the room where that transition either happens or quietly doesn't.

---

## 2. Framing and scope

**The decision this serves.** The forecast exists to support one decision: how much to commit, eight to twelve weeks ahead, by category and country. Purchasing commits orders against it, and the plan is only useful if it is feasible against stock and supplier constraints. Everything in the design follows from that — it is a commitment instrument, not a prediction exercise.

**v1 scope: category × country × week, 8–12 weeks ahead.** The horizon is a parameter, not a principle: it is set by supplier lead time plus the review cycle. If lead times are longer, the horizon extends, and the architecture does not change.

**Phase 2: SKU × week**, horizon set by lead time and MOQ, reconciled to the v1 aggregate and feeding purchase orders directly.

**What "good" means, in the brief's own words:**

| Criterion | Measured as |
|---|---|
| Faster | One working day → under an hour of planner time per cycle; baseline generated overnight |
| Scalable | Adding a country or category is data onboarding, not a new spreadsheet |
| Evaluable | Every forecast stored with its inputs and scored out-of-sample — including against the human forecast |
| Trustworthy | Every number has a driver breakdown; planners can override with a reason code; overrides are measured, not suppressed |

**What I am deliberately choosing:**

| Choice | Alternative rejected | Why |
|---|---|---|
| Aggregate v1 | SKU-first | Matches the existing decision cadence; tractable; earns trust before scale |
| Simple models | Deep learning | A few hundred series; explainability is an adoption requirement |
| Human in the loop | Full automation | The planners hold context that is not in any system yet |
| Structured context | Free-text notes | Auditable, learnable, and survives staff turnover |
| Probabilistic output | Single number | Commitment decisions depend on the tail |
| Shadow mode | Big-bang replacement | Trust is earned by comparison, not by assertion |

**Assumptions I'm making** (stated rather than hidden):

1. The manual forecast is produced at category/country level: one working day across hundreds of thousands of SKUs is implausible at SKU grain.
2. The forecast feeds purchasing/open-to-buy; the brief's references to stock availability and supplier constraints only matter if it does.
3. Historical sales are censored by stockouts and therefore understate demand.
4. Late, siloed event intelligence (streamers, launches, supplier promos) exists and materially moves demand.

**Non-goals for v1:** replacing planner judgement; SKU-level purchasing; price optimisation (that is Challenge 3); automating the planning meeting.

---

## 3. The current process: what to keep, where it breaks

The existing process is not a strawman. It earns its place:

- It carries **tacit context** — a streamer event, a supplier conversation, a competitor move — that never reaches central systems.
- It has **accountability**: a named planner owns the number and defends it.
- It has a **cadence**: the cross-functional review is where Sales, Marketing, Purchasing and Category reconcile their views.

The design should replace the mechanics, not the judgement.

Where it breaks at scale:

- **Not reproducible.** The number lives in a file and in someone's head. You cannot re-run last quarter's forecast with today's assumptions.
- **Not evaluable.** No stored forecast means no out-of-sample score. "It performs well" is a judgement, not a measurement — and it cannot separate skill from a favourable quarter.
- **Linear cost.** Every new country, category or promotion adds planner hours; growth is bought with headcount.
- **Inconsistent.** Two planners produce two methods; the same situation gets two numbers.
- **Late context leaks away.** The streamer email arrives, changes one planner's number, and is never captured, so it cannot be learned from or audited.
- **Bias is invisible.** Optimism or sandbagging by team or country is never measured, so it compounds cycle after cycle.
- **Single point of failure.** Illness, holiday or attrition takes the forecast with it.

`[you: pick the two or three you actually saw at Zooplus — the rest are plausible but weaker if you can't speak to them]`

---

## 4. Data and features

### 4.1 What the forecast consumes

Every input plays one of four roles: **signal** (observed behaviour), **driver** (a planned event that changes demand), **constraint** (what limited sales), and **context** (calendar, market). The distinction is not academic — it decides how each input is allowed to enter the model.

| Family | Examples | Role | Known at forecast time? |
|---|---|---|---|
| Demand history | orders, units, returns-adjusted sales | Signal | Yes (lagged) |
| Calendar | public holidays, school terms, Black Friday, Christmas, paydays | Context | Yes |
| Promotion | mechanic, depth, duration, channel, supplier-funded | Driver | Yes (planned) |
| Price | own price index, competitor index | Driver | Yes |
| Lifecycle | days since launch, successor announced, generation | Driver | Yes |
| Launch | launch date, new-SKU count, expected magnitude | Driver | Yes (calendar) |
| Streamer / influencer | event flag, creator tier, platform, product mapping | Driver | Often late → override |
| Availability | stockout rate, days of cover | Constraint | Partially |
| Market | category trend, search interest | Context | Yes (lagged) |

### 4.2 Sales are not demand

If a category was out of stock, observed sales understate demand — and the model learns to under-forecast precisely when supply is tight. At v1 grain this is manageable: flag constrained weeks and either exclude them or model latent demand with availability as an offset. At SKU grain it becomes unavoidable, which is one reason SKU forecasting is phase 2.

### 4.3 Features that matter in this catalogue

- **Seasonality is shared but locally shifted.** Use country-specific holiday and school calendars, not a single European one.
- **Promotions are heterogeneous.** Encode mechanic, depth, duration and funding. Twenty percent off a console is a different event from twenty percent off a cable.
- **Lifecycle is the differentiator.** Days since launch, successor announced or available, generation. This is what separates this retailer from a grocery catalogue, and it is what the manual process currently carries in people's heads.
- **Launch effects need analogs.** Estimate magnitude from comparable past launches rather than from the product's own (non-existent) history.
- **Streamer effects are sparse and heavy-tailed.** Estimate average lift by creator tier, shrink toward zero, and widen intervals. Do not fit a separate parameter per event.
- **Availability is both a constraint and a data-quality warning.** A stockout is evidence that history is unreliable, not evidence that demand fell.

### 4.4 Leakage and the latency rule

Every feature must be known at forecast time. If it isn't, it becomes an override. That single discipline keeps backtests honest and answers the "but streamer events are announced late" objection without weakening the model.

---

## 5. Modelling

### 5.1 Why v1 should be deliberately simple

At category × country × week there are only a few hundred series, each with a few years of weekly history. That is not the regime where deep learning wins. Accuracy comes from clean features, correct censoring and honest evaluation, not model capacity. Simplicity is also an adoption feature: a model whose drivers can be explained in a planning meeting is a model that gets used. Machine learning earns its place in phase 2, where there are hundreds of thousands of series and genuine cold-start products.

### 5.2 Baseline first

Seasonal naive and ETS. The baseline is not a strawman: it is the benchmark every later model must beat, and it is what makes forecast value added measurable rather than rhetorical.

### 5.3 Main model

A regularised regression with ARIMA errors (SARIMAX) over calendar, promotion, price, lifecycle and availability features; or gradient boosting on the same features where interactions matter. Pick per series family, not one global winner, and report both. At this grain the difference between the two is usually smaller than the difference made by getting promotions and censoring right.

### 5.4 Events

**Promotions.** Separate baseline from uplift, and estimate uplift with controls for season, price and availability. State the endogeneity plainly: promotions are not assigned at random — a product may be discounted because it was expected to sell, because it was overstocked, or because a supplier funded it. Naive comparisons will overstate the effect. At v1, control and backtest; at phase 2, use causal designs (difference-in-differences, synthetic control) for the categories where discount spend is largest.

**Launches.** Use analogs and the launch calendar; treat the first weeks as high-uncertainty and widen the intervals rather than pretending precision.

**Streamers.** Sparse events with large variance. Estimate lift by creator tier from past events, shrink the estimate, and route late announcements to a planner override.

### 5.5 Hierarchy and reconciliation

Reconcile category × country to the country total and to the European category total. Bottom-up where local series are stable, top-down where they are thin, minimum-trace (MinT) where the error covariance is estimable. The number the planner uses and the number finance sees must be the same number.

### 5.6 Uncertainty, not just a point

Publish quantiles. Commitment and safety-stock decisions depend on the tail, not the mean. Backtest interval coverage as well as accuracy.

### 5.7 Phase 2: SKU

Intermittent demand methods (Croston, TSB) for the long tail; gradient boosting for the head; attribute and analog models for new products; reconciliation back to the v1 aggregate. This is where machine learning is genuinely necessary, and where the purchasing constraints (lead time, MOQ, allocation) enter as feasibility rules rather than as demand features.

### 5.8 What we deliberately do not do in v1

No deep learning. No automatic price optimisation. No full SKU coverage. No attempt to model every streamer event as its own feature. Each of these is a defensible deferral, not an omission.

---

## 6. Validation and metrics

### 6.1 The evaluation must mirror the decision

Backtest with a **rolling origin**: at each forecast date, train only on data available then, and predict the full 8–12 week horizon. No random splits. Evaluate at the horizon the business uses, not one step ahead — a model that is accurate next week and useless in week ten has not solved the problem.

### 6.2 Benchmarks

Three, not one: seasonal naive, last year same week, and **the current manual forecast**. The manual forecast must be captured prospectively, otherwise there is no way to claim improvement.

### 6.3 Metrics

| Layer | Metric | Why |
|---|---|---|
| Accuracy | WAPE | Robust to low volume and interpretable in units |
| Bias | Signed error | Catches systematic over- or under-forecasting |
| Skill | MASE, FVA vs manual | Proves value added rather than activity |
| Uncertainty | Pinball loss, interval coverage | Commitments depend on the tail |
| Segments | WAPE by country, category, promo and launch weeks | Averages hide the failures that matter |
| Business | Stockouts, excess and obsolescence, service level, weeks of cover | The actual objective |
| Adoption | Override rate and override accuracy, planner hours | Trust is measurable |

### 6.4 Acceptance gates (illustrative)

- Beats the manual forecast on FVA in at least 70% of series.
- No country worse than the manual forecast by more than 5%.
- 80% prediction intervals cover at least 75% of outcomes.
- No degradation in stockout rate during champion–challenger.

### 6.5 Human versus model

Overrides are logged and scored. If planners consistently beat the model in a segment, that is a feature gap, not a nuisance. If they consistently lose, that is evidence for adoption. Either way the argument becomes empirical rather than political.

### 6.6 Guard against leakage

Maintain a feature-availability log, timestamp every event, and forbid future promotion data in training. The backtest must simulate the real decision date, including the information that was genuinely unavailable.

---

## 7. Production, monitoring, risks and tradeoffs

### 7.1 Architecture

Sources → ingestion/ELT → feature store → training and backtesting → forecast service → planning tool with driver explanations → override capture → evaluation store.

Weekly batch is sufficient. There is no real-time requirement for an 8–12 week commitment, and pretending otherwise adds cost and failure modes.

### 7.2 Reproducibility

Every forecast is stored with its data snapshot, feature version, model version and parameters. Any historical forecast can be re-run and explained. This is the difference between a model and an accountable planning system.

### 7.3 Explainability

Per-forecast driver attribution: seasonality, promotion, price, launch, availability, residual. Planners see *why* the number moved, not just the number. This is what makes an override a judgement rather than a guess.

### 7.4 Monitoring

- **Data:** missing weeks, taxonomy changes, promo coding errors, availability feed lag.
- **Model:** feature drift, error by segment, bias trend, interval coverage.
- **Adoption:** override rate and accuracy, active planners, time-to-plan.
- **Business:** stockouts, excess, markdown and clearance.

Retrain monthly, or on a regime change — a console generation, a taxonomy change, a new country.

### 7.5 Risks

| Risk | Mitigation |
|---|---|
| Planners bypass the model | Co-design the inputs; shadow mode; explainability; measured overrides |
| Late event intelligence | Latency rule → structured override with reason code and expiry |
| Censored history | Availability flags and latent-demand adjustment |
| Promo endogeneity | Controls in v1; causal designs for high-spend categories in phase 2 |
| Taxonomy and hierarchy churn | Versioned mappings; reconciliation tests |
| Feedback loops (forecast drives stock, stock drives sales) | Holdout periods; randomised or phased promotions where possible |
| Regime change (new console generation) | Analog forecasting, wider intervals, mandatory manual review |
| Incentive misalignment | Keep forecast and target separate; measure bias by team |
| Key-person dependency | Documentation and audit trail from day one |
| Over-engineering | Deliberately simple v1 with explicit deferrals |

### 7.6 Explicit tradeoffs

- **Accuracy versus explainability** → choose explainable in v1; recover accuracy with better features, not opacity.
- **Automation versus control** → human in the loop; automate the mechanics, not the judgement.
- **Granularity versus reliability** → aggregate first; SKU where data supports it.
- **Speed versus rigour** → weekly batch, not real-time.
- **Coverage versus precision** → probabilistic output so the business can choose its service level.

---

## 8. Rollout and first 90 days

| Phase | Weeks | What happens | Exit gate |
|---|---|---|---|
| 0. Baseline | 1–2 | Audit data; reproduce last cycle's manual forecast in the system | Same number, explainable |
| 1. Shadow | 3–6 | Model runs alongside the manual process; no decision changes; planners review driver explanations | Planners can explain the model's number |
| 2. Champion–challenger | 7–10 | Two categories × two countries; planners accept or override; business KPIs tracked | Beats manual on FVA without worse stockouts |
| 3. Scale | 11–13 | Expand coverage; formalise override workflow; train planners | Adoption and accuracy gates met |
| 4. SKU phase | Q2 | Item-level forecasting for purchasing; reconcile to aggregate | Feasible plans at SKU grain |

**Change management.** The operating principle is "the model proposes, the planner disposes." Publish override accuracy, make the planner the hero of the cases they catch, and never surprise them in a meeting with a number they have not seen.

**Team.** Two data scientists, two data engineers, a product manager, and the planning lead as co-owner. The planning lead is not a stakeholder to be managed; they are the design partner.

**Stop conditions.** If shadow mode cannot beat the manual forecast after a full cycle, the scope or features are wrong. Re-scope to where it does work rather than force adoption.

**What success looks like at 90 days.** Not a model that wins a benchmark — a forecast the planning team defends in their own planning meeting.

---

## Appendix: assumptions to validate in discovery

1. The manual forecast grain is category × country, not SKU.
2. The consumer is purchasing/open-to-buy.
3. Supplier lead time is 8–12 weeks.
4. Stockout flags exist and are reliable enough to adjust history.
5. Streamer and launch intelligence is available before the forecast, at least for major events.
6. The planning cycle is weekly or fortnightly, not daily.
