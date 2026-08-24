# Roadmap

Where the site is, what's next, and what needs the Founder. Update this in the same
session anything here changes — a stale roadmap is worse than none.

## Right now

| | State |
|---|---|
| Landing page | Built, screenshot-verified, committed. Design locked (TW-3). |
| Deployment | **Not live.** `twelvewords.xyz` does not resolve. Vercel and DNS pending. |
| Site restructure | Specced in `ARCHITECTURE.md` (TW-8). Not started. |
| Brand npub | Does not exist. Footer ostrich links `#`; `nostr.json` is `{"names":{}}`. |
| Writing | Nothing published. |

## Blocking the Founder (nobody else can do these)

1. **Deploy.** Create the Vercel project from this repo, add `twelvewords.xyz` and `www`
   under Domains, then add the two records at Cloudflare — **grey-cloud / DNS-only**, not
   proxied. Until this is done, nothing else about the site is externally verifiable.
2. **Answer the open questions** below — two of them get expensive to change later.
3. **The npub ceremony**, when it's time: the nsec is generated on the Founder's machine,
   written on paper, and installed only into a signer. It never appears in a session, a
   transcript, a password manager, or a screenshot. Afterwards the **hex** pubkey (not the
   npub) goes into `.well-known/nostr.json` as `{"names":{"_":"<hex>"}}` and the footer
   ostrich gets its real destination.

## Build sequence (from ARCHITECTURE.md §12)

Each stage is one concern, reviewable in minutes. Stages 1–3 are refactors whose entire
acceptance criterion is **nothing changed visibly**.

- **TW-B1 — Foundation, zero visual change.** `package.json` (marked only), `.gitignore`,
  a copy-only build script, Vercel build config, security headers, asset caching,
  `robots.txt`. Gates: G1, G3, G4, G5, plus headers present.
- **TW-B2 — Extract assets.** Font and CSS become same-origin files; font preloaded;
  `font-display` `block` → `swap`. Gates: G2, G4, G9, G10.
- **TW-B3 — Templating.** `templates/base.html` holds head, nav, footer, disclaimer; the
  landing renders through it; root `index.html` is removed only after the build output is
  proven identical. Gates: G2, G3, G6, G7.
- **TW-B4 — Content pipeline.** `marked`, front-matter parsing, `/about`, `/notes` index,
  note pages, `feed.xml`, `sitemap.xml`, and the new CSS components. Gates: G3, G5, G6,
  G7, G8, plus a design pass on the new surfaces.

TW-B4 is the only stage introducing new visual surfaces, so it is the only one needing a
design pass. The others succeed by changing nothing.

## Open questions — Founder decisions, do not guess

1. **HSTS `preload`.** The spec ships `Strict-Transport-Security` *without* `preload`.
   Submitting to the browser preload list is effectively irreversible and binds every
   current and future `*.twelvewords.xyz` subdomain to HTTPS-only. One-way door. Include?
2. **Note URL style.** `/notes/<slug>` (spec's choice — short, stable, ages well) versus
   `/notes/2026/09/<slug>`. Cheap now, breaks links later.
3. **`--ink-read` token.** Extended prose at `--ink-2` (.62) is dimmer than it should be
   over 800 words. Proposal: `--ink-read: rgba(255,255,255,.78)`, used only for long-form
   bodies. Token additions are decisions, not implementation details.
4. **Mobile nav.** Below 560px the nav links currently hide entirely. That was fine with
   three links; with Apps, Notes, About, and FAQ it hides real navigation. Options: a
   horizontally scrollable link row (the spec's assumption), dropping to two links, or a
   real menu — which would require the site's first JavaScript and therefore a decision.
5. **`robots.txt` and AI crawlers.** The spec allows all crawlers. Given a build-in-public
   posture that wants reach, allowing seems right — but it is a choice.
6. **The newsletter platform's role** now that writing lives here: mirror with canonical
   pointing home, or retired?

## Not doing

No client JavaScript without a decision. No analytics, tracking pixels, or storage APIs.
No comments. No search until roughly twenty notes exist. No tags or categories initially.
No pagination until twenty notes. No per-note og images. No light theme — the brand is
black. No email capture; the feed is the subscription mechanism.

Adding to this repo is a decision with a founder-hour price, not a default. When in doubt,
the apex's job is legibility, and the answer is usually fewer elements.
