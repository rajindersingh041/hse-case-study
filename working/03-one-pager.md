# One-page summary

**Demand forecasting for a large European digital-entertainment retailer**
HSE Data Science Technical Case — Challenge 2 · Rajinder Singh

**The problem.** The planning team builds a demand forecast in spreadsheets over a full working day. It works because it carries context that is not in any system: a streamer event, a supplier conversation, a product launch. It does not scale, cannot be evaluated, and depends on a few people.

**What I propose.** Keep the judgement; replace the mechanics.

- **Scope:** category × country × week, eight to twelve weeks ahead — the purchasing commitment window. SKU-level forecasting is phase 2, reconciled to the same aggregate.
- **Inputs:** the planners' tacit context becomes structured data, not free text — promotions, launches, lifecycle, streamer events, availability. Every input is classified as signal, driver, constraint or context, which decides how it may enter the model.
- **Modelling:** seasonal baseline plus SARIMAX or gradient boosting with event features. Deliberately simple, because a few hundred series do not reward deep learning and because explainability is an adoption requirement.
- **Uncertainty:** quantiles, not a point estimate — commitments depend on the tail.
- **Control:** planners override with a reason code; overrides are scored, not suppressed.

**Why it is different from "just build a model".**

1. **Stockout censoring.** Sales are not demand. Without availability adjustment, the model learns to under-forecast exactly when supply is tight.
2. **The latency rule.** Information that arrives after the forecast is an override, not a feature — which is how late streamer intelligence is handled honestly.
3. **Promo endogeneity.** Discounts are not random. Naive uplift overstates the effect; v1 controls for it and phase 2 uses causal designs.

**How we will know it works.** Forecast value added against the current manual forecast, not just a naive baseline; WAPE and bias by segment; interval coverage; and business KPIs — stockouts, obsolescence, planner hours. Acceptance gates are agreed before launch.

**How it gets adopted.** Shadow mode, then champion–challenger in two categories and two countries, then scale. The model proposes; the planner disposes. If it cannot beat the manual process in shadow mode, we re-scope rather than force it.

**Why me.** At Zooplus I owned outbound demand forecasting — country × day, four weeks to two years — and turned it into a four-month plan with Sales, Marketing, Purchasing and Category. I have been on both sides of the adoption problem this case describes.
