# Web Watermark Prompt · 网页水印工具 · 提示词仓库

> [English](./README.md) | 中文

一份可以直接给 AI 助手（Claude / ChatGPT / Codex / Cursor / Gemini）使用的提示词，帮助你为 **[Web Watermark Tool](https://chromewebstore.google.com/) 网页水印工具** Chrome 扩展生成配置。

## 这是什么？

Web Watermark Tool 通过灵活的匹配规则（域名 / URL 正则 / IP / Cookie），在网页上叠加自定义水印，帮助你在**生产 / 准生产 / 测试 / VPN 内网**等相似 URL 的环境之间一眼分辨——尤其当它们共用同一根域名的时候。

对新用户来说，要学 6 种规则类型 + Cookie 语法 + IP CIDR 有点繁琐。这个仓库提供一份提示词文件，你可以粘贴到任意 AI 助手里，用大白话描述你的场景，AI 就会返回一段可以直接导入扩展的 JSON 配置。

## 三种使用方式

### 方式 1 — 扩展内一键复制（推荐）

如果你装了 Web Watermark Tool **v2.0 或以上版本**：

1. 打开扩展的设置页
2. 点击 **🤖 让 AI 帮我写规则**
3. 提示词（本仓库的 `PROMPT.md`）+ 你当前的配置快照会被复制到剪贴板
4. 粘贴到 ChatGPT / Claude / Codex / Cursor
5. 用中文描述你的环境（例如 "我生产站是 app.example.com，测试站是 test.app.example.com……"）
6. AI 返回一段 JSON 数组
7. 回到扩展，点 **📋 从剪贴板导入**

### 方式 2 — 从本仓库手动复制

1. 打开 [`PROMPT.zh_CN.md`](./PROMPT.zh_CN.md)，全部复制
2. 作为第一条消息粘贴到 AI 对话里
3. 再用第二条消息描述你的场景
4. 把 AI 输出的 JSON 复制下来，在扩展里点 **导入** → 粘贴

### 方式 3 — 注册为 Skill / 项目指令

- **Claude Projects**：新建 Project → Custom Instructions → 粘贴 `PROMPT.zh_CN.md`
- **ChatGPT Custom GPTs**：Configure → Instructions → 粘贴 `PROMPT.zh_CN.md`
- **Cursor / Codex**：把 `PROMPT.zh_CN.md` 放到 `.cursor/rules/` 或 `AGENTS.md` 里
- 之后随时可以问："帮我给 cust.example.com 生成一个生产环境水印配置"

## 文件说明

| 文件 | 用途 |
|---|---|
| [`PROMPT.md`](./PROMPT.md) / [`PROMPT.zh_CN.md`](./PROMPT.zh_CN.md) | 完整的 system prompt。整份复制粘贴给 AI 即可 |
| [`EXAMPLES.md`](./EXAMPLES.md) / [`EXAMPLES.zh_CN.md`](./EXAMPLES.zh_CN.md) | 5 个真实场景的输入 → 输出示例，作为 few-shot 参考 |
| [`privacy-policy.html`](./privacy-policy.html) | Chrome Web Store 隐私政策页（用 GitHub Pages 挂载） |
| [`LICENSE`](./LICENSE) | MIT，可自由 fork / 修改 / 再分发 |

## 反馈与贡献

- AI 生成错误案例：提 [Issue](https://github.com/jinnersun/web-watermark-prompt/issues)
- 补充你自己的场景到 `EXAMPLES.md`：欢迎 PR
- 提示词优化建议：欢迎 PR

## 相关链接

- **Web Watermark Tool** Chrome 扩展：[Chrome Web Store 链接 — 上架中]
- **扩展源代码**：[v2.0 期间闭源，v3.0 后可能开源]
