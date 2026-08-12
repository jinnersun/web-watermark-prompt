# Landing Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a bilingual (EN/zh-CN) marketing landing page plus a rule-examples page for the Web Watermark Tool Chrome extension, deployed on the existing `web-watermark-prompt` GitHub Pages site.

**Architecture:** Five standalone HTML files at the repo root, each with inlined `<style>` and `<script>`, zero external dependencies, zero build step. Language switching uses real `<a href>` links (not JS) so both languages get indexed independently. Images are the only external resources.

**Tech Stack:** Hand-written HTML5 + CSS custom properties. No framework, no Tailwind, no bundler. GitHub Pages serves the repo `main` branch directly.

**Spec:** `docs/superpowers/specs/2026-08-12-landing-page-design.md`

## Global Constraints

- **Repo:** all work happens in `D:\item\chrome插件\web-watermark-prompt` (independent git repo, branch `main`, remote `origin`).
- **NEVER link to `djimnchdlbbedppeedlcmebbmehcloeb`** — that extension ID belongs to a competitor ("Web Custom Watermark"). Linking it sends traffic to a competitor.
- **The extension is NOT yet published.** No Chrome Web Store URL exists. Primary CTA must be a disabled `<button>`, never an `<a>` with a placeholder or competitor href.
- **No monetization copy anywhere**: no "Pro", "Premium", "Pricing", "Upgrade", "$", "Sponsor", "Donate", "Buy".
- **No FAQPage JSON-LD schema. No llms.txt.** Only `SoftwareApplication` + `Organization` JSON-LD. (Evidence: Ahrefs 2026-05 controlled study found no citation lift; Google's 2026-05-15 generative AI guide states no special schema is needed.)
- **Privacy claim must be exactly "1 permission"** (`storage` only). This depends on Task 0 verification.
- **Base URL:** `https://jinnersun.github.io/web-watermark-prompt/`
- **Color tokens** (match existing `privacy-policy.html`): primary `#4f46e5`, dark-mode primary `#818cf8`, text `#0f172a`, muted `#64748b`, border `#e2e8f0`, panel `#f8fafc`, bg `#ffffff`. Dark mode via `prefers-color-scheme`.
- **Font stack:** `-apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", Roboto, "Helvetica Neue", Arial, sans-serif`
- **Feedback URL:** `https://github.com/jinnersun/web-watermark-prompt/issues`
- **Repo URL:** `https://github.com/jinnersun/web-watermark-prompt`
- **Footer text:** `© 2026 jinnersun · MIT Licensed`
- **Commit style:** Conventional Commits, matching existing history (`docs(prompt): ...`, `chore: ...`, `feat: ...`).
- **Do NOT push** until Task 9. All commits stay local until the full site is ready.
- Shell is Windows PowerShell 5.1. Use `Copy-Item`, `Test-Path -LiteralPath`. Chain with `;` and `if ($?) { }`, never `&&`.

## File Structure

| File | Responsibility |
|---|---|
| `index.html` | English landing page. Canonical + x-default. Holds GSC meta placeholder. |
| `index.zh.html` | Chinese landing page. Same structure, translated copy. |
| `examples.html` | English rule examples (5 scenarios from `EXAMPLES.md`). |
| `examples.zh.html` | Chinese rule examples (5 scenarios from `EXAMPLES.zh_CN.md`). |
| `sitemap.xml` | Lists all 5 public URLs. |
| `googled3a11b0a36ad28b1.html` | GSC HTML-file verification token (moved from container dir). |
| `assets/*.png` | 3 screenshots + OG image + icon, copied from the extension repo. |
| `README.md` / `README.zh_CN.md` | Add landing page link. |
| `privacy-policy.html` | Already modified (permission count 2→1), uncommitted. Committed in Task 9. |

Each landing page is self-contained: no shared CSS file. This is deliberate — 5 files with some CSS duplication beats a shared stylesheet that adds an extra HTTP request to every page, given each page is served independently and the CSS is small.

---

## Task 0: Verify the "1 permission" claim before building anything

This is a **blocking prerequisite**. `docs/publish-guide.md:298` flags an unverified assumption: whether the toolbar badge works with only the `storage` permission, or whether Chrome strips `tab.url` and forces adding `activeTab`. The entire "only 1 permission" selling point — and the already-edited `privacy-policy.html` — depend on the answer.

**Files:**
- Read: `D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\src\background.js`
- Read: `D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\src\manifest.json`

- [ ] **Step 1: Confirm manifest declares only `storage`**

Run:
```powershell
Select-String -LiteralPath "D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\src\manifest.json" -Pattern "permissions" -Context 0,3
```

Expected: `"permissions": ["storage"]` with no `activeTab` and no `host_permissions`.

- [ ] **Step 2: Determine how badge text is set**

Read `background.js` and find the `chrome.action.setBadgeText` call. Identify where its input comes from.

Two possible designs:
- **Design A (no extra permission needed):** the content script computes the match in-page and sends it via `chrome.runtime.sendMessage`; background reads `sender.tab.id`. `sender.tab` is always available to the receiving background script and requires no permission.
- **Design B (needs `activeTab` or `tabs`):** background calls `chrome.tabs.query` / `chrome.tabs.onUpdated` and reads `tab.url` itself. `tab.url` **is stripped to `undefined`** without the `tabs` or `activeTab` permission.

- [ ] **Step 3: Record the finding and branch**

If **Design A** — the `storage`-only claim holds. The `privacy-policy.html` edit is correct. Proceed to Task 1 unchanged.

If **Design B** — STOP and report to the user. These must all change before continuing:
1. Revert the `privacy-policy.html` permission edit (restore the `activeTab` bullet).
2. Change every "1 permission" string in this plan to "2 permissions".
3. Rewrite FAQ #3 and the Privacy section copy.

Do not guess. If `background.js` is ambiguous, report the ambiguity rather than assuming.

- [ ] **Step 4: No commit**

This task produces a finding, not a code change.

---

## Task 1: Copy image assets into the repo

**Files:**
- Create: `assets/screenshot-main.png`
- Create: `assets/screenshot-scenario.png`
- Create: `assets/screenshot-smartcolor.png`
- Create: `assets/og-image.png`
- Create: `assets/icon-128.png`

**Interfaces:**
- Produces: the five asset paths referenced by every later HTML task.

- [ ] **Step 1: Create the assets directory**

```powershell
Test-Path -LiteralPath "D:\item\chrome插件\web-watermark-prompt"
New-Item -ItemType Directory -Force -Path "D:\item\chrome插件\web-watermark-prompt\assets" | Out-Null
```

- [ ] **Step 2: Copy the five files**

```powershell
$src = "D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\docs\store-assets"
$dst = "D:\item\chrome插件\web-watermark-prompt\assets"
Copy-Item -LiteralPath "$src\screenshots\01-main-panel.png"    -Destination "$dst\screenshot-main.png"
Copy-Item -LiteralPath "$src\screenshots\02-real-scenario.png" -Destination "$dst\screenshot-scenario.png"
Copy-Item -LiteralPath "$src\screenshots\04-smart-color.png"   -Destination "$dst\screenshot-smartcolor.png"
Copy-Item -LiteralPath "$src\promo-1400x560.png"               -Destination "$dst\og-image.png"
Copy-Item -LiteralPath "$src\icon-128.png"                     -Destination "$dst\icon-128.png"
```

- [ ] **Step 3: Verify all five exist with expected sizes**

```powershell
Get-ChildItem -LiteralPath "D:\item\chrome插件\web-watermark-prompt\assets" | Select-Object Name, Length
```

Expected: 5 files. Approximate sizes: `screenshot-main.png` 111KB, `screenshot-scenario.png` 79KB, `screenshot-smartcolor.png` 160KB, `og-image.png` 109KB, `icon-128.png` 6KB. A 0-byte file means the copy failed.

- [ ] **Step 4: Verify screenshot dimensions are 1280×800 and OG image is 1400×560**

```powershell
Add-Type -AssemblyName System.Drawing
Get-ChildItem -LiteralPath "D:\item\chrome插件\web-watermark-prompt\assets" -Filter *.png | ForEach-Object {
  $img = [System.Drawing.Image]::FromFile($_.FullName)
  "$($_.Name)  $($img.Width)x$($img.Height)"
  $img.Dispose()
}
```

Expected: three screenshots at `1280x800`, `og-image.png` at `1400x560`, `icon-128.png` at `128x128`.

- [ ] **Step 5: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add assets
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "chore(assets): add screenshots and OG image for landing page"
```

---

## Task 2: Build the English landing page

**Files:**
- Create: `index.html`

**Interfaces:**
- Consumes: `assets/*.png` from Task 1.
- Produces: the `.btn`, `.btn-primary`, `.btn-secondary`, `.card`, `.rule-table`, `.faq` class names and the `--primary`/`--text`/`--muted`/`--border`/`--panel`/`--bg` CSS variables reused verbatim by Tasks 3, 4, 5.

- [ ] **Step 1: Write the complete file**

Create `index.html` with exactly this content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Web Watermark Tool — Never Deploy to the Wrong Environment Again</title>
<meta name="description" content="A Chrome extension that watermarks web pages by domain, URL, IP, or cookie so you can tell production from test at a glance. Free, one permission, zero network requests." />
<link rel="canonical" href="https://jinnersun.github.io/web-watermark-prompt/" />
<link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/" />
<link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/index.zh.html" />
<link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/" />
<link rel="icon" type="image/png" href="./assets/icon-128.png" />

<!-- GSC meta-tag verification: paste the google-site-verification tag here after this page is live.
     File-based verification via googled3a11b0a36ad28b1.html is already active; this is a redundant backup. -->

<meta property="og:type" content="website" />
<meta property="og:title" content="Web Watermark Tool — Never Deploy to the Wrong Environment Again" />
<meta property="og:description" content="Watermark web pages by domain, URL, IP, or cookie. Tell production from test at a glance. Free, one permission, zero network requests." />
<meta property="og:url" content="https://jinnersun.github.io/web-watermark-prompt/" />
<meta property="og:image" content="https://jinnersun.github.io/web-watermark-prompt/assets/og-image.png" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Web Watermark Tool — Never Deploy to the Wrong Environment Again" />
<meta name="twitter:description" content="Watermark web pages by domain, URL, IP, or cookie. Free, one permission, zero network requests." />
<meta name="twitter:image" content="https://jinnersun.github.io/web-watermark-prompt/assets/og-image.png" />

<style>
  :root {
    --bg: #ffffff;
    --panel: #f8fafc;
    --text: #0f172a;
    --muted: #64748b;
    --border: #e2e8f0;
    --primary: #4f46e5;
    --code-bg: #f1f5f9;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #0f172a;
      --panel: #1e293b;
      --text: #f1f5f9;
      --muted: #94a3b8;
      --border: #334155;
      --primary: #818cf8;
      --code-bg: #1e293b;
    }
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
                 "Microsoft YaHei", Roboto, "Helvetica Neue", Arial, sans-serif;
    background: var(--bg);
    color: var(--text);
    line-height: 1.7;
    font-size: 16px;
  }
  .wrap { max-width: 880px; margin: 0 auto; padding: 0 20px; }
  a { color: var(--primary); }

  .topbar {
    display: flex; justify-content: flex-end; gap: 8px;
    padding: 16px 0;
  }
  .topbar a {
    padding: 4px 10px; border-radius: 999px;
    border: 1px solid var(--border); color: var(--muted);
    text-decoration: none; font-size: 13px;
  }
  .topbar a.active { background: var(--primary); color: #fff; border-color: var(--primary); }

  header.hero { text-align: center; padding: 24px 0 8px; }
  h1 { font-size: 34px; line-height: 1.3; margin: 0 0 12px; }
  .sub { font-size: 18px; color: var(--muted); margin: 0 auto 24px; max-width: 620px; }

  .cta-row { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; margin-bottom: 14px; }
  .btn {
    display: inline-block; padding: 12px 22px; border-radius: 8px;
    font-size: 15px; font-weight: 600; text-decoration: none;
    border: 1px solid transparent; cursor: pointer;
    font-family: inherit;
  }
  .btn-primary { background: var(--primary); color: #fff; }
  .btn-primary:disabled { background: var(--muted); cursor: not-allowed; opacity: .65; }
  .btn-secondary { background: transparent; color: var(--primary); border-color: var(--border); }

  .trust { font-size: 13px; color: var(--muted); margin-bottom: 28px; }

  .shot {
    width: 100%; height: auto; display: block;
    border: 1px solid var(--border); border-radius: 12px; margin: 8px 0 40px;
  }

  section { padding: 8px 0 32px; }
  h2 { font-size: 24px; margin: 32px 0 14px; }
  h3 { font-size: 17px; margin: 20px 0 6px; }

  .card {
    background: var(--panel); border: 1px solid var(--border);
    border-radius: 12px; padding: 18px 22px; margin: 14px 0;
  }

  .rule-table { width: 100%; border-collapse: collapse; font-size: 15px; margin: 14px 0; }
  .rule-table th, .rule-table td {
    text-align: left; padding: 10px 12px;
    border-bottom: 1px solid var(--border); vertical-align: top;
  }
  .rule-table th { color: var(--muted); font-weight: 600; font-size: 13px; text-transform: uppercase; }

  code {
    background: var(--code-bg); padding: 2px 6px; border-radius: 4px;
    font-family: "SF Mono", Monaco, Consolas, "Courier New", monospace;
    font-size: 14px;
  }

  .privacy { border-left: 3px solid var(--primary); }
  .privacy ul { margin: 8px 0 0; padding-left: 22px; }

  .faq h3 { margin-top: 22px; }
  .faq p { margin: 6px 0 0; }

  footer {
    margin-top: 48px; padding: 24px 0 40px;
    border-top: 1px solid var(--border);
    color: var(--muted); font-size: 13px; text-align: center;
  }
  footer a { margin: 0 8px; }

  @media (max-width: 600px) {
    h1 { font-size: 27px; }
    .sub { font-size: 16px; }
    .rule-table { font-size: 14px; }
    .rule-table th, .rule-table td { padding: 8px 6px; }
  }
</style>
</head>
<body>
<div class="wrap">

<nav class="topbar">
  <a href="./" class="active">English</a>
  <a href="./index.zh.html">中文</a>
</nav>

<header class="hero">
  <h1>Never Deploy to the Wrong Environment Again</h1>
  <p class="sub">Web Watermark Tool overlays a clear watermark on any page matching your rules — by domain, URL, IP, or cookie — so you always know whether you're on production or test.</p>
  <div class="cta-row">
    <button class="btn btn-primary" disabled aria-disabled="true">Coming to Chrome Web Store</button>
    <a class="btn btn-secondary" href="https://github.com/jinnersun/web-watermark-prompt">Get early access on GitHub</a>
  </div>
  <p class="trust">Free · No account · 1 permission · Zero network requests</p>
</header>

<img class="shot" src="./assets/screenshot-main.png" alt="Web Watermark Tool options page showing environment rules and live preview" width="1280" height="800" loading="eager" />

<section>
  <h2>The problem</h2>

  <div class="card">
    <h3>Test and production share a root domain</h3>
    <p><code>test.app.example.com</code> and <code>app.example.com</code> look nearly identical in a browser tab. One misplaced click runs your test script against live data.</p>
  </div>

  <div class="card">
    <h3>Internal tools accessed by raw IP over VPN</h3>
    <p>An admin panel at <code>https://192.0.2.5/</code> gives you no visual clue about what you're connected to. There's no domain name to read.</p>
  </div>

  <div class="card">
    <h3>Canary rollouts on an identical URL</h3>
    <p>The backend routes on a cookie like <code>deploy=canary</code>. Same host, same path, completely different environment — and nothing on screen says so.</p>
  </div>
</section>

<img class="shot" src="./assets/screenshot-scenario.png" alt="A production page with a red watermark and inset border warning" width="1280" height="800" loading="lazy" />

<section>
  <h2>Six rule types</h2>
  <p>A configuration fires when any of its rules match. When several configurations match at once, the most specific rule wins.</p>

  <table class="rule-table">
    <thead>
      <tr><th>Type</th><th>Matches</th><th>Example</th></tr>
    </thead>
    <tbody>
      <tr><td><code>host-exact</code></td><td>Hostname equals the value exactly</td><td><code>app.example.com</code></td></tr>
      <tr><td><code>host-suffix</code></td><td>Hostname and all its subdomains</td><td><code>example.com</code></td></tr>
      <tr><td><code>url-regex</code></td><td>Full URL against a regular expression</td><td><code>^https?://.*/admin(/.*)?$</code></td></tr>
      <tr><td><code>ip-exact</code></td><td>Hostname is exactly this IP</td><td><code>192.0.2.5</code></td></tr>
      <tr><td><code>ip-cidr</code></td><td>Hostname is an IPv4 in this subnet</td><td><code>10.0.0.0/8</code></td></tr>
      <tr><td><code>cookie</code></td><td>Cookie name, exact value, or contains</td><td><code>deploy=canary</code></td></tr>
    </tbody>
  </table>

  <p><a href="./examples.html">See five worked examples →</a></p>
</section>

<section>
  <h2>What makes it usable</h2>

  <div class="card">
    <h3>Smart contrast color</h3>
    <p>A watermark that disappears against some backgrounds is worse than none. This one inverts against whatever is behind it using CSS <code>mix-blend-mode</code>, so it stays readable on light pages, dark pages, and gradients alike.</p>
  </div>

  <div class="card">
    <h3>Immersive border</h3>
    <p>You stop noticing a watermark you see all day. A border around the viewport edge catches peripheral vision instead, which is why production deserves one.</p>
  </div>

  <div class="card">
    <h3>Fades while you work</h3>
    <p>A permanent overlay gets in the way of reading. The watermark dims during heavy typing and clicking, then comes back a couple of seconds after you stop.</p>
  </div>

  <div class="card">
    <h3>Toolbar badge</h3>
    <p>A short label such as <code>PROD</code> or <code>TEST</code> sits on the extension icon, so you can check the environment without even looking at the page.</p>
  </div>
</section>

<img class="shot" src="./assets/screenshot-smartcolor.png" alt="The same watermark staying legible over light, dark, and gradient backgrounds" width="1280" height="800" loading="lazy" />

<section>
  <h2>Privacy</h2>
  <div class="card privacy">
    <p>This extension requests <strong>one permission</strong>: <code>storage</code>.</p>
    <ul>
      <li>No network requests of any kind — no CDN, no analytics, no remote scripts</li>
      <li>No data collection, no usage tracking, no third-party services</li>
      <li>Your rules live in your own Chrome account via <code>chrome.storage.sync</code>, readable by nobody else</li>
      <li>Page content is never read or transmitted</li>
    </ul>
    <p><a href="./privacy-policy.html">Read the full privacy policy →</a></p>
  </div>
</section>

<section class="faq">
  <h2>Questions</h2>

  <h3>Is Web Watermark Tool free?</h3>
  <p>Yes — every feature is free, with no account, no sign-in, and no in-app purchases. There is no paid tier.</p>

  <h3>Can it tell environments apart when they share the same domain?</h3>
  <p>Yes. Use a <code>cookie</code> rule when the backend distinguishes environments by cookie, or a <code>url-regex</code> rule when they differ by path. Both work on hosts that are otherwise identical.</p>

  <h3>Does it collect any data?</h3>
  <p>No. The extension requests only the <code>storage</code> permission and makes zero network requests. Nothing about your browsing leaves your machine.</p>

  <h3>Can I mark environments I reach by IP address?</h3>
  <p>Yes. Use <code>ip-exact</code> for a single address such as <code>192.0.2.5</code>, or <code>ip-cidr</code> to cover a whole subnet such as <code>10.0.0.0/8</code>. This is the usual case for VPN-only internal tools.</p>

  <h3>How do I share configuration with my team?</h3>
  <p>Export your configurations to JSON and let teammates import the file. The same export also moves your setup between machines.</p>

  <h3>What languages does the extension support?</h3>
  <p>Five: English, Simplified Chinese, Traditional Chinese, Japanese, and Spanish. It follows your browser language and can be switched at any time.</p>
</section>

<footer>
  <a href="https://github.com/jinnersun/web-watermark-prompt">GitHub</a>
  <a href="./privacy-policy.html">Privacy</a>
  <a href="https://github.com/jinnersun/web-watermark-prompt/issues">Feedback</a>
  <a href="./index.zh.html">中文</a>
  <p>&copy; 2026 jinnersun &middot; MIT Licensed</p>
</footer>

</div>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "Web Watermark Tool",
  "description": "Chrome extension that overlays environment watermarks on web pages matched by domain, URL, IP, or cookie.",
  "applicationCategory": "BrowserApplication",
  "operatingSystem": "Chrome",
  "url": "https://jinnersun.github.io/web-watermark-prompt/",
  "inLanguage": ["en", "zh-CN", "zh-TW", "ja", "es"],
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "USD"
  },
  "featureList": [
    "Match pages by exact hostname",
    "Match pages by host suffix including subdomains",
    "Match pages by full URL regular expression",
    "Match pages by exact IP address",
    "Match pages by IPv4 CIDR subnet",
    "Match pages by cookie name and value",
    "Smart contrast watermark color",
    "Immersive viewport border",
    "Watermark fade during interaction",
    "Toolbar environment badge",
    "JSON import and export"
  ],
  "author": {
    "@type": "Person",
    "name": "jinnersun",
    "url": "https://github.com/jinnersun"
  }
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "jinnersun",
  "url": "https://jinnersun.github.io/web-watermark-prompt/",
  "sameAs": ["https://github.com/jinnersun"]
}
</script>

</body>
</html>
```

- [ ] **Step 2: Verify the HTML has no competitor ID and no monetization words**

```powershell
$f = "D:\item\chrome插件\web-watermark-prompt\index.html"
Select-String -LiteralPath $f -Pattern "djimnchdlbbedppeedlcmebbmehcloeb","chromewebstore","Pro\b","Premium","Pricing","Sponsor","Donate"
```

Expected: **no output at all.** Any match is a constraint violation that must be fixed before committing.

- [ ] **Step 3: Verify both JSON-LD blocks parse as valid JSON**

Create `D:\item\chrome插件\web-watermark-prompt\tmp-check-jsonld.py`:

```python
import io
import json
import re
import sys

path = sys.argv[1]
html = io.open(path, encoding="utf-8").read()
blocks = re.findall(
    r'<script type="application/ld\+json">(.*?)</script>', html, re.S
)
print("found %d JSON-LD block(s)" % len(blocks))
ok = True
for i, block in enumerate(blocks, 1):
    try:
        data = json.loads(block)
        print("block %d: OK  @type=%s" % (i, data.get("@type")))
    except ValueError as exc:
        ok = False
        print("block %d: INVALID JSON -> %s" % (i, exc))
if not ok:
    sys.exit(1)
```

Run:
```powershell
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-jsonld.py" "D:\item\chrome插件\web-watermark-prompt\index.html"
```

Expected:
```
found 2 JSON-LD block(s)
block 1: OK  @type=SoftwareApplication
block 2: OK  @type=Organization
```

- [ ] **Step 4: Verify exactly one `<h1>` and correct heading order**

```powershell
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\index.html" -Pattern "<h1|<h2|<h3" | ForEach-Object { $_.Line.Trim() }
```

Expected: exactly one `<h1>`, then `<h2>`/`<h3>` only. No `<h3>` appearing before the first `<h2>`.

- [ ] **Step 5: Open in a browser and check rendering**

```powershell
Start-Process "D:\item\chrome插件\web-watermark-prompt\index.html"
```

Confirm by eye:
- The primary button is visibly greyed out and not clickable
- All three screenshots load (no broken-image icons)
- The rule table is readable and does not overflow horizontally
- Narrow the window below 600px — layout stays readable, table does not break out
- Switch OS to dark mode — text stays legible, no black-on-black

- [ ] **Step 6: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add index.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "feat(site): add English landing page"
```

---

## Task 3: Build the Chinese landing page

**Files:**
- Create: `index.zh.html`

**Interfaces:**
- Consumes: the `<style>` block and all class names defined in Task 2's `index.html`.
- Produces: `index.zh.html`, linked from Task 2's nav/footer and from Tasks 4, 5, 6.

The structure, CSS, and element order are identical to `index.html`. Only the `<head>` metadata, the visible copy, and two JSON-LD fields differ. Everything specified below is the complete content — no invention required.

- [ ] **Step 1: Create the file skeleton by copying index.html**

```powershell
Copy-Item -LiteralPath "D:\item\chrome插件\web-watermark-prompt\index.html" -Destination "D:\item\chrome插件\web-watermark-prompt\index.zh.html"
```

Copy the `<style>` block **byte-for-byte unchanged**. Do not restyle, do not reorder, do not add rules.

- [ ] **Step 2: Replace the entire `<head>` metadata (everything above `<style>`)**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>网页水印工具 — 再也不会在生产环境上误操作</title>
<meta name="description" content="一个 Chrome 扩展，按域名、URL、IP、Cookie 给网页自动打环境水印，一眼分清生产与测试。完全免费，仅一个权限，零网络请求。" />
<link rel="canonical" href="https://jinnersun.github.io/web-watermark-prompt/index.zh.html" />
<link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/" />
<link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/index.zh.html" />
<link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/" />
<link rel="icon" type="image/png" href="./assets/icon-128.png" />

<meta property="og:type" content="website" />
<meta property="og:title" content="网页水印工具 — 再也不会在生产环境上误操作" />
<meta property="og:description" content="按域名、URL、IP、Cookie 给网页自动打环境水印，一眼分清生产与测试。完全免费，仅一个权限，零网络请求。" />
<meta property="og:url" content="https://jinnersun.github.io/web-watermark-prompt/index.zh.html" />
<meta property="og:image" content="https://jinnersun.github.io/web-watermark-prompt/assets/og-image.png" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="网页水印工具 — 再也不会在生产环境上误操作" />
<meta name="twitter:description" content="按域名、URL、IP、Cookie 给网页自动打环境水印。完全免费，仅一个权限，零网络请求。" />
<meta name="twitter:image" content="https://jinnersun.github.io/web-watermark-prompt/assets/og-image.png" />
```

Note: the GSC verification comment belongs only in `index.html`. Remove it from this file.

- [ ] **Step 3: Replace the entire `<body>` content down to (not including) the JSON-LD scripts**

```html
<body>
<div class="wrap">

<nav class="topbar">
  <a href="./">English</a>
  <a href="./index.zh.html" class="active">中文</a>
</nav>

<header class="hero">
  <h1>再也不会在生产环境上误操作</h1>
  <p class="sub">网页水印工具按你的规则给页面叠加醒目水印 —— 支持域名、URL、IP、Cookie 四类识别方式，让你随时知道自己在生产还是测试。</p>
  <div class="cta-row">
    <button class="btn btn-primary" disabled aria-disabled="true">即将上架 Chrome 应用商店</button>
    <a class="btn btn-secondary" href="https://github.com/jinnersun/web-watermark-prompt">先到 GitHub 抢先体验</a>
  </div>
  <p class="trust">完全免费 · 无需账号 · 仅 1 个权限 · 零网络请求</p>
</header>

<img class="shot" src="./assets/screenshot-main.png" alt="网页水印工具设置页，展示环境规则与实时预览" width="1280" height="800" loading="eager" />

<section>
  <h2>它解决什么问题</h2>

  <div class="card">
    <h3>测试环境和生产环境共用根域名</h3>
    <p><code>test.app.example.com</code> 和 <code>app.example.com</code> 在标签栏里几乎一模一样。一次点错，测试脚本就跑到了线上数据上。</p>
  </div>

  <div class="card">
    <h3>内网后台通过 VPN 用 IP 访问</h3>
    <p>地址栏里只有 <code>https://192.0.2.5/</code>，没有域名可读，完全没有任何提示告诉你连的是哪一套环境。</p>
  </div>

  <div class="card">
    <h3>灰度发布的地址和生产完全相同</h3>
    <p>后端靠 <code>deploy=canary</code> 这类 cookie 分流。同一个域名、同一个路径，环境却完全不同 —— 而屏幕上没有任何迹象。</p>
  </div>
</section>

<img class="shot" src="./assets/screenshot-scenario.png" alt="生产环境页面上的红色水印与沉浸式边框警示" width="1280" height="800" loading="lazy" />

<section>
  <h2>六种匹配规则</h2>
  <p>一个配置里任意一条规则命中即生效。多个配置同时命中时，越精确的规则优先。</p>

  <table class="rule-table">
    <thead>
      <tr><th>类型</th><th>匹配方式</th><th>示例</th></tr>
    </thead>
    <tbody>
      <tr><td><code>host-exact</code></td><td>hostname 与值完全相等</td><td><code>app.example.com</code></td></tr>
      <tr><td><code>host-suffix</code></td><td>该域名及其所有子域</td><td><code>example.com</code></td></tr>
      <tr><td><code>url-regex</code></td><td>用正则匹配完整 URL</td><td><code>^https?://.*/admin(/.*)?$</code></td></tr>
      <tr><td><code>ip-exact</code></td><td>hostname 恰好是这个 IP</td><td><code>192.0.2.5</code></td></tr>
      <tr><td><code>ip-cidr</code></td><td>hostname 是该网段内的 IPv4</td><td><code>10.0.0.0/8</code></td></tr>
      <tr><td><code>cookie</code></td><td>按 cookie 名、精确值或包含匹配</td><td><code>deploy=canary</code></td></tr>
    </tbody>
  </table>

  <p><a href="./examples.zh.html">查看五个完整示例 →</a></p>
</section>

<section>
  <h2>那些让它真正好用的细节</h2>

  <div class="card">
    <h3>智能对比色</h3>
    <p>在某些背景上看不见的水印，比没有水印更糟。它用 CSS <code>mix-blend-mode</code> 对背景反色，浅色页、深色页、渐变页上都保持清晰。</p>
  </div>

  <div class="card">
    <h3>沉浸式边框</h3>
    <p>天天看着的水印，你会逐渐视而不见。视口四周的边框走的是余光通道，所以生产环境值得开一个。</p>
  </div>

  <div class="card">
    <h3>操作时自动渐隐</h3>
    <p>一直挡在那儿的水印会影响阅读。密集打字和点击时它会自动变淡，停手约两秒后恢复。</p>
  </div>

  <div class="card">
    <h3>工具栏短标签</h3>
    <p><code>PROD</code>、<code>TEST</code> 这样的短标签显示在扩展图标上，不用看页面就能确认当前环境。</p>
  </div>
</section>

<img class="shot" src="./assets/screenshot-smartcolor.png" alt="同一个水印在浅色、深色、渐变背景上都保持清晰" width="1280" height="800" loading="lazy" />

<section>
  <h2>隐私</h2>
  <div class="card privacy">
    <p>本扩展只申请<strong>一个权限</strong>：<code>storage</code>。</p>
    <ul>
      <li>不发起任何网络请求 —— 不用 CDN、不接分析、不加载远程脚本</li>
      <li>不收集数据、不上报使用行为、不接入任何第三方服务</li>
      <li>规则存在你自己的 Chrome 账号里（<code>chrome.storage.sync</code>），他人无法读取</li>
      <li>从不读取、也从不上传页面内容</li>
    </ul>
    <p><a href="./privacy-policy.html">阅读完整隐私政策 →</a></p>
  </div>
</section>

<section class="faq">
  <h2>常见问题</h2>

  <h3>网页水印工具是免费的吗？</h3>
  <p>是，全部功能免费，无需账号、无需登录、没有任何内购。不存在付费版本。</p>

  <h3>两个环境用同一个域名，它还能区分吗？</h3>
  <p>能。后端靠 cookie 分流时用 <code>cookie</code> 规则；靠路径区分时用 <code>url-regex</code> 规则。两者都能在 hostname 完全相同的情况下工作。</p>

  <h3>它会收集我的数据吗？</h3>
  <p>不会。扩展只申请 <code>storage</code> 一个权限，且不发起任何网络请求。你的浏览信息不会离开本机。</p>

  <h3>用 IP 地址访问的环境能打水印吗？</h3>
  <p>能。单个地址用 <code>ip-exact</code>（如 <code>192.0.2.5</code>），整个网段用 <code>ip-cidr</code>（如 <code>10.0.0.0/8</code>）。这正是纯内网工具的典型场景。</p>

  <h3>怎么把配置共享给团队？</h3>
  <p>把配置导出成 JSON，同事导入这个文件即可。同一份导出也能把配置迁移到你的其他设备。</p>

  <h3>扩展支持哪些语言？</h3>
  <p>五种：简体中文、繁体中文、English、日本語、Español。默认跟随浏览器语言，也可以随时手动切换。</p>
</section>

<footer>
  <a href="https://github.com/jinnersun/web-watermark-prompt">GitHub</a>
  <a href="./privacy-policy.html">隐私政策</a>
  <a href="https://github.com/jinnersun/web-watermark-prompt/issues">反馈</a>
  <a href="./">English</a>
  <p>&copy; 2026 jinnersun &middot; MIT Licensed</p>
</footer>

</div>
```

- [ ] **Step 4: Adjust the two JSON-LD blocks**

Keep both blocks from `index.html`, changing exactly three values:

- `SoftwareApplication.name` → `"网页水印工具"`
- `SoftwareApplication.description` → `"Chrome 扩展，按域名、URL、IP、Cookie 匹配网页并叠加环境水印。"`
- `SoftwareApplication.url` and `Organization.url` → `"https://jinnersun.github.io/web-watermark-prompt/index.zh.html"`

Leave `featureList` in English — it describes capabilities for machine consumption and matching the English page keeps the entity consistent.

- [ ] **Step 5: Verify no competitor ID, no monetization words, valid JSON-LD**

```powershell
$f = "D:\item\chrome插件\web-watermark-prompt\index.zh.html"
Select-String -LiteralPath $f -Pattern "djimnchdlbbedppeedlcmebbmehcloeb","chromewebstore","付费","价格","赞助","捐赠","Pro\b"
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-jsonld.py" $f
```

Expected: no `Select-String` output; JSON-LD reports 2 valid blocks.

- [ ] **Step 6: Verify the file is UTF-8 and Chinese renders correctly**

```powershell
Start-Process "D:\item\chrome插件\web-watermark-prompt\index.zh.html"
```

Confirm Chinese text displays properly with no mojibake (garbled characters). If garbled, the file was written in the wrong encoding — rewrite as UTF-8.

- [ ] **Step 7: Verify language links work in both directions**

Click `English` on the Chinese page, then `中文` on the English page. Both must navigate correctly.

- [ ] **Step 8: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add index.zh.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "feat(site): add Chinese landing page"
```

---

## Task 4: Build the English examples page

**Files:**
- Create: `examples.html`
- Read: `EXAMPLES.md` (source content)

**Interfaces:**
- Consumes: Task 2's `<style>` block, plus a small addition for `<pre>` blocks.
- Produces: `examples.html`, the link target of `index.html`'s "See five worked examples →".

Content comes from `EXAMPLES.md` Examples 1–5. **Example 6 is excluded** — it documents AI fallback behavior on vague input and has no value for a human reader.

- [ ] **Step 1: Create the file**

Use the same `<!DOCTYPE>`, `<html lang="en">`, `<style>` block, `.topbar`, and `<footer>` as `index.html`. Append these CSS rules to the end of the `<style>` block so JSON is readable:

```css
  pre {
    background: var(--code-bg); border: 1px solid var(--border);
    border-radius: 8px; padding: 14px 16px; overflow-x: auto;
    font-family: "SF Mono", Monaco, Consolas, "Courier New", monospace;
    font-size: 13px; line-height: 1.5;
  }
  pre code { background: none; padding: 0; font-size: inherit; }
  blockquote {
    margin: 10px 0; padding: 10px 16px;
    border-left: 3px solid var(--border); color: var(--muted);
  }
  .why { font-size: 15px; }
  .back { display: inline-block; margin: 8px 0 24px; }
```

Head metadata:

```html
<title>Rule Examples — Web Watermark Tool</title>
<meta name="description" content="Five real-world watermark rule examples: same-domain environments, VPN internal IP, whole subnets, path-based admin routing, and cookie-driven canary deployments." />
<link rel="canonical" href="https://jinnersun.github.io/web-watermark-prompt/examples.html" />
<link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/examples.html" />
<link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/examples.zh.html" />
<link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/examples.html" />
<link rel="icon" type="image/png" href="./assets/icon-128.png" />
```

Nav must point at the Chinese examples page, not the Chinese landing page:

```html
<nav class="topbar">
  <a href="./examples.html" class="active">English</a>
  <a href="./examples.zh.html">中文</a>
</nav>
```

Body opening:

```html
<h1>Rule Examples</h1>
<p class="sub">Five real scenarios, each showing a plain-language description, the JSON configuration it maps to, and why that rule type is the right choice.</p>
<a class="back" href="./">← Back to overview</a>
```

Then one `<section>` per example. Use `<h2>` for each example title, `<blockquote>` for the user's description, `<pre><code>` for the JSON, and `<p class="why">` for the rationale. Take all five examples verbatim from `EXAMPLES.md` — the titles are:

1. `Same-domain multi-environment` (JSON: three configs — PROD red `#ef4444` with border, SIM amber `#f59e0b`, TEST green `#10b981`)
2. `VPN internal admin over IP` (JSON: ADMIN violet `#8b5cf6`, `ip-exact` `192.0.2.5`, border width 3)
3. `An entire internal subnet` (JSON: INT indigo `#6366f1`, `ip-cidr` `10.0.0.0/8`, rotation -25)
4. `Path-based admin routing on production` (JSON: ADM red `#dc2626`, `url-regex` `^https://app\\.example\\.com/admin(/.*)?$`, fontSize 28, border width 5)
5. `Cookie-driven canary environment` (JSON: CNRY orange `#f97316`, `cookie` `deploy=canary`)

Copy each JSON block exactly as it appears in `EXAMPLES.md`, including the Chinese `name` and `text` values — those are the real config values and changing them would misrepresent the tool.

Escape `<` and `&` if any appear in JSON strings. The regex in Example 4 contains no such characters, so a literal copy is safe.

Close with the same `<footer>` as `index.html`, but with the language link pointing to `./examples.zh.html`.

**No JSON-LD on this page** — it is supporting content, not the entity.

- [ ] **Step 2: Verify all five examples are present**

```powershell
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\examples.html" -Pattern "<h2" | ForEach-Object { $_.Line.Trim() }
```

Expected: exactly 5 `<h2>` lines. If 6, Example 6 was wrongly included — remove it.

- [ ] **Step 3: Verify the JSON blocks are valid**

Create `D:\item\chrome插件\web-watermark-prompt\tmp-check-examples.py`:

```python
import io
import json
import re
import sys

path = sys.argv[1]
html = io.open(path, encoding="utf-8").read()
blocks = re.findall(r"<pre><code>(.*?)</code></pre>", html, re.S)
print("found %d JSON block(s)" % len(blocks))
ok = True
for i, raw in enumerate(blocks, 1):
    text = raw.replace("&amp;", "&").replace("&lt;", "<").replace("&gt;", ">")
    try:
        data = json.loads(text)
        names = [c.get("shortLabel") for c in data]
        print("block %d: OK  %d config(s)  labels=%s" % (i, len(data), names))
    except ValueError as exc:
        ok = False
        print("block %d: INVALID JSON -> %s" % (i, exc))
if not ok:
    sys.exit(1)
```

Run:
```powershell
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-examples.py" "D:\item\chrome插件\web-watermark-prompt\examples.html"
```

Expected:
```
found 5 JSON block(s)
block 1: OK  3 config(s)  labels=['PROD', 'SIM', 'TEST']
block 2: OK  1 config(s)  labels=['ADMIN']
block 3: OK  1 config(s)  labels=['INT']
block 4: OK  1 config(s)  labels=['ADM']
block 5: OK  1 config(s)  labels=['CNRY']
```

This catches the most likely bug: a broken regex escape in Example 4 producing invalid JSON that a user would copy and fail to import.

- [ ] **Step 4: Check rendering**

```powershell
Start-Process "D:\item\chrome插件\web-watermark-prompt\examples.html"
```

Confirm: JSON blocks are monospaced and scroll horizontally rather than overflowing the page; the "Back to overview" link works.

- [ ] **Step 5: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add examples.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "feat(site): add English rule examples page"
```

---

## Task 5: Build the Chinese examples page

**Files:**
- Create: `examples.zh.html`
- Read: `EXAMPLES.zh_CN.md` (source content)

**Interfaces:**
- Consumes: Task 4's structure and extended `<style>` block.
- Produces: `examples.zh.html`, the link target of `index.zh.html`'s "查看五个完整示例 →".

- [ ] **Step 1: Create the file**

Same structure and CSS as `examples.html`, with `<html lang="zh-CN">`.

Head metadata:

```html
<title>规则示例 — 网页水印工具</title>
<meta name="description" content="五个真实的水印规则示例：同域名多环境、VPN 内网 IP、整个内网网段、按路径匹配管理页、Cookie 驱动的灰度环境。" />
<link rel="canonical" href="https://jinnersun.github.io/web-watermark-prompt/examples.zh.html" />
<link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/examples.html" />
<link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/examples.zh.html" />
<link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/examples.html" />
<link rel="icon" type="image/png" href="./assets/icon-128.png" />
```

Nav:

```html
<nav class="topbar">
  <a href="./examples.html">English</a>
  <a href="./examples.zh.html" class="active">中文</a>
</nav>
```

Body opening:

```html
<h1>规则示例</h1>
<p class="sub">五个真实场景，每个都包含用户的原始描述、对应的 JSON 配置，以及为什么该用这种规则类型。</p>
<a class="back" href="./index.zh.html">← 返回产品介绍</a>
```

Five sections taken verbatim from `EXAMPLES.zh_CN.md`, titles:

1. `同域名多环境（核心用例）`
2. `VPN 内网管理后台走 IP`
3. `整个内网网段`
4. `生产站按路径路由的管理页`
5. `Cookie 驱动的灰度环境`

Use the JSON blocks from `EXAMPLES.zh_CN.md`. Note these differ slightly from the English file: Example 2's `text` is `"Admin — 受限区域\n仅授权人员访问"` and Example 3's `text` is `"INTERNAL NETWORK\n内部网络"`. Use the Chinese file's values, not the English file's.

Footer language link points to `./examples.html`.

**No JSON-LD on this page.**

- [ ] **Step 2: Verify five examples and valid JSON**

```powershell
$f = "D:\item\chrome插件\web-watermark-prompt\examples.zh.html"
Select-String -LiteralPath $f -Pattern "<h2" | ForEach-Object { $_.Line.Trim() }
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-examples.py" $f
```

Expected: 5 `<h2>` lines; 5 valid JSON blocks with the same shortLabel sets as Task 4.

- [ ] **Step 3: Check rendering and encoding**

```powershell
Start-Process "D:\item\chrome插件\web-watermark-prompt\examples.zh.html"
```

Confirm no mojibake, and that `\n` inside JSON strings displays literally as `\n` (it is JSON source, not rendered text).

- [ ] **Step 4: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add examples.zh.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "feat(site): add Chinese rule examples page"
```

---

## Task 6: Add sitemap.xml and the GSC verification file

**Files:**
- Create: `sitemap.xml`
- Create: `googled3a11b0a36ad28b1.html` (moved from `D:\item\chrome插件\`)

**Interfaces:**
- Consumes: all four page files from Tasks 2–5.
- Produces: `sitemap.xml`, submitted to GSC in Task 10.

- [ ] **Step 1: Move the GSC verification file into the repo root**

The file currently sits in the container directory, where GitHub Pages cannot serve it.

```powershell
Test-Path -LiteralPath "D:\item\chrome插件\googled3a11b0a36ad28b1.html"
Move-Item -LiteralPath "D:\item\chrome插件\googled3a11b0a36ad28b1.html" -Destination "D:\item\chrome插件\web-watermark-prompt\googled3a11b0a36ad28b1.html"
Get-Content -LiteralPath "D:\item\chrome插件\web-watermark-prompt\googled3a11b0a36ad28b1.html"
```

Expected content, exactly one line:
```
google-site-verification: googled3a11b0a36ad28b1.html
```

Do not edit this file. Google matches it byte-for-byte.

- [ ] **Step 2: Create sitemap.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">

  <url>
    <loc>https://jinnersun.github.io/web-watermark-prompt/</loc>
    <lastmod>2026-08-12</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
    <xhtml:link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/"/>
    <xhtml:link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/index.zh.html"/>
    <xhtml:link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/"/>
  </url>

  <url>
    <loc>https://jinnersun.github.io/web-watermark-prompt/index.zh.html</loc>
    <lastmod>2026-08-12</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
    <xhtml:link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/"/>
    <xhtml:link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/index.zh.html"/>
    <xhtml:link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/"/>
  </url>

  <url>
    <loc>https://jinnersun.github.io/web-watermark-prompt/examples.html</loc>
    <lastmod>2026-08-12</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
    <xhtml:link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/examples.html"/>
    <xhtml:link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/examples.zh.html"/>
    <xhtml:link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/examples.html"/>
  </url>

  <url>
    <loc>https://jinnersun.github.io/web-watermark-prompt/examples.zh.html</loc>
    <lastmod>2026-08-12</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
    <xhtml:link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/examples.html"/>
    <xhtml:link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/examples.zh.html"/>
    <xhtml:link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/examples.html"/>
  </url>

  <url>
    <loc>https://jinnersun.github.io/web-watermark-prompt/privacy-policy.html</loc>
    <lastmod>2026-08-12</lastmod>
    <changefreq>yearly</changefreq>
    <priority>0.3</priority>
  </url>

</urlset>
```

- [ ] **Step 3: Verify the sitemap is well-formed XML with 5 URLs**

```powershell
$xml = [xml](Get-Content -LiteralPath "D:\item\chrome插件\web-watermark-prompt\sitemap.xml" -Raw)
"url count: $($xml.urlset.url.Count)"
$xml.urlset.url | ForEach-Object { $_.loc }
```

Expected: `url count: 5`, followed by the five URLs. An XML parse error means the file is malformed and Google will reject it.

- [ ] **Step 4: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add sitemap.xml googled3a11b0a36ad28b1.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "chore(seo): add sitemap and GSC verification file"
```

---

## Task 7: Update both READMEs

**Files:**
- Modify: `README.md`
- Modify: `README.zh_CN.md`

**Interfaces:**
- Consumes: the live landing page URLs.

Note: once `index.html` exists at the repo root, `https://jinnersun.github.io/web-watermark-prompt/` serves the landing page instead of rendering `README.md`. The README remains visible on the GitHub repo page itself.

`README.md` currently has two inaccuracies to fix while editing:
- Line 5 links "Web Watermark Tool" to a bare `https://chromewebstore.google.com/` — misleading, since the extension is not published
- Line 57 says "Chrome Web Store link — coming soon"

- [ ] **Step 1: Add a website link near the top of `README.md`**

Insert directly after the language-switch line (`> English | [中文](./README.zh_CN.md)`):

```markdown

**Website:** https://jinnersun.github.io/web-watermark-prompt/
```

- [ ] **Step 2: Fix the misleading store link on line 5 of `README.md`**

Change:
```markdown
A ready-to-use prompt for AI assistants (Claude / ChatGPT / Codex / Cursor / Gemini) to help you generate configuration for the **[Web Watermark Tool](https://chromewebstore.google.com/) Chrome extension**.
```

To:
```markdown
A ready-to-use prompt for AI assistants (Claude / ChatGPT / Codex / Cursor / Gemini) to help you generate configuration for the **[Web Watermark Tool](https://jinnersun.github.io/web-watermark-prompt/) Chrome extension**.
```

- [ ] **Step 3: Update the Related section of `README.md`**

Change:
```markdown
- **Web Watermark Tool** Chrome extension: [Chrome Web Store link — coming soon]
```

To:
```markdown
- **Web Watermark Tool** website: https://jinnersun.github.io/web-watermark-prompt/
- **Chrome Web Store**: not yet published
```

- [ ] **Step 4: Apply the equivalent three edits to `README.zh_CN.md`**

Read the file first to match its exact wording and structure, then:
- Add `**官网：** https://jinnersun.github.io/web-watermark-prompt/` after its language-switch line
- Point any bare `chromewebstore.google.com` link at the website instead
- Replace any "Chrome 应用商店链接 — 即将上线" style line with the website URL plus `**Chrome 应用商店**：尚未上架`

- [ ] **Step 5: Verify no bare store links remain in either README**

```powershell
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\README.md","D:\item\chrome插件\web-watermark-prompt\README.zh_CN.md" -Pattern "chromewebstore"
```

Expected: no output.

- [ ] **Step 6: Commit**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add README.md README.zh_CN.md
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "docs(readme): link the new website, drop the unpublished store link"
```

---

## Task 8: Pre-deploy audit across all pages

**Files:**
- Read: all HTML files
- Delete: `tmp-check-jsonld.py`, `tmp-check-examples.py`

This is the last gate before anything becomes public. Every check here catches a mistake that would be costly or embarrassing once live.

- [ ] **Step 1: Verify no page references the competitor extension ID**

```powershell
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\*.html" -Pattern "djimnchdlbbedppeedlcmebbmehcloeb"
```

Expected: **no output.** Any hit sends your traffic to a competitor and must be removed.

- [ ] **Step 2: Verify no monetization language anywhere**

```powershell
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\*.html" -Pattern "Premium","Upgrade","Pricing","Sponsor","Donate","付费","订阅","赞助","捐赠"
```

Expected: no output.

- [ ] **Step 3: Verify every internal link target exists**

Create `D:\item\chrome插件\web-watermark-prompt\tmp-check-links.py`:

```python
import io
import os
import re
import sys

root = r"D:\item\chrome插件\web-watermark-prompt"
pages = ["index.html", "index.zh.html", "examples.html", "examples.zh.html"]
ok = True

for page in pages:
    html = io.open(os.path.join(root, page), encoding="utf-8").read()
    refs = re.findall(r'(?:href|src)="\./([^"]+)"', html)
    for ref in sorted(set(refs)):
        target = ref.split("#")[0]
        if not target:
            continue
        full = os.path.join(root, target.replace("/", os.sep))
        if not os.path.exists(full):
            ok = False
            print("MISSING  %s -> %s" % (page, ref))
    # a bare "./" link resolves to index.html
    if 'href="./"' in html and not os.path.exists(os.path.join(root, "index.html")):
        ok = False
        print("MISSING  %s -> ./ (index.html)" % page)

print("link check: %s" % ("PASS" if ok else "FAIL"))
if not ok:
    sys.exit(1)
```

Run:
```powershell
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-links.py"
```

Expected: `link check: PASS`. This catches typos like `assets/screenshot-mian.png` that would render as a broken image.

- [ ] **Step 4: Verify hreflang reciprocity**

Google ignores hreflang annotations that are not mutually confirmed.

Create `D:\item\chrome插件\web-watermark-prompt\tmp-check-hreflang.py`:

```python
import io
import os
import re
import sys

root = r"D:\item\chrome插件\web-watermark-prompt"
base = "https://jinnersun.github.io/web-watermark-prompt/"
groups = [
    ("index.html", "index.zh.html", base, base + "index.zh.html"),
    ("examples.html", "examples.zh.html",
     base + "examples.html", base + "examples.zh.html"),
]
ok = True

for en_file, zh_file, en_url, zh_url in groups:
    for page in (en_file, zh_file):
        html = io.open(os.path.join(root, page), encoding="utf-8").read()
        found = dict(re.findall(
            r'<link rel="alternate" hreflang="([^"]+)" href="([^"]+)"', html))
        expected = {"en": en_url, "zh-Hans": zh_url, "x-default": en_url}
        for lang, url in expected.items():
            if found.get(lang) != url:
                ok = False
                print("BAD  %s  hreflang=%s  got=%s  want=%s"
                      % (page, lang, found.get(lang), url))

print("hreflang check: %s" % ("PASS" if ok else "FAIL"))
if not ok:
    sys.exit(1)
```

Run:
```powershell
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-hreflang.py"
```

Expected: `hreflang check: PASS`.

- [ ] **Step 5: Verify each page has exactly one canonical, one h1, and a title**

Create `D:\item\chrome插件\web-watermark-prompt\tmp-check-seo.py`:

```python
import io
import os
import re
import sys

root = r"D:\item\chrome插件\web-watermark-prompt"
pages = ["index.html", "index.zh.html", "examples.html", "examples.zh.html"]
ok = True

for page in pages:
    html = io.open(os.path.join(root, page), encoding="utf-8").read()
    h1 = len(re.findall(r"<h1[ >]", html))
    canon = len(re.findall(r'rel="canonical"', html))
    title = re.search(r"<title>(.*?)</title>", html, re.S)
    desc = re.search(r'<meta name="description" content="(.*?)"', html, re.S)
    problems = []
    if h1 != 1:
        problems.append("h1 count=%d" % h1)
    if canon != 1:
        problems.append("canonical count=%d" % canon)
    if not title:
        problems.append("no title")
    elif len(title.group(1)) > 65:
        problems.append("title %d chars (>65 may truncate)" % len(title.group(1)))
    if not desc:
        problems.append("no meta description")
    elif len(desc.group(1)) > 165:
        problems.append("description %d chars (>165 may truncate)"
                        % len(desc.group(1)))
    if problems:
        ok = False
        print("%s: %s" % (page, "; ".join(problems)))
    else:
        print("%s: OK" % page)

print("seo check: %s" % ("PASS" if ok else "FAIL"))
if not ok:
    sys.exit(1)
```

Run:
```powershell
py "D:\item\chrome插件\web-watermark-prompt\tmp-check-seo.py"
```

Expected: all four pages `OK`, then `seo check: PASS`. Title/description length warnings are advisory — shorten if flagged, but they do not block.

- [ ] **Step 6: Confirm the privacy claim matches the manifest one final time**

```powershell
Select-String -LiteralPath "D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\src\manifest.json" -Pattern "permissions" -Context 0,3
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\index.html" -Pattern "one permission"
Select-String -LiteralPath "D:\item\chrome插件\web-watermark-prompt\index.zh.html" -Pattern "一个权限"
```

The manifest must show only `storage`. If Task 0 concluded `activeTab` is required, these strings must have been updated already — verify they were.

- [ ] **Step 7: Delete the temporary check scripts**

```powershell
Remove-Item -LiteralPath "D:\item\chrome插件\web-watermark-prompt\tmp-check-jsonld.py","D:\item\chrome插件\web-watermark-prompt\tmp-check-examples.py","D:\item\chrome插件\web-watermark-prompt\tmp-check-links.py","D:\item\chrome插件\web-watermark-prompt\tmp-check-hreflang.py","D:\item\chrome插件\web-watermark-prompt\tmp-check-seo.py"
Get-ChildItem -LiteralPath "D:\item\chrome插件\web-watermark-prompt" -Filter "tmp-*"
```

Expected: no output from the second command.

- [ ] **Step 8: Confirm nothing unintended is staged or untracked**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" status --short
```

Expected: only ` M privacy-policy.html` (the permission fix from the design phase, still uncommitted). No `tmp-*` files, no stray artifacts.

- [ ] **Step 9: No commit**

This task only verifies and cleans up.

---

## Task 9: Commit the privacy fix and push everything

**Files:**
- Modify: `privacy-policy.html` (already edited during design, still uncommitted)

- [ ] **Step 1: Review the pending privacy policy diff**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" diff privacy-policy.html
```

Expected: the `activeTab` bullet removed in both the Chinese and English sections, replaced with a paragraph stating `storage` is the only permission and explaining that the badge is driven by the content script.

- [ ] **Step 2: Commit it**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add privacy-policy.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "fix(privacy): declare storage as the only permission" -m "The policy claimed activeTab, but src/manifest.json only requests storage. The toolbar badge is driven by the content script reporting its match result, so no activeTab or host permission is needed."
```

- [ ] **Step 3: Review the full set of commits about to go public**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" log --oneline origin/main..HEAD
git -C "D:\item\chrome插件\web-watermark-prompt" diff --stat origin/main..HEAD
```

Expected: the spec commit, asset commit, four page commits, sitemap commit, readme commit, and privacy fix. Confirm no unexpected files.

- [ ] **Step 4: Push**

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" push origin main
```

- [ ] **Step 5: Wait for Pages to rebuild, then verify every URL serves 200**

GitHub Pages typically takes under a minute. Wait, then:

```powershell
$urls = @(
  "https://jinnersun.github.io/web-watermark-prompt/",
  "https://jinnersun.github.io/web-watermark-prompt/index.zh.html",
  "https://jinnersun.github.io/web-watermark-prompt/examples.html",
  "https://jinnersun.github.io/web-watermark-prompt/examples.zh.html",
  "https://jinnersun.github.io/web-watermark-prompt/privacy-policy.html",
  "https://jinnersun.github.io/web-watermark-prompt/sitemap.xml",
  "https://jinnersun.github.io/web-watermark-prompt/googled3a11b0a36ad28b1.html",
  "https://jinnersun.github.io/web-watermark-prompt/assets/screenshot-main.png"
)
foreach ($u in $urls) {
  try {
    $r = Invoke-WebRequest -Uri $u -Method Head -TimeoutSec 20 -ErrorAction Stop
    "$($r.StatusCode)  $u"
  } catch {
    "$($_.Exception.Response.StatusCode.value__)  $u"
  }
}
```

Expected: `200` for all eight. A 404 on the root means `index.html` did not deploy; a 404 on the verification file will block Task 10.

- [ ] **Step 6: Verify the live root page is the landing page, not the rendered README**

```powershell
$r = Invoke-WebRequest -Uri "https://jinnersun.github.io/web-watermark-prompt/" -TimeoutSec 20
if ($r.Content -match "<title>(.*?)</title>") { $matches[1] }
```

Expected: `Web Watermark Tool — Never Deploy to the Wrong Environment Again`. If it shows README content, Pages has not finished rebuilding — wait and retry.

---

## Task 10: Register with Google Search Console

Manual steps for the user — no code. Requires Task 9 to be live first, because GSC verifies against the deployed site.

- [ ] **Step 1: Add the property**

Go to Google Search Console → Add property → choose **URL prefix** (the right-hand option, not Domain).

Enter exactly, including protocol and trailing slash:
```
https://jinnersun.github.io/web-watermark-prompt/
```

Do not use the Domain property option — it requires a DNS TXT record, and `github.io` DNS is not under your control.

- [ ] **Step 2: Verify via the HTML file method**

Choose **HTML file** verification. The file is already live at `https://jinnersun.github.io/web-watermark-prompt/googled3a11b0a36ad28b1.html`. Click Verify.

- [ ] **Step 3: Add the meta tag as a redundant second method**

In GSC, open the **HTML tag** verification option and copy the `<meta name="google-site-verification" ...>` tag.

Paste it into `index.html`, replacing the placeholder comment left in Task 2. It must sit inside `<head>`, before `<body>`.

Then commit and push:

```powershell
git -C "D:\item\chrome插件\web-watermark-prompt" add index.html
git -C "D:\item\chrome插件\web-watermark-prompt" commit -m "chore(seo): add GSC meta tag verification"
git -C "D:\item\chrome插件\web-watermark-prompt" push origin main
```

Wait for deployment, then click Verify on the HTML tag method too. Having both active prevents losing verification if one method breaks.

- [ ] **Step 4: Submit the sitemap**

GSC → Sitemaps → enter `sitemap.xml` → Submit.

Expected status: "Success" with 5 discovered URLs. Indexing itself takes days to weeks.

- [ ] **Step 5: Request indexing for the two landing pages**

GSC → URL Inspection → enter each URL → Request Indexing:
- `https://jinnersun.github.io/web-watermark-prompt/`
- `https://jinnersun.github.io/web-watermark-prompt/index.zh.html`

This nudges Google rather than waiting for organic discovery.

---

## Task 11: Create the `jinnersun.github.io` root repo

`robots.txt` is only honoured at the origin root. Because the site lives at a subpath, a separate repo is needed to control it.

- [ ] **Step 1: Create the repo on GitHub**

Create a public repo named exactly `jinnersun.github.io`. The name must match the username for GitHub to treat it as the user site.

```powershell
gh repo create jinnersun.github.io --public --description "Personal site"
```

If `gh` is unavailable, create it through the GitHub web UI.

- [ ] **Step 2: Clone it locally**

```powershell
Test-Path -LiteralPath "D:\item\chrome插件"
git clone https://github.com/jinnersun/jinnersun.github.io.git "D:\item\chrome插件\jinnersun.github.io"
```

- [ ] **Step 3: Create `robots.txt`**

```
User-agent: *
Allow: /

Sitemap: https://jinnersun.github.io/web-watermark-prompt/sitemap.xml
```

- [ ] **Step 4: Create a minimal `index.html`**

Avoids a bare 404 at the origin root, which looks abandoned to both users and crawlers.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>jinnersun</title>
<meta name="description" content="Projects by jinnersun." />
<style>
  body {
    margin: 0; padding: 80px 20px;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
                 "Microsoft YaHei", Roboto, Arial, sans-serif;
    background: #ffffff; color: #0f172a; line-height: 1.7;
  }
  @media (prefers-color-scheme: dark) {
    body { background: #0f172a; color: #f1f5f9; }
  }
  main { max-width: 640px; margin: 0 auto; }
  h1 { font-size: 26px; margin: 0 0 20px; }
  a { color: #4f46e5; }
  @media (prefers-color-scheme: dark) { a { color: #818cf8; } }
  li { margin: 8px 0; }
</style>
</head>
<body>
<main>
  <h1>jinnersun</h1>
  <ul>
    <li><a href="/web-watermark-prompt/">Web Watermark Tool</a> — a Chrome extension that watermarks pages by environment</li>
    <li><a href="https://github.com/jinnersun">GitHub</a></li>
  </ul>
</main>
</body>
</html>
```

- [ ] **Step 5: Commit and push**

```powershell
git -C "D:\item\chrome插件\jinnersun.github.io" add robots.txt index.html
git -C "D:\item\chrome插件\jinnersun.github.io" commit -m "chore: add robots.txt and minimal index"
git -C "D:\item\chrome插件\jinnersun.github.io" push origin main
```

- [ ] **Step 6: Enable Pages if it is not automatic**

Repo Settings → Pages → Source: `Deploy from a branch` → Branch `main`, folder `/ (root)`.

- [ ] **Step 7: Verify both root URLs serve 200**

```powershell
@("https://jinnersun.github.io/", "https://jinnersun.github.io/robots.txt") | ForEach-Object {
  try {
    $r = Invoke-WebRequest -Uri $_ -Method Head -TimeoutSec 20 -ErrorAction Stop
    "$($r.StatusCode)  $_"
  } catch {
    "$($_.Exception.Response.StatusCode.value__)  $_"
  }
}
```

Expected: `200` for both. Both returned 404 before this task.

---

## Deferred: after the extension is published

Not part of this plan. Recorded so the follow-up is not forgotten.

1. **Fill in the real CTA.** Once the extension has its own store ID, replace the disabled button in `index.html` and `index.zh.html`:

```html
<a class="btn btn-primary" href="https://chromewebstore.google.com/detail/YOUR_REAL_ID">Add to Chrome — Free</a>
```

Also update `SoftwareApplication.downloadUrl` in the JSON-LD, and the `Chrome Web Store: not yet published` lines in both READMEs.

2. **Set the store's homepage field** to `https://jinnersun.github.io/web-watermark-prompt/`.

3. **Rename the extension source directory** from the competitor's ID `djimnchdlbbedppeedlcmebbmehcloeb` to `web-watermark-tool`.

4. **Clean up stale `activeTab` references** in `docs/store-assets/privacy-policy.md` and `docs/publish-guide.md:202,340`.
