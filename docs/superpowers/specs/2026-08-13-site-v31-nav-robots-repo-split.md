# V3.1 站点导航 / robots / GitHub 链接修正 / 仓库拆分 — 设计

**日期:** 2026-08-13
**状态:** 已确认（watermask = 独立站点仓库；扩展水印仓库不动；站点 issues 指向 watermask）
**关联:** 上一篇 spec 与 plan 已随 v2.1 Task 5 删除；本文档沿用"拆分后排除、最终删除，只留 git 历史"的惯例。

## 背景与问题

1. **导航缺失**：站点目前只有页内内容和底部 footer，用户需要跨区块/跨页跳转需反复滚屏。需求：必须增加顶部横向导航（固定可见）。
2. **robots.txt 缺失**：仓库无 `robots.txt`，线上 `/robots.txt` 是 Cloudflare 自动托管的托管版（含 AI 爬虫屏蔽，但无 `Sitemap:` 声明）。需求：自建 `robots.txt`。
3. **GitHub 链接指向错误仓库**：站点所有 GitHub 链接（Get early access / GitHub / issues）指向 `github.com/jinnersun/web-watermark-prompt`——这是 **prompt 仓库**（README 定位是"给 AI 用的 prompt 文件"），而扩展源码在 `github.com/jinnersun/water-mark`（扩展仓库 remote 已确认）。概念混淆。
4. **仓库职责混杂**：`web-watermark-prompt` 一个仓库同时承载 prompt 内容与部署的网站文件。用户决定拆分。
5. **sitemap 疑似 GSC 报错**：GSC 报"无法读取此站点地图"。已核实线上 sitemap.xml 返回标准合法 XML（`application/xml`，三种请求方式均与本地逐字节一致，含 `<loc>` 标签，非纯文本；浏览器渲染成文本视图属正常现象）。GSC 属性为 `https://www.webwatermark.dpdns.org/`，与 sitemap 5 条 loc 域名一致，无跨域问题。

## 决策记录（用户逐项确认）

| 议题 | 决策 |
|---|---|
| 导航形式 | 顶部横向导航（固定可见） |
| 导航内容 | 首页 / 示例 / 隐私政策 / 反馈 + 语言切换 + GitHub 链接 |
| 导航"反馈"链接 | 锚点指向本页面反馈表单（保留 EasyForm 双路径） |
| robots.txt 内容 | 基础版：`Allow: /` + `Sitemap:` 声明 |
| **新站点仓库** | `github.com/jinnersun/watermask`（已存在 public 200）；本地克隆到 `D:\item\chrome插件\web-watermask`（当前为空，无 git） |
| **GitHub 链接指向** | 站点内所有 GitHub/issue 链接指向 `github.com/jinnersun/watermask` |
| **扩展仓库** | 扩展源码仓库 remote（water-mark）**不动**；站点 issues 指向 watermask |
| 拆分方式 | 新建独立站点仓库 watermask（不带 prompt 内容） |
| 拆分时机 | 本地准备好后一次性迁移（导航+robots+新域名+链接修正全部就绪 → 建本地 web-watermask git → 关联 watermask remote → push → CF 改连） |
| 旧 GH Pages | 让 `jinnersun.github.io/web-watermark-prompt/` 自然失效（canonical 已指向新域名） |
| 旧 prompt 仓库 | `web-watermark-prompt` 删除站点的"不相关代码"（部署文件 index.html 等），只保留 prompt 内容，README 更新 |
| sitemap 处理 | 保持内容不变，GSC 重试；仍失败则用 URL 检查工具调试 |

### 拆分执行明细（修订）

**新仓库内容（白名单迁移，排除 prompt）：**
```
.gitattributes  LICENSE  robots.txt  sitemap.xml
index.html  zh/index.html  examples.html  zh/examples.html
index.zh.html  examples.zh.html  privacy-policy.html
googled3a11b0a36ad28b1.html
assets/  (fonts + icon + og-image + screenshots)
README.md（重写为站点仓库说明，非 prompt 版）
```

**排除：** `PROMPT.md`、`PROMPT.zh_CN.md`、`EXAMPLES.md`、`EXAMPLES.zh_CN.md`、旧 README（prompt 定位）、`docs/`（v3.1 spec/plan 留在旧仓库 git 历史即可）。

**执行步骤：**
1. 本地在 `web-watermark-prompt` 完成导航/robots/链接修正 → commit → push（短暂保持 GH Pages 旧站一致）。
2. 在 `web-watermark`（repo 目录）`git clone` 站点仓库 → 检出到 `web-watermask` 本地文件夹（或 `git init` + `git remote add origin git@/https://github.com/jinnersun/watermask.git` + 推送当前树）。
3. 新仓库树：删除 prompt 内容文件，仅保留白名单 + 重写 README；`git add`/`commit` → push 到 `watermask` remote。
4. 用户操作：CF Pages 改连 `jinnersun/watermask`。
5. 新域名线上验证。
6. 旧 `web-watermark-prompt` 仓库：删除站点部署文件，保留 prompt 内容，更新 README（"站点已移至 watermask"）；旧 GH Pages 自然失效。

**风险与缓解：**
- GitHub Pages 旧地址 404：canonical 已指新域名，可接受。
- watermask 为空仓库 → 直接推当前树（全新历史），不追求迁移旧历史（用户对站点历史无依赖条件）。

## 详细设计

### 1. 顶部横向导航（所有页面）

**作用域：** `index.html`、`zh/index.html`、`examples.html`、`zh/examples.html`、`privacy-policy.html`（真实页）+ `index.zh.html`、`examples.zh.html`（stub 页）。

**结构**（放在 `<body>` 内最顶部，`<nav class="topbar">`，与现有 `.topbar` 类名冲突——现有 topbar 是语言切换条，需重命名或合并。检查后决定：**改造现有 `.topbar`** 为完整导航栏，避免两栏叠加）：

```html
<nav class="topbar">
  <a class="brand" href="./">
    <span class="mark">W</span><span>Web Watermark</span>
  </a>
  <div class="links">
    <a href="./">Home</a>
    <a href="./examples">Examples</a>
    <a href="./privacy-policy.html">Privacy</a>
    <a href="#feedback">Feedback</a>
  </div>
  <div class="side">
    <a href="https://github.com/jinnersun/watermask">GitHub</a>
    <a href="./zh/">中文</a>   <!-- EN 页显示 -->
  </div>
</nav>
```

- ZH 页文本对应中文（首页/示例/隐私政策/反馈/中文链接对应切换 EN）。
- **stub 页（index.zh.html / examples.zh.html）**: 顶部导航保留，链接语言切换部分指回 EN 主站；这些页无 `#feedback` 锚点 → "反馈"链接指向 `https://github.com/jinnersun/watermask/issues`。
- **隐私页（privacy-policy.html）**: 双语言区块，导航语言切换链接在顶部（非块内）；反馈锚点 `#feedback` 不存在于隐私页 → "反馈"链接指 issues。
- 原 `.topbar` 若为独立语言切换条，合并进新导航后**删除原条**，避免重复。

**CSS**（复用现有 token：`--panel`/`--border`/`--text`/`--muted`/`--primary`/`--mono`/`--sans`）：

```css
.topbar {
  position: sticky; top: 0; z-index: 50;
  display: flex; align-items: center; gap: 24px;
  padding: 12px 28px; background: var(--panel);
  border-bottom: 1px solid var(--border);
}
.topbar .brand { display: flex; align-items: center; gap: 8px; font: 700 16px/1 var(--sans); color: var(--text); text-decoration: none; }
.topbar .mark { width: 26px; height: 26px; border-radius: 7px; background: #4f46e5; color: #fff; display: inline-flex; align-items: center; justify-content: center; font: 700 13px/1 var(--sans); }
.topbar .links { display: flex; gap: 18px; margin-left: 8px; }
.topbar a { font: 500 14px/1 var(--sans); color: var(--text-2); text-decoration: none; }
.topbar a:hover { color: var(--primary); }
.topbar a.active { color: var(--primary); }
.topbar .side { margin-left: auto; display: flex; gap: 16px; }
@media (max-width: 720px) {
  .topbar { flex-wrap: wrap; gap: 10px; padding: 10px 16px; }
  .topbar .links { order: 3; width: 100%; justify-content: space-between; }
  .topbar .side { margin-left: 0; }
}
```

**active 态**：当前页对应链接加 `.active`（Home 在 index、Examples 在 examples、Privacy 在 privacy）。

### 2. robots.txt（仓库根）

```
User-agent: *
Allow: /

Sitemap: https://www.webwatermark.dpdns.org/sitemap.xml
```

- LF 无 BOM，单尾换行。
- 部署到 CF Pages 后，会覆盖 CF 托管的自动 robots.txt（CF Pages 优先用仓库根的文件）。
- 不屏蔽任何爬虫（用户选基础版）。

### 3. GitHub 链接改向

**规则：站点内所有** `https://github.com/jinnersun/web-watermark-prompt*` 站内链接 → `https://github.com/jinnersun/watermask*`（issues 改 `.../watermask/issues`）。

涉及文件：
- `index.html`（CTA"Get early access"、footer GitHub、footer Feedback、反馈区 issues）
- `zh/index.html`（同 EN，中文文案）
- `examples.html`、`zh/examples.html`（footer + 反馈区 issues）
- `privacy-policy.html`（反馈小节 issues 链接，两语言块）

**不改：**
- 扩展仓库 `water-mark` 源码不动；其 `options.html` 的 AI prompt 源链接指向 prompt 仓库 `web-watermark-prompt/PROMPT.md` 是**正确**的（prompt 文件仍在 prompt 仓库），保持不变。
- JSON-LD `author.url` / `sameAs` = `https://github.com/jinnersun`（个人主页），不改。

### 4. 仓库拆分（独立站点仓库）

**目标：** 新仓库 `github.com/jinnersun/watermask`（已确认，public 200），仅含部署文件；本地目录 `D:\item\chrome插件\web-watermask`。

**新仓库内容（白名单迁移）：**
```
.gitattributes  LICENSE  robots.txt  sitemap.xml
index.html  zh/index.html  examples.html  zh/examples.html
index.zh.html  examples.zh.html  privacy-policy.html
googled3a11b0a36ad28b1.html
assets/  (fonts + icon + og-image + screenshots)
README.md（重写为站点仓库说明，非 prompt 版）
```

**排除：** `PROMPT.md`、`PROMPT.zh_CN.md`、`EXAMPLES.md`、`EXAMPLES.zh_CN.md`、旧 README（prompt 定位）、`docs/`。

**执行路径（本地准备好后一次性迁移）：**
1. 本地继续在 `web-watermark-prompt` 仓库完成导航/robots/链接修正 → commit → push（维持 GH Pages 旧站短暂一致）。
2. 在 `D:\item\chrome插件\web-watermask` 初始化 git，添加 **仅部署白名单文件**（从当前站点树拷贝）。
3. push 到 `https://github.com/jinnersun/watermask.git`（全新历史；用户已确认水印仓库为空，站点历史无依赖）。
4. 用户操作：CF Pages 改连 `watermask` 仓库。
5. 新域名线上验证。
6. 旧 `web-watermark-prompt` 仓库：删除站点部署文件（index.html 等），保留 prompt 内容，更新 README（站点已迁出）；旧 GH Pages 自然失效。

**风险与缓解：**
- GitHub Pages 旧地址 404：canonical 已指新域名，v2.1 时已建立；可接受。
- 拆分时历史丢失：watermask 为空仓库 → 全新历史；旧仓库 git 历史仍保留 prompt 内容，无需迁移。

### 5. sitemap / GSC

- **不动 sitemap.xml**（线上已验证合法、域名匹配）。
- GSC 操作（用户）：Sitemap 页面点"重试"；仍失败 → URL 检查工具输入 `https://www.webwatermark.dpdns.org/sitemap.xml` 查看抓取错误码（可能 CF 偶发 TLS 握手失败，重试即可）。

## 明确不做

- 不改 URL 结构、不改相对路径、不加 meta refresh（旧站自然失效不设跳转）。
- 不动扩展仓库 `water-mark` 源码；其 `options.html` 的 AI prompt 源链接保持指向 prompt 仓库 PROMPT.md。
- 不在站点上重新加回 GH Pages 旧站。
- 不屏蔽任何爬虫（robots.txt 基础版）。
- 不简化 sitemap。

## 验证

- 导航：6 个页面渲染顶部栏，active 态正确，sticky 生效，移动端换行不破版。
- robots.txt：新仓库线上 `/robots.txt` 为自定义版（含 Sitemap 声明），非 CF 托管版。
- 链接：全站 `web-watermark-prompt` 站内链接 0 残留（`.html`/`.xml`）；`water-mark` 链接生效；issues 链接可达。
- 字节审计：无 CRLF/BOM/单尾换行、白名单、JSON-LD 不变、无变现词。
- 线上：新仓库部署后 5 页 + sitemap + 字体 byte-match；旧 GH Pages 允许 404。
- GSC：用户重试 sitemap。
