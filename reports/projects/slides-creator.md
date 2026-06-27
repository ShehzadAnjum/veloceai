# Slides Creator — A high-fidelity HTML-to-PowerPoint rendering engine (no AI yet)

## 1. Snapshot
- **One-liner:** A code pipeline that turns hand-authored, themed HTML slides into pixel-accurate, editable `.pptx` decks (with stock-photo sourcing, icon rasterization, and screenshot QA).
- **Category:** Content / Productivity AI (today: deterministic content tooling, *not* AI)
- **Target buyer:** Presentation designers, consultants, sales/marketing/enablement teams, and analysts who must ship on-brand decks repeatedly.
- **Tech stack:** Node (CommonJS) · Playwright/Chromium (DOM render + measurement) · pptxgenjs v4 (native PPTX emit) · sharp (image crop/duotone/raster) · react + react-icons + react-dom/server (icon → PNG) · Python `.venv` with markitdown/mammoth/pdfplumber/pymupdf (source ingestion). No build scripts in `package.json`; tools are run by hand via `node build/<x>.js`.
- **Maturity:** **Prototype** — The core converter (`build/html2pptx.js`, 978 lines) is genuinely sophisticated and works: it produced a real `decks/SAMPLE_ProcessorsJourney.pptx` (626 KB, 2 slides). But it is a single approved sample only; per `tasks/todo.md` all three intended decks are still unchecked, there is no CLI/packaging, no tests, and no automation around it.
- **Live URL / demo:** None. Local artifact only: `decks/SAMPLE_ProcessorsJourney.pptx`.
- **Path:** /home/anjum/dev/slides_creator

## 2. What's built today (verified)
Verified by reading source, not assumptions:
- **HTML→PPTX engine (`build/html2pptx.js`):** Loads an HTML slide in headless Chromium, reads computed styles, and emits **native, editable PowerPoint elements** (text runs, lists, images, shapes, lines) at accurate positions. Real engineering inside: pt/px/EMU/inch conversions, CSS box-shadow → PPTX shadow parsing, rotation/`writing-mode` handling, inline `<b>/<i>/<u>/<span>` run formatting, single-line width compensation, and **validation** that throws on body-size mismatch or content overflow (with a 0.5" bottom-margin rule). This is the project's crown jewel.
- **Deck assembler (`build/build_deck.js`):** Globs numbered `*.html` files in a directory, runs each through the engine, sets author/title, writes one `.pptx`. Generic and reusable.
- **Visual QA renderer (`build/render.js`):** Playwright screenshots each HTML slide to 2x PNG (960x540) for overflow/contrast review without LibreOffice.
- **Shared theme system (`slides_html/theme.css`):** A polished "Agent Factory" design system — palette tokens, light content slides, dark section/title slides, cards, KPI/stat blocks, tags, callouts, fixed 720x405pt clipping stage so decorative art can bleed without tripping overflow validation.
- **Asset pipeline:**
  - `build/icons.js` — rasterizes ~45 react-icons (FontAwesome/Simple Icons) to brand-colored 256px PNGs.
  - `build/photos.js` — queries the **Openverse** API for license-clear stock, center-crops via sharp's attention algorithm, and **writes `ATTRIBUTION.json`** (real attributions present for 4 photos).
  - `build/duotone.js` — applies a navy duotone so photos blend into the dark theme.
- **Two finished sample slides** (`slides_html/sample_title.html`, `sample_content.html`) and the rendered sample deck — proof the full chain works end to end.
- **Source ingestion capability:** Python `.venv` ships markitdown/mammoth/pdfplumber/pymupdf to extract content from source `.pptx`/`.docx` (the todo references converting a `presentaiton_1.pptx` and docx slides) — but this step is manual and not wired into any script.

## 3. Planned but missing
From `tasks/todo.md` (decisions confirmed with user, work not done):
- Deck 1 "The Processor Journey", Deck 2 "Two Deaths, Not One", Deck 3 "Three Things Changed — Together" — **all three unchecked.**
- Remaining steps unchecked: build the shared theme *helpers*, source/process per-deck photos & icons, build each deck's HTML, run visual QA thumbnail grids, fix overflow/contrast, and final delivery.
- Only the explicit checkpoint ("build ONE sample, get approval") appears satisfied.

## 4. The AI gap
**There is zero AI in this project today — verified.** A repo-wide grep for `openai|anthropic|claude|gpt|gemini|llm|langchain|prompt|api_key|completion` returns nothing. The "intelligence" — writing slide copy, choosing layouts, picking icons, structuring the narrative — is **100% human**: every slide is hand-authored HTML/CSS. The system is a deterministic *renderer and asset fetcher*, not a generator. Even photo selection is keyword search (`build/photo-queries.json`), not semantic/visual reasoning. So as a product it is a high-quality **tool an AI could drive**, but it contains no model, no agent, and no content generation.

## 5. Missing pieces to make it sellable
- **A generation front-end:** brief/outline → slide HTML. Right now a human must write the HTML by hand; that is the entire bottleneck.
- **Productization:** no `package.json` scripts, no CLI, no config, no API/web UI, no install path. It's a developer's local toolkit.
- **Brand intake:** theme is hardcoded to one "Agent Factory" palette/fonts; needs per-customer brand kits (colors, logos, fonts, templates).
- **Reliability:** no tests, no error recovery, no batch/queue, no observability; Openverse/network calls fail silently to "FAILED".
- **Asset rights at scale:** Openverse licenses include `by-nc` and `by-sa` — needs license filtering/clearance for commercial use and auto-inserted attribution slides.
- **Output QA loop:** screenshots exist but nothing automatically detects/repairs overflow or contrast; today a human reviews.

## 6. Native-AI + Autonomous-FTE upgrade
**AI presentation-designer / deck-builder agent ("brief in → branded deck out, 24/7").** The architecture is unusually well-suited to this: the engine already takes **structured HTML in and emits editable PPTX out** — the exact seam an LLM can target.
- **Content agent:** turns a one-paragraph brief, raw doc, or transcript into a slide-by-slide outline and narrative (claims, stats, speaker notes).
- **Layout/design agent:** emits theme-conformant HTML against `theme.css`'s component vocabulary (cards, KPIs, dividers), choosing icons and stock imagery semantically.
- **Self-healing QA loop:** renders via `render.js`, runs the existing overflow/contrast validators, and **re-prompts itself to fix** failures before delivery — closing the loop no human currently closes.
- **Guardrails/brand:** per-customer brand kit (palette/logo/fonts/templates), commercial-license-only image filtering with auto-attribution, and factual/citation checks on generated claims.
- **FTE framing:** a branded 12–20 slide deck is realistically ~3–6 hours of a designer/analyst; an always-on agent **replaces ~4–6 hours per deck** and runs overnight in batches — i.e., one autonomous "presentation associate."

## 7. Showcase angle (for the portal)
- **Headline:** "Your overnight presentation team — a brief at 6pm, a boardroom-ready, on-brand, *editable* deck by 7am."
- **Stat-benefit bullets:**
  - **Editable, not screenshots:** emits native PowerPoint text/shapes/charts — clients keep full control (real `.pptx` proof in `decks/`).
  - **Pixel-accurate & self-validating:** a 978-line layout engine measures every element and rejects overflow before it ships.
  - **On-brand by construction:** one design system, license-clear imagery with auto-attribution, brand-colored iconography — consistency without a designer in the loop.
- **Demo hook:** Drop a messy source doc on the left; watch slide HTML render, auto-QA, and a downloadable branded `.pptx` appear on the right.

## 8. Verdict
- **AI-native score:** **1/5** — no AI today; the only credit is an architecture (structured-in → editable-out, with a validation loop) that is ideal to wrap an agent around.
- **FTE-fit score:** **4/5** — "deck production" is a high-volume, brand-bounded, clearly-scoped knowledge-worker task with a measurable hours-saved story; the existing engine + QA loop is the hard 60% already built.
- **Maturity:** Prototype (working engine + one approved sample; zero of three target decks delivered; no packaging/automation).
- **Recommendation:** **Incubate** — Genuinely strong, differentiated rendering tech (editable output + self-validation) but it is a renderer, not a product, and contains no AI. Fund the generation + brand-intake + self-healing agent layer on top; it could graduate to a Flagship "autonomous FTE" demo, and the editable-PPTX angle is a real wedge against screenshot-grade AI slide tools.

---
*Evidence: `build/html2pptx.js`, `build/build_deck.js`, `build/render.js`, `build/photos.js`, `build/duotone.js`, `build/icons.js`, `build/photo-queries.json`, `slides_html/theme.css`, `slides_html/sample_title.html`, `slides_html/sample_content.html`, `decks/SAMPLE_ProcessorsJourney.pptx`, `tasks/todo.md`, `package.json`, `assets/photos/ATTRIBUTION.json`. AI absence confirmed via repo-wide grep (no matches).*
