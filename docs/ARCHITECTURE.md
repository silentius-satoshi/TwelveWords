# Twelve Words apex site — architecture spec v1
*2026-08-24 · Architect deliverable. Status: **DRAFT — awaiting Founder sign-off.** No code moves until this is approved. Governs `github.com/silentius-satoshi/TwelveWords` → twelvewords.xyz. Implements the Founder decision that long-form writing lives on the apex rather than on Substack.*

---

## 1. What this spec covers

Turning the apex from a single hand-authored landing page into a small static site that hosts the company's long-form writing, without giving up any of the properties TW-3 locked: matte-black brand system, zero external requests, zero JavaScript, zero analytics, zero storage APIs, disclaimer verbatim on every public page.

**In scope:** repo layout, build pipeline, templating contract, content format, URL map, asset strategy, security headers, feed, SEO surface, the design of the three new page types, and the verification gates.

**Out of scope:** the landing page's visual design (locked by TW-3 and preserved byte-for-byte), product apps, comms strategy and publishing cadence (TW-6 territory), the brand npub ceremony.

---

## 2. Decisions this spec locks

| # | Decision | Why |
|---|---|---|
| A1 | **Own the build.** A ~150-line `scripts/build-site.mjs`, mirroring BitBooks' approach. | A framework (Eleventy/Astro) saves ~1h of setup and costs a ~100-package dependency tree, permanent upgrade duty, and conventions to learn — on a project whose pitch is "audit the code yourself." The Workhorse makes writing 150 lines cheap. |
| A2 | **Exactly one dependency: `marked@18` (verified zero transitive deps).** Front-matter is parsed by ~15 lines of our own code rather than adding `gray-matter`. | Total supply chain stays at 1 package. `markdown-it` would pull 6. |
| A3 | **Markdown is the authoring format** for notes and prose pages. | Writing long-form in HTML suppresses cadence, and cadence is the entire strategic point. |
| A4 | **The landing renders through the same template as every other page.** | Keeping it hand-authored reintroduces exactly the nav/footer/disclaimer drift this spec exists to prevent. Risk of visual regression is handled by a pixel-identity gate (§11), not by hope. |
| A5 | **CSS is an external same-origin file, not inlined.** | Inlining forces `style-src 'unsafe-inline'` in the CSP. External CSS permits `style-src 'self'` — a strictly stronger policy — and caches across pages. One 6KB request. |
| A6 | **Font is an external same-origin file, preloaded.** | At N>1 pages an inlined font is re-downloaded on every navigation and cannot be cached. Same-origin is not "external" in the sense TW-3 prohibited — that rule was about CDNs and third-party tracking. |
| A7 | **Full-text RSS at `/feed.xml`.** | If writing lives here, a feed is how people subscribe without surrendering an email address. Full text because there is no ad model and no analytics — nothing is gained by forcing a click. |
| A8 | **Builds are deterministic.** No `Date.now()` in output; the feed's `lastBuildDate` is the newest note's date. | Byte-identical rebuilds are what make the verification gates meaningful. |

---

## 3. Repo layout

```
TwelveWords/
├── content/
│   ├── pages/
│   │   └── about.md
│   └── notes/
│       └── YYYY-MM-DD-slug.md
├── templates/
│   ├── base.html            ← <head>, nav, footer, disclaimer. ONE source of truth.
│   ├── landing.html         ← body of the landing page only
│   ├── page.html            ← body wrapper for content/pages/*
│   ├── note.html            ← body wrapper for a single note
│   └── notes-index.html     ← body wrapper for /notes
├── assets/
│   ├── inter-tw.woff2       ← 38,452 bytes, subset incl. ₿ U+20BF
│   ├── site.css             ← extracted verbatim from today's <style> block, then extended
│   └── og.png               ← 1200×630 link-preview card
├── static/
│   ├── .well-known/nostr.json
│   └── robots.txt
├── scripts/build-site.mjs
├── package.json
├── vercel.json
├── .gitignore
└── dist/                    ← build output, gitignored
```

`index.html` at the repo root is **removed** at step 3 of the sequence, once `dist/index.html` is proven byte-identical to it. Not before.

---

## 4. Build pipeline

`node scripts/build-site.mjs`, no arguments, deterministic, exits non-zero on any failure.

1. Delete and recreate `dist/`.
2. Copy `assets/*` → `dist/assets/`; copy `static/*` → `dist/` (preserving `.well-known/`).
3. Read `templates/base.html`.
4. Render the landing from `templates/landing.html` → `dist/index.html`.
5. For each `content/pages/*.md`: parse front-matter, render markdown, wrap in `page.html`, → `dist/<slug>.html`.
6. Read `content/notes/*.md`. Skip any with `draft: true`. Sort by `date` descending.
7. Render each note through `note.html` → `dist/notes/<slug>.html`.
8. Render the notes index through `notes-index.html` → `dist/notes/index.html`.
9. Write `dist/feed.xml` and `dist/sitemap.xml`.
10. Print a summary: page count, note count, per-file byte sizes, total.

**Failure conditions (exit 1, do not emit a partial site):** a note missing `title` or `date`; a duplicate slug; a template placeholder left unreplaced; the disclaimer absent from any emitted HTML file.

That last one deserves emphasis: **the build enforces company law #3 rather than trusting anyone to remember it.**

---

## 5. Templating contract

Deliberately primitive: `{{name}}` string substitution. No expressions, no loops in templates — loops (the notes list, the feed) are built in JS and injected as a single `{{...}}` value.

`base.html` slots:

| Placeholder | Contents | Escaped? |
|---|---|---|
| `{{title}}` | `<title>` and `og:title` | **yes** |
| `{{description}}` | meta description and `og:description` | **yes** |
| `{{canonical}}` | absolute canonical URL | **yes** |
| `{{og_type}}` | `website` or `article` | **yes** |
| `{{head_extra}}` | per-page `<head>` additions | no — trusted, authored by us |
| `{{content}}` | the page body | no — already HTML from `marked` |

**Escaping rule:** every placeholder is HTML-escaped except `{{content}}` and `{{head_extra}}`. A note titled `Bitcoin & <basis>` must not be able to break the page. The build fails if any `{{` survives into output.

---

## 6. Content contract

```markdown
---
title: What Twelve Words is for
date: 2026-09-01
summary: One sentence, used on the notes index and in the feed.
draft: false
---

Body in markdown.
```

- `title`, `date` (ISO `YYYY-MM-DD`), `summary` are **required** for notes. `draft` is optional, default false.
- Front-matter is a `---` fenced block at byte 0. Keys are `key: value`, split on the **first** colon; surrounding quotes stripped. No nesting, no multiline, no arrays — if a value needs more than that, the spec is wrong and gets amended rather than worked around.
- Slug comes from the filename with the `YYYY-MM-DD-` prefix stripped. `2026-09-01-one-key.md` → `/notes/one-key`.
- Pages under `content/pages/` need `title` and `description`; they have no date and never appear in the feed.

---

## 7. URL map

| URL | Source | Output |
|---|---|---|
| `/` | `templates/landing.html` | `dist/index.html` |
| `/about` | `content/pages/about.md` | `dist/about.html` |
| `/notes` | generated index | `dist/notes/index.html` |
| `/notes/<slug>` | `content/notes/*.md` | `dist/notes/<slug>.html` |
| `/feed.xml` | generated | `dist/feed.xml` |
| `/sitemap.xml` | generated | `dist/sitemap.xml` |
| `/robots.txt` | `static/robots.txt` | `dist/robots.txt` |
| `/.well-known/nostr.json` | `static/` | unchanged, CORS header preserved |

`vercel.json` sets `"cleanUrls": true` and `"trailingSlash": false` so `/about` serves `about.html` without the extension.

Nav gains `Notes` and `About`. At ≤560px the nav links already collapse and the CTA takes the right edge — with five links that collapse now hides real navigation, so **the mobile nav needs a design decision** (§13, open question 5).

---

## 8. Assets

**Font.** `assets/inter-tw.woff2`, 38,452 bytes, the existing verified subset (Inter variable, opsz 14–32 + wght 100–900, Latin + `₿` U+20BF + punctuation and arrows). In `base.html`:

```html
<link rel="preload" href="/assets/inter-tw.woff2" as="font" type="font/woff2" crossorigin>
```

`crossorigin` is required even same-origin — fonts are fetched in CORS mode, and omitting it causes a double fetch. `font-display` changes from `block` to `swap`: with preload the font almost always arrives before first paint, and `swap` guarantees text is never invisible. If a flash of fallback is observed on the 88px hero in practice, the fallback position is `optional` — recorded here so the tradeoff isn't re-litigated from scratch.

**CSS.** `assets/site.css` begins as a byte-verbatim extraction of today's `<style>` contents, then gains the new components in §10. Referenced with `<link rel="stylesheet">`.

**Caching.** `vercel.json` sets `Cache-Control: public, max-age=31536000, immutable` on `/assets/*`. Consequence to respect: **any change to a file under `/assets/` requires a filename change**, or cached clients keep the old bytes for a year.

**Page weight after this change:** landing HTML drops from 69,708 bytes to roughly 12–14KB, plus a 6KB stylesheet and a 38KB font that are fetched once and reused across every page. Budget from the design seat's runbook (≤80KB target, 120KB cap) is measured **per page including assets on first load** and is comfortably met.

---

## 9. Security and privacy headers

Added to `vercel.json`, applied to all routes. The site has zero JavaScript and zero third-party resources, so the policy can be near-maximal and nothing will ever break against it.

```
Content-Security-Policy: default-src 'none'; style-src 'self'; font-src 'self'; img-src 'self' data:; base-uri 'none'; form-action 'none'; frame-ancestors 'none'
Referrer-Policy: no-referrer
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Permissions-Policy: accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()
Strict-Transport-Security: max-age=63072000; includeSubDomains
```

Notes: `default-src 'none'` covers `script-src`, so no JavaScript can execute even if injected. `img-src data:` is required by the inline SVG favicon. `X-Frame-Options` is redundant against `frame-ancestors` but costs nothing for older clients.

**`preload` is deliberately omitted from HSTS.** Submitting to the browser preload list is effectively irreversible and would bind every current and future `*.twelvewords.xyz` subdomain to HTTPS-only forever. That is a one-way door and needs explicit sign-off (§13, open question 1).

The existing CORS header on `/.well-known/nostr.json` is preserved exactly.

---

## 10. New page designs

Orders 1–3 change nothing visually. This section is what Order 4 introduces, and it is **designed here, not by the implementer** — the Workhorse builds this markup exactly; it does not make visual choices.

**Proposed new token.** The existing ink scale was tuned for short marketing copy. Extended reading at `--ink-2` (.62) is dimmer than it should be over 800 words. Proposal: `--ink-read: rgba(255,255,255,.78)` (≈11.5:1 on `#000`), used **only** for long-form prose bodies. This is a token addition and therefore a TW-series decision (§13, open question 2).

**Note page** (`/notes/<slug>`)
- Reading column `max-width: 680px` (~70 characters), centered, `padding: 0 24px`.
- Header: date as a kicker (13px, `.06em`, uppercase, `--ink-3`) → `h1` at `clamp(32px, 5vw, 52px)`, weight 640, `-.035em`. Smaller than the hero — the landing keeps the biggest type on the site.
- Prose: 17px / 1.75, `--ink-read`, paragraph spacing 1.15em.
- Prose `h2` 24px/620/-.02em with 2em top margin; `h3` 19px/600.
- Prose links: `--ink` with `border-bottom: 1px solid rgba(255,255,255,.28)`, brightening to `--ink-2` border on hover. No accent colors — coral and gold stay in product context per TW-3.
- `blockquote`: 3px left hairline, `--ink-2`, no italics.
- `code` / `pre`: `#101010` ground, hairline border, 14px, system monospace stack (no additional webfont).
- Footer of the article: hairline, then a single `← All notes` link. No share buttons, no related-posts, no newsletter interstitial.

**Notes index** (`/notes`)
- Standard section header: kicker `WRITING` → `h2` "Notes." (terminal period, house signature).
- One row per note: date (`--ink-3`, 13px, tabular) → title (19px, 600) → summary (15px, `--ink-2`). Hairline between rows, matching the FAQ divider treatment.
- Whole row is the anchor; hover lifts the divider to `--line-2`, same interaction language as the product cards.
- A single line under the header: `Subscribe by RSS` linking `/feed.xml`.
- Empty state, which is what ships first: "Nothing published yet." in `--ink-2`. Honest and consistent with never marketing unshipped work.

**About page** (`/about`) — the standard prose page: `h1`, then markdown body in the reading column. Content is written separately by the comms seat; this spec only defines its rendering.

---

## 11. Verification gates

Machine-checkable throughout. No gate requires taste; anything that would is routed to the design seat instead.

| Gate | Command | Pass condition |
|---|---|---|
| **G1 byte identity** | `cmp dist/index.html index.html` | identical (Orders 1 only) |
| **G2 pixel identity** | render `dist/index.html` at 1440×900 and 390×844 (deviceScaleFactor 2), diff against the pre-change render | **zero** differing pixels (Orders 2–3) |
| **G3 disclaimer law** | `grep -L "educational and informational purposes only" dist/**/*.html` | empty output — every emitted page carries it |
| **G4 no external requests** | `grep -oE 'https?://[^"< ]+' dist/**/*.html` | only anchor `href`s and the SVG namespace; no `src=`, `@import`, or `url(http…)` |
| **G5 no JS, no storage** | `grep -ci "<script\|localStorage\|sessionStorage\|indexedDB" dist/**/*.html` | 0 |
| **G6 no unreplaced slots** | `grep -r "{{" dist/` | no matches |
| **G7 determinism** | build twice into `dist-a`/`dist-b`, `diff -r` | identical |
| **G8 feed validity** | parse `dist/feed.xml` as XML; assert required RSS elements | valid, item count matches note count |
| **G9 font integrity** | fontTools: `fvar` axes present, `U+20BF` in cmap | ₿ renders (this has broken elsewhere and is why it's a gate) |
| **G10 page weight** | sum of `index.html` + `site.css` + font | ≤ 80KB |

G2 belongs to me: I render and diff in the cloud session and read the shots. A non-zero pixel diff fails the order — no negotiation, no "looks basically the same."

---

## 12. Build order sequence

Each order is one concern and reviewable in minutes, per the sizing law.

**TW-B1 — Foundation, zero visual change.** Add `package.json` (`marked` only), `.gitignore`, a build script that so far only copies files, `vercel.json` build config + security headers + asset caching, `static/robots.txt`. `index.html` still hand-authored and untouched. Gates: G1, G3, G4, G5, plus headers present in `vercel.json`.

**TW-B2 — Extract assets.** Font → `assets/inter-tw.woff2` with preload; `<style>` → `assets/site.css`; `font-display` `block`→`swap`. Gates: G2, G4, G9, G10.

**TW-B3 — Templating.** `templates/base.html` carrying head/nav/footer/disclaimer; landing body → `templates/landing.html`; landing rendered by the build; root `index.html` deleted. Gates: G2, G3, G6, G7.

**TW-B4 — Content pipeline.** `marked` wired in, front-matter parser, `content/pages/about.md`, `/notes` index (empty state), note rendering, `feed.xml`, `sitemap.xml`, plus the §10 CSS components. Gates: G3, G5, G6, G7, G8, and a design pass from me on the new surfaces before merge.

Order 4 is the only one introducing new visual surfaces, so it is also the only one requiring the Critic→Polish pass. Orders 1–3 are refactors whose entire acceptance criterion is *nothing changed*.

---

## 13. Open questions — Founder decisions this spec cannot make

1. **HSTS `preload`.** One-way door affecting every `*.twelvewords.xyz` subdomain permanently. Spec ships without it. Include?
2. **`--ink-read` token addition** (§10). Token changes are TW-series decisions per the design charter. Approve, or keep long-form prose at `--ink-2`?
3. **`robots.txt` stance on AI crawlers.** Spec allows all crawlers. Given a build-in-public posture that wants reach, allowing seems right — but it is a choice, not a default.
4. **Note URL style.** `/notes/<slug>` (spec's choice — short, stable, ages well) vs `/notes/2026/09/<slug>` (dated, conventional for blogs, uglier). Changing later breaks links.
5. **Mobile nav.** With `Apps · Notes · About · FAQ` the current ≤560px rule hides all links behind nothing. Options: (a) horizontally scrollable link row, (b) drop to two links on mobile, (c) a real menu — which would require the site's first JavaScript and therefore a TW decision. Spec assumes (a) pending a call.
6. **Substack's role now.** If writing lives here, is Substack a mirror (with canonical pointing home), or retired? Comms decision, TW-6 territory — flagged, not decided here.

---

## 14. Non-goals

No client JavaScript, ever, without an explicit TW decision. No analytics, no tracking pixels, no storage APIs. No comments. No search (revisit past ~20 notes). No tags or categories initially. No pagination until >20 notes. No per-note og images. No light theme — the brand is black. No email capture on the site; the feed is the subscription mechanism.
