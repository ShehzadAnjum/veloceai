# WhatsApp MCP Server — the personal-WhatsApp channel that lets Aevum agents read, search and reply 24/7

## 1. Snapshot
- **One-liner:** A two-part bridge (Go ⇄ Python/MCP) that connects a *personal* WhatsApp account to AI agents — exposing search/read of message history (incl. media), contact lookup, and outbound text/media/voice sending, all over a local SQLite store and a localhost REST API.
- **Category:** Platform Capability / Agent Channel (it is plumbing, not a standalone product).
- **Target buyer:** Internal / platform — consumed by other Aevum projects (e.g. `Hackaton_0_FTE`), not sold directly to end users.
- **Tech stack:** Go bridge using `go.mau.fi/whatsmeow` (WhatsApp Web multi-device protocol) + `modernc.org/sqlite`; Python MCP server using `FastMCP` (`mcp[cli]`), `requests`/`httpx`; QR pairing via `qrterminal`; optional `ffmpeg` for Opus audio. Two SQLite DBs (`messages.db` for app data, `whatsapp.db` for whatsmeow session/contacts).
- **Maturity:** **MVP, bordering on "production for single-tenant personal use."** Justification: it is not a toy — the live install holds **32,691 messages across 1,577 chats spanning 2018→2026**, with real media (6,108 images, 2,585 audio, 1,643 video, 167 docs) and 5,430 tracked read receipts. It is actively wired into a working agent (`Hackaton_0_FTE`'s "FTE Nandos" sales watcher). But it is single-account, single-process, localhost-only with no auth/HA, and built on a personal-account integration — so it is *not* productized.
- **Live URL / demo:** None public. Runs locally: Go bridge serves a REST API on `http://localhost:8080`; MCP server runs over stdio for Claude Desktop / Cursor. (`Hackaton_0_FTE` has a `DEMO_GUIDE.md` showing the live WhatsApp flow.)
- **Path:** /home/anjum/dev/whatsapp-mcp

## 2. What's built today (verified)
Verified by reading source, not the README.

**Go WhatsApp bridge (`whatsapp-bridge/main.go`, ~1,800 lines):**
- whatsmeow client with QR-code pairing — both terminal (`qrterminal.GenerateHalfBlock`) and a **REST pairing API** added in this fork: `GET /api/status`, `GET /api/qr`, `POST /api/logout`, `POST /api/disconnect` with in-memory `clientState` (`disconnected`/`qr_pending`/`connected`) guarded by a mutex. Device name shows as "Personal Assistant" in WhatsApp Linked Devices.
- SQLite schema (`chats`, `messages`) with full media metadata columns (`url`, `media_key`, `file_sha256`, `file_enc_sha256`, `file_length`) plus a migrated **`read_at`** column.
- Live message handling: text extraction (`Conversation` + `ExtendedTextMessage`), media-info extraction for image/video/audio/document, chat-name resolution (contact full name / group info), and storage.
- **History sync**: handles whatsmeow `HistorySync` events and an **on-demand `POST /api/sync`** that builds a `BuildHistorySyncRequest` anchored on the newest stored message — lets an agent backfill a specific chat.
- **Read receipts**: `handleReceipt` tracks both `ReceiptTypeRead` (recipient read us) and `ReceiptTypeReadSelf` (you read on another device), persisting `read_at` — useful inbox/triage signal.
- **Reliability/correctness hardening** (this fork's `feat` commit): protobuf-vs-delivery **timestamp correction**, a smart UPSERT that fixes sender attribution and keeps the earliest timestamp on re-sync, and corrected group-history sender attribution (`participantUser`, top-level `Participant`).
- **Outbound send** (`POST /api/send`): text, and media by file extension → uploads via `client.Upload` and builds Image/Video/Audio/Document messages. Voice notes get real Ogg-Opus duration parsing + a synthetic 64-byte waveform (`analyzeOggOpus`, `placeholderWaveform`) and are sent as PTT.
- **Media download** (`POST /api/download`): re-fetches encrypted media using stored keys via a `MediaDownloader`, caches to `store/<chat>/<file>`, returns absolute path.
- **Incremental poll endpoint** (`GET /api/messages?since=&limit=`): returns inbound-only messages since a unix ts, and **resolves `@lid` JIDs → phone numbers + contact names** by joining whatsmeow's `whatsmeow_lid_map` / `whatsmeow_contacts`. This is the endpoint that powers the autonomous watcher.

**Python MCP server (`whatsapp-mcp-server/`):**
- `main.py` exposes **11 MCP tools**: `search_contacts`, `list_messages`, `list_chats`, `get_chat`, `get_direct_chat_by_contact`, `get_contact_chats`, `get_last_interaction`, `get_message_context`, `send_message`, `send_file`, `send_audio_message`, `download_media`.
- `whatsapp.py` reads the bridge's SQLite **directly** for queries (parameterized SQL, LIKE search, pagination, message-context windows) and calls the bridge **REST API** for send/download. `audio.py` converts arbitrary audio → Opus Ogg via ffmpeg.

**Proven consumer — `Hackaton_0_FTE` ("FTE Nandos"):** a 24/7 watcher (`src/watchers/whatsapp_watcher.py`) polls the bridge, parses sales questions with an LLM, queries Postgres, and replies into the group via `POST /api/send` (`src/services/whatsapp_service.py`). It reads history straight from the bridge SQLite via `aiosqlite` (`src/services/whatsapp_reader.py`). This is concrete evidence the capability works as an agent channel.

## 3. Planned but missing
- **No official Business API path** — entirely personal-account (whatsmeow). README itself warns ~20-day re-auth.
- **No auth on the REST API** — `:8080` is open on localhost with no token/secret; anything on the host can send messages as the user.
- **No multi-account / multi-tenant** — one device, one hardcoded port, one SQLite store.
- **No HA / queueing / retry** — single Go process; QR/connection state is in-memory; outbound has no delivery queue beyond a synchronous send result.
- **No rate limiting / anti-ban controls.**
- **MCP surface lags the bridge** — the newer bridge powers (read receipts, `/api/messages` polling, `/api/sync`) are *not* exposed as MCP tools; the live agent bypasses MCP and hits REST/SQLite directly.
- **No tests** in the repo; several stray prod binaries (`whatsapp-bridge.bak*`, `.prePatch*`) checked into the working tree.

## 4. The AI gap
This project contains **zero AI** — there is no model call, prompt, or inference anywhere in `whatsapp-mcp`. It is pure plumbing: a protocol bridge + datastore + tool/REST surface. That is the honest framing. The AI value is entirely **what an agent does over this channel** — and the only AI in the loop today lives in the *consumer* (`Hackaton_0_FTE` uses Gemini to parse sales queries). The MCP wrapper makes the channel *callable by* an LLM agent, but the intelligence is external. For the portal, never describe this as "AI that messages on WhatsApp"; describe it as "the WhatsApp nervous system our AI agents act through."

## 5. Missing pieces to make it sellable
- **ToS / account risk:** whatsmeow drives a *personal* account via the Web protocol — this violates WhatsApp's terms and risks bans; fine for internal/demo, unacceptable as a sold product. A sellable version needs the **official WhatsApp Business Cloud API** (Meta), with templates, opt-in, and the 24-hour customer-service window — a different integration entirely.
- **Compliance:** consent/opt-in tracking, data-retention controls, and audit logging (the README flags the "lethal trifecta"/prompt-injection exfiltration risk — a real concern when an LLM can both read private chats and send).
- **Multi-account & tenancy:** per-tenant sessions, isolated stores, credential management.
- **Reliability/HA:** supervised process, reconnection/backoff, outbound queue with delivery confirmation, rate limiting, observability/metrics.
- **Security:** authenticated REST (mTLS or bearer token), least-privilege, secrets in `.env`/vault (currently localhost-trust only).
- **Productization:** package the MCP surface to expose the new capabilities (polling, receipts, sync) and ship a managed deployment.

## 6. Native-AI + Autonomous-FTE upgrade
Position this as the **omnichannel nerve for autonomous FTEs** — shared infrastructure, not a product line:
- One bridge, many agents: a customer-service FTE, a sales-ops FTE, and an inbox-triage FTE all speak through the same channel. The `Hackaton_0_FTE` "FTE Nandos" sales agent already proves the pattern (group question → LLM → DB → reply) running 24/7.
- The read-receipt + incremental-poll + on-demand-sync features make it genuinely **agent-native**: an FTE can watch an inbox, know what's unread, backfill context for a specific contact, and respond — the primitives an always-on worker needs.
- Strategic move: standardize this as the WhatsApp adapter behind a provider-agnostic "messaging channel" interface (alongside email/Slack), then swap the personal-whatsmeow backend for the official Business API for any externally-facing FTE. Keep whatsmeow for internal/personal-assistant FTEs where the user owns the account.
- Pair with Aevum's agent runtime (Claude-based) so the same channel that today fronts a Gemini script becomes the conversational surface for a fleet of Claude FTEs.

## 7. Showcase angle (for the portal)
- **Headline:** "Your AI workforce, on WhatsApp — reading, triaging and replying 24/7."
- **Stat-benefit bullets:**
  - **32,000+ messages, 1,500+ chats, 8 years of history** indexed and searchable — agents act with full context, not a blank slate.
  - **11 agent tools + read-receipt and live-poll signals** — an FTE knows what's unread and answers in seconds, day or night.
  - **Text, images, video, documents and voice notes** flow both ways — a complete conversational channel, not a notification gimmick.
- **Demo hook:** Drop a question into a WhatsApp group ("what were yesterday's Nando's sales?") and watch an autonomous FTE reply in-thread within seconds — no human touched the phone. (Live in `Hackaton_0_FTE`'s `DEMO_GUIDE.md`.)

## 8. Verdict
- **AI-native score:** 1/5 — no AI in the repo at all; it is deliberately plumbing. Scores above 0 only because it is purpose-built (MCP tools + agent-friendly poll/receipt APIs) to be driven by an LLM.
- **FTE-fit score:** 4/5 — an excellent, already-proven channel for always-on agents; loses a point because the personal-account backend caps it at internal/demo use until the Business API path exists.
- **Maturity:** MVP (functionally production-grade for single-tenant personal use; not productized — no auth, HA, multi-tenant, or compliance).
- **Recommendation:** **Platform capability — Incubate as shared infrastructure.** Do not market as a standalone product. Fold it behind a unified messaging-channel interface, harden security/HA, and plan the official Business API backend for any customer-facing FTE. It is the WhatsApp limb of the autonomous workforce, not a product on its own.
