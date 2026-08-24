# CLAUDE.md — Twelve Words apex site

Operating brief for any AI agent working in this repo. Read this first, every session.
Kept short on purpose: everything here competes for attention with the actual work.
Depth lives in `docs/` — go read the one the task needs.

## What this repo is

The parent-brand site for **Twelve Words**, a studio building bitcoin-native
personal-finance apps. Deploys to **twelvewords.xyz** via Vercel on push to `main`.

Twelve words of one Recovery Key sign into every app in the suite. The products live
in their own repos on their own subdomains — **never build product features here**:

- Personal ₿LOC — shipped, `personal-bloc.twelvewords.xyz`, accent coral `#E8836A`
- BitBooks — preview, `bitbooks.twelvewords.xyz`, accent gold `#e8b04f`

Founder is pseudonymous (`silentius-satoshi`), works 10–15 hrs/week, $0 capital.
Founder-hours are the company's scarcest resource. Cost every proposal in hours first.

## Current state

- `index.html` — the landing page. Single self-contained file, 69,708 bytes, zero JS,
  zero external requests. Screenshot-verified. **Locked design** (see DECISIONS.md).
- `vercel.json` — CORS header on `/.well-known/nostr.json`.
- `.well-known/nostr.json` — `{"names":{}}`, placeholder until the brand npub exists.
- **Not yet deployed.** twelvewords.xyz does not resolve; Vercel and DNS are pending.
- Restructuring into a small static site is specced but **not started** — `docs/ROADMAP.md`.

## Read before you act

| Doing this | Read first |
|---|---|
| Anything at all | this file |
| Changing markup, CSS, colors, type | `docs/BRAND_SYSTEM.md` |
| Restructuring the site, adding pages, the build | `docs/ARCHITECTURE.md` |
| Wondering what to work on next | `docs/ROADMAP.md` |
| Wondering why something is the way it is | `docs/DECISIONS.md` |
| Before claiming any UI work is done | `docs/VERIFICATION.md` |
| Writing anything a visitor will read | `docs/COMPANY_LAW.md` |
| What the company is and is for | `docs/NORTH_STAR.md` |

## Hard rules

These are not preferences. Breaking one is a defect regardless of how good the result looks.

**Never push to `main`.** Work on `build/<short-name>`. The Founder merges. Never
force-push, rebase, amend published history, or `reset --hard` over uncommitted work.

**The disclaimer is verbatim.** Every public page carries the paragraph in
`docs/COMPANY_LAW.md` §3 exactly — not paraphrased, not shortened, not restyled away.
Once the build exists, this is enforced by a build failure. Until then it is on you.

**Zero external requests.** No CDNs, no Google Fonts, no analytics, no tracking pixels,
no third-party anything. Same-origin files are fine. If a change needs a network
resource, the change is wrong.

**Zero JavaScript**, unless a TW decision says otherwise. The site has none today. Adding
the first line of JS is an architecture decision, not an implementation detail — stop
and ask.

**No storage APIs.** No localStorage, sessionStorage, IndexedDB, cookies.

**Never market unshipped work as shipped.** BitBooks is "Preview" until it launches.
Dates only when a gate has actually been passed.

**Pseudonymity holds.** No real names, locations, employers, or biographical detail in
code, comments, commits, or copy. Strip EXIF from any image. Check screenshots for
usernames and file paths before committing them.

**Never add a dependency** without the Founder approving it. The site currently has
zero runtime dependencies. Every addition is a supply-chain decision.

**Never touch `.well-known/` or secrets** unless the task explicitly says to.

**Product accents stay in product context.** Coral and gold appear only inside their own
product's card, badge, or link. Parent-brand CTAs are neutral white.

**Stop and ask rather than guess.** A blocked task reported honestly is a success. A
guessed task that looks finished is a latent defect.

## Commands

There is no build step yet — `index.html` is served as-is. The build arrives in TW-B1
(`docs/ROADMAP.md`). Until then, to preview: open `index.html` in a browser.

Verification (screenshots, gates) is described in `docs/VERIFICATION.md` and is required
before any UI change is called done.

## What needs the Founder, not you

This repo governs **the site**. Company-level strategy — revenue, pricing, publishing
cadence, what the next product is, anything about the brand npub or key material — is
decided by the Founder and recorded outside this repo. If a task seems to require one of
those, stop and say so.

Also the Founder's alone: merging to `main`, Vercel and DNS configuration, generating or
handling the nsec, and answering the open questions in `docs/ROADMAP.md`.

## Definition of done

A change is done when: the hard rules above are unbroken, the relevant gates in
`docs/VERIFICATION.md` pass with output you actually looked at, screenshots have been
rendered **and read** at 1440 and 390 for anything visual, and the work is committed on a
`build/*` branch. "It probably renders" is not verification.
