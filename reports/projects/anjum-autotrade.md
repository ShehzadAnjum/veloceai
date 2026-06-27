# Anjum AutoTrade — A governed, ML-filtered MT4 trading-EA lab with a tier/rulebook risk regime

## 1. Snapshot

- **One-liner:** A fleet of MetaTrader-4 Expert Advisors (VSA/FVG/Bollinger/Price-Action/LWMA strategies) for Gold & GBPUSD, gated by XGBoost ML quality-filtering and governed by a strict written "rulebook" — tier promotion, loss caps, lot/EmgKill ratios, and weekly War Room reviews.
- **Category:** Algorithmic Trading / Fintech (retail systematic FX/metals).
- **Target buyer:** Self-directed retail/prop traders and small trading desks who want disciplined, rules-enforced semi-automated execution; secondarily, an internal showcase of "AI risk-officer governance."
- **Tech stack:** MQL4 (MT4 EAs, `.mq4`/`.ex4` + `.set` presets); Python 3.12 (XGBoost, scikit-learn, imbalanced-learn/SMOTE, joblib); ZeroMQ (EA↔Python signal bus); FastAPI dashboard (port 8501); PostgreSQL (trade store); WhatsApp/Twilio notifications; Windows Task Scheduler automation; Tickstory 99.9% tick backtesting. Heavy Markdown-as-governance layer.
- **Maturity:** **Prototype (research-grade), running on a DEMO account.** Justification: real, working infrastructure (ML training pipeline, ZMQ server, dashboard, DB sync, weekly auto-brief) but **zero EAs at LIVE tier**, **all activity on XM Demo account 79232294**, Season 1 net **−$455 ("tuition")**, Season 2 baseline reset to $0, and **no strategy has yet demonstrated a durable live/real-tick edge** — several have been formally proven dead at 99.9% modelling. Infra is MVP-quality; the *product* (a profitable autonomous trader) is unproven.
- **Live URL / demo:** None public. Local FastAPI dashboard at `http://localhost:8501` (webapp/run.py); operates against MT4 on a Windows VPS. Repo: `github.com/ShehzadAnjum/anjum_autotrade` (private).
- **Path:** /home/anjum/dev/anjum_autotrade

## 2. What's built today (verified)

- **EA fleet (MQL4):** ~30 `.mq4`/`.ex4` files in `src/mt4/` across families — ATS_VSA_FVG (V7.1→V8.1.0), ATS_P_A (PA, V7.3.2→V7.3.7), BB_Brkout/BrkoutLmt (P700/P703), BB_RegimePyramid_P706, ATS_LWMA_Cross_G4908, G5908_ATR_Range. Plus ~40 tuned `.set` presets in `resources/expert_settings/`.
- **XGBoost ML pipeline (real, disciplined):** `src/python/train_xgboost_v2.1.py` does expanding-window **walk-forward CV with purge rows** (`walk_forward_cv()`), SMOTE for class imbalance, PR-curve threshold optimization, Walk-Forward Efficiency metric, and computes feature importance. Engineers ~52 features from ~25 raw EA inputs. 15 trained `model_*.joblib` artifacts in `src/models/` (each magic isolated: own model, own ports, own server process).
- **Live ML inference path (one EA only):** `src/python/server_v2.1.py` — ZeroMQ PUSH/PULL server, non-blocking loop, fail-open on the EA side. In production only for **G4908 LWMA (magic 49334908)** on ports 32788/32789 with the pf140 model (CV AUC 0.799, threshold 0.492). A macro server (`server_v3.0.py`) pushes DXY/yields → PostgreSQL.
- **Governance system (the differentiator):** `references/rulebook.md` is a genuine, maintained operating constitution — 3-tier system (LIVE/TRIAL/LAB/BENCH/RETIRED), explicit promotion/demotion/retire thresholds, per-tier daily loss caps (LAB −$15, TRIAL −$20, **portfolio hard stop −$50/day**), lot↔EmgKill scaling rules, a "No-Touch" data-integrity rule, a full Config Registry audit trail, a War Room Log, and an Amendments Log. `references/lessons.md` holds 44 numbered hard-won lessons.
- **Automation/ops:** `scripts/warroom_brief.py` auto-generates a weekly decision brief (Windows Task `WarRoomBrief`, Sun 07:00) that maps live DB state onto rulebook thresholds; `trade_sync_service.py` syncs MT4 logs/journals → PostgreSQL every 30s; FastAPI dashboard with auto EA-discovery, analytics endpoints, Swagger; WhatsApp brief/query bridge; ATR watchdog; 7 Claude Code skills (ea-backtest-analyze, edge-analysis, ea-version-deploy, mt4-monitor, sync-parent, etc.).
- **Validation rigor:** 99.9% Dukascopy-tick Tickstory backtests used as the bar of truth; documented multi-run A/B/C chains (e.g., BB P703 v7/v8/v9 + bugfix re-verification) that correctly diagnosed and **retired** dead strategies rather than shipping them.

## 3. Planned but missing

- **Server persistence / 24-7 reliability:** LWMA XGBoost server runs in a foreground PowerShell window — dies on window close or VPS reboot; NSSM/Windows-Service wrapper is a noted TODO (live-setup.md). EA fail-opens silently, so outages are invisible without log inspection.
- **End-to-end ML coverage:** only 1 of ~6 active EAs actually uses the ZMQ→XGBoost link; the pf328/88801888 ZMQ path is benched ("not proven end-to-end").
- **A shadow/A-B model framework** (v2.2 trainer/server exists in `resources/update_frm_tm/` but deferred).
- **Reliability of automation:** WhatsApp 2-way channel and scheduled tasks (`WarRoomBrief`, `WhatsAppQuery`) have repeatedly gone silently dead; a 14-day VPS outage (May 11–25) halted everything.
- **A live (real-money) track record, multi-year backtests for low-frequency strategies (P706 needs 2-3yr), and any productization** (multi-tenant, licensing, packaging) — none exist.

## 4. The AI gap

**XGBoost IS real ML, and the methodology is unusually honest for retail algotrading** — but its proven contribution to edge is thin, and there is **no LLM/agentic component** in what ships (the "AI" doing strategy reasoning is the human + Claude Code workflow, not the product).

- **Leakage control — good:** the rulebook enforces a hard deploy gate — `CV AUC ≥ 0.55` AND `feature_importance['Spread'] < 0.5` AND `n_estimators_ > 5` — explicitly to catch the classic Spread look-ahead leak and degenerate stump models (rejected examples like a "STUMP" 2-tree model are archived). Walk-forward CV uses **purge rows** between train/test, which is the correct guard against temporal leakage. This is materially better than the TimeSeriesSplit it replaced (Lesson #16).
- **Edge — weak and honest about it:** ML adds genuine value on exactly one strategy (LWMA, AUC ~0.80/0.69), and **near-zero on VSA (AUC 0.52 = coin flip)** per the project's own tracking. So the ML is a quality *filter* on one trend EA, not a portfolio-wide alpha engine.
- **Risk governance — the strongest part:** the rulebook functions as a manual risk-officer SOP. Breakeven-WR math is applied per EA (e.g., GBPUSD config needs 80% WR to break even and gets 67% → correctly paused; PA needs 67%, gets 50% → demoted). Tight-SL/wide-TP strategies are blanket-rejected without 99.9% proof. This discipline is real and repeatedly prevented bad live deployments — but it is **enforced by a human reading Markdown**, not by software in the loop.
- **Bottom line:** the system's intelligence is mostly *process* (governance + tick-validation rigor), partly *classical ML* (one EA), and **not at all generative/agentic**. No proven live edge has emerged; most validated strategies are sub-breakeven or dead.

## 5. Missing pieces to make it sellable

- **Track record:** No live/real-money results. Demo-only, Season 1 −$455, no EA promoted to LIVE. Any sellable trading product needs an audited, real-capital, multi-month equity curve with verified PF > 1 across regimes — this does not exist yet.
- **Licensing/compliance:** Selling trading signals/EAs or "autonomous FTEs that trade" triggers heavy regulation (investment-advice / CTA / MiFID-type rules vary by jurisdiction). Needs disclaimers, "past performance" language, and likely a "tool, not advice" positioning. Currently zero compliance layer.
- **Risk controls (productized):** loss caps/EmgKill rules exist as *policy in Markdown*, not as a tamper-proof software kill-switch enforced independently of the operator (note: a human already overrode a DailyLossPercent cap 10→70 to stop an EA halting — exactly the failure mode a sellable risk layer must prevent).
- **Packaging:** single-operator, manual MT4 chart attachment, hand-edited `.set` files, foreground servers. No installer, no multi-account/multi-tenant, no broker abstraction, no onboarding. Heavily coupled to one XM demo VPS.
- **Robustness:** several strategies are mathematically losing or unproven; the few promising ones are low-frequency (1-2 setups/month), so statistical significance is years away.

## 6. Native-AI + Autonomous-FTE upgrade

The single most compelling AI-native pivot here is to **turn the rulebook into the agent.** Today a human reads `warroom_brief.md` every Sunday and hand-applies tier/risk decisions. Replace that loop:

- **AI Risk-Officer / Trading-Desk agent (24/7):** an LLM-driven agent that (a) ingests the live PostgreSQL trade feed and EA logs continuously, (b) **enforces `rulebook.md` deterministically** — promotion/demotion/retire triggers, per-tier and portfolio loss caps, EmgKill≤$5 checks, the No-Touch rule, lot/EmgKill ratio validation before any deploy, (c) auto-generates the War Room brief *and executes the low-risk, unambiguous actions* (freeze on −$50 portfolio stop, pause an EA past its loss cap) while escalating judgment calls to a human, and (d) maintains the Config Registry / Amendments audit trail automatically. The hard guardrails (caps, kill-switches) run as deterministic code the LLM cannot override — the LLM does narration, triage, and the gray-zone reasoning.
- **LLM market-context layer:** summarize macro/news (DXY, yields, gold drivers) into a regime tag the risk-officer uses to widen/tighten which EAs are allowed to trade — replacing the manual `macro-plan.md` "Gold Holy Trinity" work.
- **"Replaces ~N hours":** realistically **~5-8 hrs/week of a trading-ops / risk-analyst role** (Sunday War Room, daily dashboard glances, brief generation, config-change auditing, statement reconciliation) plus continuous after-hours risk monitoring a human can't sustain.
- **Mandatory guardrails + disclaimers:** demo-first, hard-coded loss limits independent of the LLM, no autonomous real-capital scaling without human sign-off, and prominent "research tool, not financial advice; past/backtested performance ≠ future results" framing. Given the honest finding that edge is unproven, the agent should be sold as a **discipline/risk-governance FTE**, not a profit machine.

## 7. Showcase angle (for the portal)

- **Headline:** "The Risk Officer That Never Sleeps — an AI agent that enforces a trading rulebook 24/7, on real ticks, without ego."
- **Stat-benefit bullets (performance-claim-careful):**
  - **Governs a multi-strategy EA fleet by an auditable rulebook** — 5 tiers, hard loss caps (−$50/day portfolio kill-switch), and a full config/amendments audit trail, all enforced automatically.
  - **Validates on 99.9% real-tick data before risking a cent** — the same rigor that has *correctly retired multiple losing strategies* instead of shipping them (a discipline most retail bots skip).
  - **Frees ~5-8 analyst hours/week** by auto-generating the weekly decision brief and executing the unambiguous risk actions; humans only handle the judgment calls.
- **Demo hook:** Live dashboard tour (FastAPI) showing the EA tier board + a simulated portfolio breaching the −$50 cap → the agent auto-freezes every EA, writes the War Room log entry, and posts the rationale to WhatsApp — narrated by the LLM in plain English. Emphasize **demo account; research tool; not investment advice.**

## 8. Verdict

- **AI-native score:** **2/5** — real, methodologically honest classical ML (walk-forward + leakage gates) but it governs only one EA, adds little proven edge, and there is **no generative/agentic AI** in the shipped product. The "AI" is mostly disciplined process.
- **FTE-fit score:** **3/5** — the rulebook→autonomous-risk-officer mapping is genuinely strong and differentiated as an FTE concept, but the autonomy is not built (human-in-loop for every decision), reliability is shaky (silent task deaths, VPS outages), and the domain demands heavy guardrails.
- **Maturity:** Prototype (research-grade), demo-only, no LIVE-tier EA, no real-money track record, no proven edge.
- **Recommendation:** **Incubate.** Not a flagship — no edge, demo-only, and a compliance minefield as a trading product. But the **governance IP (the rulebook + tier/War-Room/audit discipline) and the "AI risk-officer that enforces a rulebook 24/7" narrative are genuinely novel and demo-worthy.** Incubate by building the deterministic-guardrail risk-officer agent as a showcase, explicitly positioned as a discipline/risk FTE — not a profit guarantee.

---

### Relationship to parent `auto_trade`

anjum_autotrade is the **more-evolved fork** of `/home/anjum/dev/auto_trade` (read-only per CLAUDE.md DO-NOT #1). What the fork *adds over the parent* is the **governance and validation layer**, not new alpha: the formal `rulebook.md` tier/promotion/loss-cap regime, the War Room cadence + auto-brief, the Config Registry/Amendments audit trail, the lot↔EmgKill safety ratios, the 99.9%-tick "prove-it-or-retire" mandate, the lessons archive (44 entries), the PostgreSQL+FastAPI dashboard, and a `sync-parent` skill to pull strategy updates from the parent. In showcase terms: the parent is the strategy R&D line; **anjum_autotrade is the risk-governance / operationalization line** — and that governance is precisely what makes the AI-risk-officer FTE story credible.
