# SME Product & Price Research Engine — Pakistan-aware procurement shortlister that turns a plain-language need into a ranked, sourced top-5

## 1. Snapshot
- **One-liner:** Takes a plain-language SME procurement requirement, scrapes Pakistani retailers (and review sites for SaaS), and returns a ranked top-5 with PKR pricing, source links, and score breakdowns — exported as Web UI / JSON / Excel / CSV / Markdown.
- **Category:** Procurement / Research (product-sourcing assistant).
- **Target buyer:** Pakistani SMEs (Karachi-centric) and their office/admin/procurement staff buying laptops, printers, networking gear, or SaaS; secondary: IT resellers and procurement consultants.
- **Tech stack:** Python 3, Playwright (headless Chromium) for hardware scraping, httpx + BeautifulSoup/lxml for software pages, `ddgs` (keyless DuckDuckGo) for discovery, Pydantic v2 models, FastAPI + Jinja2 web UI, openpyxl for Excel, YAML-driven source config, pytest. Optional Anthropic Claude layer (default OFF).
- **Maturity:** **MVP.** Justify: end-to-end pipeline actually runs (CLI + web UI), produces all four export formats (real artifacts present in `outputs/`), has 20 passing pure-logic tests, and `tasks/todo.md` documents a live demo that returned 5 real PriceOye laptops. But it is narrow: of 5 wired retailers only 2 (PriceOye, Telemart) returned data in the recorded run; the AI layer is unproven (see §4). Functional and demoable, not production-hardened.
- **Live URL / demo:** None deployed. Runs locally: `uvicorn webui.app:app --port 8000`. Static demo screenshots exist (`outputs/ui_home.png`, `outputs/ui_results.png`, `resources/sc1.png`).
- **Path:** /home/anjum/dev/price_research

## 2. What's built today (verified)
All verified by reading source, not the README claims.

- **NL requirement parser** (`requirement.py`) — pure regex/keyword logic: classifies hardware vs software, infers category (laptop/printer/crm/…), extracts budget (handles `150k`, `1.5 lac`, `Rs 150,000`), quantity (seats/units), specs (i5/16gb/512gb ssd), and keywords. Deterministic, keyless.
- **Keyless discovery** (`search.py`) — DuckDuckGo via `ddgs`, with a query builder and domain-restricted (`site:`) search. Returns `[]` gracefully on rate-limit.
- **Hardware scraping** (`scrape/base.py`, `scrape/generic.py`, `scrape/sites.py`) — Playwright session with realistic UA, `networkidle` wait, SHA-keyed HTML disk cache, and bot-wall detection (captcha/cloudflare markers). Extractor tries JSON-LD `Product`/`Offer` first, then a heuristic DOM scan for link+title+price blocks. Per-site failure is logged and skipped, never aborts the run.
- **Normalization** (`normalize.py`) — cleans heuristic name noise (cuts at price marker, strips "% OFF" and "4.00 (0)" ratings), parses prices to PKR, re-extracts specs from names, de-dupes.
- **Relevance gate** (`relevance.py`) — drops off-category items (the documented "foot massager vs laptop" problem) using category tokens + brand+CPU hints, with a fallback so it never returns empty.
- **Deterministic ranker** (`rank.py`) — weighted score out of 100: relevance 30 · price/value 25 · local availability 20 · trust 15 · SME-fit 10. Hard-coded per-domain trust table (priceoye 0.95 … daraz 0.7). Produces a per-component breakdown and a templated one-line rationale.
- **Software path** (`software.py`) — DDG search ("best …" + `site:g2/capterra/getapp`), dedupes by title, best-effort price-hint scrape ($/mo) from the top vendor pages. Locality weighting is neutralized for SaaS.
- **Exporters** (`report/`) — JSON, Markdown (table + per-item detail + score breakdown), Excel (openpyxl), CSV. Real outputs exist for 4+ sample queries in `outputs/`.
- **Web UI** (`webui/app.py` + templates) — FastAPI form → ranked cards with score breakdowns → one-click download of all four formats; `/health` endpoint; path-traversal guard on downloads.
- **Config-driven sources** (`config/sources.yaml`) — single place to enable/disable sites, caps, timeouts, delays, cache toggle.
- **Tests** — 20 pure-logic tests (requirement/normalize/rank/relevance), no network.

## 3. Planned but missing
From `tasks/todo.md` ("Possible follow-ups, not started") and code reality:

- **Real per-site adapters.** Despite the README implying dedicated adapters, `scrape/sites.py` shows `priceoye`, `czone`, and `mega` are **thin wrappers that all call the same generic extractor** — there is no site-specific selector logic. Czone, Mega, and Paklap returned **0 items** in the recorded live run (logged in `lessons.md`). So effectively 2 of 5 retailers work today.
- **Daraz** adapter present but disabled (heavy anti-bot); not hardened.
- **Per-product detail-page enrichment** (full specs, warranty, stock) — not started; availability is mostly "unknown".
- **Paid/robust search backends** (Brave / SerpAPI) — referenced in `.env.example` but not wired into `search.py`.
- **No persistence, accounts, history, scheduling, alerting, or deployment** — single-shot local tool only.

## 4. The AI gap
**The shipped/default pipeline contains no AI.** Intent parsing, ranking, and rationales are all regex + weighted arithmetic. The "ranking/LLM reasoning" in the brief is, by default, deterministic scoring — not reasoning.

There is an **optional Claude layer (`llm.py`), default OFF**, gated behind a key plus an explicit enable flag, with two hooks: `enhance_requirement()` (better intent parsing) and `annotate_results()` (SME-fit verdicts + natural-language rationale). It is honest about being optional. However:

- **It is unverified and likely non-functional as written.** It calls `client.messages.parse(..., output_format=Model)` and reads `resp.parsed_output`, plus `thinking={"type":"adaptive"}` and `output_config={"effort":"low"}`. These are not methods/params of the current Anthropic Python SDK (which uses `messages.create`; structured output is done via tool-use). As written these calls would raise — and because every function is wrapped in `try/except` returning the un-enhanced input, the AI layer would **silently no-op** even when "enabled."
- **It has never executed.** There is a latent bug in `pipeline.py` (`meta["llm"] = True` references `meta` before it is assigned) that would `NameError` the moment the LLM path is taken — proof the enabled path was never run. No tests cover `llm.py`.
- **Model id `claude-opus-4-8`** is referenced as the default — plausible but the integration around it is aspirational, not validated against a live API.

Net: AI is a stubbed-out enhancement, not a shipped capability. Treat the product as a deterministic scraper-ranker.

## 5. Missing pieces to make it sellable
- **Make the AI layer real and default.** Rewrite `llm.py` against the actual Anthropic SDK (tool-use/structured output), fix the `meta` bug, add tests, and make LLM intent-parsing + fit reasoning the primary path (deterministic as fallback). This is the single biggest gap.
- **Reliable sourcing.** Real adapters for Czone/Mega/Paklap, hardened Daraz, and a paid search fallback — today coverage is effectively 2 retailers, which undermines "researches the web."
- **Trust & freshness.** Price/stock verification, scrape timestamps, "as-of" labeling, and detail-page enrichment (warranty, true availability).
- **Decision artifacts buyers pay for:** total-cost-for-quantity math (qty × unit, already partially modeled), shortlist comparison narrative, and a buyer-ready recommendation memo.
- **Productization:** hosted deployment, auth, saved searches/history, re-run-on-schedule with price-drop alerts, and a shareable result link.
- **Quality bar:** the sample output shows all 5 "under-150k" laptops actually over budget — needs explicit budget-fit handling and a "nothing fits, here's the closest / raise budget to X" message.

## 6. Native-AI + Autonomous-FTE upgrade
Reframe as an **AI Procurement Analyst FTE** for SMEs:
- **Researches 24/7:** monitors target categories across PK retailers + SaaS vendors, re-checks prices/stock on a schedule, and surfaces deals proactively instead of one-shot on request.
- **Reasons, not just scores:** an LLM agent reads listings, reconciles specs against the requirement, flags mismatches/gotchas (refurb, no warranty, gray import), and writes a defensible recommendation memo with trade-offs.
- **Compares & prepares negotiation:** builds a per-vendor comparison, computes total cost for the requested quantity, drafts an RFQ/email to vendors, and assembles negotiation talking points (competitor prices, bundle leverage).
- **Reports & closes the loop:** delivers an Excel BOM + Markdown memo to the buyer, tracks decisions, and re-quotes when prices move.
- **"Replaces ~15–25 hours/week** of a junior procurement/research associate's manual price-hunting, spec-matching, and quote-collation."
- **Guardrails:** every claim cites a source URL with scrape timestamp; never invents prices/specs (already a design principle); budget/over-budget always disclosed; human approval gate before any outbound vendor contact; confidence + freshness labels on each recommendation.

## 7. Showcase angle (for the portal)
- **Headline:** "Describe what you need. Your AI procurement analyst returns a sourced, ranked shortlist — in PKR, for your city."
- **Stat-benefit bullets:**
  - One sentence in → top-5 ranked buys out, each with price, source link, and a scored why.
  - Pakistan-native: PKR pricing and Karachi/local availability weighted highest, not generic global results.
  - Five formats from one run — Web UI, Excel, CSV, JSON, Markdown memo — ready to forward or file.
- **Demo hook:** Type "5 office laptops for an accounting SME in Karachi, Core i5, 8GB RAM, under 150k PKR" and watch live PriceOye/Telemart listings get scraped, gated, and ranked into a downloadable shortlist in seconds. (Pre-baked cached run recommended for reliability on stage.)

## 8. Verdict
- **AI-native score:** 1/5 — default pipeline is 100% deterministic; the Claude layer is off, untested, and as-written would not run.
- **FTE-fit score:** 2/5 — the procurement-analyst framing is strong and the data plumbing exists, but autonomy is shallow (single-shot, no monitoring/negotiation/agency) and sourcing is thin.
- **Maturity:** MVP (functional, demoable, narrow; brittle scraping; AI stubbed).
- **Recommendation:** **Incubate.** Genuine, defensible wedge (PK SME procurement with PKR/Karachi weighting and multi-format outputs) and clean, honest engineering — but it needs a working AI reasoning core and broader retailer coverage before it can be sold as an autonomous FTE. Strong candidate for an AI-first rebuild on top of the existing pipeline.
