# Q&A bank — 90-minute panel

Two sections: the decisions they will attack, and the questions they will ask. Answers are short on purpose — say the headline, then one supporting sentence, then stop.

---

## A. Decision defence (the "why did you choose X" table)

| Decision | What I chose | Rejected alternative | Why | What would change my mind |
|---|---|---|---|---|
| Challenge | Demand forecasting | Customer valuation / pricing | I have owned it end to end; I can talk about failure modes, not theory | — |
| Grain | Category × country × week | SKU-first | One-day manual process implies aggregate; tractable; earns trust first | If purchasing actually consumes SKU-level and the manual process is top-N only |
| Horizon | 8–12 weeks | 4 weeks / 12 months | Commitment window set by lead time plus review | Longer supplier lead times |
| Model class | SARIMAX / GBM | Deep learning | A few hundred series; explainability matters | SKU phase, hundreds of thousands of series |
| Autonomy | Human in the loop | Full automation | Tacit context and accountability; adoption is the binding constraint | Once override accuracy and coverage are stable |
| Rollout | Shadow → champion | Big-bang | Trust is earned by comparison | If a low-risk subset can go live faster |
| Consumer | Purchasing / OTB | Marketing / finance | The brief's stock and supplier-constraint language only makes sense for purchasing | If the panel says finance owns the number |
| Build | Build on warehouse | Buy a forecasting suite | Context schema and evaluation are the differentiator | If a suite already encodes the promo/launch logic |

---

## B. Questions

### Choice and scope

**1. Why Challenge 2 and not 1 or 3?**
I've owned it. At Zooplus I ran outbound demand forecasting and the planning cycle. Challenge 1 is a diagnosis problem in a live model; Challenge 3 is causal pricing. Both are good, but I'd be theorising. Here I can speak from failure modes I've actually seen — and the brief's hardest requirement, trust from a team that already has a good process, is the part I've done.

**2. Why category × country and not SKU?**
The manual process takes one working day. Across hundreds of thousands of SKUs that is only possible at aggregate grain, so the decision the business runs on today is aggregate. v1 matches that decision and earns trust; SKU is phase 2, where intermittent demand and cold start make ML necessary. Starting at SKU would be a long path to first value and a credibility risk.

**3. Why 8–12 weeks?**
It is the commitment window: long enough to cover supplier lead time plus review, short enough that promo and launch events are known. Longer horizons are dominated by seasonality and belong in a separate annual plan. The horizon is a parameter, not a principle.

**4. Isn't 8–12 weeks too short for supplier lead times?**
The horizon must cover lead time plus the review period. If lead time is 8–12 weeks, the forecast window is at least that, and the architecture is unchanged. I would confirm the actual lead-time distribution in discovery before fixing the number.

**5. Why not just improve the spreadsheet?**
You should, for a quarter — but a spreadsheet cannot give reproducibility, backtesting, or scale. The point is to keep its judgement and remove its mechanics.

### Modelling and data

**6. Why not deep learning / a transformer?**
At a few hundred aggregate series with a few years of weekly data, deep models overfit and cannot be explained in a planning meeting. The gains come from censoring, promo coding, lifecycle features and honest backtests. ML is genuinely needed at SKU scale — that is exactly where I put it.

**7. Promotions are not randomly assigned. How do you handle that?**
Acknowledge it, control for it, and don't call correlation causation. v1 uses explicit controls — season, price, availability, prior sales — separates baseline from uplift, and backtests on held-out promo periods. For high-spend categories, phase 2 uses difference-in-differences or synthetic control where a credible counterfactual exists. I'd also say clearly that a full causal pricing model is Challenge 3; this design consumes its outputs.

**8. How do you forecast a console launch with no history?**
Analog forecasting: match on platform, price tier, brand, category and prior generation; use pre-orders, search and traffic as leading indicators; use supplier guidance; widen the interval. Treat launch weeks as a separate regime, not as noise to smooth away.

**9. Streamer events are announced late. Doesn't that break the model?**
No — the latency rule handles it. If it is known after the forecast is produced, it is an override, not a feature. I model average lift by creator tier for events known in time, and give planners a structured override with reason code, expected uplift and expiry. Overrides are then scored, so we learn rather than forget.

**10. Stockouts censor demand. What do you do?**
Flag constrained weeks, model latent demand with availability as an offset, or exclude constrained periods from training. At SKU level, reconstruct unconstrained demand. Skip this and the model learns "we sell nothing when we have nothing" — the most common silent failure in retail forecasting.

**11. How do you handle country differences?**
Country-specific holiday and school calendars, local price and promo calendars, and reconciliation to a European total. Share strength across countries where series are thin — partial pooling or a hierarchical model — rather than forcing one seasonality shape.

**12. How do you avoid overfitting the backtest?**
Rolling origin, decision-time feature availability, no future information, a held-out final period, and segment-level reporting. Keep the model simple enough that it cannot be tuned to death.

**13. How do you handle taxonomy changes and hierarchy churn?**
Versioned mappings, reconciliation tests, and a rule that a category redefinition is a data event with an owner — not a silent break in the series.

### Validation and adoption

**14. How do you prove it beats the spreadsheet?**
Forecast value added against the manual forecast, not just against a naive baseline. Capture the manual forecast prospectively, backtest at the decision horizon, and report by segment. Then business KPIs: stockouts, obsolescence, planner hours. Accuracy alone does not prove value.

**15. Why would planners trust it?**
Because it doesn't ask them to give up control. It reproduces their process, shows driver attribution, lets them override with a reason, and measures their overrides rather than hiding them. Shadow mode first, champion–challenger, and they co-design the input schema. Trust is earned by being auditable, not by being accurate.

**16. What if the planners are simply better?**
In some segments they will be — launches and relationships especially. That is useful: it tells you where the model needs features, or where an override is legitimate. The goal is not to win everywhere; it is to be better on the bulk and free planner time for the cases where judgement matters.

**17. What is the biggest risk?**
Adoption, not accuracy. A model that is 10% better and ignored is worth zero. The mitigation is the design: shadow mode, explainability, override, and a planner who owns the number.

**18. What would make you stop?**
If shadow mode cannot beat the manual forecast on FVA after a full cycle, the scope or features are wrong. Re-scope to where it works rather than force adoption.

### Production and scale

**19. How does this scale to 300k SKUs?**
Phase 2. Aggregate forecasts set the envelope; SKU forecasts allocate within it. Intermittent-demand methods for the tail, GBM for the head, analog models for new products, reconciled to the aggregate. Compute is not the constraint; data quality and taxonomy are.

**20. Build or buy?**
Build the forecast layer on the warehouse; buy commodity pieces if they fit — planning tool, orchestration. The differentiator is the context schema and the evaluation, not the algorithm.

**21. What is the ROI?**
Planner time (a day per cycle × cycles × countries), fewer stockouts on promoted and launch items, less obsolete stock. I would instrument the baseline before and after rather than promise a number in the interview.

**22. What about marketing and finance using the same number?**
One number, reconciled. If the planner's number and finance's number differ, you have two truths and the forecast loses authority. The hierarchy exists precisely so the same number serves both.

**23. How do you monitor it in production?**
Four layers: data quality, model error and drift, adoption (override rate and accuracy), and business KPIs. Retrain monthly or on a regime change.

**24. What would you do differently with more time?**
Move faster on the input schema and slower on the model. The schema is the hard part and the adoption lever; the model is comparatively commodity.

---

## C. If you only remember three things

1. **The incumbent is good, and the design says so.** Replace the mechanics, not the judgement.
2. **Three technical points carry the answer:** censoring, the latency rule, promo endogeneity.
3. **Success is adoption measured by FVA and business KPIs**, not a benchmark score.
