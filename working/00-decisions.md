# HSE case — decision log

Updated: 2026-11-26

## Locked

- **Challenge:** 2 — Demand forecasting.
- **Why this challenge:** owned outbound demand at Zooplus (country × day, 4 weeks–2 years) and drove the 4-month planning cycle with Sales, Marketing, Purchasing and Category. Adjacent to article-level auto-SARX and purchasing constraints. Justification = lived both the modelling and the adoption problem.
- **v1 scope:** category × country × week, 8–12 week commitment window.
- **Consumer (assumed):** purchasing / open-to-buy. Evidence: the brief cites stock availability and supplier constraints, which only matter if the forecast drives purchase commitments.
- **KPIs (assumed):** stockouts, excess/obsolescence, service level; WAPE and bias as the technical gate.
- **Phase 2:** SKU × week, horizon set by lead time / MOQ, reconciled to the v1 aggregate.
- **Spine:** don't replace the planners, replace the spreadsheet — statistical baseline + structured context capture + hierarchical reconciliation + human control + backtestability.
- **Deliverable:** 5–6 page pre-read + 1-page summary; ~15 min talk; 90 min panel.
- **Send:** Monday 09:00. Freeze Sunday night.

## Built (v0.1 — for your challenge)

- `02-pre-read-draft.md` — full pre-read, sections 1–8 plus assumptions appendix.
- `03-one-pager.md` — one-page summary for the Monday send.
- `04-qa-bank.md` — decision-defence table plus 24 likely panel questions.
- `05-talk-outline.md` — 15-minute, six-slide talk with timing.

## Still open

- Which failure modes in section 3 you actually witnessed (placeholder in the doc).
- Quantified ROI — deliberately left qualitative until baseline is measured.
- Whether the panel's view of the consumer is purchasing or finance.

## Assumptions to state in the doc

- The manual process runs at category/country level: one working day across hundreds of thousands of SKUs is implausible at SKU grain.
- The forecast feeds the 4-month purchasing/planning cycle; SKU-level is phase 2.
- Historical sales are censored by stockouts and therefore understate demand.
- Event intelligence (streamers, launches, supplier promos) arrives late and unstructured.
