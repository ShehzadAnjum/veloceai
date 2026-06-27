# Voice Card-Blocking Demo — Bilingual (English/Urdu) speech IVR prototype for bank card blocking

## 1. Snapshot
- **One-liner:** A local desktop voice app that lets a caller authenticate and "block" a bank card by speaking English or Urdu, using OpenAI Whisper for speech-to-text and OpenAI TTS for replies, against a mock customer database.
- **Category:** Fintech / Voice AI (conversational banking self-service)
- **Target buyer:** Pakistani / South-Asian retail banks and card issuers; call-center / IVR operations teams wanting bilingual card-services automation.
- **Tech stack:** Python 3.12; OpenAI Whisper (`whisper-1`) STT; OpenAI TTS (`tts-1`, voice "nova"); Google Gemini (`gemini-2.0-flash`) and OpenAI chat/Assistant for "agent" replies; `gtts` fallback for Urdu; `sounddevice`/`scipy`/`numpy` mic capture; `playsound` playback; ffmpeg; `python-dotenv`. Mock data via `.env` env vars. No web/server framework, no telephony.
- **Maturity:** **Prototype.** Justification: runs only as a local CLI script bound to the dev machine's microphone/speaker; the entire banking backend (customers, CNIC, TPIN, cards) is mock data hardcoded in `.env`; "blocking a card" just plays a static "blocked" audio phrase and writes a log line — no transaction occurs. Three parallel half-finished versions exist (V1 monolith, V2 Assistant-API "beta/compatibility issues", V2-simple) plus dozens of ad-hoc test/demo scripts, indicating exploration rather than a shipped product.
- **Live URL / demo:** None. No deployment, no server, no hosted demo. Windows `.bat`/local `python card_blocking_app.py` only.
- **Path:** /home/anjum/dev/card_blocking_demo

## 2. What's built today (verified)
Verified by reading `card_blocking_app.py`, `card_blocking_app_v2_simple.py`, `config.py`, `.env`, and `README*.md`:
- **Working bilingual speech loop (real APIs).** `transcribe_audio()` calls OpenAI `whisper-1` with per-input-type prompts (digits/TPIN/CNIC) and language hints; `speak()` calls OpenAI `tts-1` and plays MP3. This part genuinely works against live OpenAI APIs (`card_blocking_app.py:581`, `:612`, `:668`).
- **Static audio cache.** `StaticAudioManager` pre-generates and hashes common phrases to `static_audio_cache/*.mp3` (28 cached files present) to cut TTS cost/latency for fixed prompts (`:192`). Real, functioning optimization.
- **Scripted card-blocking flow.** `run()` walks: welcome → language pick → identify customer (existing vs NTB) → CNIC (+ TPIN for existing) → "block card?" → card selection by last-4 digits → confirm → "blocked" → optional new-card application (`:1106`–`:1385`).
- **Mock authentication.** `validate_authentication()` does plain string membership against `MOCK_TPIN_DATABASE` / `MOCK_CNIC_DATABASE` lists from `.env` (`:864`). Customer type from a single hardcoded `MOCK_CLI_NUMBER`.
- **Number-pronunciation handling** for digit strings/CNIC/TPIN, and a `transfer_to_agent()` stub that just prints "transfer to human agent" (`:881`).
- **Test/demo scripts.** ~30 `test_*.py` / `demo_*.py` files (mic diagnostics, Urdu recognition, number recognition, multilingual). These are mostly manual/standalone scripts, not a CI suite; claimed "95% coverage" in docs is unverified marketing.

## 3. Planned but missing
README/summaries list as "Future Enhancements" (i.e. NOT built): real database integration, voice biometric authentication, predictive/recommendation analytics, multi-modal (image/document) support, microservices + load balancing, Redis session layer, CDN audio distribution, mobile (React Native) and web interfaces, Docker production deployment. Docs reference test files (`test_agent_tools.py`, `benchmark_v2.py`, `compare_versions.py`) that do not exist in the repo.

## 4. The AI gap
- **STT and TTS are real AI; the "agent"/intelligence is not.** Whisper and OpenAI TTS do real work. But the conversation is a hardcoded state machine, not AI-driven.
- **Language detection is a giant hand-written misrecognition list**, not a model — e.g. mapping "we're going to do" → "Urdu" (`:1132`–`:1166`). Brittle and embarrassing under scrutiny.
- **Gemini/LLM output is discarded.** Auth "validation" calls `get_gemini_response()` but throws the result away (`_ = self.get_gemini_response(...)`, `:1201`, `:1225`); the actual decision is a string match against mock lists. The LLM adds no functional value in V1.
- **Intent handling is rule-based** (`normalize_yes_no`, keyword/substring matching), so it does not generalize to free-form caller speech ("I lost my wallet, kill all my cards").
- **No fraud detection, no risk scoring, no real authentication, no real card action.** Provider sprawl (Whisper + TTS + Gemini + OpenAI Assistant + gTTS across 3 app versions) signals prototype churn rather than a coherent AI design.

## 5. Missing pieces to make it sellable
- **Telephony/IVR integration** (Twilio/SIP/contact-center) — today it is mic-and-speaker on one laptop; it cannot take a phone call.
- **Real core-banking / card-management API integration** (block, hot-list, reissue, dispute) replacing all mock env-var data.
- **Real identity & auth** (secure CNIC/OTP/TPIN verification, rate limiting, lockout, audit trail) — current string-match is a demo only.
- **LLM-driven NLU** for open-ended intent (block/dispute/balance/lost-card) instead of yes/no keyword scripts.
- **Concurrency / multi-session server**, observability, and a deployable (containerized) service; consolidate the 3 app versions into one.
- **Security & compliance**: PCI-DSS handling, PII redaction (logs currently print CNICs/TPINs/card lists to console), encryption, consent/recording disclosures.
- **QA**: a real automated test suite and accuracy benchmarks for Urdu STT on real telephony-quality audio.

## 6. Native-AI + Autonomous-FTE upgrade
Rebuild as an **AI multilingual card-services voice FTE** that staffs the phone line 24/7:
- **Telephony-native**: answers real inbound calls (Twilio/SIP), streaming STT + low-latency TTS, barge-in, and natural turn-taking in English and Urdu (plus Punjabi/Sindhi expansion).
- **LLM intent + tool-calling agent** that actually resolves: block/replace a card, report lost/stolen, dispute a transaction, check balance/last transactions — by calling secured core-banking tools (not mock data).
- **Authentication & fraud guardrails**: OTP + knowledge-based + optional voice-biometric step-up, velocity/anomaly checks before any block or dispute, hard human-handoff on low confidence or high-risk actions.
- **FTE framing:** one agent handles unlimited concurrent calls; a single bilingual card-services call-center seat covers ~160 hrs/month — this **replaces roughly 150+ agent-hours/month per seat** on repetitive card-block/lost-card calls (the highest-volume, lowest-complexity contact type), with sub-second response and zero after-hours gap.
- **Auditability**: every action logged, recorded, and PII-redacted for compliance.

## 7. Showcase angle (for the portal)
- **Headline:** "Block a stolen card by voice — in Urdu or English — 24/7, no agent, no hold music."
- **Stat-benefit bullets:**
  - Bilingual (English + Urdu) speech in and out — built on Whisper STT + OpenAI TTS.
  - Replaces ~150+ call-center agent-hours/month per seat on lost/stolen-card calls.
  - Sub-second cached prompts; one agent answers every call at once, around the clock.
- **Demo hook:** Live screen-record of a caller saying "میرا کارڈ بلاک کر دیں" / "block my card", authenticating, and the agent confirming the block — narrated as the autonomous night-shift card-services FTE.

## 8. Verdict
- **AI-native score:** 2/5 — real STT/TTS, but intent/auth/decisioning are hardcoded rules; the LLM output is literally discarded in V1.
- **FTE-fit score:** 3/5 — the use case (24/7 bilingual card-block call handling) is a strong, high-volume FTE-replacement fit; the current build delivers almost none of the autonomy needed.
- **Maturity:** Prototype (local CLI, mock backend, no telephony, no deployment).
- **Recommendation:** **Incubate.** The domain and bilingual angle are genuinely sellable, but it needs a near-total rebuild (telephony + real banking APIs + LLM agent + auth/fraud guardrails) before it is more than a demo. Strong story, weak substance today.
