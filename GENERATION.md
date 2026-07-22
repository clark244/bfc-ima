# Generating a Client Report (AI-assisted)

**This repo (`bfc-ima`) is the template of record.** Its `index.html` is the canonical
starting point for every new client report. Other client repos (e.g. `snorkl-ima`) are
completed deliverables, not templates — never start a new build from one. If the chrome needs
a fix or a new component, fix it **here** and re-push, then propagate.

There is no build step. The markup, JS, and SVG live in `index.html`; all CSS lives alongside
it in `styles.css` (linked from the `<head>`). Keep these two files together — `index.html`
references `styles.css` by relative path, so they must travel as a pair. The fonts load from
Typekit/Google CDNs; everything else is local.

The currently-filled version (Breathe for Change) doubles as a **gold-standard example**:
every content region is filled with real, well-formed copy. To make a new client report, you
clone the file and **rewrite the content regions** while leaving the **chrome** untouched.

> **Anchors, not line numbers.** This guide deliberately locates things by *search string*
> rather than line number. Line numbers drift the moment anyone edits the file, and stale
> pointers were the main way an earlier version of this guide went wrong.

---

## Files in this repo

| File | Role |
|---|---|
| `index.html` | The report. Markup + JS + SVG. **Canonical template.** |
| `styles.css` | All CSS. Chrome — do not edit per client. |
| *(no PDF file)* | The Doc's PDF is embedded **inside** `index.html` as base64 — not a separate repo file. |
| `bundle.py` | Inlines CSS into a single portable HTML in `dist/` (for email). |
| `GENERATION.md` | This guide. |
| `.gitignore` | Ignores `dist/`. |

---

## Order of operations (the real workflow)

The HTML is **not** the first artifact, and the downloadable PDF is **not** a render of the
HTML. The sequence is:

1. **Draft the Google Doc version** of the report (the Word/GDoc deliverable).
2. **Clark reviews and edits** the Doc until it is final.
3. **Generate the HTML report** from the finalized Doc's content (this guide).
4. **Export the final Google Doc to PDF**, base64-encode it, and embed it in `index.html`
   as `PDF_B64` (see "The Download PDF button"). This is a scripted step, not a manual one.
5. Update `PDF_NAME` to the filename the client should receive on download.
6. Post to GitHub; share the Word/PDF plus the GitHub Pages link.

Because the HTML and the PDF are two independent renderings of the same report, **build the
HTML only after the Doc is genuinely final.** If the Doc changes later, re-export the PDF and
re-commit it — the HTML will not update itself, and vice versa.

---

## How to generate a new report

1. Get the template with `git clone https://github.com/clark244/bfc-ima.git` — **not** a raw
   URL fetch, which truncates around 80KB and silently loses the `<script>` block. Copy
   `index.html` → `dist/<client>.html` and `styles.css` alongside it (keep the originals as the
   template). The copied HTML still links `styles.css` by relative path.
2. Feed an LLM: (a) the source material for the new client — discovery notes, questionnaire,
   public info; (b) this guide; (c) the filled BFC file as the example.
3. Instruct it to rewrite **only** the content regions listed below, preserving the chrome
   and all the coupling rules.
4. Run the **post-generation checklist** at the bottom before sending.

The model is *clone + rewrite-in-place*, not *regenerate from scratch*. The chrome (layout,
CSS, JS logic, SVG geometry) is hard-won and identical for every client — never rebuild it.

### Exporting a portable single file (for email)

The working copy links `styles.css`, so the two files must stay together. To produce **one
self-contained `.html`** (CSS inlined, no local dependencies):

```
python3 bundle.py                      # -> dist/bfc-bundled.html
python3 bundle.py dist/<client>.html   # custom output path
```

`bundle.py` needs only system Python 3. It reads `index.html` + `styles.css`, inlines the CSS
into a `<style>` block, and writes the bundle to `dist/` (gitignored). It never modifies the
source files. Because the PDF is embedded in `index.html` rather than sitting beside it, the
bundled file keeps a working Download PDF button — nothing extra to attach.

---

## Structure of the report

Sections, in document order. **"About This Report" is a modal, not a section** — it opens from
the cover buttons via `openAboutModal()`.

| Section id | Title | Notes |
|---|---|---|
| `#keyfindings` | Key Findings | Full-bleed tinted band, 2×2 card grid. Unnumbered. |
| `#model` | The Impact Process Model | Interactive SVG + print-only description table. |
| `#evidence` | Evidence Landscape | Table with inline source tooltips. |
| `#opportunities` | Measurement Opportunities | Framing prose + dot legend. |
| `#track1` | Track 1 — *(client-specific title)* | Priority accordion. |
| `#track2` | Track 2 — *(client-specific title)* | Priority accordion. |
| `#nextsteps` | Recommended Next Steps | Numbered list. |

**One track vs. two.** BFC is a two-track report (`#track1` + `#track2`). Most clients have a
single track. To go single-track: delete the `#track2` section, remove its nav sub-item,
remove its `navMap` entry, and renumber the priorities (e.g. `1a…1d` → `1…4`). Keep one
`.priority-matrix-key` at the top of whichever track sections remain.

---

## Content regions to rewrite (per client)

**Word counts are targets, not hard limits.** They come from the filled file, which is tuned
to the layout's rhythm — cards, matrix cells, and diagram boxes are sized for copy in these
ranges. Land inside the range and the report keeps its density. Run long and cards grow
uneven, prose blows past the `--measure` width, or diagram labels overflow their boxes. When
in doubt, cut toward the low end — this is an executive brief.

| # | Region | Where (search for) | Words (target) | Notes |
|---|--------|--------------------|----------------|-------|
| 1 | Cover + meta | `cover-label`, `cover-client`, `cover-meta-value` | labels only | Client name, doc type, "prepared for" contacts, date. **"Cobalt Collective" is the report author — constant, do not change.** |
| 2 | Key Findings (4 cards) | `class="kf-card"` | title **8–14**, desc **20–28** (≈**100** total desc) | The executive summary. Keep the four descriptions balanced so the cards align. |
| 3 | About modal | `id="about-modal"` | intro **60–90** | Mostly boilerplate; swap client name, source dates, discovery-conversation reference, and the `.about-sources` list. |
| 4 | Process model prose | `id="model"` intro `<p>` | **55–85** | Narrative framing of the diagram. |
| 5 | **The diagram** | `id="model-svg"` + the `nodes` object | box labels **2–5 words each**; each node `know`/`gaps` **30–60** | **The hardest region. See "The diagram trap" below.** |
| 6 | Evidence Landscape | `id="evidence-table"` + closing `<p>` | **6 rows**; closing para **80–110** | Rows = the client's existing evidence assets. Keep cell copy terse. |
| 7 | Opportunities framing | `id="opportunities"` intro `<p>`s | **3 paras, ≈200 total** (buyer-reality para **80–90**) | Includes the two-track framing. |
| 8 | Matrix column labels | `priority-matrix-key` | labels only | The four audience/buyer columns. Must match the `.pm-dots` order in every card. |
| 9 | Priority cards | `class="priority-card"` | body **170–230** (intro **90–140** + a 4-item list + one italic note **25–40**) | The detailed payload. BFC has 6 (1a–1d, 2a–2b). |
| 10 | Next Steps | `next-steps-list` | **6–7 steps, 25–45 each** | |
| 11 | Nav labels | `<nav>` in `id="sidebar"` | labels only | Must **mirror** the section titles and track titles. |
| 12 | Embedded PDF | `const PDF_B64` + `const PDF_NAME` | scripted | Regenerate from the final Doc. See below. |

---

## The Download PDF button

The button does **not** print the HTML. It serves the PDF export of the **final Google Doc**,
embedded directly in `index.html` as base64. This is the standard for all IMA HTML
deliverables — the file is self-contained and has no external dependency.

```js
const PDF_NAME = 'BFC_Impact_Measurement_Roadmap_May2026.pdf';
const PDF_B64  = "JVBERi0xLjQ...";   // the whole PDF, one line
```

`downloadPDF()` decodes `PDF_B64` with `atob`, builds a `Uint8Array` → `Blob`, and triggers an
`<a download>` click.

**Regenerating it per client** (scripted — never paste base64 by hand):

1. Export the final Doc: Drive `download_file_content` with `exportMimeType: application/pdf`.
   The response is base64 already. Large responses are written to a tool-results file that the
   sandbox can read directly at `.claude/projects/.../tool-results/` — decode from there so the
   base64 never has to pass through a model context.
2. `base64.b64encode(pdf_bytes)` → splice in as the `PDF_B64` literal, replacing the old one.
3. Update `PDF_NAME`.

**Always verify the round-trip.** Extract `PDF_B64` back out of the written file, decode it, and
assert the bytes are identical to the source PDF (compare SHA-256) and that it still parses at
the expected page count. A corrupted embed produces a button that downloads a broken file — it
will not announce itself.

**Expected size.** The embed adds roughly 4/3 the PDF's size, so `index.html` lands in the
hundreds of KB to low MB (BFC: 869KB for a 580KB, 9-page PDF). That is normal, not a mistake.

**The embed is a snapshot.** If the Doc changes, re-export and re-embed — the HTML will not
update itself.

---

## The diagram trap (region 5) — read before touching the model figure

The figure is **one SVG** (`id="model-svg"`) containing both the model and the measurement
overlay. There is **no** `switchView()` and **no** second `#view-meas` SVG — an earlier
version of this template worked that way and the guide has been corrected. The measurement
layer is a group, `id="meas-layer"`, shown/hidden by the `toggleBadges()` switch above the
figure.

The model has **7 nodes**: `product, impl, behavior, outcome, mech1, mech2, user`.
For each node, content lives in coupled places that must stay in sync:

1. **SVG `<text>` labels** — the visible box text, positioned by hand-tuned `x`/`y`
   coordinates. Rewrite the *words*; keep them short enough to fit; **do not change
   coordinates** unless you re-check the layout visually.
2. **Clickable `<rect ... onclick="show('<id>')">`** — the hit region. The `id` string must
   match a key in the `nodes` object. Don't rename ids.
3. **The `nodes` JS object** (search `const nodes`) — the drawer content shown when a box is
   clicked: `cat`, `title`, `tip`, `badge`, `badgeText`, `know`, `gaps`. Real per-client
   prose. The `badge` field (`ok` / `par` / `gap`) drives the colored confidence pill — set it
   to reflect how well-understood that part of the chain is.
4. **Box styling encodes confidence.** White + solid border = confirmed. Green = context.
   Blue = mechanism. **Amber + dashed border = needs confirmation / needs definition.** If you
   change a node's `badge` to `gap`, restyle its `<rect>` to amber-dashed and make sure the
   legend still matches.
5. **Measurement overlay** (`id="meas-layer"`) — per priority: a `.priority-badge-group`
   (`data-priority`, `onclick="openPriorityCard('<id>')"`), one or more `.connector-line`
   elements with the same `data-priority`, and the `.dim-overlay` rects. Badge ids must match
   the `priorityData` keys and the priority card ids.
6. **The `priorityData` JS object** (search `const priorityData`) — fills the priority drawer.
   One entry per priority; keep consistent with the priority cards (region 9).

**Rule:** if you change a node's meaning, update all of the above for that node. If you change
the *number or identity* of nodes, you are redrawing the figure — that's a layout job, not a
content swap. Do it deliberately and eyeball the result.

---

## Do NOT touch (chrome — identical for every client)

- The entire `styles.css` file (all CSS, including `--measure` and the layout variables).
  If a client genuinely needs a new component, add a rule to `styles.css` and treat it as a
  template improvement — don't inline styles into one client's HTML.
- All JS **functions/logic**: `show()`, `toggleCard()`, `toggleBadges()`, `openPriorityCard()`,
  `hidePriorityPopup()`, `toggleFullscreen()`, `openAboutModal()` (+ its close handler),
  `downloadPDF()`, the scroll-spy `IntersectionObserver` + `navMap`, and the sticky-TOC
  `--toc-pin` logic. You edit the JS **data** — `nodes`, `priorityData`, `navMap` keys, and
  the `PDF_B64` / `PDF_NAME` values — never the function bodies.
- SVG **geometry**: `<rect>`/`<line>`/`<marker>`/`<defs>` coordinates, arrowheads, filters,
  the dim-overlays and connector lines. Only the `<text>` *words* and the confidence
  fill/stroke are content.
- The cover topbar, the Cobalt Collective logo SVG, the footer brand, and the
  `cobaltcollective.org` author identity.
- The `.print-only` Model Description block — hidden on screen, restored in print.

---

## Coupling to keep in sync

- **Nav ↔ sections ↔ navMap:** every section `id` needs a nav `<a href="#x">` *and* a `navMap`
  entry, or scroll-spy highlighting breaks. Track sub-items use `.nav-item.sub` with a
  `.nav-badge` count that must equal the number of priority cards in that track.
- **Matrix ↔ priorities:** the `.pm-dots` in each card header must have the same number and
  order of columns as the `.priority-matrix-key` above it. Each track section carries its own
  copy of the key row as the first child of its `.priority-cards.priority-matrix` container.
- **Badges ↔ cards ↔ priorityData:** the badge `data-priority`, the card `id="card-<x>"`, and
  the `priorityData` key are the same identifier. All three must agree.
- **Track counts:** nav badge number = cards in that track = badges in the figure for that
  track.

---

## Post-generation checklist

- [ ] No previous-client specifics left anywhere — grep for: `Breathe for Change`, `BFC`,
      `Ilana Nankin`, `Michael Fenchel`, `Samuel Levine`, `William Jewell`, `Yoga Ed`,
      `Human Intelligence`, `M.S.Ed.`, `capstone`, `Perceived Stress Scale`.
- [ ] "Cobalt Collective" author identity intact (cover topbar logo, About modal, footer).
- [ ] `PDF_B64` regenerated from **this client's** final Doc, and `PDF_NAME` updated.
      Verified by round-trip: extracted, decoded, SHA-256 matches the source PDF.
      Then click the button in a browser and confirm the downloaded file opens correctly.
- [ ] Open in a browser: every model-diagram box click opens a drawer with matching content.
- [ ] Toggle "Show measurement priorities": badges appear; hovering a badge highlights its
      connector lines and dims the other nodes; clicking opens the right priority drawer.
- [ ] Full-screen button on the figure expands and collapses.
- [ ] All priority cards expand/collapse; the dot columns line up under the key row.
- [ ] Scroll top→bottom: the sticky TOC active state tracks the visible section, including
      the track sub-items.
- [ ] About modal opens and closes from both cover buttons (topbar and the narrow-screen one).
- [ ] Print/PDF preview looks right (the print-only Model Description renders; TOC and topbar
      are hidden; priority card bodies are expanded).
- [ ] Prose still respects the `--measure` width — no paragraph runs the full page width.
- [ ] Responsive: below ~860px the TOC hides and the About button moves under the cover meta.
