# Verification

No UI change is done until this passes. An unread screenshot is theatre, and "it probably
renders" is not verification.

## 1. Setup (once per machine)

```bash
npm install -D playwright
npx playwright install chromium
npm install -D fonttools 2>/dev/null || pip install fonttools brotli
```

## 2. The gates

| Gate | Command | Pass condition |
|---|---|---|
| **G1** byte identity | `cmp dist/index.html index.html` | identical — refactors that must change nothing |
| **G2** pixel identity | render before/after at 1440×900 and 390×844, diff | **zero** differing pixels |
| **G3** disclaimer law | `grep -L "educational and informational purposes only" dist/**/*.html` | empty — every page carries it |
| **G4** no external requests | `grep -oE 'https?://[^"< ]+' dist/**/*.html \| sort -u` | only anchor `href`s and the SVG namespace — no `src=`, `@import`, or `url(http…)` |
| **G5** no JS, no storage | `grep -ci "<script\|localStorage\|sessionStorage\|indexedDB" dist/**/*.html` | `0` |
| **G6** no unreplaced slots | `grep -r "{{" dist/` | no matches |
| **G7** determinism | build twice, `diff -r dist-a dist-b` | identical |
| **G8** feed validity | parse `dist/feed.xml` as XML | valid; item count matches note count |
| **G9** font integrity | fontTools: `fvar` axes present, `U+20BF` in cmap | ₿ renders |
| **G10** page weight | `index.html` + `site.css` + font | ≤ 80KB |

Gates G1, G6–G8 only apply once the build exists (see `ARCHITECTURE.md`).

## 3. Screenshot protocol

```js
const { chromium } = require('playwright');
const browser = await chromium.launch();
const page = await browser.newPage({
  viewport: { width: 1440, height: 900 }, deviceScaleFactor: 2
});
page.on('pageerror', e => { throw e; });   // any page error fails the run
await page.goto('file://' + process.cwd() + '/index.html');
```

**Traps, each of which has bitten a real session:**

- `html { scroll-behavior: smooth }` animates programmatic scrolls — scroll with
  `behavior: 'instant'` or shots catch mid-scroll blur.
- Open `<details>` via `el.open = true` inside `page.evaluate`, not by clicking — clicks
  race the 180ms icon transition. Then wait ~250ms.
- Always `deviceScaleFactor: 2`. At 1×, hairline borders and glyph defects are invisible.
- Full-page mobile screenshots can exceed 8000px and fail some uploaders — capture the
  hero plus section crops for sharing, and read the full-page shot locally.
- For detail work, crop and upscale 2× rather than squinting at a full-page thumbnail.

## 4. Shot matrix — render **and read** every one

| Shot | Viewport | Checking |
|---|---|---|
| Desktop full page | 1440×900, fullPage | rhythm, section borders, no stray selectors |
| Desktop hero | 1440×900 | headline tracking and line breaks, lede measure, CTA pair, nav |
| FAQ open (≥2 items) | 1440 | plus→minus flip, answer padding, divider integrity |
| Footer closeup | 1440, scrolled | columns, disclaimer verbatim, baseline row |
| Icon row zoom | crop + 2× | each glyph legible at size, tiles aligned |
| Mobile full | 390×844, fullPage | stack order, nothing crushed or orphaned |
| Mobile nav/hero | 390×844 | CTA at the right edge once links hide, clamp sizes sane |
| ₿ check | any card shot | `U+20BF` renders in "Personal ₿LOC" — not tofu |

Never fewer than: desktop full, mobile full, every interactive state, one type-detail crop.
Read each before moving on — a fix can break a different shot, which is exactly how the
`.how div` selector bug happened.

## 5. Font subset recipe (reproducible)

```bash
npm pack inter-ui@4.1.1 && tar xzf inter-ui-*.tgz
pyftsubset package/variable/InterVariable.woff2 \
  --unicodes="U+0020-007E,U+00A0,U+00A9,U+00B7,U+2010-2015,U+2018-201D,U+2022,U+2026,U+2192,U+20BF" \
  --layout-features="kern,liga,calt,cv11" \
  --flavor=woff2 --output-file=inter-tw.woff2
```

Expect ~38,452 bytes. `@font-face` uses `font-weight: 100 900`. After any font change,
verify with fontTools that `fvar` shows opsz 14–32 and wght 100–900 and that the cmap
contains `U+20BF` and `U+2192` — a subset that silently drops ₿ turns the flagship
product's name into "Personal BLOC".

## 6. Before you say it's done

- [ ] Weight within budget; zero fetched resources; zero JS; zero storage APIs
- [ ] Font axes and ₿ verified if the font was touched
- [ ] Accessibility walk per `BRAND_SYSTEM.md` §7; no unregistered sev-A debt
- [ ] `pageerror` clean on every load
- [ ] Full shot matrix rendered **and read**; every claim points at a shot
- [ ] Disclaimer verbatim; "Preview" labelling intact; no hype introduced by layout
- [ ] Every color, radius, and size traces to `BRAND_SYSTEM.md` — no magic values
- [ ] Committed on a `build/*` branch; open items named
