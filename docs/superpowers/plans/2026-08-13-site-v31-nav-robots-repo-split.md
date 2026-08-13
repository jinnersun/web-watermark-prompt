# 站点 v3.1：顶部导航 / robots.txt / GitHub 链接改向 / 仓库拆分 — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为站点全部页面加固定顶部导航栏，新增自建 `robots.txt`，把站内 GitHub/issues 链接从 prompt 仓库改向到新站点仓库 `github.com/jinnersun/watermask`，并把站点部署文件拆分到独立仓库 watermask（本地 `D:\item\chrome插件\web-watermask`），旧 prompt 仓库清成纯 prompt 内容。

**Architecture:** 纯静态 HTML（每页内联 `<style>`），无构建/测试框架。改动全部落在 5 个真实页 + 2 个 stub 页 + 新增 robots.txt。验证方式是字节级审计脚本（node）+ 链接 grep，非单元测试。仓库拆分：本地把白名单文件拷进新目录，`git init` 全新历史推送到 `watermask.git`。

**Tech Stack:** 静态 HTML/CSS（无 JS 框架）、node 审计脚本、git、GitHub、Cloudflare Pages。

## Global Constraints

- 文件编码：LF 无 BOM，单尾换行；`.gitattributes` 已强制 `eol=lf`。
- 不引入任何外部 JS/CSS 库；导航 CSS 复用现有 token（`--panel/--border/--text/--text-2/--muted/--primary/--sans/--mono`）。
- 不改 URL 结构、不改相对路径规则；`#feedback` 锚点仅出现在有反馈表单的页（index/examples）。
- 站点内所有 GitHub 链接目标：`https://github.com/jinnersun/watermask`（issues 为 `/issues`）。
- JSON-LD `author.url`/`sameAs`（`https://github.com/jinnersun`）不改。
- sitemap.xml 内容不动。
- 不用 emoji；中文文案用简体。

---

### Task 1: 新增 robots.txt（仓库根）

**Files:**
- Create: `robots.txt`

- [ ] **Step 1: 创建 robots.txt**

```text
User-agent: *
Allow: /

Sitemap: https://www.webwatermark.dpdns.org/sitemap.xml
```

写入后确认结尾恰为一个换行符、无 BOM。

- [ ] **Step 2: 验证**

用字节检查（见 Task 7 审计脚本会覆盖），此处先确认文件存在且内容逐字节如上。

- [ ] **Step 3: Commit**

```bash
git add robots.txt
git commit -m "feat(site): add custom robots.txt with Sitemap declaration"
```

---

### Task 2: index.html — 顶部导航 + 反馈锚点 + GitHub 链接改向

**Files:**
- Modify: `index.html`（顶部 CSS 块约 L77-L97；`<nav class="topbar">` L365-368；feedback section L708；footer L726-729；CTA L376）

**Interfaces:**
- Consumes: `--text-2` token 已存在于 index.html `:root`。
- Produces: 导航栏 markup + CSS 作为 Task 3/4/5 的模板基准。

- [ ] **Step 1: 替换顶部导航 CSS 块**

将 index.html 中这一段（含 dark-mode 内 `nav.topbar a.active { color: #0f172a; }` L77）：

```css
nav.topbar {
  display: flex; gap: 8px; padding: 20px 0 0; justify-content: flex-end;
}
nav.topbar a {
  font: 600 13px/1 var(--mono); letter-spacing: .04em;
  color: var(--muted); text-decoration: none;
  padding: 7px 11px; border-radius: 6px; border: 1px solid var(--border);
}
nav.topbar a.active { background: var(--primary); border-color: var(--primary); color: #fff; }
```

替换为：

```css
nav.topbar {
  position: sticky; top: 0; z-index: 50;
  display: flex; align-items: center; gap: 24px;
  padding: 14px 28px;
  background: var(--panel);
  border-bottom: 1px solid var(--border);
}
nav.topbar .brand {
  display: flex; align-items: center; gap: 8px;
  font: 700 16px/1 var(--sans); color: var(--text); text-decoration: none;
}
nav.topbar .brand .mark {
  width: 26px; height: 26px; border-radius: 7px;
  background: var(--primary); color: #fff;
  display: inline-flex; align-items: center; justify-content: center;
  font: 700 13px/1 var(--mono);
}
nav.topbar .links { display: flex; gap: 18px; }
nav.topbar a { font: 500 14px/1 var(--sans); color: var(--text-2); text-decoration: none; }
nav.topbar a:hover { color: var(--primary); }
nav.topbar a.active { color: var(--primary); }
nav.topbar .side { margin-left: auto; display: flex; gap: 16px; align-items: center; }
section.feedback { scroll-margin-top: 72px; }
@media (max-width: 720px) {
  nav.topbar { flex-wrap: wrap; gap: 10px; padding: 12px 16px; }
  nav.topbar .links { order: 3; width: 100%; justify-content: space-between; }
  nav.topbar .side { margin-left: 0; }
}
```

并删除 dark-mode media 块内的 `nav.topbar a.active { color: #0f172a; }` 一行。

- [ ] **Step 2: 把导航栏移到 `<div class="wrap">` 之前（body 直属子元素）**

当前结构为 `<body>` → `<div class="wrap">` → `<nav class="topbar">`。改为 `<body>` → `<nav class="topbar">` → `<div class="wrap">`，使 sticky 导航横向铺满视口。

- [ ] **Step 3: 替换导航 markup**

```html
<nav class="topbar">
  <a class="brand" href="./">
    <span class="mark">W</span><span>Web Watermark</span>
  </a>
  <div class="links">
    <a href="./" class="active">Home</a>
    <a href="./examples">Examples</a>
    <a href="./privacy-policy.html">Privacy</a>
    <a href="#feedback">Feedback</a>
  </div>
  <div class="side">
    <a href="https://github.com/jinnersun/watermask">GitHub</a>
    <a href="./zh/">中文</a>
  </div>
</nav>
```

- [ ] **Step 4: 给反馈区加锚点 id**

`<section class="feedback">` → `<section class="feedback" id="feedback">`

- [ ] **Step 5: GitHub 链接改向**（4 处）

- L376 `href="https://github.com/jinnersun/web-watermark-prompt"` → `https://github.com/jinnersun/watermask`
- L720 `.../web-watermark-prompt/issues` → `.../watermask/issues`
- L726 `.../web-watermark-prompt` → `.../watermask`
- L729 `.../web-watermark-prompt/issues` → `.../watermask/issues`

JSON-LD（L1016/L1027 `https://github.com/jinnersun`）不动。

- [ ] **Step 6: 验证**（Task 7 审计会覆盖全部，此处跑快速检查）

```bash
rg -n "web-watermark-prompt" index.html
```
预期：无输出（0 残留）。再确认 `nav.topbar` 在 body 直属、`id="feedback"` 存在。

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat(site): add top nav bar and point GitHub links to watermask repo"
```

---

### Task 3: zh/index.html — 中文导航 + 反馈锚点 + GitHub 链接改向

**Files:**
- Modify: `zh/index.html`（顶部 CSS 块；topbar L365-368；feedback L708；footer L726-729；CTA L376）

- [ ] **Step 1: 替换导航 CSS**

与 Task 2 Step 1 完全相同的 CSS 块替换（zh/index.html 同样有 `--text-2` token），并删除 dark-mode 内的 `nav.topbar a.active { color: #0f172a; }`。

- [ ] **Step 2: 导航移到 `<div class="wrap">` 之前**

- [ ] **Step 3: 替换导航 markup（中文）**

```html
<nav class="topbar">
  <a class="brand" href="./">
    <span class="mark">W</span><span>网页水印</span>
  </a>
  <div class="links">
    <a href="./" class="active">首页</a>
    <a href="./examples">示例</a>
    <a href="../privacy-policy.html">隐私政策</a>
    <a href="#feedback">反馈</a>
  </div>
  <div class="side">
    <a href="https://github.com/jinnersun/watermask">GitHub</a>
    <a href="../">English</a>
  </div>
</nav>
```

- [ ] **Step 4: 反馈区加 id**

`<section class="feedback">` → `<section class="feedback" id="feedback">`

- [ ] **Step 5: GitHub 链接改向**（4 处，同 Task 2 Step 5 对应中文文案所在行 L376/L720/L726/L729）

- [ ] **Step 6: 验证**

```bash
rg -n "web-watermark-prompt" zh/index.html
```
预期：无输出。

- [ ] **Step 7: Commit**

```bash
git add zh/index.html
git commit -m "feat(site): add zh top nav and point GitHub links to watermask"
```

---

### Task 4: examples.html — 导航 + 反馈锚点 + GitHub 链接改向

**Files:**
- Modify: `examples.html`（顶部 CSS 块 L80-92；topbar L239-242；feedback L423；footer L441-444）

- [ ] **Step 1: 替换导航 CSS**

examples.html 的 `.wrap` 是 `max-width: 940px`。替换其顶部 `nav.topbar` 三行 CSS 块，采用与 Task 2 Step 1 相同的完整 CSS 块（含 `--text-2`）。

- [ ] **Step 2: 导航移到 `<div class="wrap">` 之前**

- [ ] **Step 3: 替换导航 markup**

```html
<nav class="topbar">
  <a class="brand" href="./">
    <span class="mark">W</span><span>Web Watermark</span>
  </a>
  <div class="links">
    <a href="./">Home</a>
    <a href="./examples" class="active">Examples</a>
    <a href="./privacy-policy.html">Privacy</a>
    <a href="#feedback">Feedback</a>
  </div>
  <div class="side">
    <a href="https://github.com/jinnersun/watermask">GitHub</a>
    <a href="./zh/examples">中文</a>
  </div>
</nav>
```

- [ ] **Step 4: 反馈区加 id**

`<section class="feedback">` → `<section class="feedback" id="feedback">`

- [ ] **Step 5: GitHub 链接改向**（3 处）

- L435 `.../web-watermark-prompt/issues` → `.../watermask/issues`
- L441 `.../web-watermark-prompt` → `.../watermask`
- L444 `.../web-watermark-prompt/issues` → `.../watermask/issues`

- [ ] **Step 6: 验证**

```bash
rg -n "web-watermark-prompt" examples.html
```
预期：无输出。

- [ ] **Step 7: Commit**

```bash
git add examples.html
git commit -m "feat(site): add examples top nav and point GitHub links to watermask"
```

---

### Task 5: zh/examples.html — 中文导航 + 反馈锚点 + GitHub 链接改向

**Files:**
- Modify: `zh/examples.html`（顶部 CSS 块；topbar L239-242；feedback L423；footer L441-444）

- [ ] **Step 1: 替换导航 CSS**（同 Task 4 Step 1 完整块）

- [ ] **Step 2: 导航移到 `<div class="wrap">` 之前**

- [ ] **Step 3: 替换导航 markup（中文）**

```html
<nav class="topbar">
  <a class="brand" href="../">
    <span class="mark">W</span><span>网页水印</span>
  </a>
  <div class="links">
    <a href="../">首页</a>
    <a href="./examples" class="active">示例</a>
    <a href="../privacy-policy.html">隐私政策</a>
    <a href="#feedback">反馈</a>
  </div>
  <div class="side">
    <a href="https://github.com/jinnersun/watermask">GitHub</a>
    <a href="../examples">English</a>
  </div>
</nav>
```

- [ ] **Step 4: 反馈区加 id**

`<section class="feedback">` → `<section class="feedback" id="feedback">`

- [ ] **Step 5: GitHub 链接改向**（3 处，同 Task 4 Step 5 对应行 L435/L441/L444）

- [ ] **Step 6: 验证**

```bash
rg -n "web-watermark-prompt" zh/examples.html
```
预期：无输出。

- [ ] **Step 7: Commit**

```bash
git add zh/examples.html
git commit -m "feat(site): add zh examples top nav and point GitHub links to watermask"
```

---

### Task 6: privacy-policy.html — 导航 + 语言切换入导航 + GitHub 链接改向

**Files:**
- Modify: `privacy-policy.html`（顶部 CSS 块 L57/L91-109/L134-143；topbar L158-161；header meta 的 lang-switch L168-171；issues 链接 L212/L253）

**Interfaces:**
- Consumes: `switchLang()` JS（L264 起）不变，依赖 `.lang-switch a` 与 `.lang-block`。
- Produces: 导航栏内嵌 `.lang-switch`，沿用现有 JS。

- [ ] **Step 1: 调整 body 与 container 顶距**

privacy 的 body 是 `padding: 40px 20px;`。改为 `padding: 0 0 40px;`，并给 `.container` 加 `margin-top: 40px;`，让全宽 sticky 导航贴顶、卡片容器下移留白。

- [ ] **Step 2: 替换导航 CSS 块**

将现有 `nav.topbar`（L134-143）与 `.lang-switch`（L91-109）两段替换为：

```css
nav.topbar {
  position: sticky; top: 0; z-index: 50;
  display: flex; align-items: center; gap: 24px;
  padding: 14px 28px;
  background: var(--panel);
  border-bottom: 1px solid var(--border);
}
nav.topbar .brand {
  display: flex; align-items: center; gap: 8px;
  font: 700 16px/1 var(--sans); color: var(--text); text-decoration: none;
}
nav.topbar .brand .mark {
  width: 26px; height: 26px; border-radius: 7px;
  background: var(--primary); color: #fff;
  display: inline-flex; align-items: center; justify-content: center;
  font: 700 13px/1 var(--mono);
}
nav.topbar .links { display: flex; gap: 18px; }
nav.topbar a { font: 500 14px/1 var(--sans); color: var(--muted); text-decoration: none; }
nav.topbar a:hover { color: var(--primary); }
nav.topbar a.active { color: var(--primary); }
nav.topbar .side { margin-left: auto; display: flex; gap: 16px; align-items: center; }
nav.topbar .side .lang-switch { display: flex; gap: 4px; }
nav.topbar .side .lang-switch a {
  font: 500 14px/1 var(--sans); color: var(--muted);
  border: none; padding: 0; background: none; border-radius: 0;
}
nav.topbar .side .lang-switch a.active { color: var(--primary); }
@media (max-width: 720px) {
  nav.topbar { flex-wrap: wrap; gap: 10px; padding: 12px 16px; }
  nav.topbar .links { order: 3; width: 100%; justify-content: space-between; }
  nav.topbar .side { margin-left: 0; }
}
```

删除 dark-mode media 内的 `nav.topbar a.active { color: #0f172a; }`。

- [ ] **Step 3: 导航移到 `<div class="container">` 之前（body 直属）**

- [ ] **Step 4: 替换导航 markup**

```html
<nav class="topbar">
  <a class="brand" href="./">
    <span class="mark">W</span><span>Web Watermark</span>
  </a>
  <div class="links">
    <a href="./">首页 / Home</a>
    <a href="./examples">示例 / Examples</a>
    <a href="./privacy-policy.html" class="active">隐私政策 / Privacy</a>
    <a href="https://github.com/jinnersun/watermask/issues">反馈 / Feedback</a>
  </div>
  <div class="side">
    <a href="https://github.com/jinnersun/watermask">GitHub</a>
    <span class="lang-switch">
      <a data-lang="zh" class="active" onclick="switchLang('zh')">中文</a>
      <a data-lang="en" onclick="switchLang('en')">English</a>
    </span>
  </div>
</nav>
```

- [ ] **Step 5: 从 header `.meta` 中删除原 `.lang-switch` div（L168-171），保留前两行信息**

原 `.lang-switch` 移到导航栏后，header 内不再重复。

- [ ] **Step 6: GitHub 链接改向**（2 处，L212/L253）

`.../web-watermark-prompt/issues` → `.../watermask/issues`（两处链接的 href 与可见文本都改，可见文本为完整 URL）。

- [ ] **Step 7: 验证**

```bash
rg -n "web-watermark-prompt" privacy-policy.html
```
预期：无输出。确认 `.lang-switch` 只在导航栏内出现一次，`switchLang` 函数未被改动。

- [ ] **Step 8: Commit**

```bash
git add privacy-policy.html
git commit -m "feat(site): add privacy top nav, move lang switch into nav, point issues to watermask"
```

---

### Task 7: stub 页评估（index.zh.html / examples.zh.html）

**Files:**
- Read-only: `index.zh.html`, `examples.zh.html`

- [ ] **Step 1: 确认现状并记录结论**

两个 stub 页均带 `meta http-equiv="refresh" content="0; url=..."` 即时跳转（秒级 301 等价）。因跳转发生在任何导航渲染之前，**不添加导航**（加了也不可见，属死代码）。此结论写进 plan 说明即可，不改文件。

- [ ] **Step 2: 无变更，无 commit**

---

### Task 8: 全站审计脚本 + 运行

**Files:**
- Create: `C:\Users\ZhuanZ\AppData\Local\Temp\opencode\audit-site.mjs`（临时目录，不入库）

- [ ] **Step 1: 写审计脚本**

```js
import { readFileSync, statSync } from 'node:fs';
import { join } from 'node:path';

const ROOT = 'D:/item/chrome插件/web-watermark-prompt';
const TARGETS = [
  'index.html', 'zh/index.html', 'examples.html', 'zh/examples.html',
  'privacy-policy.html', 'index.zh.html', 'examples.zh.html',
  'sitemap.xml', 'robots.txt'
];
const NAV_PAGES = ['index.html', 'zh/index.html', 'examples.html', 'zh/examples.html', 'privacy-policy.html'];
const GH_REPO_OLD = 'github.com/jinnersun/web-watermark-prompt';
const GH_REPO_NEW = 'github.com/jinnersun/watermask';
let fail = 0;

function err(msg) { fail++; console.error('FAIL:', msg); }

for (const f of TARGETS) {
  const p = join(ROOT, f);
  if (!statSync(p).isFile()) { err(f + ' missing'); continue; }
  const buf = readFileSync(p);
  const s = buf.toString('utf8');
  if (buf.includes(0x0d)) err(f + ': contains CRLF');
  if (buf[0] === 0xef && buf[1] === 0xbb) err(f + ': has BOM');
  if (!s.endsWith('\n')) err(f + ': no trailing newline');
  else if (s.endsWith('\n\n')) err(f + ': double trailing newline');
  if (f.endsWith('.html')) {
    if (s.includes(GH_REPO_OLD)) err(f + ': residual old repo link');
    if (!s.includes(GH_REPO_NEW)) err(f + ': missing new repo link');
  }
}
for (const f of NAV_PAGES) {
  const s = readFileSync(join(ROOT, f), 'utf8');
  if (!s.includes('<nav class="topbar">')) err(f + ': no topbar nav');
}
const robots = readFileSync(join(ROOT, 'robots.txt'), 'utf8');
if (!robots.includes('Sitemap: https://www.webwatermark.dpdns.org/sitemap.xml')) err('robots.txt: missing Sitemap');
if (fail === 0) console.log('AUDIT PASSED');
else { console.log('AUDIT FAILED: ' + fail); process.exit(1); }
```

- [ ] **Step 2: 运行**

```bash
node "C:/Users/ZhuanZ/AppData/Local/Temp/opencode/audit-site.mjs"
```
预期：`AUDIT PASSED`。任何 FAIL 先修复再继续。

---

### Task 9: 审计通过后提交与推送旧仓库

- [ ] **Step 1: 全量检查待提交文件**

```bash
git status --short
```
预期：仅上述 5 个 html + robots.txt + 审计临时文件（临时文件不在仓库内，应显示 clean 之外仅目标文件）。

- [ ] **Step 2: Commit + Push**

```bash
git add -A
git commit -m "feat(site): v3.1 top nav, robots.txt, GitHub links to watermask"
git push origin main
```

- [ ] **Step 3: 确认 push 后 GH Pages / CF 旧站仍可访问（短暂一致期）**

```bash
curl.exe -sS -o NUL -w "%{http_code}" https://www.webwatermark.dpdns.org/
```
预期：200。

---

### Task 10: 创建独立站点仓库 watermask（本地 web-watermask + GitHub）

**Files:**
- Create: `D:\item\chrome插件\web-watermask`（git init，全新历史）
- Create: `D:\item\chrome插件\web-watermask\README.md`（站点仓库版）

- [ ] **Step 1: 初始化本地仓库**

```bash
git init
git config user.name jinnersun
git config user.email 358042175@163.com
git remote add origin https://github.com/jinnersun/watermask.git
```
（workdir: `D:\item\chrome插件\web-watermask`）

- [ ] **Step 2: 拷贝白名单文件**

从 `D:\item\chrome插件\web-watermark-prompt` 拷贝以下到 `web-watermask`：

```
.gitattributes  LICENSE  robots.txt  sitemap.xml
index.html  zh/index.html  examples.html  zh/examples.html
index.zh.html  examples.zh.html  privacy-policy.html
googled3a11b0a36ad28b1.html
assets/  (递归)
```

**排除：** PROMPT.md、PROMPT.zh_CN.md、EXAMPLES.md、EXAMPLES.zh_CN.md、README.md（旧 prompt 版）、README.zh_CN.md、docs/、.gitignore（新仓库不需要）、.superpowers/。

- [ ] **Step 3: 写新 README.md**

```markdown
# Web Watermark Tool — 站点

Web Watermark Tool 的静态站点，部署于 Cloudflare Pages（`www.webwatermark.dpdns.org`）。

纯静态 HTML，无构建步骤。页面：首页（EN/ZH）、示例（EN/ZH）、隐私政策。

- 反馈/Issue：https://github.com/jinnersun/watermask/issues
- 扩展源码（另仓）：本仓库不包含扩展源码与 prompt 文件。
```

- [ ] **Step 4: 字节审计新仓库**

把 Task 8 的 `ROOT` 指向 `D:/item/chrome插件/web-watermark`，改 TARGETS 去掉 sitemap/robots 之外保持原样，再跑一遍，预期 PASSED（同时确认无 `web-watermark-prompt` 残留、新链接存在）。

- [ ] **Step 5: Commit + Push**

```bash
git add -A
git commit -m "chore: initial site repo (from web-watermark-prompt v3.1)"
git push -u origin main
```
若 GitHub 上仓库非空导致 push 被拒，先 `git pull origin main --allow-unrelated-histories` 合并后重推（用户确认仓库无冲突文件时再做）。

- [ ] **Step 6: 交付用户操作清单**

CF Pages 连接仓库改为 `jinnersun/watermask`（构建配置留空，输出目录 `/`）。此为用户操作，计划在此暂停等待用户完成。

---

### Task 11: 线上验证新仓库

- [ ] **Step 1: 用户 CF 改连 watermask 后，验证**

```bash
curl.exe -sS -o NUL -w "%{http_code}" https://www.webwatermark.dpdns.org/
curl.exe -sS -o NUL -w "%{http_code}" https://www.webwatermark.dpdns.org/robots.txt
curl.exe -sS -o NUL -w "%{http_code}" https://www.webwatermark.dpdns.org/sitemap.xml
curl.exe -sS -o NUL -w "%{http_code}" https://www.webwatermark.dpdns.org/examples
```
预期：全部 200。

- [ ] **Step 2: robots 内容核对**

```bash
curl.exe -sS https://www.webwatermark.dpdns.org/robots.txt
```
预期输出逐字节等于自建版（含 `Sitemap:` 行），非 CF 托管版。

- [ ] **Step 3: 首页导航人工目检**（用户浏览器打开首页，确认 sticky 顶栏、active 高亮、Feedback 锚点滚到表单）

---

### Task 12: 清理旧 prompt 仓库（删除站点文件）

**Files:**
- Delete: `D:\item\chrome插件\web-watermark-prompt` 下的站点部署文件
- Modify: `D:\item\chrome插件\web-watermark-prompt\README.md`

- [ ] **Step 1: 删除站点文件**

```bash
git rm -r index.html zh examples.html zh/examples.html index.zh.html examples.zh.html privacy-policy.html sitemap.xml robots.txt assets googled3a11b0a36ad28b1.html docs
```
（docs/ 含本 spec/plan，按旧仓库惯例随清理删除，git 历史保留。）

- [ ] **Step 2: 更新 README.md**

在开头加一行说明：

```markdown
> 站点已迁移到独立仓库 [jinnersun/watermask](https://github.com/jinnersun/watermask)（部署 www.webwatermark.dpdns.org）。本仓库仅包含 prompt 内容。
```

保留 PROMPT.md / EXAMPLES.md 相关说明，移除"Website:"指向 GH Pages 的旧行（改成新站点域名或删除）。

- [ ] **Step 3: 验证**

```bash
git status --short
rg -n "web-watermark-prompt" README.md PROMPT.md PROMPT.zh_CN.md EXAMPLES.md EXAMPLES.zh_CN.md README.zh_CN.md 2>$null
```
预期：剩余引用仅为说明站点迁移到 watermask 的文案；无 index.html 等站点文件残留（`git ls-files` 确认）。

- [ ] **Step 4: Commit + Push**

```bash
git add -A
git commit -m "chore: remove site deployment files, repo is prompt-only now"
git push origin main
```

- [ ] **Step 5: 确认旧 GH Pages 自然失效（可接受 404）**

```bash
curl.exe -sS -o NUL -w "%{http_code}" https://jinnersun.github.io/web-watermark-prompt/
```
预期：404 或 3xx；若仍 200（缓存），等 GitHub 重建即可，canonical 已指新域名。

---

### Task 13: GSC sitemap 重试

- [ ] **Step 1: 用户操作**：GSC 属性 `https://www.webwatermark.dpdns.org/` → Sitemaps → 对 `sitemap.xml` 点"重试"。

- [ ] **Step 2: 仍失败则 URL 检查调试**：URL 检查工具输入 `https://www.webwatermark.dpdns.org/sitemap.xml`，查看抓取状态码；此前已确认线上 XML 合法（`application/xml`、5 条 loc 域名匹配），大概率是 CF 偶发 TLS，重试即过。

---

## Self-Review 检查点

- **spec 覆盖**：导航（Task 2-7）✓；robots.txt（Task 1）✓；GitHub 链接→watermask（Task 2-6,8）✓；仓库拆分 watermask（Task 10）✓；旧 prompt 仓库清理（Task 12）✓；sitemap 不动 + GSC 重试（Task 8 不校验内容 + Task 13）✓；JSON-LD 不改（Task 2 Step 5 明确不动）✓。
- **占位符扫描**：无 TBD/TODO；CSS 与 markup 均为完整代码。
- **类型/命名一致**：导航类名 `.topbar/.brand/.mark/.links/.side` 五个页面统一；`--text-2`（index/examples）与 `--muted`（privacy）按各页 token 集分别使用，privacy 单独给了 `.lang-switch` 覆写。
