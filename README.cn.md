<div align="center">

```
██████╗ ██╗   ██╗██╗ █████╗ ██████╗ ██╗
██╔══██╗██║   ██║██║██╔══██╗██╔══██╗██║
██║  ██║██║   ██║██║███████║██████╔╝██║
██║  ██║██║   ██║██║██╔══██║██╔═══╝ ██║
██████╔╝╚██████╔╝██║██║  ██║██║     ██║
╚═════╝  ╚═════╝ ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝
```

# duiapi-bulk-apikey

**批量创建 DuiAPI 账户并自动生成 API 密钥 — 兼容 OpenAI 的推理 + 9Router 连接器。**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 快速开始](#-quick-start) · [🎁 免费额度](#-free-tier--how-much-you-get) · [🤖 模型](#-models--all-17-of-them) · [🧪 使用 API](#-how-to-use-the-api) · [✅ 已验证](#-verified-working--every-key-is-tested) · [🧭 工作原理](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [⚙️ 选项](#️-cli-options) · [🛰️ 代理](#️-behind-a-proxy) · [🐞 问题](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ 为什么有这个项目

DuiAPI 是一个 New API 系列面板：在 Cloudflare Turnstile 之后进行邮箱 + 密码注册，并且**每个新账户都有 1,000,000 的初始额度**（`quota_per_unit` 为 500,000；第一个账户的余额显示为 1,000,000）。目录中包含跨折扣分组（`default`、`deepseek-sale`、`glm-sale`、`qwen-sale`、`kimi-discount`）的 **17 个模型 id**，经聊天验证的默认模型是 `deepseek-v4.1-flash`。额度存在于账户上——这就是为什么此工具**每个密钥创建一个账户**。

## 🎁 免费额度 — 你能获得多少

**每个新的 DuiAPI 账户初始都有 1,000,000 额度单位。**

| 项目 | 值 |
| --- | --- |
| 初始额度 | 每个账户 **1,000,000**（在全新账户上测得，`quotaStart`） |
| 注册 | 邮箱 + 密码 + Cloudflare Turnstile |
| 邮箱验证 | 来自收件箱的 6 位字符验证码（实测往返 18 秒） |
| 模型目录 | **17 个 id**（面板定价）；零比率免费文本模型：无——额度就是免费层级 |
| 速率限制 | 注册端点在突发下返回 `429`（24 次尝试：7 个密钥，4 次注册 429，5 次邮件验证码未命中）——控制运行节奏，重试是安全的 |
| 可扩展的单位 | **一个账户 = 一个密钥**（额度按账户计算） |

> 💡 免费层级由 DuiAPI 设定，可能会变化。在 [duiapi.com](https://duiapi.com) 查看实时数字——并让工具自身的关卡来判定：

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 模型 — 全部 17 个

`GET /v1/models` 提供面板目录——**17 个 id**（deepseek / glm / qwen / kimi 折扣分组）。经聊天验证的默认模型：

| 模型 ID | 备注 |
| --- | --- |
| `deepseek-v4.1-flash` | 验证默认模型（`default` 分组，比率 0.5）——聊天返回 200 且有真实内容，每次小调用额度变化约 180 单位 |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 如何使用 API

基础 URL **`https://duiapi.com/v1`** — 兼容 OpenAI。

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

余额检查（工具把关的依据）：使用账户令牌调用 `GET /api/user/self`，或进行一次 1 token 的聊天调用。

## 🛠 快速开始

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI 将其注册/发送验证码/登录调用置于 Cloudflare Turnstile 之后，因此每次调用消耗一个全新的令牌（每个账户三个）：在任意浏览器中于 <https://duiapi.com> 解决该组件并粘贴 `cf-turnstile-response` 值，或将 `--token-cmd` 指向一个在 stdout 上打印令牌的求解器。`keys.txt` **仅在**一次实时聊天完成后才写入；`accounts.json` 保留用户名 / 邮箱 / 密钥 id。

### 🖱️ 启动器

启动器会询问代理列表（可选——没有代理时 CLI 直接运行）和数量，记住它们，并运行 CLI——铸造、验证、筛选出口、设置、帮助、连接到 9Router。

## ✅ 已验证可用 — 每个密钥都经过测试

| 检查 | 结果 |
| --- | --- |
| `GET /api/status` | **200**，注册+邮箱验证+turnstile 均为 true，sitekey 存在 |
| `POST /api/user/register`（Turnstile 令牌） | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 来自收件箱的 6 位字符验证码（`DuiAPI email verification`） |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions`（`deepseek-v4.1-flash`）——对**每个**密钥在写入之前 | **200**，带有真实的 `choices[0].message.content` 和额度变化 |
| 未通过聊天检查的密钥 | 从不写入——账户保留在 `accounts.json` 中 |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

## 🧭 工作原理

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| 步骤 | 调用 | 备注 |
| --- | --- | --- |
| status | `GET /api/status` | 关卡：注册开启、邮箱验证开启、turnstile 开启（sitekey 由面板提供） |
| signup | 面板注册页面 → Turnstile（在浏览器中解决）→ `POST /api/user/register` | 令牌是一次性的；每次注册/发送验证码使用一个全新的 |
| verify | `GET /api/verification?email=…` | 6 位字符验证码到达收件箱；品牌邮件被过滤掉 |
| login | `POST /api/user/login` | 明文密码（加密关闭）；会话令牌在手 |
| key | `POST /api/token/`（+ 显示） | 密钥仅返回一次，立即写入磁盘 |
| proof | `POST /v1/chat/completions` | 聊天返回 200 且有内容 = 密钥可用 |

## ⚙️ CLI 选项

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| 环境变量 | 默认值 | 含义 |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | 出口池，每行一个代理——可选：没有池时 CLI 直接连接 |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | 用于实时验证的模型 |

## 🛰️ 在代理之后

**可选——在规模化时推荐（按 IP 限制、封禁）。** 该工具在没有代理时也能工作。

- **有池时：** 传入 `--pool <file>`（或设置 `DUIAPI_PROXIES`），每个请求都通过该列表中的一个出口——一条失效线路会使该次尝试失败，工具会继续前进。注册是有损耗的步骤（突发下 `429`）——工具会用下一个出口重试并按设计控制节奏。
- **没有池时：** CLI 打印 `running WITHOUT a proxy (direct)` 并直接连接。注册、密钥铸造或验证没有任何变化——只是数据包从何处发出不同。
- 对任一模式运行 `--screen`：它会在真实运行之前针对 DuiAPI 测试你拥有的出口（或直接路由）。

## 📄 文件

| 路径 | 内容 |
| --- | --- |
| `keys.txt` | 每行一个经实时验证的密钥——已被 git 忽略 |
| `accounts.json` | 用户名 / 邮箱 / 密钥 id——恢复数据 |
| `proxies.txt` | 你的出口——已被 git 忽略 |
| `outputs/` | 运行日志——已被 git 忽略 |

## 🔌 连接到 9Router

[9Router](https://9router.com) 是一个自托管、兼容 OpenAI 的路由器。DuiAPI 使用 OpenAI 协议，因此注册一次后每个密钥都成为一个连接：

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

默认值：`--prefix duiapi`、`--base-url https://duiapi.com/v1`、`--model deepseek-v4.1-flash`。模型以 `duiapi/<model id>` 形式提供。

## 🔌 连接到 OmniRoute

[OmniRoute](https://omniroute.online) 是一个自托管、兼容 OpenAI 的路由器。DuiAPI 使用 OpenAI 协议，因此注册一次后每个密钥都成为一个连接：

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

默认值：`--prefix duiapi`、`--base-url https://duiapi.com/v1`、`--model deepseek-v4.1-flash`。模型以 `duiapi/<model id>` 形式提供。
## 🔑 一个密钥能给你什么

| | |
| --- | --- |
| 🏢 提供商 | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 提供商提供什么 | 每个账户 1,000,000 初始额度 |
| 🔑 一个密钥能给你什么 | 一个密钥限定于其自己的账户：在 `https://duiapi.com/v1` 上使用 17 个目录 id，按你账户的额度 |
| 🌐 API 形态 | 兼容 OpenAI：`POST https://duiapi.com/v1/chat/completions` |
| 🔒 你的数据 | 仅联系 `duiapi.com` 和你的收件箱后端 |

## 🐞 问题

如果 DuiAPI 更改了注册形态、Turnstile sitekey 或额度步骤，工具会在此停止并说明。**[提交 issue](../../issues)** 并附上命令和确切输出。

## 📄 许可证

MIT © 2026 — 见 [LICENSE](LICENSE)。

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>