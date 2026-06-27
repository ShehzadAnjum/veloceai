# Explainable Phishing Email Scanner — ML-primary triage with cost-controlled LLM escalation and plain-English verdicts

## 1. Snapshot
- **One-liner:** A phishing email scanner that scores risk 0–100, labels emails Safe/Suspicious/Phishing, explains *why* in plain English, and calls an LLM only for borderline or high-impact emails to keep costs down.
- **Category:** Security (email security / anti-phishing)
- **Target buyer:** SMB IT/security teams and MSSPs; secondarily SOC analyst tooling and security-awareness platforms. Buyer cares about analyst time saved and explainable, auditable verdicts.
- **Tech stack:** Python, scikit-learn (LogisticRegression + RandomForest), pandas/numpy, Flask web frontend (HTML/CSS), Anthropic Claude API (default `claude-3-5-sonnet-20241022`) with an OpenAI GPT fallback behind a provider abstraction, python-dotenv. Vercel/Procfile deployment scaffolding present.
- **Maturity:** **MVP** — justified below. Working end-to-end pipeline (feature extraction → trained model on real data → 3-tier classification → conditional LLM explanation → formatted output) is wired into a runnable Flask app with a trained 29 MB model artifact committed. It is past Prototype (it runs on real data, not toy stubs) but short of Production (accuracy below its own targets, no email-system integration, no auth/multi-tenancy, a hardcoded heuristic patch in the request path, no live deployment).
- **Live URL / demo:** None live. `submission/vercel.json` + `submission/Procfile` + `submission/api/index.py` exist as deployment scaffolding, but no deployed URL was found. Demo = run `python frontend/app.py` locally (port 5000); 12 curated test emails are built into the `/test-samples` endpoint.
- **Path:** /home/anjum/dev/spam_email

## 2. What's built today (verified)

This is **mostly working code**, not a paper design. The design brief (`Explainable Phishing Email Scanner with Controlled LLM Escalation.md`) is a 1-page spec/assignment, and the repo actually implements it.

- **Feature extraction (real, ~40 hand-crafted features)** — `phishing_scanner/feature_extractor.py`. Linguistic (urgency/threat/financial keyword counts, ALL-CAPS, exclamation, generic greetings), structural (link counts, external-link ratio, HTML/script presence, suspicious attachment extensions, length), and metadata (domain mismatch vs. claimed brand, suspicious TLDs `.tk/.ml/.ga`, free-email providers, reply-to mismatch). Also flags `is_high_impact` via finance/HR/executive/security keyword lists.
- **Trained ML classifier (real model on real data)** — `phishing_scanner/classifier.py` + committed `data/models/phishing_classifier.pkl` (29 MB). `train_model.py` trains on the **real Kaggle phishing dataset** (`data/raw/phishing_email.csv` ~100 MB; component sets Enron, Nazario, Nigerian_Fraud, SpamAssassin, CEAS, Ling also present), sampling 20k rows. Processed train/val/test splits exist (14k/3k/3k). Outputs a risk score (P(phishing)×100) and feature importances.
- **3-tier verdict from thresholds** — `classify_from_risk_score`: ≤44 Safe, 45–70 Suspicious, >70 Phishing. Note: the trained label is **binary** (safe=0/phishing=1); the "Suspicious" middle band is derived purely from score thresholds, not a learned class.
- **Controlled LLM escalation logic (real, wired in)** — `phishing_scanner/llm_escalator.py`: escalate **only** if score is borderline (45–70) OR email is high-impact; includes an escalation-rate tracker with a <30% target. This gate is actually called in the Flask `/analyze` path (`frontend/app.py`), so cheap rules/ML run first and the LLM is invoked conditionally — the core "controlled escalation" claim is genuinely implemented.
- **LLM explainer with provider abstraction** — `phishing_scanner/llm_explainer.py`: abstract `LLMProvider` with concrete `AnthropicProvider` (default, Claude 3.5 Sonnet) and `OpenAIProvider` (GPT-4). The prompt explicitly instructs the model to **explain, not override** the ML score (matches the design constraint), targets 2–4 plain-English sentences, and **fails gracefully** (returns None, app continues without LLM review).
- **Template-based (deterministic) explanations + user guidance** — `explainer.py` maps feature names → plain-English bullets and emits action guidance per verdict (e.g., Phishing → "Do not click… Report to IT… Delete"). This is the always-on explainability layer; the LLM only enriches borderline/high-impact cases.
- **Flask web app** — `frontend/app.py` with `templates/index.html` + `static/style.css`: paste sender/subject/body, get verdict, risk breakdown, top-10 feature importances, escalation analysis, email stats, and the formatted verdict. 12 built-in sample emails for instant demo.
- **Honest evaluation** — documented test metrics: **Accuracy 78.0%, Precision 79.0%, Recall 78.6%, F1 0.788** (`docs/PROJECT_ANALYSIS_AND_EXECUTION.md`), explicitly flagged as below the project's own targets (85/90/85). Logistic Regression scored 71.7%; switched to Random Forest at 78%. Failure modes (non-English email, novel tactics, sender-feature gaps) documented in `docs/REFLECTION.md`.

## 3. Planned but missing

- **Live/hosted demo** — deployment configs exist but nothing is deployed; no public URL.
- **Email-system integration** — no Gmail/M365/IMAP/gateway ingestion. Input is a manual paste form for one email at a time; no inbox connector, no batch/stream, no quarantine action.
- **URL/link & attachment analysis** — `/analyze` passes `links=[]` and `attachments=[]` with a `# TODO: Extract links from body`; link/attachment features therefore never fire from the web path despite being implemented in the extractor.
- **Real header/auth signals** — `spf_fail` is a hardcoded `0` placeholder; DKIM/DMARC/SPF, reply-to, and true sender-name checks are stubbed or unavailable without real headers.
- **Hitting its own accuracy targets** — 78% vs. 85% target; no model iteration (embeddings/transformer features, ensembles, calibration) done yet.
- **Sender-feature training gap** — `train_model.py` hardcodes `sender='unknown@example.com'` and empty subject for every training row, so domain-mismatch / suspicious-TLD / free-provider features are effectively dead during training and only activate at inference. The model largely learned body text only.
- **Productionization** — no auth, no multi-tenant, no rate limiting, no persistence/audit log, no feedback loop, no monitoring of live escalation rate/cost.

## 4. The AI gap

What's "AI" today is modest and honest: a classical scikit-learn classifier over hand-crafted features (the bulk of decisions), plus a **conditionally-invoked LLM** that writes a plain-English explanation for borderline/high-impact emails. The LLM is real and correctly scoped (explain-not-override, graceful fallback), but it is an explanation layer, not a detection brain — it never changes the verdict. There is no LLM-based detection, no agentic investigation (no enrichment, no pivoting on sender/URL reputation), no learning from analyst feedback, and the default model (`claude-3-5-sonnet-20241022`) is a now-older Claude that should be refreshed for a showcase. A small but notable wrinkle: a hardcoded heuristic in `app.py` manually subtracts up to 40 points from the model's score when "safe indicators" are present — a band-aid for false positives that signals the underlying model isn't well-calibrated. Net: solid, explainable, cost-aware plumbing; thin actual intelligence.

## 5. Missing pieces to make it sellable

- **Inbox/gateway connectors** (Gmail/M365 Graph/IMAP/journaling) so it scans real mail, not pasted text.
- **Real link + attachment + header analysis** (URL reputation/expansion, SPF/DKIM/DMARC parsing) — wire up the features already designed.
- **Model quality + calibration** — pass the 85%+ bar, calibrate probabilities, and retire the hardcoded score-adjustment hack; train with realistic sender/header fields.
- **Throughput** — batch/stream scanning, queueing, and a measured per-email cost + escalation-rate dashboard (the tracker exists; expose it).
- **Actioning + workflow** — quarantine/release, analyst review queue, "report phish" feedback that retrains, and SIEM/SOAR/ticketing hooks.
- **Tenant readiness** — auth, RBAC, audit log, data retention, PII handling.
- **Trust artifacts** — benchmark vs. known phishing corpora, false-positive rate on legit business mail, and a verdict audit trail (the explainability is a genuine selling point — productize it).

## 6. Native-AI + Autonomous-FTE upgrade

Reframe from a paste-box scanner into an **autonomous phishing-triage analyst** that runs 24/7 on the mail stream:

- **Always-on triage FTE:** auto-ingests every inbound email, runs cheap ML/rules first, and escalates only ambiguous/high-impact cases to an LLM — exactly the cost-control architecture already prototyped, now operating continuously instead of on demand.
- **Explainable verdicts by default:** every decision ships with What/Why/What-to-do (already built), giving auditable, analyst-grade write-ups for each flagged email — the differentiator vs. opaque filters.
- **Agentic enrichment (the real upgrade):** before deciding, the agent pivots on sender domain age/reputation, expands and detonates URLs, checks auth headers, and cross-references known campaigns — turning the LLM from "explainer" into "investigator."
- **Replaces ~N hours of a SOC analyst:** L1 phishing triage is high-volume and repetitive; an agent that auto-clears the obvious safe/malicious bulk and writes triage notes for the rest plausibly offloads **3–6 analyst-hours/day per 1–2k emails** — concrete, since escalation rate is already targeted/measured at <30%.
- **Guardrails:** LLM explains but cannot override the deterministic risk score (already enforced); add human-in-the-loop approval for destructive actions (quarantine/delete), confidence thresholds, full audit logging, and per-tenant cost caps on escalation.

## 7. Showcase angle (for the portal)

- **Headline:** "The phishing analyst that never sleeps — explains every verdict, and only pays for an LLM when it actually needs one."
- **Stat-benefit bullets:**
  - **<30% LLM escalation** by design — cheap ML/rules clear the easy 70%+, so you pay for AI only on the hard cases.
  - **Plain-English verdict on every email** — risk score, top signals, and "what to do," not a black-box spam label.
  - **Trained on the real ~100MB Kaggle phishing corpus**, with a runnable demo and 12 built-in example emails — working today, not a slide.
- **Demo hook:** Paste the built-in "CEO wire-transfer" or "PayPal suspended" sample → watch it score the risk, show the top features that fired, and (because it's borderline/high-impact) escalate to Claude for a human-readable explanation — live in one click.

## 8. Verdict
- **AI-native score:** 2/5 — real but narrow LLM use (explanation-only, correctly cost-gated); core detection is classical ML and the LLM never investigates or decides.
- **FTE-fit score:** 3/5 — phishing triage is a textbook autonomous-FTE use case and the escalation/explainability scaffolding fits it well, but it lacks inbox ingestion, actioning, and feedback to actually replace analyst hours today.
- **Maturity:** MVP (working end-to-end locally on real data; below its own accuracy targets; not deployed/integrated).
- **Recommendation:** **Secondary** — strong, demoable anchor for the "Email Security" product line (sibling: `/home/anjum/dev/email_spam` Email Threat Classifier) and a credible base for the autonomous triage-analyst story, but it needs inbox integration, model-quality work, and agentic enrichment before it's flagship-grade. Best paired/merged with the sibling classifier into one Email Security offering.
