# Vision Alert — live AI camera monitoring that sits beside a Dahua NVR and raises zone-based alerts

## 1. Snapshot
- **One-liner:** A single-GPU service that pulls per-channel RTSP from an existing Dahua NVR, runs YOLO11 detection + ByteTrack tracking, evaluates intrusion/loiter/line-cross zone rules, and pushes live annotated video, an alert log, and WhatsApp snapshots to a browser dashboard.
- **Category:** Computer Vision / Physical Security (video analytics / VMS add-on).
- **Target buyer:** SMB/industrial sites that already own a Dahua (or similar) NVR — warehouses, yards, retail, residential compounds, small factories — plus security integrators who want analytics without ripping out the recorder.
- **Tech stack:** Python 3.12, PyAV/FFmpeg + GStreamer/NVDEC (decode), Ultralytics YOLO11 on torch-CUDA (MVP detector), custom dependency-free ByteTrackLite (tracking), Shapely-style point-in-polygon/line-side rules, FastAPI (MJPEG + SSE + WebSocket), SQLModel/SQLite, Next.js + react-konva frontend, WhatsApp Cloud bridge for notifications. Production targets (scaffolded): NVIDIA Triton + TensorRT, go2rtc WebRTC, Redis Streams, Postgres, MinIO, Docker Compose.
- **Maturity:** **MVP** — justified: the full live path (RTSP decode → detect → track → zone rules → annotated MJPEG → SQLite alerts → SSE/dashboard → WhatsApp) is implemented and the decode+track stage is verified against a real NVR (live stairwell snapshot, ~12 FPS, 2026-06-17); 24 unit tests pass. It is past prototype but short of production: the GPU-served production stack (Triton/TensorRT/NVDEC) is unproven, there is no multi-channel scaling, and no Redis/MinIO/Postgres yet.
- **Live URL / demo:** None public. Local only: `make dev` → backend `:8000`, frontend `http://localhost:3100`. Proof artifacts: `snapshots/live_nvr2_ch1.jpg` (real NVR frame), `snapshots/gpu_proof.jpg` (YOLO+track annotated).
- **Path:** /home/anjum/dev/vision_alert

## 2. What's built today (verified)
Read and confirmed in source; `pytest` → **24 passed**.
- **Live RTSP ingest, NVR-proven.** `pipeline/engine.py` opens the Dahua sub-stream via PyAV (`av.open`, TCP, 5s socket timeout) with capped exponential-backoff reconnect. Live-proven 2026-06-17 against NVR B (DHI-NVR4216-4KS2, H.264) at ~12 FPS; `snapshots/live_nvr2_ch1.jpg` is a genuine decoded frame. Codec-aware GStreamer/NVDEC pipeline-string builder exists and is unit-tested (`ingest/pipeline.py`, H.264/H.265, hw `nvh26{4,5}dec` + sw `avdec` fallback).
- **Detection, GPU-proven offline.** `detect/ultralytics_detector.py` runs YOLO11 on CUDA, class-filtered to person/vehicles (COCO 0,1,2,3,5,7). `scripts/dev_gpu_proof.py` ran it on an RTX 4050 → 5 detections + 5 stable track IDs, annotated snapshot written. A `Detector` seam (`detect/base.py`/`factory.py`) lets `DETECTOR_BACKEND` switch to Triton.
- **Tracking, real and tested.** `track/bytetrack.py` is a from-scratch ByteTrack-style tracker: two-stage IoU association, class-aware, constant-velocity, min-hits/max-age. 5 unit tests (IoU, greedy match, ID stability, age cull, person≠car).
- **Rules engine, real and tested.** `rules/engine.py` implements **intrusion** (foot-point in polygon), **loiter** (dwell ≥ N s with fired-state), and **line_cross** (directed sign-flip) with per-(zone,track) cooldown/debounce. Zones stored normalized [0,1]. Tested in `tests/test_rules.py`.
- **Full backend API.** `api/app.py`: per-channel annotated MJPEG stream with connecting/no-signal placeholders, zone CRUD with validation, alert history + live alert SSE, a detections WebSocket (for WebRTC overlay), and a WhatsApp snapshot-send endpoint. Pipelines are reference-counted per viewer (`pipeline/manager.py`), one GPU detector shared under a lock.
- **Persistence + notifications.** SQLite store (`store/db.py`) for zones and alerts; alert snapshots saved to disk. `whatsapp.py` sends an annotated snapshot via a local WhatsApp bridge (working integration pattern, reused from sibling apps).
- **Frontend (≈394 LOC).** Next.js app with channel selector, live MJPEG view, react-konva zone/line editor, alert panel, and a WebRTC view component.
- **Engineering discipline.** `docs/architecture.md` (decided stack + rejected alternatives + GPU budget tiering), `tasks/todo.md` with honest done/pending checkboxes, and a substantive `tasks/lessons.md` recording real debugging corrections.

## 3. Planned but missing
Scaffolded or design-only — **not** running:
- **Production serving stack unproven.** Triton model-repo, `export_engine.sh`, gRPC `TritonDetector`, and Compose `triton` service exist but were never built/run on a GPU box (engine, tensor-name reconciliation, health check all marked pending). Same for GStreamer+NVDEC hardware decode — the live path actually uses PyAV, not the documented NVDEC pipeline.
- **Architecture-only, no code:** Redis Streams event bus, MinIO object store, Postgres (only SQLite exists), evidence-clip capture from the main stream, camera-health monitor (non-ML heuristics), after-hours-schedule and crowd-count rules.
- **No PPE model and no fire/smoke model** — described in the GPU-budget tiering but not implemented or trained.
- **No multi-channel scale.** One detector serialized under a lock, explicitly "fine for 1-2 viewed channels"; Triton dynamic batching across 9–32 streams is the goal, not the state.
- **Never caught a live detection of an actual person/vehicle** (detector proven on a sample image; live scenes were empty) — environmental, not a code gap.

## 4. The AI gap
The "AI" today is **classical computer-vision perception**: YOLO11 boxes + IoU tracking + geometric zone rules. That is real, working ML — but it is detection, not reasoning. There is **no generative/LLM/VLM layer**: the system cannot describe what it sees ("a person climbing the perimeter fence near Gate 2"), cannot judge intent or severity, cannot suppress obvious false positives (a delivery van vs. an intruder), and cannot triage or escalate contextually. An alert is a row — channel, zone, rule, track ID, class — not a narrated incident. For a company selling *autonomous FTEs*, the gap is the entire "judgment" half of a security-guard's job: watch, **understand**, decide, communicate. The bones (frames on GPU, stable track IDs, a detector seam, a notification channel) are exactly the right substrate to bolt a VLM reasoning agent onto — but that agent does not exist yet.

## 5. Missing pieces to make it sellable
- **Prove the GPU production path end-to-end** (Triton/TensorRT engine + NVDEC) on real hardware, with a measured multi-channel FPS/utilization number — the core scale claim is currently unverified.
- **Multi-channel concurrency** (batched inference, per-camera codec config in DB) to credibly cover 9–32 streams, not 1–2.
- **Hardening:** Postgres + Redis for durable events/debounce, MinIO + evidence-clip capture, auth on the API/dashboard (currently open CORS to any localhost), multi-tenant/per-site config.
- **A reasoning/triage layer** (see §6) to cut false positives and produce human-readable incidents — without it, operators drown in raw zone trips.
- **Onboarding UX:** auto-discover NVR channels (partially there via `nvr.get_channels`), guided zone setup, alert rule templates per vertical, and notification routing (Telegram/email/webhook beyond the single WhatsApp path).
- **Packaging:** one-line installer/appliance image, licensing, and a deployment story for an on-prem GPU box next to the customer's NVR.

## 6. Native-AI + Autonomous-FTE upgrade
Position Vision Alert as the perception backbone for an **AI Security-Guard FTE** — a tireless monitoring-room seat that watches every feed 24/7. The upgrade: pipe each rule-break (already gated by track ID + debounce) plus the annotated snapshot into a **VLM reasoning agent** that (1) describes the incident in plain language, (2) classifies severity and likely intent, (3) **filters false positives** (couriers, staff, animals, shadows) against site context and schedules, and (4) escalates — silent log for benign, WhatsApp/Telegram with a written summary for real, phone/siren for critical. Layer in loss-prevention (dwell at high-value shelving, sweethearting cues) and safety-monitor variants (no-PPE in a zone, person-down, restricted-area entry) using the same crops. **FTE framing:** replaces ~1 monitoring-room seat per site (≈ 8–24 guard-hours/day, ~$2.5–6k/mo loaded) at one-GPU cost, never blinking, never bored. **Guardrails:** require track-ID persistence + debounce before a VLM call (cost + noise control), a confidence/severity threshold for human escalation, an audit trail (snapshot + reasoning stored per alert), and a "needs human review" queue so the agent assists rather than silently dismisses real threats.

## 7. Showcase angle (for the portal)
- **Headline:** "Turn the NVR you already own into a 24/7 AI security guard."
- **Stat-benefit bullets:**
  - **No rip-and-replace** — reads existing Dahua RTSP read-only; never touches recording. Live-proven on a real NVR at ~12 FPS.
  - **Draw a zone, get alerts** — browser zone/line editor + intrusion/loiter/line-cross rules with debounce; alerts stream live and ping WhatsApp with a snapshot.
  - **One GPU, many cameras** — YOLO11 + ByteTrack tracking on a single NVIDIA card, architected for 9–32 channels via batched Triton inference.
- **Demo hook:** Live dashboard — drag a polygon across a doorway on the real stairwell feed, walk through it, watch the green track box + alert fire and a WhatsApp snapshot land in seconds.

## 8. Verdict
- **AI-native score:** 3/5 — genuine, working ML perception (YOLO11 + custom ByteTrack + geometric rules), but no generative/agentic reasoning layer; it detects, it doesn't yet understand.
- **FTE-fit score:** 4/5 — security/monitoring-room watching is a textbook autonomous-FTE replacement, and the plumbing (24/7 feeds, alerts, snapshot notifications) is already pointed at exactly that seat; only the judgment/triage layer is missing.
- **Maturity:** MVP (live decode+detect+track+rules+UI+notify working and tested; production GPU-serving stack and scale unproven).
- **Recommendation:** **Secondary** — a strong, demoable showcase of a real working vision pipeline with an obvious, high-value upgrade path. Promote to **Flagship** once the VLM reasoning/triage agent lands and the multi-channel Triton/NVDEC path is hardware-proven.
</content>
</invoke>
