# 真实场景示例

> [English](./EXAMPLES.md) | 中文

5 个真实场景，展示用户描述如何映射到 JSON 配置。作为 AI 生成配置时的 few-shot 参考。

---

## 示例 1 — 同域名多环境（核心用例）

**用户输入：**

> 我有 3 个共用同一个根域名的环境：
> - `https://cust.adentrd.com/` 是生产
> - `https://test.cust.adentrd.com/` 是测试
> - `https://bvi2sim.cust.adentrd.com/` 是准生产
>
> 我需要用不同的水印区分它们，防止误操作。

**期望输出：**

```json
[
  {
    "name": "生产环境",
    "shortLabel": "PROD",
    "rules": [{ "type": "host-exact", "value": "cust.adentrd.com" }],
    "text": "生产环境 - 请谨慎操作",
    "color": "#ef4444",
    "opacity": 0.15,
    "border": { "enabled": true, "color": "#ef4444", "width": 4 }
  },
  {
    "name": "准生产",
    "shortLabel": "SIM",
    "rules": [{ "type": "host-exact", "value": "bvi2sim.cust.adentrd.com" }],
    "text": "准生产环境",
    "color": "#f59e0b",
    "opacity": 0.15
  },
  {
    "name": "测试环境",
    "shortLabel": "TEST",
    "rules": [{ "type": "host-exact", "value": "test.cust.adentrd.com" }],
    "text": "测试环境",
    "color": "#10b981",
    "opacity": 0.15
  }
]
```

**为什么这么写**：三个域名各不相同，`host-exact` 是最精确的规则。生产用红色 + 沉浸式边框（最高警觉）。准生产橙色。测试绿色。

---

## 示例 2 — VPN 内网管理后台走 IP

**用户输入：**

> 我们的管理后台只能走公司 VPN 用 `https://10.20.30.5/` 访问，没有域名，直接是 IP。我想用紫色水印标注 "Admin — 受限区域"。

**期望输出：**

```json
[
  {
    "name": "管理后台",
    "shortLabel": "ADMIN",
    "rules": [{ "type": "ip-exact", "value": "10.20.30.5" }],
    "text": "Admin — 受限区域\n仅授权人员访问",
    "color": "#8b5cf6",
    "opacity": 0.18,
    "border": { "enabled": true, "color": "#8b5cf6", "width": 3 }
  }
]
```

**为什么这么写**：浏览器 hostname 就是 `10.20.30.5`，用 `ip-exact` 最合适。开启边框强化 "受限" 感。

---

## 示例 3 — 整个内网网段

**用户输入：**

> 我想让内网所有 `10.0.0.0/8` 段的地址都打上 "INTERNAL" 水印。外部站点不打。

**期望输出：**

```json
[
  {
    "name": "内部网络",
    "shortLabel": "INT",
    "rules": [{ "type": "ip-cidr", "value": "10.0.0.0/8" }],
    "text": "INTERNAL NETWORK\n内部网络",
    "color": "#6366f1",
    "opacity": 0.12,
    "rotation": -25
  }
]
```

**为什么这么写**：`10.0.0.0/8` 是 A 类私有子网，1600 万个地址。只在用户访问的页面 hostname 是该段内 IPv4 时命中（典型场景是纯内网工具）。

---

## 示例 4 — 生产站按路径路由的管理页

**用户输入：**

> 我的生产站 `cust.adentrd.com` 主应用是正常的，但只要 URL 以 `/admin` 开头的页面就要打大红色 DANGER 水印。普通页面不打。

**期望输出：**

```json
[
  {
    "name": "生产管理页",
    "shortLabel": "ADM",
    "rules": [
      { "type": "url-regex", "value": "^https://cust\\.adentrd\\.com/admin(/.*)?$" }
    ],
    "text": "DANGER — 生产管理页面\n此页可修改线上数据",
    "color": "#dc2626",
    "opacity": 0.2,
    "fontSize": 28,
    "border": { "enabled": true, "color": "#dc2626", "width": 5 }
  }
]
```

**为什么这么写**：用户要按**路径**匹配而非按 host，只有 `url-regex` 能处理完整 URL。JSON 里反斜杠转义 `.`。

---

## 示例 5 — Cookie 驱动的灰度环境

**用户输入：**

> 我们有一个灰度部署，URL 跟生产一样是 `cust.adentrd.com`，但后端会根据 `deploy=canary` 这个 cookie 路由。我在灰度上时，希望打橙色 "CANARY" 水印。

**期望输出：**

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

**为什么这么写**：URL、host 完全一样，只有 cookie 能区分。`deploy=canary` 是精确等值。如果 value 可能有多个（如 `canary-v1`、`canary-v2`），改用 `deploy~=canary`（包含）即可。

---

## AI 助手须知

- 用户场景不能完全套用上述示例时，**组合多个示例的思路**
- 拿不准规则类型时，**先反问澄清**再生成
- 文案和名称匹配用户使用的语言——用户写中文就输出中文的 `text` 和 `name`
- 颜色遵循 `PROMPT.zh_CN.md` 里的环境色规范，用户明确指定才覆盖
