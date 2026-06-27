# Auto Trade System — MT4 + XGBoost signal-filtering for Gold (XAUUSD), forward-tested across a demo-account fleet

## 1. Snapshot
- **One-liner:** A multi-strategy MetaTrader 4 trading stack for Gold whose expert advisors (EAs) stream each candidate signal over ZeroMQ to a Python XGBoost server that replies TRADE / NO based on a per-model probability threshold.
- **Category:** Algorithmic Trading / Fintech (retail FX/CFD, MT4 ecosystem).
- **Target buyer:** Retail/prop algo traders, signal-service operators, small trading desks wanting an ML "approval filter" layered on rule-based EAs. NOT institutional-grade.
- **Tech stack:** MQL4 EAs (MT4); Python 3.12 (XGBoost, scikit-learn, imbalanced-learn/SMOTE, pandas, numpy, joblib); ZeroMQ PUSH/PULL bridge; optional PostgreSQL logging; server_v3.0 adds Alpha Vantage news-sentiment + FRED/Alpha Vantage macro (DXY/VIX/yields) background fetchers. Models persisted as `.joblib`. CSV-driven training.
- **Maturity:** **MVP (forward-test / demo stage).** Justification: the full pipeline works end-to-end (EA → ZeroMQ → XGBoost → reply) and has run live on **6 demo accounts / ~20 EA instances** for months with weekly P&L tracking; one live-validation snapshot (9 Apr 2026) showed live win-rate matching backtest within <1%. But there is **no live funded account** (promoting the first one is still an open to-do), P&L swings violently week to week (+$1,036 one week, −$1,824 the next at $0.01–0.10 lots), and the ML filter is frequently bypassed, stumped, or leakage-driven. It is a serious research/forward-testing workflow, not a production live-trading product.
- **Live URL / demo:** None. No UI, no hosted service. Runs locally / on a Blink VPS against MT4 terminals. Artifacts are MT4 `.mq4/.ex4` files, Python scripts, `.joblib` models, and Markdown analysis reports.
- **Path:** /home/anjum/dev/auto_trade

**Relationship to the Algo-Trading line:** This is the **parent**. The sibling `/home/anjum/dev/anjum_autotrade` (covered separately) is a fork extending the same MT4+XGBoost approach to Gold **and GBPUSD**. Together they form the "Algo-Trading" product line; auto_trade is the larger, deeper-tracked codebase (extensive `resources/` history, multiple EA families, weekly account reviews).

## 2. What's built today (verified)
- **Working EA ↔ Python ZeroMQ bridge** (`src/python/server_v2.1.py`, `server_v2.2.py`). PULL socket receives the signal feature string from MT4, server scores it with XGBoost, PUSH socket replies `TRADE | NO | ECHO | ERROR`. One server process per magic number, each with its own port pair and `.joblib` model. Non-blocking recv loop with hit-rate logging.
- **Multiple EA families** (`src/mt4/`): VSA+FVG (V6→V8.1.0), Price-Action+S/R (P_A V7.3.2–7.3.5), Bollinger breakout/limit (BB_P700/P701/P703/P705/P706), LWMA-cross G4908, G5908 ATR-range, plus a documented base template (`ATS_P_A_BASE_V7.3.4.mq4`, 1082 lines) and a versioned portfolio with compiled `.ex4` files and `.set` parameter files.
- **Genuine XGBoost training pipeline with real ML hygiene** (`train_xgboost_v2.2.py`, 591 lines): expanding-window **walk-forward CV with purge rows** between train/test; **SMOTE applied to the training fold only** (and skipped below 200 rows); optimal threshold chosen from the precision-recall curve; a **Walk-Forward Efficiency (WFE)** metric and explicit overfit warnings; a stated **deploy gate of CV AUC ≥ 0.55**; feature engineering that must mirror the server exactly (documented in CLAUDE.md as a silent-error hazard). This is materially better discipline than typical retail "backtest-and-pray."
- **15 trained model artifacts** (`src/models/*.joblib`) and ~45 training CSVs (`src/logs/`).
- **server_v3.0.py (built, not in live results):** background threads for macro data (DXY via EUR/USD inverse, VIX, 10Y yield), **gold news sentiment via Alpha Vantage** (not an LLM — a keyword/relevance-weighted news API), PostgreSQL fire-and-forget logging, and a rule-based macro/news pre-filter. Degrades gracefully to v2.1 behavior if keys/DB absent.
- **Disciplined, unusually honest tracking** (`tasks/tracking.md`, `tasks/xgboost_tracking.md`): weekly per-account P&L reviews, root-cause writeups (e.g., the 12.5× ping-latency finding explaining a directional asymmetry), and repeated self-correction ("last week's PF 109 was favorable conditions, not strategy edge").
- **Live-vs-backtest validation evidence** (`src/logs/LIVE_RESULTS.md`): PA V7.3.4 live WR 57.9% vs 58.2% backtest; BB P703 live 75.0% vs 74.2% — small samples but honestly logged.

## 3. Planned but missing
- **First live funded account** — never deployed; "promote demo876 70022704 to live funded" remains an open action item.
- **XGBoost V3 dynamic-scoring redesign — PARKED** awaiting three user decisions. The parked note itself diagnoses the core problem: "CV AUC 0.539 (near random); Score=78 constant; 44 of 52 features have zero importance; model approves 98.9% of trades with no filtering power."
- **server_v3.0 external-intelligence stack** (macro/news/Postgres) is coded but absent from live results — effectively unproven/experimental.
- **EA-side wire-format coordination** for newer features (BB_P700 7-column schema, echoing prior XGProbability back over ZMQ) is "pending" — several feedback features are logged to CSV but never sent live.
- **No autonomous deployment/retraining** — every EA build, compile, parameter set, and server launch is manual and handed to an external coder (the codebase carefully scrubs the word "developer" and all AI/system-path references from output packages).

## 4. The AI gap
**XGBoost is real ML, and the engineering hygiene here is above retail average** — walk-forward CV, train-only SMOTE, purging, PR-curve thresholds, a WFE overfit metric, and a documented AUC≥0.55 deploy gate. The team also *catches its own failures honestly*, which is the strongest signal in the repo.

**But on this problem the ML largely does not work, and the tracking says so:**
- **Leakage / non-generalizing features:** V7.4 models 88625886 / 88645886 had **`Spread` at 87–90% of feature importance** — flagged DO-NOT-DEPLOY because spread differs between backtest and live brokers. CV AUC of 0.87 was called "misleading."
- **Stumps:** 5 of 7 retrains in one bundle collapsed to ≤2 trees after SMOTE early-stopping; multiple "single-tree stump" rejects.
- **Near-random edge:** repeated CV AUC of 0.52–0.54; one V8 model scored 0.534 with two of three folds *below* random and F1 *worse* than the majority-class baseline.
- **No live edge demonstrated:** models that "approve 98.9% of trades" add no filtering; live weeks where the XGBoost call count was **0** (performance was pure rule logic); and an XGBoost server **timeout that cost −$691** in one week because the filter silently never ran.
- **Constant-output bug:** a deployed V7.4 model emitted 0.517 for every signal in production until fixed.

**Verdict on modeling:** the *process* is robust; the *result* is mostly an overfit/under-powered backtest filter that has not shown a durable live edge on ~100–880-row datasets. There is **no LLM, no agentic, no autonomous component** — the only "external intelligence" (news sentiment) is a third-party API, not generative AI.

## 5. Missing pieces to make it sellable
- **Regulatory / licensing exposure is severe.** Selling or signaling on a trading system can trigger investment-adviser, CTA/CPO (US), or equivalent registration, plus MiFID/ESMA leverage and marketing rules in the EU/UK. Any sale needs jurisdiction scoping, "past performance ≠ future results" disclaimers, and likely a "tool, not advice" framing with no managed money.
- **No audited track record.** All results are demo accounts at micro lots with wild week-to-week variance and acknowledged regime-luck. A sellable claim needs a multi-month *funded, third-party-verified* (e.g., Myfxbook/broker statement) record — explicitly not yet started.
- **Risk controls are EA-local and fragile.** Daily-loss limit, emergency-kill, cooldowns exist per-EA, but incidents like a 10× lot misconfiguration (−$349) and grid-ladder stacking (−$691 family) show portfolio-level risk management is missing.
- **Broker/latency dependence:** identical configs produced opposite results across 20 ms VPS vs 250 ms local ping — a productized offering must pin execution environment.
- **No packaging, onboarding, UI, billing, or support model.** Today it ships as raw MT4 files + Python scripts to a single operator.

## 6. Native-AI + Autonomous-FTE upgrade
Reframe from "EA + ML filter" to an **AI Quant-Analyst / Trading-Desk FTE** that runs the loop a human is currently doing by hand every week:
- **Regime detection** (range vs trend) as a first-class gating layer — the tracking already proves regime, not edge, drives most P&L; an agent that classifies regime and sizes/suspends strategies accordingly is the single highest-value addition.
- **LLM news/macro analyst:** replace the keyword news API with an LLM that reads gold-relevant macro releases, FOMC/CPI calendars, and headlines, producing a structured risk posture and pre-trade veto with a written rationale.
- **Autonomous risk manager 24/7:** portfolio-level exposure caps, lot-size sanity guards (would have blocked the 10× incident), kill-switch on server timeouts (would have blocked the −$691 loss), and drawdown circuit-breakers.
- **Self-auditing retrain agent:** automatically retrains, runs the existing walk-forward/leakage checks, *rejects* Spread-dominated or stump models per the existing AUC≥0.55 gate, and writes the weekly review the operator writes manually today.
- **FTE positioning:** "Replaces ~8–12 hours/week of a discretionary quant's monitoring, weekly performance review, model-vetting, and risk-oversight work" — framed strictly as a **research/ops copilot with hard guardrails and disclaimers**, never as an autonomous money manager.

## 7. Showcase angle (for the portal)
- **Headline:** "An AI co-pilot for an algo-trading desk — it filters every Gold signal through a walk-forward-validated model and shows its work."
- **Stat-benefit bullets (claims must stay carefully hedged):**
  - **6 demo accounts, ~20 EA instances, months of forward-tested weekly P&L** — a real operating track record (demo, micro-lot; not a return guarantee).
  - **Live win-rate matched backtest within <1%** on a validation snapshot (PA 57.9% vs 58.2%; BB P703 75.0% vs 74.2%) — small-sample, but honest.
  - **Disciplined ML governance built in:** walk-forward CV, leakage detection, and an enforced AUC≥0.55 deploy gate that *rejects* its own bad models — sellable as trustworthy process, not magic returns.
- **Demo hook:** Live "signal inspector" — feed a Gold candle's features into the ZeroMQ server and watch it return TRADE/NO with the probability, threshold, and top contributing features; toggle a deliberately leakage-prone model to show the system flagging and rejecting it.

## 8. Verdict
- **AI-native score:** 2/5 — real XGBoost with strong ML hygiene, but the ML adds little demonstrated live edge (near-random AUC, leakage, stumps, frequently bypassed), and there is no LLM/agentic/autonomous layer.
- **FTE-fit score:** 2/5 — today it is heavily human-in-the-loop (manual weekly reviews, external coder, manual deploys), but the domain maps cleanly onto a compelling "autonomous quant-analyst/risk-manager FTE" vision.
- **Maturity:** MVP (forward-test/demo); research-grade, not live-funded or production.
- **Recommendation:** **Incubate** — keep as the anchor of the Algo-Trading line and invest in the regime-detection + autonomous-risk + LLM-analyst upgrade; do NOT showcase performance claims until a funded, third-party-verified track record and regulatory framing exist.

SLUG=auto-trade | NAME=Auto Trade System | CATEGORY=Algorithmic Trading / Fintech | MATURITY=MVP (forward-test/demo) | AI=2/5 | FTE=2/5 | VERDICT=Incubate | HOOK=AI co-pilot that filters every Gold signal, shows its work
