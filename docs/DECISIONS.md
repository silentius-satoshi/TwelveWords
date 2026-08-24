# Decisions

Append-only, numbered, dated. **An undocumented decision doesn't exist.** Never edit a
prior entry except to add a dated "superseded by" line — reversals are new entries, not
deletions. The log is the memory of *why*, and a scrubbed log is worthless.

Scope: this file records decisions about **the apex site and the brand system**. Company
strategy — revenue, pricing, publishing cadence, the product calendar, key material — is
decided by the Founder and recorded privately. Site decisions here carry the `TW-` numbers
from that wider log, so the numbering has gaps. That is expected, not an error.

---

## TW-3 — Apex landing v1: brand decisions (2026-07-19)

The landing shipped as a single self-contained `index.html`: zero JS, zero external
fetches, no analytics or storage. Locked by this ship:

- **Thesis line:** "One key. Every ledger." Fresh copy — products keep their own headlines
  and the apex never borrows them.
- **The mark:** a 12-dot grid (4×3 round dots), twelve dots for twelve words. Nav, footer,
  favicon.
- **Type:** Inter *variable* (opsz + wght), subset to Latin plus `₿` U+20BF, punctuation
  and arrows, base64-embedded — 38,452-byte woff2 from `inter-ui@4.1.1` via pyftsubset.
- **Tokens:** the full set now in `BRAND_SYSTEM.md`.
- **Structure:** sticky blur nav → hero → "Two ledgers, so far." product cards with
  Live/Preview badges → "How the suite works" → 5-question FAQ (native `<details>`,
  plus→minus) → four-column footer + verbatim disclaimer + baseline line.

Screenshot-verified at 1440 full page, 390 mobile, FAQ open and closed, footer and icon
closeups, and the ₿ glyph.

## TW-5 — D1 contrast fix and cross-surface review (2026-07-19)

The footer disclaimer rendered at 12px in `--ink-3` — 3.4:1, below the 4.5:1 floor for
body-size legal text. Changed to `rgba(255,255,255,.5)` (5.28:1 computed, screenshot
re-verified). Registered and closed as debt D1.

Reviewed the apex against the live BitBooks landing: the two share the suite grid
faithfully — same ground, card, and hairline tokens; blur nav with a single white pill CTA;
Apple-register hero scale; `{Live, Preview}` badge vocabulary; native-details FAQ;
four-column footer with adapted verbatim disclaimers. Intentional differences confirmed
by design: product landings may center-stage a live demo, the apex stays artifact-free.
Conventions recorded in `BRAND_SYSTEM.md` §9.

**Codified caution:** `bitbooks-design-preview.html` is a superseded v2-era design (IBM
Plex Mono, blue-tinted ground, a hype strip) and must never be used as a design reference.

## TW-7 — Build execution model (2026-08-24)

Repo work is specified by a Brain (Claude, in chartered seats: decides, specs, reviews,
owns judgment-gated quality) and executed by a Workhorse (an agent running on the
Founder's machine). **The Founder is the only party who merges to `main`.**

Standing fences, which are now the hard rules in `CLAUDE.md`: never `main`; push the work
branch so nothing lives only in a local clone; never force-push or rewrite history; never
touch files outside the task's scope; **never edit a test to make it pass**; never add a
dependency the task didn't name; never touch secrets or `.well-known/`; stop and ask on
ambiguity rather than guessing.

**Judgment-gated quality never routes to an executor** — UI quality, the
screenshot-verification law, brand conformance, and published copy stay with the Founder
and the design seat. A model cannot self-assess whether a screen looks right.

*Superseded in part, 2026-08-24:* with Claude Code running natively in the repo, the
Brain/Workhorse split collapses for this repo — one agent both specs and executes. The
fences and the merge boundary survive unchanged; they are what actually mattered.

## TW-8 — The apex hosts the writing (2026-08-24)

Long-form writing lives on twelvewords.xyz rather than on a hosted newsletter platform.
Rationale: publishing to rented land before owning an archive means migrating later, and
platform dependency is a named company risk. The site therefore grows from one landing
page into a small static site with `/about`, `/notes`, and a full-text RSS feed.

Architecture decisions A1–A8, the build pipeline, the templating and content contracts,
the URL map, security headers, and the designs for the new page types: `ARCHITECTURE.md`.
Headlines: own the ~150-line build script rather than adopt a framework; exactly one
dependency (`marked`, verified zero transitive deps); markdown as the authoring format;
the landing renders through the same template as every other page, gated on pixel identity;
CSS and font become same-origin files so the CSP can drop `unsafe-inline` and assets cache
across pages; builds are deterministic.

*Status:* specced, **not yet implemented.** Open questions in `ROADMAP.md`.

## TW-9 — Repo becomes the operating context (2026-08-24)

The governing documents move into the repo — `CLAUDE.md` (auto-loaded, kept short),
`AGENTS.md` (identical, for tools that read that name), and `docs/` — so any agent working
here is briefed from the repo rather than from a chat history that does not travel.

*Boundary:* the repo is the source of truth for the **site** and the **brand system**.
Company strategy stays private with the Founder. If a task appears to need a company-level
decision, stop and ask.

*Kill criterion:* if these docs drift from what is shipped and are not corrected in the
same session the drift is noticed, they are worse than nothing — delete and start over
rather than trusting a stale map.
