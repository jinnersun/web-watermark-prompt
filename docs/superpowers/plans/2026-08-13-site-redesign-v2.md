# 站点改版 v2 实施计划

> **给执行者：** 必备子技能 —— 使用 superpowers:subagent-driven-development 或 superpowers:executing-plans 按任务逐条实施。步骤使用 `- [ ]` 复选框跟踪。

**目标：** 把已上线的 v1 双语站点改版为「environment indicator」定位、视觉留白充足、含可实操演示的两页双语站点，并部署到 GitHub Pages。

**架构：** 手写静态 HTML，逐页内联 `<style>`（刻意重复，已批准），零依赖零构建。字体自托管于 `assets/fonts/`，`unicode-range` 限定拉丁范围，系统栈兜底。首屏为纯 CSS + canvas 的可交互演示，逐行复现扩展真实渲染算法。

**技术栈：** HTML5 / CSS3 / 原生 JS（无框架）；fontTools 4.60.1 + brotli 1.2.0 做字体子集；GitHub Pages 部署。

**规范来源：** `docs/superpowers/specs/2026-08-13-site-redesign-v2.md`

## 全局约束

- `djimnchdlbbedppeedlcmebbmehcloeb` 是**竞品**扩展 ID，仅作本地源码目录名。**永不链接或引用。**
- 扩展未上架 → 主 CTA 必须是 `<button disabled aria-disabled="true">`，**绝不用 `<a>`**，无 Chrome Web Store 链接。
- 零变现：无 Pro / Premium / Pricing / Upgrade / Sponsor / Donate / 专业版 / 定价 / 赞助 / 捐赠；`$` 不作价格。合法例外：JSON 示例中 `$` 作正则结束锚点。
- 无 llms.txt；不加 FAQPage / HowTo JSON-LD。仅 landing 页放 `SoftwareApplication` + `Organization`，支撑页零 JSON-LD。
- `inLanguage` 保持 `["en","zh-CN","zh-TW","ja","es"]`（扩展 locale 列表，**不改为 zh-Hans**）。
- 逐页内联 `<style>` 重复是**刻意且已批准**的，不得作为缺陷提出。
- 页脚固定 `© 2026 jinnersun · MIT Licensed`；反馈指向 `https://github.com/jinnersun/web-watermark-prompt/issues`。
- 所有文件 **LF 换行、无 BOM**。`.gitattributes` 已含 `* text=auto eol=lf`，但编辑工具曾两次静默写入 CRLF，**每次提交前必须字节级复核**。
- 颜色 token：accent `#4f46e5`、prod `#dc2626`、test `#16a34a`、text `#0f172a`、muted `#64748b`、border `#e2e8f0`、panel `#f8fafc`、bg `#ffffff`。红/绿严格语义化，不作装饰。
- 字号阶梯 14 / 18(正文) / 20 / 28 / 40 / 60；section 留白 96px 桌面 / 64px 移动；正文宽 58–68ch；容器 1180px。
- 字重只用 Inter 400/600/700 与 JetBrains Mono 400，**不得出现 650 / 680**。
- Windows PowerShell 5.1：`&&` 不可用，用 `;` 或 `if ($?) { }`。路径含 `chrome插件` 必须双引号。Python 命令是 `py`，`rg` 未安装。
- **禁止多层引号一行流**，改写为 `tmp-*.py` 脚本文件执行；脚本用完即删，不得提交。
- 禁止用 PowerShell 管道传输 UTF-8/二进制（`git show | py` 会损坏字节），改用 `subprocess` + `git cat-file`。

---

## 文件结构

| 文件 | 职责 | 状态 |
|---|---|---|
| `assets/fonts/*.woff2` | 4 个拉丁子集字体（76.1 KB） | 新建 |
| `assets/fonts/OFL.txt` | Inter + JetBrains Mono 许可证 | 新建 |
| `index.html` | 英文首页，含可实操演示 | 重写 |
| `zh/index.html` | 中文首页 | 新建 |
| `examples.html` | 英文示例页 | 重写 |
| `zh/examples.html` | 中文示例页 | 新建 |
| `index.zh.html` | 旧 URL → canonical 薄页 | 改为薄页 |
| `examples.zh.html` | 旧 URL → canonical 薄页 | 改为薄页 |
| `privacy-policy.html` | 隐私政策 | 仅更新导航/字体 |
| `sitemap.xml` | 5 条 URL | 重新生成 |

---

### Task 1: 字体子集与自托管

**Files:**
- Create: `assets/fonts/inter-latin-400.woff2`, `inter-latin-600.woff2`, `inter-latin-700.woff2`, `jetbrains-mono-latin-400.woff2`, `assets/fonts/OFL.txt`

**Interfaces:**
- Produces: 4 个 woff2 文件路径，供后续所有页面的 `@font-face` 引用；`--sans` / `--mono` CSS 变量定义。

- [ ] **Step 1: 下载并解压源字体到临时目录**

```powershell
$tmp="C:\Users\ZhuanZ\AppData\Local\Temp\opencode\fonts"; New-Item -ItemType Directory -Path $tmp -Force | Out-Null
$ProgressPreference='SilentlyContinue'
Invoke-WebRequest -Uri "https://github.com/rsms/inter/releases/download/v4.1/Inter-4.1.zip" -OutFile "$tmp\inter.zip" -TimeoutSec 120
Invoke-WebRequest -Uri "https://github.com/JetBrains/JetBrainsMono/releases/download/v2.304/JetBrainsMono-2.304.zip" -OutFile "$tmp\jb.zip" -TimeoutSec 120
```

- [ ] **Step 2: 生成子集（写成脚本文件，勿用一行流）**

创建 `C:\Users\ZhuanZ\AppData\Local\Temp\opencode\mkfonts.py`：

```python
import os, zipfile, shutil, subprocess
TMP = r'C:\Users\ZhuanZ\AppData\Local\Temp\opencode\fonts'
OUT = r'D:\item\chrome插件\web-watermark-prompt\assets\fonts'
os.makedirs(OUT, exist_ok=True)
WORK = os.path.join(TMP, 'work'); os.makedirs(WORK, exist_ok=True)

SRC = {
    'inter.zip': {'Inter-Regular.ttf': 'inter-latin-400.woff2',
                  'Inter-SemiBold.ttf': 'inter-latin-600.woff2',
                  'Inter-Bold.ttf': 'inter-latin-700.woff2'},
    'jb.zip': {'JetBrainsMono-Regular.ttf': 'jetbrains-mono-latin-400.woff2'},
}
for zf, mapping in SRC.items():
    with zipfile.ZipFile(os.path.join(TMP, zf)) as z:
        for base, out_name in mapping.items():
            cand = [n for n in z.namelist()
                    if n.endswith('/' + base) and 'Variable' not in n]
            assert cand, 'not found: ' + base
            with z.open(cand[0]) as fs, open(os.path.join(WORK, base), 'wb') as fd:
                shutil.copyfileobj(fs, fd)

UNI = 'U+0000-00FF,U+2000-206F,U+2190-21BB'
for zf, mapping in SRC.items():
    for base, out_name in mapping.items():
        r = subprocess.run(['py', '-m', 'fontTools.subset',
                            os.path.join(WORK, base), '--unicodes=' + UNI,
                            '--layout-features=kern,liga', '--flavor=woff2',
                            '--output-file=' + os.path.join(OUT, out_name)],
                           capture_output=True, text=True)
        assert r.returncode == 0, r.stderr[:300]
        print('%-32s %6.1f KB' % (out_name, os.path.getsize(os.path.join(OUT, out_name)) / 1024))
```

运行：`py "C:\Users\ZhuanZ\AppData\Local\Temp\opencode\mkfonts.py"`

- [ ] **Step 3: 验证体积与字形覆盖**

预期输出（允许 ±1 KB 浮动）：

```
inter-latin-400.woff2              20.2 KB
inter-latin-600.woff2              20.6 KB
inter-latin-700.woff2              20.6 KB
jetbrains-mono-latin-400.woff2     14.6 KB
```

创建校验脚本断言：每个子集含站点所需全部 ASCII 与 `→ § © ·`；**且不含任何 CJK 码位**（`0x4E00–0x9FFF` 为空）。

- [ ] **Step 4: 写入 OFL.txt**

内容为 SIL Open Font License 1.1 全文，头部标注两款字体的版权行：
`Copyright (c) 2016 The Inter Project Authors` 与 `Copyright (c) 2020 The JetBrains Mono Project Authors`。

- [ ] **Step 5: 确认 .gitattributes 已把 woff2 当二进制**

`.gitattributes` 必须含 `*.woff2 binary`（若无则追加），否则 `text=auto` 可能损坏字体文件。

- [ ] **Step 6: 提交**

```powershell
git add .gitattributes assets/fonts
git commit -m "feat(fonts): self-host Inter + JetBrains Mono latin subsets"
```

---

### Task 2: 英文首页

**Files:**
- Modify: `index.html`（整页重写）

**Interfaces:**
- Consumes: Task 1 的 4 个字体文件。
- Produces: `buildTile()` / `render()` / `paint()` JS 函数与 `.viewport` / `.wm` / `.bd` CSS 类，Task 3 的中文页原样复用。

- [ ] **Step 1: 写 `<head>`：字体、meta、JSON-LD**

`@font-face` 四条声明，均带 `font-display: swap` 与
`unicode-range: U+0000-00FF, U+2000-206F, U+2190-21BB;`
`<link rel="preload" as="font" type="font/woff2" crossorigin>` 只给 `inter-latin-400.woff2`。
canonical 指向 `https://jinnersun.github.io/web-watermark-prompt/`，hreflang 三向（en / zh-Hans / x-default）。
JSON-LD 仅 `SoftwareApplication` + `Organization`，`inLanguage` 保持五语数组。

- [ ] **Step 2: 首屏 —— H1 + 双窗口可实操演示**

H1 使用 environment indicator 主导词。演示技术要求：

- 两个**完整独立**浏览器窗口，各有标签栏、地址栏、扩展图标、角标
- `.viewport { position: relative; isolation: isolate; overflow: hidden; }` —— `isolation` 必需，否则 `mix-blend-mode: difference` 会穿透祖先层
- `.wm { z-index: 2 }`、`.bd { z-index: 3 }` —— 边框必须压在水印之上，否则边框像素被 difference 反色算歪
- CTA：`<button class="btn-primary" disabled aria-disabled="true">`

`buildTile()` 必须逐行复现 `watermark-core.js`：

```js
var gap = Math.max(50, parseInt(density, 10) || 300);
var dpr = Math.min(2, window.devicePixelRatio || 1);
canvas.width = gap * dpr; canvas.height = gap * dpr;
ctx.scale(dpr, dpr);
ctx.font = fontSize + 'px ' + fontFamily;
ctx.fillStyle = color; ctx.globalAlpha = parseFloat(opacity);
ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
ctx.translate(gap / 2, gap / 2);
ctx.rotate((rotation * Math.PI) / 180);
var lineHeight = fontSize * 1.2;
var startY = -((lines.length - 1) * lineHeight) / 2;
```

智能色：`light` → `#d1d5db`，`dark` → `#1f2937`；smart 模式下 canvas `globalAlpha = 1`，CSS `opacity` 承担透明度。

- [ ] **Step 3: 控制面板，范围必须与 `options.html` 全等**

| 控件 | min | max | step | 默认 |
|---|---|---|---|---|
| fontSize | 10 | 80 | 1 | 24 |
| opacity | 0.03 | 1 | 0.01 | 0.15 |
| density | 120 | 600 | 10 | 300 |
| rotation | -90 | 90 | 5 | -30 |
| border-width | 1 | 12 | 1 | 4 |
| fade-opacity | 0 | 0.5 | 0.01 | 0.03 |
| fade-resume | 300 | 8000 | 100 | 2000 |

强度条权重：不透明度 55% + 字号 25% + 密度倒数 20%，档位「很低调 / 适中 / 明显 / 非常强烈」。
淡出监听 `mousemove` / `wheel` / `touchmove`，**不监听 click**；文案须说明「打字」在演示中试不出来（非可聚焦 div 收不到 `keydown`）。

- [ ] **Step 4: 其余 7 节**

02 哪一个是生产环境（答案前置 + 序号槽三情况）
03 六种规则类型 + **打分制**优先级表（host-exact 1000 / ip-exact 900 / ip-cidr 800 / url-regex 700 / cookie 650·600·550 / host-suffix 500，各含长度加成）
04 智能对比色（difference 真实演示 + 基调切换）
05 其余能力：沉浸式边框、交互淡出、工具栏角标、**规则测试器**、**全局总开关**
06 隐私（删 CDN 后「零网络请求」成为事实）
07 问答（可见 `<h3>` + 自包含 40–75 词回答）
08 真实截图作佐证，**明确标注为截图**

- [ ] **Step 5: 验证**

脚本断言：滑块范围 7/7 与 `options.html` 全等；默认值 4/4 与 `makeDefaultConfig()` 一致；无 `650`/`680` 字重；无外部域名引用；CTA 为 disabled button 而非 `<a>`；LF 无 BOM。

- [ ] **Step 6: 提交**

```powershell
git add index.html
git commit -m "feat(site): rebuild the English home page around environment indicator"
```

---

### Task 3: 中文首页

**Files:**
- Create: `zh/index.html`

**Interfaces:**
- Consumes: Task 2 的 CSS 与 JS，原样复制后仅替换文案。

- [ ] **Step 1: 复制 Task 2 结构，替换全部文案为简体中文**

`<html lang="zh-Hans">`。canonical 指向 `.../web-watermark-prompt/zh/`。字体路径改为 `../assets/fonts/`。

- [ ] **Step 2: 中文特有校验**

CJK 由系统字体承担 —— 确认 `--sans` 栈内 `'PingFang SC'`、`'Microsoft YaHei'` 在 Inter 之后。
等宽 `pre` 内的 CJK 与「」引号需人工目视复核（记入待办）。

- [ ] **Step 3: 验证并提交**

断言：`lang="zh-Hans"`；canonical 自指；hreflang 三向；LF 无 BOM；无变现词。

```powershell
git add zh/index.html
git commit -m "feat(site): add the Chinese home page"
```

---

### Task 4: 示例页（中英）

**Files:**
- Modify: `examples.html`
- Create: `zh/examples.html`

**Interfaces:**
- Consumes: Task 1 字体；Task 2 的 token 与排版类。

- [ ] **Step 1: 英文示例页**

内容源 `EXAMPLES.md` 五个示例。独特手法：序号超大悬挂左槽，JSON 配置作视觉纹理。
新增两节：**规则测试器**（保存前用 URL + Cookie 验证）与优先级打分说明。
`.rule-table th, .rule-table td` 必须保留 `overflow-wrap: anywhere`。
支撑页**零 JSON-LD**。

- [ ] **Step 2: 中文示例页**

内容源 `EXAMPLES.zh_CN.md`。**示例 2–3 中英内容本就不同，保持差异**，不得强行统一。

- [ ] **Step 3: 验证并提交**

断言：两页均无 JSON-LD；canonical 自指新路径；JSON 代码块中的 `$` 仅作正则锚点；LF 无 BOM。

```powershell
git add examples.html zh/examples.html
git commit -m "feat(site): rebuild the examples pages with the rule tester"
```

---

### Task 5: 旧 URL 薄页与 sitemap

**Files:**
- Modify: `index.zh.html`, `examples.zh.html`, `sitemap.xml`, `privacy-policy.html`

- [ ] **Step 1: 把两个旧中文页改为 canonical 薄页**

GitHub Pages 无服务端 301 能力，故保留文件但改为极简页：`<link rel="canonical">` 指向新 URL + `<meta http-equiv="refresh" content="0; url=...">` + 一句可见说明与链接。**不得留下重复正文**（会构成重复内容）。

- [ ] **Step 2: 重新生成 sitemap.xml**

5 条 URL：`/`、`/zh/`、`/examples`、`/zh/examples`、`/privacy-policy.html`。
旧薄页**不入 sitemap**。

- [ ] **Step 3: privacy-policy.html 同步字体与导航**

仅改 `@font-face` 引用与顶部导航链接，保留 `<header class="doc-head">` 左对齐（**不用 `header.hero`**）与自指 canonical。

- [ ] **Step 4: 验证并提交**

断言：sitemap 恰含 5 条且均可解析；薄页含 canonical 且正文 < 500 字节；`privacy-policy.html` 的 `documentElement.lang` 仍输出 `zh-Hans`。

```powershell
git add index.zh.html examples.zh.html sitemap.xml privacy-policy.html
git commit -m "chore(seo): retire legacy URLs to canonical stubs and rebuild sitemap"
```

---

### Task 6: 删除扩展的 Google Fonts 链接

**Files:**
- Modify: `D:\item\chrome插件\djimnchdlbbedppeedlcmebbmehcloeb\src\options.html:7-10`

**注意：** 这是**扩展仓库**的改动，与站点分开提交。

- [ ] **Step 1: 删除整个 `<link>` 标签（第 7–10 行）**

```html
<link
  href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
  rel="stylesheet"
/>
```

- [ ] **Step 2: 验证扩展已零对外请求**

脚本断言：全 `src/` 目录内 `fonts.googleapis` / `fonts.gstatic` / `@import url` 出现次数为 **0**；`options.css` 的 `font-family` 仍以 `"Inter"` 开头（系统栈兜底，不需改动）。

- [ ] **Step 3: 提交（在扩展仓库内）**

```
fix(privacy): drop the Google Fonts CDN link

This was the extension's only outbound request, contradicting the
privacy policy's no-network claim. Inter now falls back to the system
font stack already declared in options.css.
```

---

### Task 7: 全站审计

- [ ] **Step 1: 字节级审计脚本**

对每个 `.html` / `.xml` 断言：无 CRLF、无 BOM、`lang` 属性正确、canonical 自指、无外部域名（除 schema.org / 自站 / GitHub issues）、无变现词、无竞品 ID、无商店链接、CTA 非 `<a>`。

- [ ] **Step 2: 承重 CSS 复核**

`nav.topbar a.active` 的 `nav` 前缀存在（深色 `@media` 在基础规则之前，(0,2,2) 胜 (0,2,1)）；`.btn-primary:disabled` 使用 `var(--muted)`；`.rule-table` 单元格有 `overflow-wrap: anywhere`。

- [ ] **Step 3: 字体引用路径正确性**

根目录页用 `./assets/fonts/`，`zh/` 下用 `../assets/fonts/`。断言所有引用文件真实存在。

- [ ] **Step 4: 提交修正（如有）**

---

### Task 8: 推送与线上验证

- [ ] **Step 1: 推送**

```powershell
git push origin main
```

- [ ] **Step 2: 等待 Pages 构建后逐 URL 验证**

断言全部返回 200：`/`、`/zh/`、`/examples`、`/zh/examples`、`/privacy-policy.html`、`/sitemap.xml`、4 个 woff2、`/googled3a11b0a36ad28b1.html`（须仍为 53 字节且字节精确）。
并断言线上 HTML 与本地磁盘**逐字节一致**。

- [ ] **Step 3: 字体实际可下载且 MIME 正确**

断言 woff2 返回 `content-type: font/woff2` 或 `application/octet-stream`，且体积与本地一致。

---

## 后续（不属本计划）

- **另立项目**：`jinnersun.github.io` 根仓库 + `robots.txt` + 极简 `index.html`。独立仓库，独立计划。
- **GSC 注册**（需人工）：URL-prefix 属性 `https://jinnersun.github.io/web-watermark-prompt/`，HTML 文件验证，提交 sitemap。
- **视觉目视核验**（需人工，无浏览器的代理无法完成）：深色模式 nav 药丸可读性、360px 表格换行、长正则横向滚动、等宽 `pre` 内 CJK 字形与引号。
