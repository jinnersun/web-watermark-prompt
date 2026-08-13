# v2.1 内容修复 + 迁移 www.webwatermark.dpdns.org — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 英文示例页中文化、反馈双路径（EasyForm + GitHub issues）、全站迁移到 Cloudflare Pages 域名 `https://www.webwatermark.dpdns.org/`、删除 `docs/`。

**Architecture:** 纯静态站点仓库，无构建。三块独立改动各自 commit：①`EXAMPLES.md` 英文化后重新生成 `examples.html` 的 5 段 JSON；②四页面加 EasyForm 反馈区 + 隐私页说明；③全局字符串替换绝对 URL 域名。最后删 `docs/`、push、线上验证。

**Tech Stack:** 纯 HTML/CSS/JS 静态站（GitHub Pages 现状 → Cloudflare Pages）；验证用临时 Python 脚本（仓库无测试框架）；EasyForm 一行 script 嵌入。

## Global Constraints

- 全站 LF 无 BOM、无 CRLF；每个文本文件以单个换行结尾（Google 验证文件除外，53 字节无尾换行）
- 新域名统一带 `www`：`https://www.webwatermark.dpdns.org/`；旧前缀 `https://jinnersun.github.io/web-watermark-prompt/` 全站零残留
- 相对路径（`./assets/`、`../assets/`、`./examples` 等）一律不改
- URL 结构不变：`/`、`/zh/`、`/examples`、`/zh/examples`、`/privacy-policy.html`
- EN 页 5 个 JSON 块 byte-match `EXAMPLES.md` 且不含 CJK；ZH 页 byte-match `EXAMPLES.zh_CN.md`
- 反馈接收邮箱 `wpspps@163.com`（`data-email` 唯一值）
- 外部资源白名单 = `jinnersun.github.io`、`github.com/jinnersun`、`schema.org`、`www.w3.org`、**新增 `www.easyform.dpdns.org`**；除此之外零第三方
- 真实 5 页 indexable；stub 页（`index.zh.html`、`examples.zh.html`）`noindex, follow`
- JSON-LD 仅 `index.html`/`zh/index.html` 各 2 块（SoftwareApplication + Organization），支撑页 0 块
- 无变现词、无竞品 ID、无商店链接、无 FAQPage/HowTo、无 llms.txt、无 meta refresh
- CTA 为 disabled button；footer 版权 `© 2026 jinnersun · MIT Licensed` 不变
- woff2/png/jpg/ico 保持 `text: unset`
- 本计划所有 spec/plan 文档最终随 Task 5 删除，只留 git 历史

---

### Task 1: 示例页英文化（EN）

**Files:**
- Modify: `EXAMPLES.md`
- Modify: `examples.html`（5 个 `<pre><code>` JSON 块）
- Test: 临时 `tmp-verify-en-examples.py`（用后删除）

**Interfaces:**
- Consumes: spec §1.1 的英文化映射表
- Produces: `EXAMPLES.md` 全英文 text/name；`examples.html` 5 块与之 byte-match

- [ ] **Step 1: 修改 `EXAMPLES.md` 的 `text`/`name`**

仅替换下面这些字段值，其余（`type`/`value`/`color`/`opacity`/`border`/`rotation`/`fontSize` 及结构）原样不动：

| 旧值 | 新值 |
|---|---|
| `"name": "生产环境"` | `"name": "Production"` |
| `"name": "准生产"` | `"name": "Staging"` |
| `"name": "测试环境"` | `"name": "Test"` |
| `"name": "管理后台"` | `"name": "Admin Console"` |
| `"name": "内部网络"` | `"name": "Internal"` |
| `"name": "生产管理页"` | `"name": "Production Admin"` |
| `"name": "灰度环境"` | `"name": "Canary"` |
| `"text": "生产环境 - 请谨慎操作"` | `"text": "Production - handle with care"` |
| `"text": "准生产环境"` | `"text": "Staging"` |
| `"text": "测试环境"` | `"text": "Test"` |
| `"text": "Admin — Restricted\n仅授权人员访问"` | `"text": "Admin — Restricted"` |
| `"text": "DANGER — 生产管理页面\n此页可修改线上数据"` | `"text": "DANGER — production admin page\nEditing live data"` |
| `"text": "CANARY 灰度环境"` | `"text": "CANARY ENVIRONMENT"` |

`INTERNAL NETWORK` 已是英文，不改。用临时脚本 `tmp-edit-examples-md.py` 逐对 replace 并 `assert` 每对都命中，保持 LF 无 BOM 写出。

- [ ] **Step 2: 从改后 `EXAMPLES.md` 重新生成 `examples.html` 的 5 个 JSON 块**

写 `tmp-regenerate-examples.py`：
1. 读 `EXAMPLES.md`，用 `re.findall(r'```json\n(.*?)```', md, re.S)` 取 5 块（rstrip 每块尾部 `\n`）
2. 读 `examples.html`，用 `re.findall(r'<pre><code>(.*?)</code></pre>', s, re.S)` 定位 5 块，逐一替换为 `<pre><code>{block}</code></pre>`
3. 断言替换后 `s.count('<pre><code>') == 5` 且旧中文 `"text": "生产环境` 不在文件
4. LF 无 BOM 写回

- [ ] **Step 3: 验证 EN 页与 md byte-match 且无 CJK**

写 `tmp-verify-en-examples.py`：
- 断言 `examples.html` 5 个 `<pre><code>` 块内容（HTML 反转义后）与 `EXAMPLES.md` 5 个 ```json 块逐字节相等
- 断言 `examples.html` 中 `<pre><code>` 区域内无任何 `\u4e00-\u9fff` 字符
- 断言 `EXAMPLES.md` 中 JSON 块内无 `\u4e00-\u9fff`
- 断言 `examples.html` 无 CRLF、无 BOM
- 断言 `zh/examples.html` 与 `EXAMPLES.zh_CN.md` 仍 byte-match（回归）

Expected: 全部 PASS。

- [ ] **Step 4: Commit**

```bash
git add EXAMPLES.md examples.html
git commit -m "feat(site): localize the English example configs to English
EXAMPLES.md and the EN examples page now use English watermark text and
config names, matching what the extension actually generates for English
users. The ZH page and EXAMPLES.zh_CN.md are unchanged."
```

---

### Task 2: 反馈双路径（EasyForm 表单 + GitHub issues）

**Files:**
- Modify: `index.html`、`zh/index.html`、`examples.html`、`zh/examples.html`
- Modify: `privacy-policy.html`
- Test: 临时 `tmp-verify-feedback.py`（用后删除）

**Interfaces:**
- Consumes: EasyForm 接口（`easyform.js`：任意 `form[data-easyform]`，submit 时 JSON POST 到脚本同源 `/submit/<btoa(data-email)>`，成功替换表单为成功提示，失败 `alert`；页面加载发 `/beacon?d=<hostname>`）
- Produces: 四页面含 `data-easyform` 表单与 `<script src="https://www.easyform.dpdns.org/easyform.js" data-email="wpspps@163.com"></script>`

- [ ] **Step 1: 四页面追加反馈区 HTML**

在每个页面 `</footer>` 之前插入（EN 页用英文、ZH 页用中文；`<style>` 追加见 Step 2）：

EN（`index.html`、`examples.html`）：
```html
<section class="feedback">
  <h2>Report a problem</h2>
  <p class="answer">Found a bug or want a feature? The form below lands straight in the author's inbox, or open an issue on GitHub — both work.</p>
  <div class="feedback-grid">
    <form data-easyform>
      <div class="fb-row"><label for="fb-name">Name <span>(optional)</span></label><input type="text" id="fb-name" name="name" autocomplete="name" /></div>
      <div class="fb-row"><label for="fb-email">Email <span>(optional)</span></label><input type="email" id="fb-email" name="email" autocomplete="email" /></div>
      <div class="fb-row"><label for="fb-msg">Message</label><textarea id="fb-msg" name="message" required rows="4"></textarea></div>
      <button type="submit" class="btn btn-primary">Send feedback</button>
    </form>
    <div class="fb-alt">
      <p>Prefer GitHub?</p>
      <a href="https://github.com/jinnersun/web-watermark-prompt/issues">Open an issue</a>
    </div>
  </div>
</section>
```

ZH（`zh/index.html`、`zh/examples.html`）：
```html
<section class="feedback">
  <h2>报告问题</h2>
  <p class="answer">发现 bug 或想要新功能？下面这个表单会直接发到作者的邮箱，也可以到 GitHub 提 issue —— 两条路都可以。</p>
  <div class="feedback-grid">
    <form data-easyform>
      <div class="fb-row"><label for="fb-name">名字 <span>（选填）</span></label><input type="text" id="fb-name" name="name" autocomplete="name" /></div>
      <div class="fb-row"><label for="fb-email">邮箱 <span>（选填）</span></label><input type="email" id="fb-email" name="email" autocomplete="email" /></div>
      <div class="fb-row"><label for="fb-msg">问题描述</label><textarea id="fb-msg" name="message" required rows="4"></textarea></div>
      <button type="submit" class="btn btn-primary">发送反馈</button>
    </form>
    <div class="fb-alt">
      <p>习惯用 GitHub？</p>
      <a href="https://github.com/jinnersun/web-watermark-prompt/issues">去提 issue</a>
    </div>
  </div>
</section>
```

- [ ] **Step 2: 四页面 `<style>` 追加反馈区 CSS**

在各页 `</style>` 前追加（复用现有 token）：
```css
  .feedback { padding-top: 72px; }
  .feedback .feedback-grid { display: flex; gap: 28px; align-items: flex-start; flex-wrap: wrap; }
  .feedback form { flex: 1 1 420px; display: flex; flex-direction: column; gap: 14px; }
  .feedback .fb-row { display: flex; flex-direction: column; gap: 5px; }
  .feedback .fb-row label { font: 600 13px/1.3 var(--mono); color: var(--text-2); }
  .feedback .fb-row label span { color: var(--muted); font-weight: 400; }
  .feedback input, .feedback textarea {
    font: 400 15px/1.5 var(--sans); color: var(--text);
    padding: 10px 12px; border: 1px solid var(--border); border-radius: 8px;
    background: var(--bg);
  }
  .feedback input:focus, .feedback textarea:focus { outline: 2px solid var(--primary); outline-offset: 1px; }
  .feedback .fb-alt {
    flex: 0 0 220px; padding: 18px 20px;
    border: 1px solid var(--border); border-radius: 12px; background: var(--panel);
    font-size: 15px;
  }
  .feedback .fb-alt p { color: var(--text-2); margin: 0 0 10px; }
```

- [ ] **Step 3: 四页面在 `</body>` 前追加 EasyForm 脚本**

每个页面（EN/ZH 相同）在 `</body>` 前插入一行：
```html
<script src="https://www.easyform.dpdns.org/easyform.js" data-email="wpspps@163.com"></script>
```

- [ ] **Step 4: `privacy-policy.html` 追加 EasyForm 说明**

在"6. 变更"（zh）与"6. Changes"（en）之间各加一个小节（两个 `lang-block` 都要有）：

ZH block 内新增：
```html
<h2>6. 网站反馈表单</h2>
<p>本网站的"报告问题"表单会把提交内容发送到 EasyForm（dpdns.org）以便转发到作者的邮箱。提交内容仅用于接收与处理反馈，不会被用于其它用途。你也可以选择直接在 GitHub Issues 提交反馈。</p>
```
EN block 内新增：
```html
<h2>6. Website Feedback Form</h2>
<p>The "Report a problem" form on this site sends your submission to EasyForm (dpdns.org) so it can be forwarded to the author's inbox. Submissions are used only to receive and act on feedback, never for anything else. You can also report via GitHub Issues.</p>
```
原 6/7 两节序号顺延（6. 变更→7. 变更、7. 联系方式→8. 联系方式；EN 同理）。

- [ ] **Step 5: 验证**

写 `tmp-verify-feedback.py`，对四页面断言：
- 恰好 1 个 `<form data-easyform>`
- 字段 `name="name"`、`name="email"`、`id="fb-msg" name="message" required`
- `data-email="wpspps@163.com"` 且脚本行 `<script src="https://www.easyform.dpdns.org/easyform.js" data-email="wpspps@163.com"></script>` 恰好 1 次
- issues 链接 `https://github.com/jinnersun/web-watermark-prompt/issues` 仍存在
- `.feedback` CSS 存在；无 CRLF、无 BOM
- 对 `privacy-policy.html` 断言：两个 lang-block 各含 `EasyForm` 说明且出现 `wpspps` 不在其中、`dpdns.org` 说明文本存在
- 全局断言：外部资源白名单新增 `www.easyform.dpdns.org` 后，除白名单外无其它 `http(s)` 引用

Expected: 全部 PASS。

- [ ] **Step 6: Commit**

```bash
git add index.html zh/index.html examples.html zh/examples.html privacy-policy.html
git commit -m "feat(site): add a feedback form via EasyForm alongside GitHub issues
Every content page gets a report-a-problem section in the footer area. The
form posts to EasyForm (dpdns.org) and lands in the author's inbox, which
works on networks where GitHub is unreachable; the GitHub issues link stays
for developers. The privacy policy now discloses the EasyForm submission."
```

---

### Task 3: 域名迁移到 www.webwatermark.dpdns.org

**Files:**
- Modify: `index.html`、`zh/index.html`、`examples.html`、`zh/examples.html`、`privacy-policy.html`、`index.zh.html`、`examples.zh.html`、`sitemap.xml`
- Test: 临时 `tmp-verify-domain.py`（用后删除）

**Interfaces:**
- Consumes: Task 2 已加入的 `www.easyform.dpdns.org` 脚本 URL（不在替换范围，因为前缀不同）
- Produces: 全站绝对 URL 均以 `https://www.webwatermark.dpdns.org/` 开头

- [ ] **Step 1: 全局字符串替换**

写 `tmp-migrate-domain.py`：
- `OLD = 'https://jinnersun.github.io/web-watermark-prompt'`；`NEW = 'https://www.webwatermark.dpdns.org'`
- 对上面 8 个文件：`s.replace(OLD, NEW)`；断言 `NEW in s` 且 `OLD not in s`
- 保持 LF 无 BOM 写回
- 打印每文件替换次数：index.html 9、zh/index.html 9、examples.html 4、zh/examples.html 4、privacy-policy.html 1、index.zh.html 3、examples.zh.html 3、sitemap.xml 17（若实际与这些不符，以实际为准但必须 ≥1）

- [ ] **Step 2: 验证新域名一致性**

写 `tmp-verify-domain.py`：
- 全仓库 `.html`/`.xml` 中 `jinnersun.github.io` 出现次数 == 0
- 每个真实页面 canonical == `https://www.webwatermark.dpdns.org/` + 对应路径（`/`、`/zh/`、`/examples`、`/zh/examples`、`/privacy-policy.html`）
- stub 页 canonical 指向 `/zh/`、`/zh/examples` 且 `meta refresh` 指向同一新域名
- `sitemap.xml` 5 条 `loc` 均以 `https://www.webwatermark.dpdns.org/` 开头，alternates 同域名
- `og:image`/`twitter:image` == `https://www.webwatermark.dpdns.org/assets/og-image.png`
- JSON-LD `url` 字段以新域名开头
- 相对路径（`./assets`、`../assets`、`./`、`./examples`）保持不变（抽样断言 `url('./assets/fonts/` 仍在）

Expected: 全部 PASS。

- [ ] **Step 3: Commit**

```bash
git add index.html zh/index.html examples.html zh/examples.html privacy-policy.html index.zh.html examples.zh.html sitemap.xml
git commit -m "feat(site): migrate to the Cloudflare Pages domain www.webwatermark.dpdns.org
All canonical, og, twitter, JSON-LD and sitemap URLs now point at the new
domain. Relative asset paths and the URL structure are untouched, so the
same repository deploys identically to Cloudflare Pages."
```

---

### Task 4: 全站字节审计（新域名断言）

**Files:**
- Test: 临时 `tmp-audit-site.py`（用后删除）

**Interfaces:**
- Consumes: Task 1–3 全部产物
- Produces: 审计通过证明（0 失败）

- [ ] **Step 1: 写审计脚本**

复用 v2 审计逻辑（`audit-site.py` 思路），断言改为：
- 字节完整性：全文本文件无 CRLF/无 BOM、单尾换行（Google 验证文件除外）
- `lang`/`canonical`：新域名 self-referencing
- 外部白名单 = `jinnersun.github.io`、`github.com/jinnersun`、`schema.org`、`www.w3.org`、`www.easyform.dpdns.org`
- 无旧域名残留（`jinnersun.github.io` 全仓 0 次）
- JSON-LD 仅 2 页各 2 块、类型正确；支撑页 0 块
- `sitemap.xml` 5 条且与 canonical 集合一致；`loc` 均新域名
- 无变现词、无竞品 ID、无商店链接、无 FAQPage/HowTo、无 meta refresh、无 llms.txt
- footer 版权、反馈 issues 链接、`data-email="wpspps@163.com"` 各页存在
- woff2 二进制 `text: unset`；字体引用可解析

- [ ] **Step 2: 运行并修复（如有）**

Expected: `RESULT: 0 failure(s)`。若出现失败，逐条修复后重跑至 0 失败。

- [ ] **Step 3: Commit（如有修正）**

```bash
git add -u
git commit -m "fix(site): audit corrections before domain switch"
```
若无修正，跳过本步。

---

### Task 5: 删除 docs/

**Files:**
- Delete: `docs/`（整个目录，含 v2/v2.1 的 specs 与 plans）

- [ ] **Step 1: 删除**

```bash
git rm -r docs/
git commit -m "chore: remove the superpowers planning docs from the public repo
The brainstorm, spec and plan files are process artifacts with no product
value, and the v2.1 spec contains the decision to remove them. They remain
recoverable in git history if ever needed."
```

---

### Task 6: 推送与线上验证

**Files:**
- Test: 临时 `tmp-verify-live.py`（用后删除）

**Interfaces:**
- Consumes: Task 1–5；用户完成 CF Pages 部署 + GSC 后
- Produces: 线上验证通过证明

- [ ] **Step 1: 推送**

```bash
git push origin main
```

- [ ] **Step 2: 等用户完成 CF 手动步骤**（代理无法执行，见下方"用户操作"）

- [ ] **Step 3: 线上验证**

写 `tmp-verify-live.py`（参照 v2 的 live verify，BASE 改新域名）：
- `https://www.webwatermark.dpdns.org/` 及 `/zh/`、`/examples`、`/zh/examples`、`/privacy-policy.html`、`/sitemap.xml` 全部 200 且与本地逐字节一致
- 4 个 woff2 200、字节一致、`content-type` 含 `font/woff2`
- `https://www.easyform.dpdns.org/easyform.js` 200
- 旧 GH Pages 仍 200：`https://jinnersun.github.io/web-watermark-prompt/` 等 5 页 + sitemap，且 body 内 canonical 指向新域名
- Expected: `RESULT: 0 failure(s)`

- [ ] **Step 4: 报告用户剩余项**（GSC 提交 sitemap 等，若用户尚未做）

---

## 用户操作（代理无法执行）

1. **CF Pages**：Workers & Pages → Create → Connect to Git → 选 `jinnersun/web-watermark-prompt` 仓库 → Build command 留空、Output directory `/` → Deploy
2. **Custom domain**：在 Pages 项目里 Add custom domain → `www.webwatermark.dpdns.org`（zone 已在 CF，自动验证 + 自动 HTTPS）
3. **GSC**：注册 URL-prefix 属性 `https://www.webwatermark.dpdns.org/` → 用 DNS TXT 验证（CF 的 zone 加 `google-site-verification` TXT）→ 提交 `sitemap.xml`
4. **可选**：裸域 `webwatermark.dpdns.org` 是否也要到达站点（默认不做）

## 明确不做

- 不改相对路径、不改 URL 结构、不加 meta refresh
- 不动 `EXAMPLES.zh_CN.md`、`zh/examples.html` 的 JSON 文本
- 不动扩展仓库（`e20f8e2` 的推送需用户单独确认）
- 不迁移 `jinnersun.github.io` 根仓库项目（独立计划）
