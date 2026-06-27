# Email Threat Classifier MVP — Read-only Gmail CLI that labels selected emails as spam/phishing/spoofing using keyword rules

## 1. Snapshot
- **One-liner:** A Python CLI that authenticates to one Gmail account, fetches a user-selected email by message ID, and tags it legitimate / spam / phishing / spoofing using regex + keyword rules and the auth verdicts Gmail already computed.
- **Category:** Security (email threat detection / anti-phishing)
- **Target buyer:** SMB IT admin / individual power user wanting on-demand triage of suspicious emails; aspirationally a SOC/email-security team (not yet served — no multi-user, no inbound feed).
- **Tech stack:** Python 3.11+, `google-api-python-client` + `google-auth-oauthlib` (Gmail OAuth2, `gmail.readonly`), stdlib `email` parsing, SQLite logging, PyYAML-driven rules. No web framework, no Docker, no ML/LLM libraries.
- **Maturity:** **Prototype** (leaning early-MVP on paper only). Justification: the spec/plan/tasks are detailed and the Gmail-fetch + parse + rule-classify happy path is coded, but ~3 of 6 planned subsystems (validators/, analyzers/, threat_intel/) do not exist as code, the `tests/` tree is **empty (0 test files)** despite a TDD plan and `pytest.ini`, there is a confirmed runtime bug in the phishing path, and there is no evidence it has ever been run (no generated SQLite DB). It is not a working, verified MVP.
- **Live URL / demo:** None. CLI only; no web UI, no hosted demo, no Dockerfile (despite plan claiming containerized deployment).
- **Path:** /home/anjum/dev/email_spam

## 2. What's built today (verified)
Verified by reading source under `src/email_threat_classifier/` (4,694 LOC):
- **Gmail OAuth2 + fetch (real):** `gmail/auth.py`, `client.py`, `fetcher.py`, `token_manager.py` implement the desktop OAuth2 flow, token storage, and `messages.get` fetch by ID. Read-only scope (`gmail.readonly`) per `config/gmail_api_scopes.yaml`. This is the most complete module.
- **Email parsing:** `parser/email_parser.py` + `parser/header_extractor.py` extract From/To/Subject/Return-Path/Received chain and pull SPF/DKIM/DMARC **verdicts by substring-matching Gmail's `Authentication-Results` / `Received-SPF` headers** (e.g. `'dkim=pass' in auth_lower`).
- **Three rule classifiers** (`classifiers/`): `spam_classifier.py`, `phishing_classifier.py`, `spoofing_classifier.py` over a shared `base_classifier.py`. Detection = keyword/regex matching against `config/classification_rules.yaml` (spam words like "viagra"/"you have won", phishing phrases like "verify your account", brand display-name spoofing for paypal/amazon/etc., From vs Return-Path domain mismatch, and reading the parsed SPF/DKIM/DMARC fail flags).
- **Scoring:** base_classifier computes a 0–1 confidence and 0–10 risk score from indicator severity weights. CLI runs all three classifiers and picks the result with the **highest risk score** (`max(results, key=risk_score)`).
- **CLI** (`cli/main.py`): commands `gmail-auth`, `analyze <id>`, `list`, `logs`. Results print to terminal; `--verbose` shows per-classifier scores and auth headers.
- **SQLite logging:** `storage/sqlite_logger.py` + `schema.sql` persist each classification for the `logs` command.
- **Data models:** typed models for EmailMessage, ClassificationResult, ThreatIndicator, Attachment, GmailConnection.
- **Strong planning artifacts:** full SDD set under `specs/001-email-threat-classifier/` (spec, plan, data-model, research, 88-task `tasks.md`, OpenAPI contract, requirements checklist).

## 3. Planned but missing
The plan (`plan.md`) and `tasks.md` describe far more than exists. Missing entirely:
- **`validators/` module — does not exist.** No DNS-based SPF, DKIM (`dkimpy`), or DMARC (`checkdmarc`) validation. Grep confirms no `import dns`, no `dkim` verification, no `checkdmarc`. `dnspython` is listed in `requirements.txt` but **unused**. "SPF/DKIM/DMARC verification" is really "trust whatever Gmail already stamped in the header."
- **`threat_intel/` module — does not exist.** No DNSBL/Spamhaus/SURBL lookups, no URL reputation, no IP reputation, no caching, no graceful-degradation code. All classifiers hardcode `threat_intel_status = UNAVAILABLE`. `data/threat_patterns/` is **empty** (no `spam_keywords.json`, `attack_signatures.json`, etc.).
- **`analyzers/` module — does not exist.** No URL extraction/analysis, no homograph/typosquatting/lookalike-domain detection, no attachment scanning (config lists `.exe`/`.scr` rules but no code consumes them), no dedicated social-engineering analyzer beyond inline keyword checks.
- **Tests — none.** `tests/{unit,integration,contract,fixtures}/` directories exist but contain **zero `.py` files**. Every Success Criteria number (95% phishing recall, 90% spam, <5% false-positive, 5s latency, batch-100) is **unmeasured and unverified**.
- **No batch analysis, no dashboard/UI, no multi-user, no Docker/containerization** (plan claims all of these as future/optional or shipped).

## 4. The AI gap
- **There is no AI of any kind.** No ML model, no embeddings, no LLM, no anthropic/openai dependency anywhere. The README's "intelligent classification system" is implemented as static YAML keyword lists and regex substring matching.
- **Brittle by construction:** detection depends on literal phrases ("nigerian prince", "verify your account"). Any rewording, obfuscation, image-based phish, or novel lure passes as legitimate. This is exactly the class of attack modern phishing uses, so real-world recall would be low.
- **Decision logic is naive and false-positive-prone:** each classifier flags a category on a single indicator (the `if >=2 ... elif >=1 ...` branches return the *same* category — dead-code thresholds), and "final = max risk score" means one "discount"/"sale" keyword can label a legitimate newsletter as spam. The hardcoded `UNAVAILABLE` threat-intel status also docks every confidence score by 0.3.
- **Confirmed runtime bug:** `phishing_classifier.py` references `IndicatorType` at line 69 but only imports it at line 95 inside a *different* method — so the moment a credential-harvesting keyword actually matches, `analyze` throws `NameError`. The core "detect phishing" path is broken for true positives.
- **Relationship to sibling `spam_email`:** the sibling project (`/home/anjum/dev/spam_email`, "Explainable Phishing Scanner with LLM escalation") is the one that actually contemplates LLM-based reasoning and explainability. `email_spam` is the pure rules-engine cousin with no AI. If both ship, `email_spam` is the weaker, redundant half unless merged in as the cheap pre-filter stage of the sibling.

## 5. Missing pieces to make it sellable
1. **Make it actually run and prove it:** fix the `IndicatorType` NameError, write the missing unit/integration tests, and demonstrate one real labeled inbox end-to-end. Today nothing proves the pipeline works.
2. **Real authentication validation:** implement the `validators/` module (live SPF/DKIM/DMARC via DNS) so the product does something Gmail's own UI doesn't already show the user.
3. **Real detection intelligence:** replace literal-keyword matching with a model — minimally an ML classifier, ideally an LLM that reads the full email + URLs + auth context and returns a label + rationale. Add URL/attachment/lookalike-domain analysis.
4. **Threat-intel integration:** wire up at least one live feed (URL/domain reputation, DNSBL) with caching and the promised graceful degradation.
5. **A surface a buyer can use:** web UI or Gmail add-on, inbound/streaming analysis instead of paste-the-message-ID, multi-account/multi-tenant, and packaging (the missing Dockerfile).
6. **Trust + reporting:** explainable per-indicator output, recommended action (block/quarantine/allow), and an audit trail beyond the local SQLite log.

## 6. Native-AI + Autonomous-FTE upgrade
Reframe from "CLI you run per message" to an **autonomous email-security-triage agent** that watches a mailbox 24/7:
- **AI SOC / email-triage agent:** on every inbound message, deterministically gather signals (live SPF/DKIM/DMARC, URL/domain/IP reputation, attachment detonation flags) and hand the full context to an LLM that classifies, scores, and writes a plain-language rationale and recommended action.
- **Phishing-response agent:** auto-quarantine high-confidence threats, draft user warnings, open/annotate a ticket, notify the SOC channel, and escalate ambiguous cases to a human with its evidence attached — closing the loop, not just labeling.
- **Replaces ~N hours:** Tier-1 phishing triage is a heavy manual queue; an agent handling first-pass triage on, say, a few hundred reported/suspicious emails a day plausibly **replaces roughly 4–8 hours/day of an L1 security analyst's inbox-triage time**, with humans reviewing only escalations.
- **Guardrails:** read-only-by-default with explicit approval gates for quarantine/delete; confidence thresholds that force human review; full decision audit log; deterministic auth/reputation checks gating the LLM so it reasons over verified facts, not just prose; PII handling and least-privilege OAuth scopes.

## 7. Showcase angle (for the portal)
- **Headline:** "An AI analyst that triages every suspicious email 24/7 — verdict, reason, and action, before your team sees it."
- **Stat-benefit bullets:**
  - Offloads ~4–8 hours/day of L1 phishing triage to an always-on agent.
  - Goes beyond Gmail's own checks: live SPF/DKIM/DMARC + URL/domain reputation + LLM reasoning on intent.
  - Every verdict ships with a plain-English rationale and a recommended action (block / quarantine / allow).
- **Demo hook:** Forward a real phishing lure; watch the agent fetch it, validate authentication, reason about the lure, label it phishing, and draft the quarantine + user-warning — live.
- **Honest caveat for internal use:** the demo above describes the *target* agent, not today's code. The current repo is a rules-only CLI with no AI and a broken phishing path; do not present it as shipped.

## 8. Verdict
- **AI-native score:** 0.5 / 5 (zero AI today; static keyword/regex rules; the only "intelligence" is reading Gmail's own auth headers).
- **FTE-fit score:** 2 / 5 (the *security-triage* job is a genuinely strong autonomous-FTE fit, but this codebase delivers almost none of the autonomy — no inbound feed, no action-taking, no AI, on-demand single-message CLI).
- **Maturity:** Prototype (detailed specs; partial happy-path code; no tests; confirmed runtime bug; never demonstrably run).
- **Recommendation:** **Incubate** — keep the Gmail OAuth/fetch/parse scaffolding and the SDD planning, but rebuild detection around AI and fold it into the sibling `spam_email` (LLM scanner) rather than shipping it standalone. As-is it is not flagship- or even secondary-ready.
