# Dream Ventures — Project Handoff Notes

> Read this first when resuming work in a fresh session. It captures the
> project structure, deploy flow, and every "house rule" we've established so
> far, so past decisions are never lost when the chat context fills up.
>
> This file is committed to the repo and **publicly reachable** at
> `https://dreamventuresindia.com/NOTES.md` — keep it free of anything
> sensitive (credentials, private contacts, etc.).

---

## 1. What this project is

A **static marketing website** for Dream Ventures, a B2B supply-chain,
logistics (3PL), trading/sourcing, and AI-operations services company.
No backend, no build step in the repo — the pages are plain HTML/CSS/JS.

- **Live:** https://dreamventuresindia.com
- **Repo:** https://github.com/libra33020/dream-ventures.git (branch `main`)
- **Hosting:** GitHub Pages, serving from repo root (`/`), custom domain
  `dreamventuresindia.com` (HTTPS enforced).

---

## 2. Deploy flow (how changes go live)

Every change is: **edit → commit → push `main` → GitHub Pages auto-builds**.

```powershell
git add -A
git commit -m "describe the change"
git push origin main
```

- Pages build & deploy automatically after each push to `main`.
- Check build status:
  ```powershell
  gh api repos/libra33020/dream-ventures/pages/builds/latest --jq '{status, commit}'
  ```
  Wait for `status: "built"` (it goes `building` → `built`; usually under a
  minute, but allow ~30s and poll if needed).
- Note: PowerShell prints `git push`'s stderr as a red "error" block. That is
  **normal** — a successful push still shows `old..new main -> main`. Don't be
  alarmed.
- Verify live with a cache-busting fetch:
  ```powershell
  Invoke-WebRequest "https://dreamventuresindia.com/index.html?v=$(Get-Random)" -UseBasicParsing
  ```

### Local preview

A background preview server usually runs at http://127.0.0.1:8080:

```powershell
python -m http.server 8080 --bind 127.0.0.1
```

Run it from the repo root. Pages are at `http://127.0.0.1:8080/<file>.html`.

---

## 3. Page / file map

### HTML pages (10 total, all in repo root)

| File | Purpose |
|---|---|
| `index.html` | Homepage |
| `about.html` | About / capabilities / "21 years" trust strip |
| `logistics-3pl.html` | 3PL & warehousing service |
| `supply-chain-solutions.html` | Supply-chain consulting / S&OP |
| `supply-chain-ai.html` | AI & control-tower product |
| `trading-sourcing.html` | Trading / sourcing / QC service |
| `insights.html` | Knowledge hub / articles |
| `partner.html` | Partner program |
| `contact.html` | Contact + consultation form |
| `global-trade.html` | Redirect stub → `supply-chain-solutions.html` |

### Static assets (repo root)

- `logo.png` (nav logo), `logo-full.png` (footer logo)
- `favicon.svg`, `favicon.png`, `favicon-32x32.png`
- `icons.svg`, `hero-video.mp4`, `hero-video-poster.jpg`, `scene.splinecode`
- `CNAME` (contains `dreamventuresindia.com`), `.nojekyll`

### Bundled assets (`assets/`)

Only **four** files are referenced directly by pages:

- `assets/main-CxuXYb2s.js`
- `assets/scroll-animate-BM37x40c.js`
- `assets/style-BqoKOdt1.css`  ← the CSS source of truth
- `assets/style-Dc5fqKfi.js`

Other `assets/*.js`/`assets/*.css` files are **code-split chunks** loaded at
runtime by the four above. Do **not** delete them unless you've verified (via
grep across the bundles) that nothing imports them.

---

## 4. Tech stack & constraints

- **Tailwind CSS**, but **prebuilt** — the CSS file is compiled already.
  → Only utility classes that actually appear in `assets/style-BqoKOdt1.css`
    will render. Arbitrary values (e.g. `bg-[#1E293B]`, `min-w-[2.75rem]`)
    are often **missing** from the prebuilt set, so **do not add new arbitrary
    classes** expecting them to work. Reuse classes already used elsewhere on
    the page instead.
- **GSAP**-style scroll animations via `scroll-animate` + `data-animate`
  attributes and `gsap-counter` elements.
- **Fonts:** Google Fonts (`inter`, `jakarta`, `fraunces`, `muted`, etc.).

---

## 5. Content & house rules (IMPORTANT)

These are decisions the owner has made. Do **not** reverse them silently.

### a) No unverified numeric claims
- Remove any **percentage / savings / performance / metric** figures
  (e.g. "99.5% SLA", "15–30% savings", "40% stockout reduction").
- **Keep the block and its wording** — just drop the number. Where a number
  sits in a card title or hero strip, replace the digit with the label
  (e.g. "OTIF Delivery SLA", "Forecast Accuracy") or a check-mark tile so the
  layout doesn't break.
- **The only verified figure allowed is "21" years of domain experience**
  (on the `about.html` bottom stats strip).
- **If a new numeric claim appears anywhere, confirm with the owner before
  changing or keeping it.** This rule is non-negotiable.

### b) No industry / sector disclosure
- The owner does **not** want the site to reveal **which industries** they
  work in. Remove references to specific verticals (automotive, FMCG, pharma,
  electronics, retail sectors, D2C, healthcare, etc.).
- The homepage "Target Sectors / Industries We Empower" block and the
  `about.html` "6 Key Focus Industry Ecosystems" block have been **deleted**.
- Generic terms are fine and should be used as replacements:
  - "industry intelligence" → "market intelligence"
  - "industry experts" → "domain experts"
  - "industrial corridors" → "freight corridors"
  - "industry problem" → "the challenge"
  - "FMCG networks" → "distribution networks"
  - "pharma & FMCG" → "temperature-sensitive cargo"
- **"manufacturer/manufacturing" is allowed** — it describes *who they source
  from* (suppliers), not which industry they serve.

### c) Other allowed content
- Structural/model terms are intentionally kept: `3PL`/`4PL`, `3D cubing`,
  `4-tier audits`, `5-Stage S&OP`, `B2B`, section numbering (`01`…`06`,
  `Pillar 02`, `Module 03`), article dates, `© 2026`.
- Equipment specs (e.g. `32ft/24ft/20ft`, `-20°C to +8°C`) are fine — they
  describe hardware, not results.

---

## 6. Editing checklist (when touching a shared element)

Three places repeat on **every** page and must stay in sync:

1. **Desktop nav** (top `<nav>`)
2. **Mobile drawer** (hamburger menu)
3. **Footer**

Any nav/footer change (e.g. adding/removing a page) must be applied to **all
10 HTML files**. Grep first:

```powershell
Select-String -Path *.html -Pattern 'your-term'
```

---

## 7. File-safety rules (encoding & line endings)

These bit us once — do not repeat the mistake.

- **Do NOT use PowerShell `Get-Content` / `Set-Content`** to edit these files.
  It corrupted UTF-8 (em dash `—` became `â€"`), added a BOM, and rewrote line
  endings.
- Use the **edit tool**, or **Python with byte-level writes** (`open(..., 'rb')`
  / `'wb'` + explicit `.decode/.encode('utf-8')`) for regex removals.
- Repo files are **CRLF** with `core.autocrlf=true`. Expect a harmless
  `warning: LF will be replaced by CRLF` on commit.
- Always **validate UTF-8** after edits (no BOM, no mojibake markers):
  ```powershell
  python -c "import glob; [print(f,'BOM' if open(f,'rb').read().startswith(b'\xef\xbb\xbf') else 'ok') for f in glob.glob('*.html')]"
  ```

---

## 8. Verification workflow

### HTML balance check (run after any structural edit)

```powershell
python -c "import html.parser,glob
class P(html.parser.HTMLParser):
    def __init__(s):
        super().__init__(); s.stack=[]; s.err=[]
        s.void={'area','base','br','col','embed','hr','img','input','link','meta','source','track','wbr'}
    def handle_starttag(s,t,a):
        if t not in s.void: s.stack.append(t)
    def handle_endtag(s,t):
        if t in s.void: return
        if s.stack and s.stack[-1]==t: s.stack.pop()
        elif t in s.stack:
            while s.stack and s.stack[-1]!=t: s.err.append('unclosed <%s>'%s.stack.pop())
            s.stack.pop()
        else: s.err.append('stray </%s>'%t)
for f in sorted(glob.glob('*.html')):
    p=P(); p.feed(open(f,encoding='utf-8').read())
    if p.stack or p.err: print('FAIL',f,p.stack[:3],p.err[:3])
print('done')"
```

### Headless Chrome screenshot (visual check)

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu --hide-scrollbars --force-prefers-reduced-motion --window-size=1440,1200 --virtual-time-budget=6000 --screenshot="<path>.png" "<url>"
```

- Use `--force-prefers-reduced-motion` so scroll animations don't hide content.
- Anchor (`#section`) and very-tall full-page screenshots are **unreliable**
  (scroll animations + `vh` units). Screenshot the viewport, not the full page.

---

## 9. How to resume a new session

Give the next session a one-line brief, e.g.:

> "Dream Ventures static marketing site, deployed via GitHub Pages from `main`
> (repo libra33020/dream-ventures, live dreamventuresindia.com). Read NOTES.md
> first."

`NOTES.md` (this file) is the source of truth for the rules above. The repo
itself is always current — any committed change is safe regardless of chat
state.

---

## 10. Recent commit history (landmarks)

```
a8b671d Remove Target Sectors block from homepage
5275399 Remove unreferenced logo variants
cfa941d Remove orphaned demo pages
afe6e54 Remove stale build copies and debug screenshots
abb6547 Remove Industries page; drop unverified numeric claims site-wide
8d91f3c Remove Next-Gen B2B Trade badge from hero
```

(History grows; use `git log --oneline` for the latest.)
