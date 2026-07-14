# Real-World Examples

> English | [中文](./EXAMPLES.zh_CN.md)

Five scenarios showing how a user's description maps to a JSON config. Use these as few-shot references when helping users generate configs.

---

## Example 1 — Same-domain multi-environment (the core use case)

**User input:**

> I have three environments sharing the same base domain:
> - `https://app.example.com/` is production
> - `https://test.app.example.com/` is test
> - `https://staging.app.example.com/` is pre-production
>
> I need distinct watermarks so I don't accidentally operate on the wrong one.

**Expected output:**

```json
[
  {
    "name": "生产环境",
    "shortLabel": "PROD",
    "rules": [{ "type": "host-exact", "value": "app.example.com" }],
    "text": "生产环境 - 请谨慎操作",
    "color": "#ef4444",
    "opacity": 0.15,
    "border": { "enabled": true, "color": "#ef4444", "width": 4 }
  },
  {
    "name": "准生产",
    "shortLabel": "SIM",
    "rules": [{ "type": "host-exact", "value": "staging.app.example.com" }],
    "text": "准生产环境",
    "color": "#f59e0b",
    "opacity": 0.15
  },
  {
    "name": "测试环境",
    "shortLabel": "TEST",
    "rules": [{ "type": "host-exact", "value": "test.app.example.com" }],
    "text": "测试环境",
    "color": "#10b981",
    "opacity": 0.15
  }
]
```

**Why**: Each hostname is distinct, so `host-exact` is the most precise rule. Production gets red + inset border (highest alarm). Pre-production amber. Test green.

---

## Example 2 — VPN internal admin over IP

**User input:**

> Our admin panel is only accessible over the office VPN at `https://192.0.2.5/`. There's no domain, just the IP. I want a purple watermark saying "Admin — Restricted".

**Expected output:**

```json
[
  {
    "name": "管理后台",
    "shortLabel": "ADMIN",
    "rules": [{ "type": "ip-exact", "value": "192.0.2.5" }],
    "text": "Admin — Restricted\n仅授权人员访问",
    "color": "#8b5cf6",
    "opacity": 0.18,
    "border": { "enabled": true, "color": "#8b5cf6", "width": 3 }
  }
]
```

**Why**: Browser hostname is literally `192.0.2.5`, so `ip-exact` is the right rule. Border ON to reinforce the "restricted" feel.

---

## Example 3 — Entire internal subnet

**User input:**

> Everything on our internal network `10.0.0.0/8` should get a "INTERNAL" watermark. External sites should not.

**Expected output:**

```json
[
  {
    "name": "内部网络",
    "shortLabel": "INT",
    "rules": [{ "type": "ip-cidr", "value": "10.0.0.0/8" }],
    "text": "INTERNAL NETWORK",
    "color": "#6366f1",
    "opacity": 0.12,
    "rotation": -25
  }
]
```

**Why**: `10.0.0.0/8` is a Class A private subnet — 16 million addresses. Only matches when the user is accessing a page whose hostname is a raw IPv4 in that range (typical for internal-only tools).

---

## Example 4 — Path-based admin routing on production

**User input:**

> On our production site `app.example.com`, the entire main app is normal but I want a big red DANGER watermark on any URL starting with `/admin`. Regular pages should have no watermark.

**Expected output:**

```json
[
  {
    "name": "生产管理页",
    "shortLabel": "ADM",
    "rules": [
      { "type": "url-regex", "value": "^https://app\\.example\\.com/admin(/.*)?$" }
    ],
    "text": "DANGER — 生产管理页面\n此页可修改线上数据",
    "color": "#dc2626",
    "opacity": 0.2,
    "fontSize": 28,
    "border": { "enabled": true, "color": "#dc2626", "width": 5 }
  }
]
```

**Why**: The user wants **path-based** matching, not host-based. `url-regex` is the only rule type that inspects the full URL. Backslashes in JSON escape the `.` in the regex.

---

## Example 5 — Cookie-driven canary environment

**User input:**

> We have a canary deployment where the URL is the same as production (`app.example.com`) but the backend routes based on a cookie `deploy=canary`. When I'm on the canary, I want an orange "CANARY" watermark.

**Expected output:**

```json
[
  {
    "name": "灰度环境",
    "shortLabel": "CNRY",
    "rules": [{ "type": "cookie", "value": "deploy=canary" }],
    "text": "CANARY 灰度环境",
    "color": "#f97316",
    "opacity": 0.15
  }
]
```

**Why**: Same URL, same host — the only distinguisher is the cookie. `cookie: deploy=canary` matches exact value. If the value could vary (e.g. `canary-v1`, `canary-v2`), use `deploy~=canary` (contains) instead.

---


---

## Example 6 — Empty or vague input → AI asks back (no JSON)

**User input (empty / only the placeholder):**

> `(describe your production / test / staging / VPN environments here)`

**Expected output (NOT a JSON array):**

> Sure — could you share a bit more so I can generate the right config?
>
> 1. How many environments do you need to distinguish (e.g. prod / staging / test / VPN)?
> 2. What identifies each one — a hostname, full URL, IP, or a Cookie flag?
> 3. Any preferred watermark text and color, or should I use the defaults (prod=red, staging=amber, test=green, VPN=violet)?
> 4. Any special needs — inset border, or a short badge label like `PROD` / `TEST`?
>
> Reply with the details and I'll return the JSON.

**Why**: The input has no hostname / URL / IP / cookie, so the assistant must **not** invent one. It falls back to a short bulleted question list in the user's language (English here because the message is English). Once the user answers concretely, generate the JSON on the next turn.

## Notes for AI assistants

- If the user's scenario doesn't match any example pattern above, **combine ideas from multiple examples**
- If unsure about the rule type, **ask a clarifying question first** rather than guessing
- Text and names should match the user's language — if they wrote in Chinese, output Chinese `text` and `name`
- Colors follow the environment convention in `PROMPT.md` — override only if user asks
