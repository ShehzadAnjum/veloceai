# Safe Journey Voice Agent — AI voice telesales rep for travel insurance (demo-stage)

## 1. Snapshot

- **One-liner:** A voice-driven travel-insurance telesales agent that greets a prospect, walks a scripted 7-stage sales flow, and captures lead contact details — with a working speech demo plus a half-built spec-driven voice backend.
- **Category:** Conversational/voice AI · outbound telesales & lead capture (insurance vertical).
- **Target buyer:** Insurance brokers, travel-insurance MGAs, and BPO/call-center operators who want to augment or replace tele-sales seats; also a reference demo for selling "autonomous FTE" call agents.
- **Tech stack:** Python 3.12; OpenAI Whisper (STT) + OpenAI TTS (`gpt-4o-mini-tts`/`tts-1`, "nova" voice) in the working demo; `sounddevice` + `webrtcvad` + `pygame`/`pydub` for capture/playback; Flask web UI. A separate newer backend targets Deepgram Aura-2 (TTS) + Nova-3 (STT) with OpenAI fallback on FastAPI + SQLModel + Neon Postgres + Alembic. OpenAI Agents SDK is imported but unused. SpecKit Plus (SDD) tooling drives the docs/specs/ADRs.
- **Maturity:** **Prototype.** Justification: the root-level "MVP v1" is a genuinely runnable voice loop with a 209 MB demo video (`insurance_travel_mvp_v1.mp4`), but its "AI" is deterministic keyword matching, not an LLM. The newer Deepgram backend is partial scaffolding: server boots and health endpoints return 200, but every voice endpoint returns "All TTS providers failed" (placeholder API keys), no database is provisioned, integration/E2E tests never ran, and ~41% of planned tasks are checked off (62 done / 88 open in `tasks.md`).
- **Live URL / demo:** No hosted URL. Local only: Flask UI at `http://0.0.0.0:5000` (`web_app.py`) and FastAPI at `http://0.0.0.0:8000` (`backend/`). Offline demo video: `insurance_travel_mvp_v1.mp4`.
- **Path:** /home/anjum/dev/travel_insurance_ai_agent

## 2. What's built today (verified)

There are effectively **two codebases** in the repo:

**A. Root "MVP v1" (Sept 2025) — a working voice demo with a fake brain.**
- `voice_module.py` (425 lines): real, working STT/TTS. Mic capture with VAD endpointing, OpenAI Whisper transcription, OpenAI TTS with sentence chunking (max ~200 chars), text normalization, 1.2 s/120 ms pause pacing, and layered playback fallbacks (pygame → playsound → Windows Media Player). This is the strongest, most production-credible code in the repo.
- `ai_agent.py` (494 lines): the conversation "engine." **It is a hardcoded `if/elif` keyword state machine** across ~11 stages (intro → verification → travel plans → duration → dates → persons → concerns → budget → recommendation → contact → closing). It imports `from agents import Agent, Runner` and instantiates an `Agent(...)`, but **`Runner` is never called and the Agent object is never used** — no LLM actually generates dialogue. The class is `SafeJourneyAgent`, yet its own `test_ai_agent()` calls a non-existent `TravelInsuranceAgent()`, so the bundled self-test is broken. Customer name is hardcoded ("Mr. Shehzad").
- `main_controller.py` (CLI loop) and `web_app.py` (Flask: `/api/start_conversation`, `/api/process_audio`, `/api/text_chat`, WebM→WAV conversion) wire voice + agent together. Plus ~15 ad-hoc `test_*.py` audio scripts.
- Evidence it ran: the demo video and committed test audio/MP3 artifacts.

**B. `backend/` (Dec 2025) — SpecKit Plus rewrite, voice I/O layer only, not yet operational.**
- ~2,800 lines under `backend/src`: FastAPI app (`main.py`), TTS service (Deepgram Aura-2 + OpenAI fallback, cost tracking), STT service (Deepgram Nova-3 + Whisper fallback, streaming endpoint), `text_processor.py` (377 lines: normalization + chunking), audio converter, SQLModel models (`VoiceInteractionSession`, `TranscriptionRecord`, `SpeechSynthesisRecord`), settings, logging, JWT auth middleware stub. 6 voice endpoints exist.
- Verified state from `backend/LIVE_TEST_REPORT.md`: server + `/health` + `/` return 200; **TTS endpoint returns 503 "All TTS providers failed"** (placeholder keys); STT/process-turn/session endpoints untested. Unit tests 38/38 pass — but only cover models + text processor; integration/E2E never run; no DB. Repo also contains junk nested empty dirs (`backend/backend/backend/...`).
- Heavy SDD documentation: a 1,199-line constitution, full `specs/001-deepgram-voice-integration/` (spec, plan, tasks, data-model, research, contracts), 4 ADRs, PHRs, and ~30 `.claude` subagent/skill definitions. Documentation maturity far exceeds code maturity.

## 3. Planned but missing

- **The actual AI sales brain.** The new backend has no conversation orchestrator/agent (no `*orchestrat*`/`*agent*` module exists in `backend/src`). The only "logic" — the keyword state machine — lives in the un-migrated old `ai_agent.py`. So the modern stack can speak and listen but cannot decide what to say.
- **Real LLM reasoning / RAG.** Despite the `ai_agent.py` docstring claiming "RAG-based responses," there is none. No retrieval, no LLM dialogue generation, no objection-handling beyond keyword branches.
- **Telephony.** Researched only (`docs/TELEPHONY_INTEGRATION_RESEARCH.md` recommends SignalWire + Telnyx, Twilio fallback). No Twilio/SignalWire integration, no inbound/outbound calling, no OpenAI Realtime API. Voice is request/response, not live duplex streaming. Phase 2.
- **Database persistence.** Neon Postgres is specified and models exist, but no instance is provisioned and migrations haven't been generated/applied; conversation state is in-memory only.
- **Dynamic pricing, admin dashboard, email/SMS automation, lead scoring, CRM** — all Phase 2 per the constitution; none built.
- **Deepgram path unproven.** No Aura-2/Nova-3 call has succeeded (no keys). The <900 ms latency, >95% STT accuracy, and <$0.15/conversation targets are all unvalidated.
- **Compliance enforcement in code.** Constitution mandates auto-terminate after 2 objections, consent capture, TCPA/DNC handling — these are documented, not implemented (the keyword machine has soft "not interested" exits only).

## 4. The AI gap

The headline gap: **this is sold as an "AI voice agent," but the shipped conversation intelligence is rule-based keyword matching, not AI.** `process_user_input()` is a deterministic branch tree; the OpenAI Agents SDK `Agent`/`Runner` are imported and instantiated as dead code. Outcomes:
- **Brittle:** anything off-script ("can my dog be covered?", a number out of order, a multi-part answer) falls through to a generic catch-all. No genuine understanding, memory, or reasoning.
- **Not generalizable:** the flow is hardwired to one persona ("Mr. Shehzad") and one product set; it can't adapt tone, handle novel objections, or personalize.
- **The genuinely strong AI parts are I/O, not cognition:** Whisper STT and neural TTS work well. The modern Deepgram backend is also pure I/O plumbing. The "agent" layer — where the value is — is the least AI-native part and the least finished.
- **Net:** the repo's AI is currently a thin transcription/synthesis wrapper around a scripted IVR. To be credibly "AI-native," the keyword machine must be replaced by an LLM-driven agent (tool-calling for pricing/quotes, RAG over policy docs, guardrails), which does not yet exist in either codebase.

## 5. Missing pieces to make it sellable

1. **Replace the keyword state machine with a real LLM agent** (function/tool calling for quote, pricing, eligibility; RAG over policy wordings; structured slot-filling for trip details). This is the core unlock.
2. **Wire the brain into the modern backend** — migrate the sales flow into a Conversation Orchestrator service in `backend/src`, so Deepgram voice I/O and the agent run as one deployable system.
3. **Provision real infra:** live Deepgram + OpenAI keys, a Neon DB with applied migrations, and an actual end-to-end run that proves a full conversation + lead capture. Right now nothing in the new stack has completed one turn.
4. **Telephony for real call handling** (SignalWire/Twilio Media Streams) so it can take/make phone calls, not just browser mic demos.
5. **Implement the compliance guardrails in code** (objection limit, consent capture, DNC/TCPA, call recording disclosure) — these are a selling point for regulated buyers but currently only exist as prose.
6. **A hosted demo** (one URL a prospect can talk to) and a clean repo (remove duplicate `backend/backend/backend`, broken self-tests, 209 MB video and audio blobs from VCS).
7. **Latency/accuracy/cost validation** against the spec's own targets, with a metrics dashboard, before claiming "production-ready."

## 6. Native-AI + Autonomous-FTE upgrade

Travel-insurance telesales is a near-ideal "autonomous FTE" use case, and the project already has the scaffolding and domain research for it.

- **AI voice sales rep (inbound + outbound), 24/7.** Stand up an LLM-driven agent on SignalWire/Twilio that fields inbound quote calls and runs outbound follow-up campaigns, fully replacing the dial-talk-log loop of a tele-sales seat. **Positioning: replaces ~6–8 productive talk-hours of a call-center seat per shift**, with no ramp time and instant scale from 1 → 100+ concurrent calls (the spec already targets <100 concurrent).
- **Quote-and-bind agent.** Add tool calls for real-time pricing, eligibility checks, and policy issuance/email-quote, turning the agent from "lead capture" into "closes and binds," measurable as quotes issued and policies bound per day.
- **Guardrails / compliance as a feature.** The documented TCPA stance (auto-terminate after 2 objections, consent capture, DNC, call-recording disclosure, encrypted PII) is exactly what regulated insurance buyers demand — implement it and sell it as "compliant-by-construction," with full transcripts and cost-per-conversation logged for audit.
- **Human-in-the-loop escalation** for edge cases, so the FTE story is "handles the routine 90%, warm-transfers the rest."

## 7. Showcase angle (for the portal)

- **Headline:** "Your always-on travel-insurance tele-sales rep — quotes, objections, and lead capture, by voice, 24/7."
- **Stat-benefit bullets:**
  - Replaces the talk-hours of a full call-center seat — no ramp, no shift gaps, scales 1→100+ calls instantly.
  - Sub-second voice loop with neural TTS and domain-tuned STT — natural enough that testers rate it "human," not robotic.
  - Compliance-by-construction: consent capture, objection limits, DNC/TCPA handling, and full audited transcripts on every call.
- **Demo hook:** Hand the prospect a phone number (or "Start Conversation" button), let them play the skeptical traveler, and watch the agent verify, qualify the trip, handle "I'm not interested," recommend a plan, and capture an email — end to end, hands-free. (Today this requires the keyword demo or the offline `insurance_travel_mvp_v1.mp4`; the LLM upgrade makes it robust.)

## 8. Verdict

- **AI-native score:** 2/5 — excellent voice I/O, but the "agent" is a keyword IVR with the LLM imported as dead code; the modern backend has no brain wired in.
- **FTE-fit score:** 4/5 — textbook call-center-seat-replacement use case, with compliance and telephony already researched; only the execution gap holds it back.
- **Maturity:** Prototype (working but rule-based demo + partial, non-operational SDD backend).
- **Recommendation:** **Incubate.** The domain fit, voice tech, and SDD groundwork are strong assets, but it needs a real LLM agent, live infra, and telephony before it can be a flagship. A focused 2–4 week push (LLM orchestrator on the existing Deepgram backend + hosted demo) could promote it to Secondary, and to Flagship once quote-and-bind + telephony land.
