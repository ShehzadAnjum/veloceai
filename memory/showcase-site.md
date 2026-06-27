---
name: showcase-site
description: The site/ showcase — structure, the adopted CMS design system, run command, and rebrand tokens.
metadata:
  type: project
---

**The site (`site/`) is built and verified** (Playwright render, no console errors). Static
HTML/CSS/JS, **no build step**.
- `index.html` — landing: hero with an AI command bar + live fleet shift-log, thesis pillars,
  the **Watch → Reason → Act → Report** pipeline, the 5-unit fleet grid (+ a "20 more in the
  lab" card), the platform stack (Live/Roadmap tags), the honesty band, count-up stats, CTA.
- `products/*.html` — 5 flagship dossiers: `complainthub`, `personal-ai-employee`,
  `aria-assistant`, `operanova`, `vision-alert`. Each splits **Running today** (cyan card) vs
  **On the roadmap** (amber card) with an explicit honesty note + a "replaces" band + demo steps.
- `lab.html` — the 20 non-flagship products in 3 tiers (field-ready / in the lab / platform),
  filter chips, AI/FTE meter bars; each card links to its `../reports/projects/<slug>.md` dossier.
- `assets/styles.css` + `assets/app.js` — shared design system + JS (reveal, typing, clock, filter).

**Design system (ported from the CMS enterprise "AI spectrum" kit,
`/home/anjum/dev/cms/marketing/enterprise.css` + `enterprise.html`):** Space Grotesk / Inter /
JetBrains Mono; void background + floating cyan/violet/magenta orbs + masked grid; the
cyan→azure→violet→magenta gradient; **glass** gradient-bordered cards; AI command bar;
reveal-on-scroll. LIVE/ROADMAP honesty encoded as **cyan = live, amber = roadmap, cyan-glow =
production**.

**Run:** from the repo **root** → `python3 -m http.server 8055` →
http://localhost:8055/site/index.html. Serve from root (not `site/`) so the lab's `../reports/`
dossier links resolve.

**To rebrand:** replace `AEVUM` and `hello@aevum.ai` across `site/`; design tokens are CSS
custom properties at the top of `assets/styles.css`. Brand context: [[identity-and-positioning]].
