# Web Watermark Prompt

> English | [中文](./README.zh_CN.md)

A ready-to-use prompt for AI assistants (Claude / ChatGPT / Codex / Cursor / Gemini) to help you generate configuration for the **[Web Watermark Tool](https://chromewebstore.google.com/) Chrome extension**.

## What is this?

The Web Watermark Tool matches web pages by rules (hostname / URL regex / IP / Cookie) and injects a customized watermark to help you tell **production / staging / test / VPN internal** environments apart at a glance — especially when they share the same domain.

For new users, learning 6 rule types + Cookie syntax + IP CIDR can be tedious. This repo provides a prompt file you can paste into any AI assistant, describe your scenario in plain language, and get back a JSON config ready to import into the extension.

## How to use

### Option 1 — In-extension one-click copy (recommended)

If you have Web Watermark Tool **v2.0 or later** installed:

1. Open the extension options page
2. Click **🤖 Let AI write rules for me**
3. The prompt (this repo's `PROMPT.md`) + your current config snapshot are copied to your clipboard
4. Paste into ChatGPT / Claude / Codex / Cursor
5. Describe your environment ("I have a production site at app.example.com and a test site at test.app.example.com…")
6. The AI replies with a JSON array
7. Back in the extension, click **📋 Import from clipboard**

### Option 2 — Manual copy from this repo

1. Open [`PROMPT.md`](./PROMPT.md), copy the whole thing
2. Paste it as the first message to your AI assistant
3. Then describe your scenario as a second message
4. Copy the JSON output, then in the extension click **Import** → paste

### Option 3 — Register as a Skill / Project instruction

- **Claude Projects**: New Project → Custom Instructions → paste `PROMPT.md`
- **ChatGPT Custom GPTs**: Configure → Instructions → paste `PROMPT.md`
- **Cursor / Codex**: add `PROMPT.md` to your `.cursor/rules/` or `AGENTS.md` file
- Then you can ask any time: "generate a watermark config for prod cust.example.com"

## Files

| File | Purpose |
|---|---|
| [`PROMPT.md`](./PROMPT.md) | The complete system prompt. Copy the whole file, paste to your AI. |
| [`EXAMPLES.md`](./EXAMPLES.md) | 5 real-world scenarios with input → output examples. Used as few-shot references. |
| [`LICENSE`](./LICENSE) | MIT. Free to fork, modify, redistribute. |

## Feedback & contributions

- Report AI misgeneration cases via [Issues](https://github.com/jinnersun/web-watermark-prompt/issues)
- Add your own scenario to `EXAMPLES.md` via PR
- Suggest prompt refinements to improve AI output quality

## Related

- **Web Watermark Tool** Chrome extension: [Chrome Web Store link — coming soon]
- **Source code**: [private for now, will open source after v3.0]
