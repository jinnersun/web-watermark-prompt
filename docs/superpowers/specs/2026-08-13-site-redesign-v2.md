# 站点改版设计 v2 — Web Watermark Tool

日期：2026-08-13
状态：待评审
仓库：`web-watermark-prompt`（GitHub Pages 已上线）
前置：`docs/superpowers/specs/2026-08-12-landing-page-design.md`（v1，已实施并部署）

---

## 1. 为什么要改版

v1 已上线并通过 30/30 项审计，但存在三类问题：

1. **词汇错位（最严重）** — 这个品类被搜索的词是 *environment indicator / environment marker / ribbon / banner*，而全站只说 *watermark*。扩展源码里 `indicator` / `marker` / `ribbon` 出现次数为 **0**。
2. **视觉密度过高** — section 垂直内边距只有 `8px 0 32px`，正文 15–16px，13–34px 之间塞了 9 个字号档位，没有 display 层级，每一节视觉权重相同，页面没有焦点。
3. **差异化被埋没** — 我们独有的能力（IP/CIDR 匹配、Cookie 匹配、`mix-blend-mode` 智能对比色）只是页面中部的普通 `<h3>`；同时有 **两个已完整实现的功能页面从未提及**（规则测试器、全局总开关）。

### 非目标

- 不重命名扩展本体（商店名仍为 Web Watermark Tool）
- 不做变现（无 Pro / 定价 / 赞助 / 捐赠）
- 不新增 IP/CIDR 专题页（属关键词拆页，会造成同类争食）

---

## 2. 关键前置事实（已验证）

### 2.1 隐私声明的主语是「扩展」，不是「网站」

这是本次改版中我最初判断错误、后经原文核对纠正的一点：

| 位置 | 原文 | 约束对象 |
|---|---|---|
| `index.html:231` | "This **extension** requests one permission: `storage`." | 扩展 |
| `index.zh.html:228` | 「本**扩展**只需要一项权限：`storage`。」 | 扩展 |
| `privacy-policy.html:155` | 「本**扩展**不与任何外部服务器通信」 | 扩展 |

全站检索 `本站` / `本网站` / `this site` / `this website`，**没有任何一句声称网站自身不发起请求**。
→ 站点自托管字体不违反任何既有承诺。
→ 但 `options.html:8` 的 Google Fonts 仍必须删除：那是**扩展**在承诺不联网的同时挂 CDN，是真实矛盾。

### 2.2 品牌色来自扩展本体

`#4f46e5` 是扩展 `options.css` 中的 `--primary`，不是随手取的 Tailwind 靛蓝。站点与扩展同色是资产，保留。

扩展完整 token（站点应对齐而非另创）：

```
--primary #4f46e5   --primary-hover #4338ca   --primary-light #eef2ff
--text-main #0f172a --text-secondary #475569  --text-muted #94a3b8
--border-light #e5e7eb --danger #ef4444 --success #10b981 --warning #f59e0b
--radius-sm 8px --radius-md 12px --radius-lg 16px
字体：Inter + JetBrains Mono
```

### 2.3 扩展渲染的真实算法（首屏演示必须逐行复现）

- `watermark-core.js` `buildTile()`：`gap = max(50, density)`；`dpr = min(2, devicePixelRatio)`；`textAlign/textBaseline` 居中；`translate(gap/2, gap/2)`；`rotate(rotation × π / 180)`；`lineHeight = fontSize × 1.2`；`startY = -((lines-1) × lineHeight) / 2`
- `content.js:118-122` 智能色填充：`light` → `#d1d5db`，`dark` → `#1f2937`
- `content.js:148-149`：smart 模式下 canvas `globalAlpha = 1`、CSS `opacity` 承担透明度、`mix-blend-mode: difference`
- `content.js:155-172` 边框：`inset 0 0 0 Npx`，`width = max(1, parseInt || 4)`，**层级必须压在水印之上**，否则边框像素会被 difference 反色算歪（源码注释明确写了）
- 淡出触发：`mousemove` / `keydown` / `wheel` / `touchmove` —— **没有 click 监听**
- badge：白字 `#ffffff` + 边框色底，按 4 单位（ASCII 1、CJK 2）截断

### 2.4 控件真实范围（来自 `options.html`，演示必须一致）

| 控件 | min | max | step | 默认 |
|---|---|---|---|---|
| fontSize | 10 | 80 | 1 | 24 |
| opacity | 0.03 | 1 | 0.01 | 0.15 |
| density | 120 | 600 | 10 | 300 |
| rotation | -90 | 90 | 5 | -30 |
| border-width | 1 | 12 | 1 | 4 |
| fade-opacity | 0 | 0.5 | 0.01 | 0.03 |
| fade-resume | 300 | 8000 | 100 | 2000 |

规则优先级由**打分**决定（`findMatches` 取分值最高者），并非固定链条。基础分：

| 规则类型 | 基础分 | 加成 |
|---|---|---|
| `host-exact` | 1000 | + 目标长度 |
| `ip-exact` | 900 | — |
| `ip-cidr` | 800 | — |
| `url-regex` | 700 | + min(正则长度, 上限) |
| `cookie` | 650 / 600 / 550 | + 键长（三种匹配形态分值不同） |
| `host-suffix` | 500 | + 目标长度 |

由此得出的实际优先级顺序：`host-exact` → `ip-exact` → `ip-cidr` → `url-regex` → `cookie` → `host-suffix`。
**站点发布优先级表时必须说明是打分制**，因为同类型规则间会因长度加成而互相压过 —— 更长更具体的匹配胜出。

### 2.5 GitHub Pages 支持无扩展名 URL

已实测 `/examples` 与 `/examples.html` 均返回 200。关键词路径用 `dir/index.html` 即可，无需 Jekyll 或构建步骤。需注意重复内容：canonical 必须指向单一形式。

### 2.6 CJK 占比（决定字体子集策略）

| 页面 | CJK | Latin | CJK 占比 |
|---|---|---|---|
| `index.html` | 4 | 2965 | 0% |
| `index.zh.html` | 882 | 383 | **70%** |
| `examples.html` | 76 | 2323 | 3% |
| `examples.zh.html` | 528 | 956 | 36% |
| `privacy-policy.html` | 402 | 1765 | 19% |

---

## 3. 视觉系统

### 3.1 方向：Technical / mono-flavored

不是任意选择：扩展 `options.css` 本就使用 Inter + JetBrains Mono，该方向即产品既有身份。mono 用于 UI chrome（eyebrow 标签、规则类型、badge、JSON），sans 用于正文。

### 3.2 字体：自托管 + 系统兜底（方案 B）

**决定**：Inter 与 JetBrains Mono 的 woff2 放入仓库 `assets/fonts/`，同域加载，`font-display: swap`，系统栈兜底。

不用 Google Fonts CDN 的理由：访客 IP 暴露给第三方，且 `fonts.googleapis.com` 国内常超时 —— `index.zh.html` 有 70% 中文内容，读者大概率在墙内。自托管同样是「网络加载 + 系统兜底」，只是服务器换成自己的。

```css
@font-face {
  font-family: 'Inter';
  src: url('./assets/fonts/inter-latin-400.woff2') format('woff2');
  font-weight: 400; font-display: swap;
  unicode-range: U+0000-00FF, U+2000-206F, U+2190-21BB;
}
/* 同上分别声明 600 / 700，以及 JetBrains Mono 400 / 600 */

--sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI',
        'PingFang SC', 'Microsoft YaHei', sans-serif;
--mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, Consolas, monospace;
```

硬性要求：

1. **必须限定 `unicode-range`** — Inter 不含 CJK 字形。不加限制，中文页会下载整套拉丁字体却一个字用不上；加了之后中文页几乎不产生字体请求，CJK 自动落到 PingFang SC / 微软雅黑。
2. **字重只取 400 / 600 / 700** — 设计中出现的 650 / 680 必须归拢到这三档。
3. 使用 `swap`，**不用 `optional`**（否则慢网络永远看不到 Inter）。
4. 首屏字体加 `<link rel="preload" as="font" type="font/woff2" crossorigin>`。
5. 仓库内附 `assets/fonts/OFL.txt` — Inter 与 JetBrains Mono 均为 SIL OFL，可自由分发。

### 3.3 Token

```
字号   14 / 18(正文) / 20 / 28 / 40 / 60
行高   正文 1.6，display 1.05–1.15
字距   display -0.025em，正文 0
颜色   accent #4f46e5（克制使用）
       prod #dc2626 / test #16a34a —— 严格语义化，绝不作装饰
       bg #ffffff，panel #f8fafc，text #0f172a，muted #64748b，border #e2e8f0
留白   section 96px（桌面）/ 64px（移动）
宽度   正文 58–68ch，容器 1180px
```

**背景保持纯白 + `#f8fafc` 面板**（不用暖白）：页面充满浏览器窗口模型，必须让它们读作真实的 Chrome 窗口。

红/绿保持语义是对「单一强调色」通则的有意例外 —— 在本产品中，生产/测试配色本身就是内容。

---

## 4. 页面结构：按意图分三页

每页只保留**一个**独特手法。

### 4.1 首页 `/` — 意图：environment indicator

**独特手法**：可实操的双窗口装置本身。不是首屏配图，而是能玩的产品，独占首屏。

| # | 分节 | 要点 |
|---|---|---|
| 01 | 首屏 | H1 + 双窗口实操装置 + 强度条 + 4 个预设。CTA 保持 `<button disabled aria-disabled="true">` |
| 02 | 哪一个是生产环境？ | 答案前置段 + 序号槽列表三情况（共用根域名 / 裸 IP / 灰度） |
| 03 | 六种规则类型 | mono 表格，**首次显式写出优先级顺序** |
| 04 | 智能对比色 | difference 真实演示 + 基调切换，作为差异化证据 |
| 05 | 其余能力 | 沉浸式边框 / 交互淡出 / 工具栏角标 + **规则测试器、全局总开关**（页面从未提及） |
| 06 | 隐私 | 删除 CDN 后「零网络请求」成为事实 |
| 07 | 问答 | 可见 `<h3>` 提问 + 自包含 40–75 词回答 |
| 08 | 真实截图 | 置于最后作佐证，**明确标注为截图**，绝不与 CSS 演示混淆 |

首屏装置技术要求：

- 两个**完整独立**的浏览器窗口，各有标签栏、地址栏、扩展图标、角标
- 共用一套控件，拖动即时重绘两窗口
- 水印文字用 `PROD / TEST` 斜杠分隔分别喂给两窗口，角标同步按 4 单位截断
- 视口容器必须 `isolation: isolate`（否则 `difference` 会穿透祖先层混合）
- 水印强度条：不透明度 55% + 字号 25% + 密度倒数 20% 加权，显示「很低调 / 适中 / 明显 / 非常强烈」
- 淡出只在窗口范围内监听，**文案须说明「打字」在演示中试不出来**（非可聚焦 div 收不到 `keydown`），避免访客以为坏了

### 4.2 对比页 `/vs-environment-marker-alternatives` — 意图：比较

**独特手法**：超大 mono 序号 + 一张**诚实的**能力表，包含我们不如对手的格子。

| # | 分节 |
|---|---|
| 01 | 一句话结论（谁该选、谁不该，40–75 词） |
| 02 | 能力对照表 |
| 03 | 我们独有：IP/CIDR、Cookie 匹配、difference 智能对比色 |
| 04 | 他们更强：Environment Marker 自 2015 年运营、周活 35,000、Google「精选」；我们尚未上架 |
| 05 | 迁移说明 |

> **⚠ 全案最大事实风险**
> 竞品调研只取得了用户量、评分与大致定位，**未逐项验证它们支持哪些匹配方式**。写错对手比写错自己更糟。
> **硬前置任务**：实施对比页之前，必须实际安装 Environment Marker、Environment Indicator、URLColors 三者并逐项实测。**验证不了的行直接删除，不得猜测。**

### 4.3 示例页 `/examples` — 意图：配置

**独特手法**：序号超大悬挂左槽，JSON 配置本身作视觉纹理。

| # | 分节 |
|---|---|
| 01 | 五个示例（沿用 `EXAMPLES.md`；中英版在示例 2–3 内容本就不同，保持） |
| 02 | 优先级说明 + 真实顺序表 |
| 03 | 规则测试器（现有页面完全未提） |
| 04 | 导入 / 导出，团队共享配置 |

三页全部双语，共 6 个文件。

---

## 5. URL 与迁移

新结构（keyword-rich 路径为副产品，非改版理由 —— Mueller 称 URL 关键词是「very very lightweight」因素）：

| 新 URL | 语言 | 取代 |
|---|---|---|
| `/` | en | `index.html` |
| `/zh/` | zh-Hans | `index.zh.html` |
| `/examples` | en | `examples.html` |
| `/zh/examples` | zh-Hans | `examples.zh.html` |
| `/vs-environment-marker-alternatives` | en | 新增 |
| `/zh/vs-environment-marker-alternatives` | zh-Hans | 新增 |
| `/privacy-policy.html` | zh-Hans（JS 切换） | 保持不变 |

处理要求：

1. 每页 canonical 自指新 URL；hreflang 三向（en / zh-Hans / x-default）
2. 旧 `.html` 路径保留为**含 canonical 指向新 URL 的薄页**，避免既有外链与 GSC 收录 404（GitHub Pages 无服务端 301 能力）
3. `sitemap.xml` 重新生成，含 6 个新 URL + `privacy-policy.html`
4. README 与 README.zh_CN 的站点链接复核
5. JSON-LD 规则不变：仅 landing 页放 `SoftwareApplication` + `Organization`，支撑页零 JSON-LD；**不加 FAQPage / HowTo**，问答靠可见 `<h3>` + `<p>` 实现
6. `inLanguage` 保持 `["en","zh-CN","zh-TW","ja","es"]`（扩展 locale 列表，不改为 zh-Hans）

---

## 6. 文案修正（现有页面的事实错误）

| 位置 | 现状 | 改为 |
|---|---|---|
| 特性区淡出说明 | "fades during typing and clicking" | 移动鼠标 / 打字 / 滚轮 / 触屏拖动 —— **源码无 click 监听** |
| 功能清单 | 未提规则测试器 | 补充：保存前用 URL + Cookie 验证规则 |
| 功能清单 | 未提全局总开关 | 补充：一键停用全部水印 |

定位词调整：**environment indicator 为主导词，watermark 作为机制说明**。不改扩展商店名。

---

## 7. 扩展侧改动（与站点分开提交）

删除 `options.html:8` 的 Google Fonts `<link>`（含第 7–10 行整个标签）。该行是**整个扩展唯一的对外请求**；删除后隐私承诺成为事实，字体靠系统栈兜底。

注意：扩展与站点在字体上采取**相反策略**且均正确 —— 站点是营销页，自托管字体无隐私承诺冲突；扩展是本地隐私工具，其隐私页明确承诺不联网。

---

## 8. 不变的约束（继承自 v1）

- `djimnchdlbbedppeedlcmebbmehcloeb` 是**竞品**扩展 ID，仅作本地源码目录名。**永不链接或引用。**
- 扩展未上架 → 主 CTA 必须是 `<button disabled aria-disabled="true">`，**绝不用 `<a>`**，无 Chrome Web Store 链接
- 零变现：无 Pro / Premium / Pricing / Upgrade / Sponsor / Donate / 专业版 / 定价 / 赞助 / 捐赠，`$` 不作价格。合法例外：JSON 示例中 `$` 作正则结束锚点；`付费` 仅出现在「不存在付费版本」的否认句中
- 无 llms.txt
- 逐页内联 `<style>` 重复是**刻意且已批准**的做法，不得作为缺陷提出
- CSS 承重项：`nav.topbar a.active`（`nav` 前缀必需 —— 深色 `@media` 位于基础规则之前，(0,2,2) 胜 (0,2,1)）；`.rule-table th, td` 的 `overflow-wrap: anywhere`；`.btn-primary:disabled` 使用 `var(--muted)` 且豁免 WCAG；文档页用 `<header class="doc-head">` 左对齐，**不用 `header.hero`**
- 页脚 `© 2026 jinnersun · MIT Licensed`；反馈指向 `https://github.com/jinnersun/web-watermark-prompt/issues`
- 手写 HTML，零依赖，零构建
- 已接受的偏差：示例 1 标题省略「（核心用例）」；`examples.zh.html` 4 处散文使用「」；EN/ZH JSON `text` 值在示例 2–3 合理不同

---

## 9. 交付顺序

1. 竞品实测（对比页硬前置）
2. 字体子集与自托管落地 + OFL
3. 首页（en → zh）
4. 示例页（en → zh）
5. 对比页（en → zh）
6. 旧 URL canonical 薄页 + sitemap 重建
7. 扩展 `options.html:8` 删除（独立提交）
8. 全站审计（EOL / BOM / lang / canonical / 无外链泄漏）
9. **最后**：`jinnersun.github.io` 根仓库 + `robots.txt` + 极简 `index.html`

## 10. 仍待人工完成

- **GSC 注册**：URL-prefix 属性，`https://jinnersun.github.io/web-watermark-prompt/`，HTML 文件验证（文件已确认可访问），随后提交 `sitemap.xml`
- **视觉核验**：子代理无浏览器，渲染从未经人眼确认。需人工检查深色模式 nav 药丸可读性、360px 表格换行、示例 4 长正则横向滚动、等宽 `pre` 中的 CJK 字形与引号
- 遗留：meta description 偏长（`index.html` 169 字符、`examples.html` 162）可能在 SERP 截断
