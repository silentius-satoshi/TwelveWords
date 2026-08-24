# Brand system

The shipped `index.html` is the reference implementation. **If this file and the shipped
page disagree, the page wins and this file gets corrected** in the same session.

## 1. Color tokens

| Token | Value | Use |
|---|---|---|
| `--bg` | `#000` | Page ground. Pure black, no tint. |
| `--card` | `#101010` | Cards, tiles. |
| `--card-2` | `#141414` | Reserved second elevation. |
| `--line` | `rgba(255,255,255,.07)` | Hairlines: borders, dividers, nav bottom. |
| `--line-2` | `rgba(255,255,255,.14)` | Hover borders, ghost-button borders. |
| `--ink` | `#f2f2f2` | Primary text. ~19:1 on black. |
| `--ink-2` | `rgba(255,255,255,.62)` | Secondary text — ledes, body, answers. ~7.9:1. |
| `--ink-3` | `rgba(255,255,255,.38)` | Tertiary — kickers, column heads, resting icons. ~3.4:1, restricted (see §7). |
| `--coral` | `#E8836A` | Personal ₿LOC. Product context ONLY. |
| `--gold` | `#e8b04f` | BitBooks. Product context ONLY. |

Accents appear exclusively inside the owning product's card, badge, or link. Never as
parent decoration, never mixed, never on a CTA. Parent CTAs are white pills (`#fff` on
black, `#000` text). No gradients, no shadows, no glows — elevation is surface value plus
hairline, nothing else. Green and red are reserved for money meaning inside product apps
and do not exist here.

## 2. Typography

Inter **variable** (opsz 14–32, wght 100–900), `font-optical-sizing: auto` — do not
disable it; the opsz axis picks display cuts at headline sizes. Variable weights (620,
640) are intentional; do not round to 600/700.

| Role | Size | Weight | Tracking | Notes |
|---|---|---|---|---|
| Hero H1 | `clamp(46px, 8.6vw, 88px)` | 640 | −.045em | line-height 1.02; break lines by meaning with `<br>` |
| Section H2 | `clamp(30px, 4.6vw, 44px)` | 620 | −.035em | line-height 1.08; terminal period is the house signature |
| Card H3 | 23px | 620 | −.025em | |
| Sub-card H3 | 18.5px | 600 | −.02em | |
| Lede | 18.5px | 420 | −.014em | `--ink-2`, max-width ~600px |
| Body | 15–15.5px | 400 | −.01em | line-height 1.6–1.7 |
| Kicker | 13px | 560 | +.06em uppercase | `--ink-3`; always paired with an H2 carrying the meaning |
| FAQ question | 17px | 540 | −.018em | |
| Nav / CTA | 14–15px | 480–560 | −.01em | |
| Legal | 12px | 400 | 0 | line-height 1.7, color `rgba(255,255,255,.5)` |

Tracking tightens as size grows. Positive tracking only on uppercase micro-labels. The
base `letter-spacing: -.011em` on body is deliberate — Inter set loose at text sizes reads
generic.

## 3. Space and layout

Container `max-width: 1080px`, `padding: 0 24px`. Sections `96px 0` desktop / `72px 0`
≤820px, each opened by a hairline top border — the page's only section separators. Hero
`180px 0 108px` (clears the 60px fixed nav) / `150px 0 84px` mobile. Card padding
`34px 32px 30px` (products), `30px 28px` (how). Grid gaps 16px; footer columns 40px (32 at
≤820px). Radii: cards 20px, icon tiles 12px, pills 980px, focus outline 6px.

Rhythm inside blocks: kicker→H2 14px, H2→sub 14px, sub→grid 44px, card-top→copy 14px,
copy→link 26px (link pinned bottom via `margin-top: auto`).

Breakpoints: ≤820px products and how-cards stack, footer to two columns. ≤560px nav links
hide and the CTA takes `margin-left: auto`, hero CTAs stack, footer to one column.

There is no hamburger menu by design — three anchors don't justify one. A future surface
needing real navigation is a decision, not a component to quietly add.

## 4. Components and their states

- **Nav** — fixed, 60px, `rgba(0,0,0,.62)` + `backdrop-filter: saturate(1.4) blur(18px)`
  (plus the `-webkit-` twin), hairline bottom. Links `--ink-2` → `--ink` on hover, 150ms.
  One CTA maximum.
- **White pill CTA** — `#fff`/`#000`, radius 980px, `9px 20px` (nav `7px 16px`, hero
  `12px 26px`), weight 560. Hover is `opacity: .85` and nothing else moves. Ghost variant:
  transparent with `--line-2` border → `rgba(255,255,255,.3)` on hover.
- **Product card** — `--card`, hairline, radius 20. The whole card is the anchor. Hover
  raises the border to `--line-2` and slides the link arrow 3px. Badge: 11.5px uppercase
  pill, accent text, 28%-alpha accent border, 10%-alpha accent fill. Exactly two badge
  words exist: **Live**, **Preview**. A third needs a decision.
- **How card** — number (13px, 560, `--ink-3`) → 18.5px H3 → 15px body. The CSS selector
  is `.how > div`; **the `>` is load-bearing** — a bare descendant selector styles the
  number chips as nested cards. This shipped as a bug once, which is why it's written down.
- **FAQ** — native `<details>`. Summary is the 17px question plus a two-bar plus icon
  (14×14, 2px bars, `--ink-3` → `--ink` on hover/open); `details[open]` scales the vertical
  bar to 0, 180ms. `list-style: none` plus `::-webkit-details-marker { display: none }`.
  No JS accordion, no auto-closing siblings — native semantics *are* the accessibility story.
- **Footer** — brand column (mark, wordmark, one-liner, max-width 280px) + link columns
  (heads 12.5px uppercase `--ink-3`; links 14.5px `--ink-2` → `--ink`) + Connect icon row
  (38px tiles, radius 12, hairline, icon and border brighten on hover). Below a hairline:
  the disclaimer, then the baseline row. No copyright line — pseudonymity posture.

## 5. Iconography and the mark

Inline SVG only. `aria-hidden="true"` on the SVG plus `aria-label` on the wrapping anchor.
`stroke-width` ~1.7 for line icons. `currentColor` always, so icons inherit ink states.

**The 12-dot mark:** 4 columns × 3 rows of round dots (viewBox 32; cx 4/12/20/28, cy
9/16.5/24, r 2.6), white on transparent; favicon variant on a `#000` rounded-7 square.
Twelve dots for twelve words — never rearrange, recount, or recolor the grid.

Wordmark: mark at 17px + "Twelve Words", 15.5px, weight 620, −.02em, 10px gap.

Current set: nostr ostrich (stroke), GitHub (fill), Substack (fill), mail (stroke). New
icons match the nearest neighbour's style and optical weight at 17–18px.

## 6. Surface rules

One self-contained HTML file per static surface today: inline CSS, base64 font, SVG
favicon as a data URI, zero JS, zero analytics, zero storage APIs, zero external requests
(outbound links are fine — inbound *resources* are not). `scroll-behavior: smooth` with a
reduced-motion override. Meta set: title, description, og:title/description/url/type,
theme-color `#000000`.

`ARCHITECTURE.md` changes some of this when the build lands — read it before restructuring.

## 7. Accessibility standard (WCAG 2.2 AA posture)

- **Contrast floors on black:** `--ink` 19:1 and `--ink-2` 7.9:1 — use freely. `--ink-3`
  is 3.4:1 and permitted **only** for text ≥18.5px, uppercase micro-labels that duplicate
  adjacent full-contrast content, or resting icon states with a full-contrast hover.
  Body-size load-bearing text needs ≥4.5:1; `rgba(255,255,255,.5)` (≈5.3:1) is the approved
  minimum step, and is what the footer disclaimer uses.
- **Focus:** `:focus-visible` outline `2px solid rgba(255,255,255,.5)`, offset 3px, on
  every interactive element including `<summary>`. Never `outline: none` without a visible
  replacement.
- **Semantics:** one `<h1>`; sections carry `<h2>`; `<nav aria-label>`; FAQ stays native
  `<details>/<summary>` (keyboard and screen readers work for free — do not replace with
  div-buttons); icon links get `aria-label`; decorative SVGs get `aria-hidden`.
- **Motion:** every transition sits inside a `prefers-reduced-motion: reduce` override.
- **Targets and reflow:** interactive targets ≥24×24 CSS px; the page reflows without
  horizontal scroll at 320px and at 200% zoom.

## 8. Debt register

Append-only. Severity: **A** fix on next touch · **B** fix with the next feature · **C** cosmetic.

| # | Sev | Item | Status |
|---|---|---|---|
| D1 | A | Disclaimer rendered at `--ink-3` (3.4:1), below the floor for body-size legal text. Fixed to `rgba(255,255,255,.5)` = 5.28:1. | **FIXED** |
| D2 | C | Footer ostrich reads "bird" more than "ostrich" at 18px. Revisit when the npub ships and the icon gains a real destination. | OPEN |
| D3 | B | No `og:image` — link previews show no card. Candidate: 1200×630 black card with the mark and wordmark. | OPEN |

## 9. Cross-surface consistency

Shared conventions the product landings also hold: matte black ground, `#101010` cards,
`.07`/`.14` hairlines; sticky blur nav with a single white pill CTA; Apple-register hero;
the `{Live, Preview}` badge vocabulary; native-`<details>` FAQ; four-column footer with a
Connect icon row; the disclaimer verbatim, adapted only in its opening words.

Intentional differences, not to be "fixed": product landings may center-stage a live demo;
the apex stays artifact-free because its job is legibility, not demonstration. Product
headlines belong to products ("Every sat, accounted for.") and the apex never borrows them.

⚠️ There exists an older `bitbooks-design-preview.html` from a superseded v2-era design
(IBM Plex Mono, blue-tinted `#0a0d12` ground, a "Free. Unlimited. Forever." strip). It is
**not** a reference. Never take cues from it.
