# Input inventory — v1 (category × country × week, 4w–4m)

**Status:** candidate list to validate and correct. Nothing here is final until checked against what the planning team actually receives.

## The frame: every input plays one of four roles

| Role | What it is | How it is used | Trap |
|---|---|---|---|
| **Signal** | Observed customer behaviour | Features / leading indicators | Lagging signals look predictive but arrive too late |
| **Driver** | Planned event that changes demand | Regressors / uplift features | Treating a driver as exogenous when it was chosen *because* of expected demand |
| **Constraint** | What limited sales (stock, allocation, MOQ) | Uncensor history; feasibility of plan | Using it as a demand driver inflates/deflates the forecast |
| **Context** | Calendar, market, competition | Seasonality, adjustments | Overfitting local holidays |

Two rules worth stating explicitly in the doc:

1. **Constraints are not demand drivers.** A stockout explains why observed sales were low, not why demand was low. Use it to reconstruct latent demand and to translate demand into a feasible plan.
2. **Latency rule.** If an input is known later than the forecast is produced, it cannot be a feature — it can only be a planner override. This is the honest answer to "what about streamer events announced late?"

## Already available (assumed — validate)

### Sales
| Input | Grain / frequency | Likely home | Role |
|---|---|---|---|
| Order and unit history | category × country × week | DW | Signal |
| Channel mix actuals (web, marketplace, store, B2B) | channel × country × week | DW | Signal |
| B2B / key-account orders | account × week | ERP / CRM | Signal (lumpy — often separate model) |
| Regional demand feedback | free text / calls | email, meetings | Context |

### Marketing
| Input | Grain / frequency | Likely home | Role |
|---|---|---|---|
| Promotion calendar (mechanic, depth, duration, channel) | promo × country × week | promo planning sheet/tool | Driver |
| Campaign calendar (email, paid, social, affiliate) | campaign × country × week | marketing calendar | Driver |
| Streamer / influencer schedule (who, when, platform, product, exclusivity) | event | email, agency decks | Driver (late, siloed) |
| Sponsored content schedule | event | marketing | Driver |
| Pre-orders, wishlists | product × country × day | e-commerce platform | Signal (leading) |

### Purchasing
| Input | Grain / frequency | Likely home | Role |
|---|---|---|---|
| Purchase orders and inbound ETAs | PO × SKU | ERP | Constraint |
| Lead times, MOQ, supplier allocation | supplier × SKU | ERP / contracts | Constraint |
| Supplier-funded promotions / rebates | promo × SKU | email / contracts | Driver |
| Cost and price changes | SKU × date | ERP | Driver |

### Category management
| Input | Grain / frequency | Likely home | Role |
|---|---|---|---|
| Assortment: new listings, delistings | SKU × date | PIM | Context / driver |
| Lifecycle status and successor mapping | SKU × date | PIM + planner knowledge | Driver |
| Launch calendar (publishers, manufacturers) | product × date | external + category | Driver |
| Pricing and discount plan | category × week | pricing tool | Driver |
| Competitive intelligence | category × country | manual | Context |

### Central data / web
| Input | Grain / frequency | Likely home | Role |
|---|---|---|---|
| Stockout / availability flags | SKU/category × country × day | WMS | Constraint (critical) |
| Traffic, sessions, conversion, PDP views, search terms | category × country × day | web analytics | Signal (leading) |
| Price history | SKU × date | DW | Driver |
| Returns | SKU × country × week | DW | Signal / context |
| Public holidays and school calendars | country × date | reference | Context |
| Competitor prices / promos | category × country × day | scraping (if any) | Context |

## What to request (candidates, not a wish list)

- Streamer / influencer event registry: date, platform, creator, product mapping, expected reach, lead time.
- Launch calendar as a feed, not a deck.
- Web analytics aggregated to category × country × day (traffic is the best leading indicator you can get cheaply).
- Stockout and availability flags at the forecast grain — without these, history is unusable.
- Promo mechanics as structured fields (type, depth, duration, channel, funding), not free text.
- Competitor price and promo feed.
- Wishlist / pre-order / search volume as leading demand signals.
- Planner override log with reason codes — this becomes both an input and an evaluation set.

## Prioritisation test

For each candidate: expected forecast lift vs cost and latency to obtain. Anything with latency longer than the forecast horizon is an override, not a feature. Anything that only explains the past is context, not signal.

## Discovery questions per team

**Sales** — What do you know before the number moves? Which accounts or regions do you adjust by hand today, and why?
**Marketing** — Which events are on the calendar more than four weeks out? Which ones land late? What does a streamer event actually do to orders, in your words?
**Purchasing** — Where do stockouts and allocation change what we *could* have sold? Which supplier constraints are known at forecast time?
**Category** — Which products are entering, leaving, or being succeeded? Which launches have a reliable date?

## The strategic point

The planning team's advantage is early, siloed context. The design should not try to out-model that — it should **instrument** it. This inventory is the trust story: you are not replacing their information advantage, you are giving it a schema, an audit trail, and a backtest.
