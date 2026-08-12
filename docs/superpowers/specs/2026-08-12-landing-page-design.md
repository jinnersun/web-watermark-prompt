# 落地页设计方案 — Web Watermark Tool

日期：2026-08-12
状态：待实施
仓库：`web-watermark-prompt`（GitHub Pages 已在线）

---

## 1. 背景与目标

Chrome 扩展 Web Watermark Tool（本地 v2.0.0，尚未上架）需要一个营销落地页。

### 目标演化过程

初始需求是「申请 Lemon Squeezy 需要提供网址」。经过讨论，变现路线经历两次调整，最终确定：

1. 曾考虑 Lemon Squeezy 付费版 → 放弃（付费方案未定，且现有核心功能已写入商店描述，转 Pro 有虚假宣传风险）
2. 曾考虑设置页挂广告 → 放弃（MV3 禁止远程代码；AdSense 不允许投放扩展，规避有封号风险；目标用户是开发者/QA/SRE，广告拦截率极高；收益预估月个位数美元，而代价是失去「零网络请求」这一核心信任卖点）
3. **最终：扩展彻底免费，不做变现，不申请 Lemon Squeezy，不放赞助链接**

### 落地页现在的实际目标

- 提供 Chrome 商店 homepage 字段所需的官网链接（信任信号）
- 承接搜索流量，覆盖「区分测试与生产环境」类真实痛点查询
- 完整展示产品能力（商店限 5 张截图且顺序固定，落地页可讲完整故事）

### 非目标

- 不做定价页、退款政策、服务条款（这些是 Lemon Squeezy 审核要素，已不需要）
- 不做变现入口（含赞助/捐赠链接）
- 不做博客或内容营销体系

---

## 2. 关键前置事实

### 2.1 扩展尚未上架，目标扩展 ID 属于竞品

`djimnchdlbbedppeedlcmebbmehcloeb` 是被借鉴的**竞品**扩展 ID，不属于本项目。实测该 ID 在商店返回 HTTP 200，名为 **Web Custom Watermark**，描述为「按域名显示自定义文字水印，可调颜色、透明度、密度」。

**落地页的 CTA 绝对不能指向此链接**，否则会将流量导向竞品。

本项目扩展 ID 需在首次上传 Chrome 商店后才会分配。

竞品功能显著弱于本项目 v2.0.0（无 IP/CIDR、无 Cookie 匹配、无智能对比色、无 badge），落地页有真实差异化可讲。

### 2.2 本地目录名沿用了竞品 ID

扩展源码位于 `D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\`。此目录名易造成混淆，建议后续重命名为 `web-watermark-tool`。**本方案不包含此重命名**，仅记录为待办。

### 2.3 GitHub Pages 现状（实测）

| URL | 状态 |
|---|---|
| `https://jinnersun.github.io/web-watermark-prompt/` | 200（当前渲染 README.md） |
| `https://jinnersun.github.io/web-watermark-prompt/privacy-policy.html` | 200 |
| `https://jinnersun.github.io` | 404（个人主页站不存在） |
| `https://jinnersun.github.io/robots.txt` | 404 |

repo 根目录无 `index.html`，因此新增 `index.html` 会直接接管该 URL，无需修改任何 Pages 设置。

`web-watermark-prompt` 为独立 repo，Pages 从分支直接发布，push 到 `main` 即自动上线，无构建步骤。

### 2.4 隐私政策权限数已修正（本轮已完成，未提交）

`privacy-policy.html` 原声称扩展申请 `storage` + `activeTab` 两个权限，但 `src/manifest.json` 实际只有 `storage`。已修改中英文两处，改为声明唯一权限并说明 badge 由 content script 上报实现。

此改动与 `docs/publish-guide.md:468` 记录的「方案 A」一致，并与 README 说法对齐。

**遗留风险**：`docs/publish-guide.md:298` 记录了一项未验证事项 —— badge 在仅有 `storage` 权限下是否真能工作（`tab.url` 是否被 Chrome 剥离）。若实测证明必须补 `activeTab`，则此修改需回滚，且落地页「仅 1 个权限」的表述必须同步修改。**落地页实施前应先完成该实测。**

---

## 3. 调研结论（推翻了部分初始假设）

### 3.1 GSC 验证方式

对子路径 URL，可用方式：

- **URL 前缀属性 + HTML meta 标签**：可行。meta 须置于该前缀首页 `<head>` 内
- **HTML 文件上传**：可行，文件放 repo 根目录
- **域名属性**：不可用，需 DNS TXT 记录，`github.io` 的 DNS 权限不在用户手中

GSC 申请时填写的网址（须带协议与末尾斜杠）：

```
https://jinnersun.github.io/web-watermark-prompt/
```

用户已获取 HTML 验证文件 `googled3a11b0a36ad28b1.html`（内容为单行 `google-site-verification: googled3a11b0a36ad28b1.html`），当前位于 `D:\item\chrome插件\`，**需移动到 repo 根目录**才能生效。

meta 标签验证码待页面上线后再获取并回填。

### 3.2 「长尾词需要大量页面」—— 此说法不适用于本项目

- Ahrefs 数据：单个页面排上目标词后，平均同时排 1000+ 相关长尾词
- Google 自 2024 年 3 月起打击 **scaled content abuse**（"生成大量页面主要为操纵排名，对用户价值很低"）
- 2026 年 3 月核心更新中，批量薄内容站掉 40–80% 流量
- **「AI 翻译扩量」被列为重点惩罚模式** —— 将同一薄内容翻译成多语言以增加页面数

结论：本项目应采用**少而实**的页面策略。双语版本可行（2 种语言、原创撰写、各有真实受众），但禁止为凑页面数生成关键词变体页。

`EXAMPLES.md` 已有的 6 个真实场景是有实质价值的长尾内容，适合单独成页。

### 3.3 GEO 结论修正 —— 放弃 FAQPage schema 与 llms.txt

初始方案曾主张「FAQPage 是 AI 引用率最高的格式」。**该说法证据不支持，已撤回。**

实证数据：

- **Ahrefs 对照实验（2026-05）**：1885 个新增 JSON-LD 的页面 vs 4000 对照页，AI 引用无显著提升（AI Overviews -4.6%，AI Mode +2.4%，ChatGPT +2.2%，后两者属噪声）
- **Google 官方生成式搜索指南（2026-05-15）**：明确说明出现在 AI Overviews / AI Mode 不需要任何特殊 schema.org 标记
- **FAQ 富结果已于 2026-05-07 下线**（schema 类型本身未废弃，但 SERP 展示效果消失）
- **searchVIU 测试**：ChatGPT / Claude / Perplexity / Gemini 检索时读取可见 HTML，不解析 JSON-LD
- **SE Ranking 数据**：正文含可见 FAQ 区块的页面平均 4.9 次引用 vs 无的 4.4；仅有 FAQ schema 的反为 3.6 vs 4.2
- **llms.txt**：Ahrefs 查 13.7 万站点，97% 的 llms.txt 在 2026-05 零请求；Gary Illyes 称 Google 无支持计划，John Mueller 比作已废弃的 keywords meta 标签

因此：

- **主力手段是页面上真实可见的问答区块**
- JSON-LD 仅保留精简 `SoftwareApplication` + `Organization`，视为基础设施而非增长手段
- 不实现 FAQPage schema
- 不实现 llms.txt

---

## 4. 技术方案

### 4.1 技术栈

每页单个 HTML 文件，内联 `<style>` 与 `<script>`，零外部依赖，零构建。

**不使用 Tailwind CDN**，理由：

1. 外部 JS 拖慢 LCP，而 Core Web Vitals 是排名因素
2. 与产品「零网络请求、不依赖 CDN」的核心主张自洽
3. 单页落地页 CSS 量有限，构建工具收益不足

沿用 `privacy-policy.html` 既有模式：CSS variables + Indigo `#4f46e5` / Slate 配色 + `prefers-color-scheme: dark`。

图片是唯一外部资源。

### 4.2 双语实现 —— 拆分为独立文件

语言切换使用**真实 `<a href>` 链接**，不使用 JS 切换。原因：JS 切换（如 `privacy-policy.html` 的做法）使两种语言共用同一 URL，Google 只索引默认语言版本，中文内容无法获得搜索流量。

`<html lang>` 各文件写死，不由 JS 修改。

hreflang 三向互指：

```html
<link rel="alternate" hreflang="en" href="https://jinnersun.github.io/web-watermark-prompt/">
<link rel="alternate" hreflang="zh-Hans" href="https://jinnersun.github.io/web-watermark-prompt/index.zh.html">
<link rel="alternate" hreflang="x-default" href="https://jinnersun.github.io/web-watermark-prompt/">
```

### 4.3 文件结构

```
web-watermark-prompt/
├── index.html                      英文落地页（canonical 自指，x-default）
├── index.zh.html                   中文落地页
├── examples.html                   英文规则示例页
├── examples.zh.html                中文规则示例页
├── privacy-policy.html             已存在，本轮已修权限段
├── sitemap.xml                     5 个 URL
├── googled3a11b0a36ad28b1.html     GSC 验证文件（从容器目录移入）
├── assets/
│   ├── screenshot-main.png
│   ├── screenshot-scenario.png
│   ├── screenshot-smartcolor.png
│   ├── og-image.png
│   └── icon-128.png
├── README.md                       加落地页链接
└── README.zh_CN.md                 加落地页链接
```

图片来源（跨 repo 复制，非引用）：

| 目标 | 来源 |
|---|---|
| `assets/screenshot-main.png` | `docs/store-assets/screenshots/01-main-panel.png` |
| `assets/screenshot-scenario.png` | `docs/store-assets/screenshots/02-real-scenario.png` |
| `assets/screenshot-smartcolor.png` | `docs/store-assets/screenshots/04-smart-color.png` |
| `assets/og-image.png` | `docs/store-assets/promo-1400x560.png` |
| `assets/icon-128.png` | `docs/store-assets/icon-128.png` |

仅用 3 张截图控制页面体积（5 张共约 520KB）。

### 4.4 SEO 基建（每页）

- `<title>` 与 `meta description` 按搜索意图撰写，非产品名堆砌
- `canonical` 自指
- `hreflang` 三向互指
- Open Graph / Twitter Card，图用 `og-image.png`（1400×560）
- 语义化 HTML：单个 `<h1>`，`<section>` + `<h2>` 层级清晰
- GSC 验证 meta 标签占位注释（上线后回填）

关于两种 GSC 验证方式并存：HTML 文件方式（§3.1）已可独立完成验证，meta 标签为冗余备份。Google 允许多方式同时生效，保留两者可防单一方式失效导致掉验证。meta 占位仅需放在 `index.html`（前缀首页），其余页面不需要。

JSON-LD 仅两类：

- `SoftwareApplication`：名称、`applicationCategory: BrowserApplication`、`operatingSystem: Chrome`、`offers.price: 0`、`featureList`
- `Organization`：作者身份，含 `sameAs` 指向 GitHub

### 4.5 新建 `jinnersun.github.io` repo

用途：取得源站根目录控制权。`robots.txt` 仅在源站根目录生效，子路径无法放置。

内容：

- `robots.txt`：允许全站抓取，声明 sitemap 位置
- `index.html`：极简个人页，避免根目录 404

`github.io` 在 Public Suffix List 上，GSC 将 `jinnersun.github.io` 视为独立站点，可正常验证与收录。

---

## 5. 落地页内容结构

`index.html` / `index.zh.html` 章节顺序：

### ① Hero

- `<h1>`：`Never Deploy to the Wrong Environment Again`（中文：`再也不会在生产环境上误操作`）
- 副标题：按域名、URL、IP、Cookie 自动给网页打环境水印
- 主 CTA：`Coming to Chrome Web Store`，置灰不可点
- 次 CTA：`Get early access on GitHub` → repo
- 信任标签行：`Free · No account · 1 permission · Zero network requests`
- 主截图 `screenshot-main.png`

CTA 说明：扩展 ID 未分配，商店链接必须集中为单一常量，便于上线后一次性替换。

主 CTA 实现方式：使用 `<button disabled>` 而非 `<a>`，配 `aria-disabled="true"`，避免出现指向空地址或竞品的链接。上线后替换为正常 `<a>`。

### ② 痛点（3 个真实场景）

复用 `docs/store-assets/descriptions/en-detail.txt` 既有内容：

- 同根域名：`test.app.example.com` 与 `app.example.com` 肉眼难辨
- VPN 内网 IP 访问后台，无任何环境视觉线索
- 灰度分流：同一域名，靠 cookie 区分 canary / prod

配 `screenshot-scenario.png`。

### ③ 六种匹配规则

表格形式（类型 / 说明 / 示例）。内容密度高，对 SEO 与 AI 抓取均有价值。每行链至 `examples.html` 对应场景。

| 类型 | 语义 | 示例 |
|---|---|---|
| `host-exact` | hostname 完全相等 | `app.example.com` |
| `host-suffix` | 匹配所有子域 | `example.com` |
| `url-regex` | 全 URL 正则 | `^https?://.*/admin(/.*)?$` |
| `ip-exact` | hostname 为 IP 时精确匹配 | `192.0.2.5` |
| `ip-cidr` | IPv4 CIDR 网段 | `10.0.0.0/8` |
| `cookie` | cookie 键值匹配 | `deploy=canary` |

### ④ 核心特性（4 个）

每个用一句话说明「解决什么问题」而非「是什么」：

- 智能对比色（配 `screenshot-smartcolor.png`）
- 沉浸式边框
- 鼠标/键盘交互渐隐
- 工具栏 badge

### ⑤ 隐私（视觉上强调）

- 仅 `storage` 一个权限
- 零网络请求、零数据收集、零第三方
- 配置仅存于用户自己的 Chrome 账号（`chrome.storage.sync`）
- 链至 `privacy-policy.html`

此章节表述必须与 `manifest.json` 实际权限严格一致（见 2.4 遗留风险）。

### ⑥ FAQ（可见问答区块）

GEO 主力手段。`<h2>` 用问句形式，答案第一句即自成完整答案，便于被直接摘取。

初版 6 个问题：

1. Is Web Watermark Tool free? — 是，全功能免费，无账号、无内购
2. Can it tell environments apart when they share the same domain? — 能，用 cookie 或 URL 正则规则
3. Does it collect any data? — 不，仅 `storage` 权限，零网络请求
4. Can I mark environments accessed by IP address? — 能，支持精确 IP 与 CIDR 网段
5. How do I share config with my team? — JSON 导入导出
6. What languages does it support? — 5 种（en / zh_CN / zh_TW / ja / es）

### ⑦ 页脚

GitHub、隐私政策、Issues 反馈、语言切换、`© 2026 jinnersun · MIT`

### 文案原则

- **不出现任何付费 / Pro / 价格字样**，与商店描述保持一致
- **不夸大**，隐私表述必须与 manifest 实际权限对得上
- 不使用竞品扩展 ID 或其商店链接

---

## 6. 示例页内容结构

`examples.html` / `examples.zh.html` 基于 `EXAMPLES.md` / `EXAMPLES.zh_CN.md`。

仅使用 Example 1–5。**Example 6 排除** —— 它描述的是 AI 在输入模糊时的兜底追问行为，对人类读者无价值。

每个场景保留三段结构：

1. 用户原话描述（真实语境，长尾词天然载体）
2. 对应 JSON 配置（`<pre><code>` 展示）
3. 「为什么这样选」说明（规则类型选择理由）

五个场景对应的长尾意图：

| 场景 | 覆盖查询意图 |
|---|---|
| 同域名多环境 | 区分同根域名的测试与生产 |
| VPN 内网 IP 后台 | 按 IP 地址标记环境 |
| 整个内网网段 | CIDR 网段批量标记 |
| 生产环境路径路由 | 正则匹配特定路径 |
| Cookie 驱动灰度 | 同 URL 靠 cookie 区分灰度 |

页面需含返回落地页链接与语言切换。

---

## 7. sitemap.xml

包含 5 个 URL：

```
https://jinnersun.github.io/web-watermark-prompt/
https://jinnersun.github.io/web-watermark-prompt/index.zh.html
https://jinnersun.github.io/web-watermark-prompt/examples.html
https://jinnersun.github.io/web-watermark-prompt/examples.zh.html
https://jinnersun.github.io/web-watermark-prompt/privacy-policy.html
```

---

## 8. README 改动

`README.md` 与 `README.zh_CN.md` 顶部加落地页链接。

注意：repo 根目录新增 `index.html` 后，`https://jinnersun.github.io/web-watermark-prompt/` 将不再渲染 README，而是显示落地页。README 仍在 GitHub repo 页面可见。

---

## 9. 部署顺序

存在硬性顺序约束：**GSC 验证要求验证文件/meta 标签已在线**，因此不能先验证再部署。

```
1. 开发全部页面（CTA 用占位常量，集中单处）
2. 移入 GSC 验证文件到 repo 根目录
3. 一次性 commit + push（含本轮 privacy-policy.html 的权限修正）
4. 确认 Pages 生效（各 URL 返回 200）
5. GSC 添加「网址前缀」属性，填 https://jinnersun.github.io/web-watermark-prompt/
6. 验证通过后提交 sitemap
7. 新建 jinnersun.github.io repo，放 robots.txt + 极简 index
8. 打包提交 Chrome 商店，homepage 字段填落地页 URL
9. 获得自己的扩展 ID 后，回填真实 CTA 链接，再 push 一次
```

第 3 步为单次提交，避免中间态上线。

---

## 10. 实施前必办事项

1. **验证 badge 在仅 `storage` 权限下是否可用**（`docs/publish-guide.md:298`）。结果决定「仅 1 个权限」这一核心卖点能否成立，以及 2.4 的隐私政策修改是否需回滚。

---

## 11. 记录在案的待办（不在本方案范围）

- 扩展源码目录从竞品 ID `djimnchdlbbedppeedlcmebbmehcloeb` 重命名为 `web-watermark-tool`
- 清理 `docs/store-assets/privacy-policy.md` 与 `docs/publish-guide.md:202,340` 中残留的 `activeTab` 表述
- 扩展上架后回填落地页 CTA 链接
