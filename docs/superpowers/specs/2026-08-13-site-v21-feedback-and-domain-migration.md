# v2.1 站点内容修复 + 迁移到 www.webwatermark.dpdns.org

日期：2026-08-13
状态：已批准（brainstorming 三轮收敛后用户确认）

## 背景

Web Watermark Tool 营销站在 `https://jinnersun.github.io/web-watermark-prompt/`
（GitHub Pages）。三个问题需要处理，同时用户决定把站点迁移到 Cloudflare Pages，
使用已注册并托管到 Cloudflare 的免费域名 **`https://www.webwatermark.dpdns.org/`**
（注意带 `www.`）。

## 目标

1. 英文示例页不再出现大量中文。
2. 反馈入口在国内网络下也能用（双路径：GitHub issues + EasyForm 表单）。
3. 站点迁移到 CF Pages + `www.webwatermark.dpdns.org`，旧 GH Pages 站点保留并被
   canonical 引导到新域名。
4. 删除仓库中的 `docs/` 过程文档目录（spec/plan 随本迭代写入后一并归档进 git 历史）。

## 决策记录（来自 brainstorming）

| 议题 | 决策 |
|---|---|
| 英文示例页中文 | `EXAMPLES.md` 的 `text`/`name` 英文化，`examples.html` 重新生成（byte-match）；`EXAMPLES.zh_CN.md` 与 `zh/examples.html` 不动 |
| 反馈入口 | 双路径：保留 GitHub issues 链接 + 底部独立反馈区（EasyForm 表单） |
| 反馈表单接收邮箱 | `wpspps@163.com` |
| 新域名 | `www.webwatermark.dpdns.org`（已注册、已托管到 CF，含 zone） |
| 部署形态 | CF Pages，CF 控制台连接 GitHub 仓库（build 留空） |
| 旧 GH Pages | 保留，靠 canonical 引导；不放 meta refresh（新旧站同仓库无法区分部署环境，放 refresh 会让新站自我跳转） |
| docs/ 目录 | 整个删除 |

## 第一部分：内容修复

### 1.1 示例页英文化（EN）

`EXAMPLES.md` 中 5 段示例配置仅 `name`/`text` 字段英文化；规则类型、`color`、
`opacity`、`border`、结构一概不动：

| 中文（现在） | 英文（改后） |
|---|---|
| name: 生产环境 | Production |
| name: 准生产 | Staging |
| name: 测试环境 | Test |
| name: 管理后台 | Admin Console |
| name: 内部网络 | Internal |
| name: 生产管理页 | Production Admin |
| name: 灰度环境 | Canary |
| text: 生产环境 - 请谨慎操作 | Production - handle with care |
| text: 准生产环境 | Staging |
| text: 测试环境 | Test |
| text: Admin — Restricted\n仅授权人员访问 | Admin — Restricted |
| text: INTERNAL NETWORK | INTERNAL NETWORK（不变） |
| text: DANGER — 生产管理页面\n此页可修改线上数据 | DANGER — production admin page\nEditing live data |
| text: CANARY 灰度环境 | CANARY ENVIRONMENT |

`examples.html` 从改后 `EXAMPLES.md` 重新生成，保持 byte-match；EN 页的
"why" 文字与新的英文 text 天然吻合（例 2 引用 "Admin — Restricted" 即等于 text），
无需改动。`EXAMPLES.zh_CN.md`、`zh/examples.html` 保持不变（已是中文）。

> 原 spec 中"示例 2–3 中英 JSON text 有差异"的偏差记录随之消失；新约束为
> EN 页 byte-match EN md、ZH 页 byte-match ZH md，两套各成体系。

### 1.2 反馈双路径（底部独立反馈区）

在 `index.html`、`zh/index.html`、`examples.html`、`zh/examples.html` 的
footer 上方新增 `<section class="feedback">`：

- 标题（EN：Report a problem / ZH：报告问题）
- 表单字段：Name（选填）、Email（选填）、Message（必填 textarea）
- EasyForm 嵌入：`<script src="https://www.easyform.dpdns.org/easyform.js" data-email="wpspps@163.com" async defer></script>` + 表单 `data-easyform` 属性，AJAX 提交到 dpdns.org
- 表单下方一行：或到 GitHub issues 提交（保留现有链接，双路径）

连带变更：

- `privacy-policy.html`：新增小节，说明"网站反馈表单向 EasyForm（dpdns.org）
  提交以转发邮件，仅用于接收反馈"。
- 审计脚本外部资源白名单加入 `www.easyform.dpdns.org`；其余零第三方约束不变。

## 第二部分：迁移到 www.webwatermark.dpdns.org

### 2.1 替换清单（全局字符串替换）

把以下旧域名前缀替换为新域名前缀：

```
https://jinnersun.github.io/web-watermark-prompt/  →  https://www.webwatermark.dpdns.org/
```

涉及文件（按 probe 结果）：

| 文件 | 出现次数 | 涉及 |
|---|---|---|
| index.html | 9 | canonical、og:url、og:image、twitter:image、2×JSON-LD url |
| zh/index.html | 9 | 同上 |
| examples.html | 4 | canonical、hreflang |
| zh/examples.html | 4 | canonical、hreflang |
| privacy-policy.html | 1 | canonical |
| index.zh.html（stub） | 3 | canonical、meta refresh、可见链接 |
| examples.zh.html（stub） | 3 | 同上 |
| sitemap.xml | 17 | 5×loc + hreflang alternates |

相对路径（`./assets/`、`../assets/`、`./`、`./examples` 等）一律不改——新域名
根目录下天然正确。仓库结构与文件名不变，URL 结构不变：`/`、`/zh/`、`/examples`、
`/zh/examples`、`/privacy-policy.html`。

### 2.2 GSC（用户手动）

- 注册新属性：URL-prefix `https://www.webwatermark.dpdns.org/`
- 验证：DNS TXT（在 CF 的 zone 里加 `google-site-verification` TXT 记录，最干净，
  无需在仓库放验证文件）
- 验证后提交 `sitemap.xml`

### 2.3 旧 GH Pages（保留 + canonical 引导）

`jinnersun.github.io/web-watermark-prompt/` 继续部署同一仓库。新旧站内容相同、
唯一差异是域名，旧站 canonical 指向新域名后搜索引擎自动收敛，不构成重复内容。
**不放 meta refresh**（见决策记录）。

## 第三部分：CF Pages 部署（用户手动步骤）

1. CF 控制台 → Workers & Pages → Create → 连接 GitHub 仓库 `jinnersun/web-watermark-prompt`
2. Build command 留空，输出目录 `/`，直接发布静态文件
3. Add custom domain：`www.webwatermark.dpdns.org`（zone 已在 CF，自动验证 + 自动 HTTPS）
4. 如需裸域 `webwatermark.dpdns.org` 也到达站点，在 CF DNS 加 `webwatermark` AAAA/CNAME 规则或另行决定（本 spec 不强制，默认只挂 www）

## 不变约束（继续生效）

- 全站 LF 无 BOM、无 CRLF；相对路径与站点结构不变
- `noindex, follow` 仅用于 stub 页；真实 5 页 indexable
- JSON-LD 仅首页 2 块（SoftwareApplication + Organization），支撑页 0 块
- CTA 为 disabled 按钮；无变现词、无竞品 ID、无商店链接、无 FAQPage/HowTo
- footer 版权 `© 2026 jinnersun · MIT Licensed` 不变
- woff2 等二进制 `text: unset`
- 5 种语言 `inLanguage` 不变

## 验证标准

1. EN 页 5 个 JSON 块 byte-match `EXAMPLES.md` 且不含 CJK；ZH 页 byte-match `EXAMPLES.zh_CN.md`
2. 四个页面均有 feedback 区：表单字段、`data-easyform`、`data-email="wpspps@163.com"`、issues 链接
3. `privacy-policy.html` 含 EasyForm 说明
4. 全站绝对 URL 无旧域名残留；canonical/og/sitemap 均为 `https://www.webwatermark.dpdns.org/...`
5. 字节审计（复用 v2 的 audit 脚本，断言改为新域名 + 白名单加 easyform）
6. `docs/` 已从 HEAD 删除
7. 推送后线上验证（等用户完成 CF/GSC 后）：
   - 新域名 5 页 + sitemap + 4 woff2 全部 200 且与本地逐字节一致，woff2 MIME 正确
   - 旧 GH Pages 各 URL 仍 200，canonical 指向新域名
   - `sitemap.xml` 可解析

## 用户手动项（代理无法执行）

1. CF Pages 连接 GitHub 仓库 + 添加 custom domain `www.webwatermark.dpdns.org`
2. GSC 新属性 DNS TXT 验证 + 提交 sitemap
3. （可选）裸域 `webwatermark.dpdns.org` 是否也要到达站点

## 明确不做

- 不迁移 `jinnersun.github.io` 根仓库项目（独立计划，仍待办）
- 不动 EasyForm 本体
- 不改扩展仓库（除非用户另要求推送 e20f8e2）
- 不加 meta refresh、不改 URL 结构、不加 FAQPage/HowTo
