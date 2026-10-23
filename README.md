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

**Bulk-create DuiAPI accounts with auto-generated API keys — OpenAI-compatible inference + 9Router connector.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Quick start](#-quick-start) · [🎁 Free tier](#-free-tier--how-much-you-get) · [🤖 Models](#-models--all-17-of-them) · [🧪 Use the API](#-how-to-use-the-api) · [✅ Verified](#-verified-working--every-key-is-tested) · [🧭 How it works](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Options](#️-cli-options) · [🛰️ Proxies](#️-behind-a-proxy) · [🐞 Issues](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Why this exists

DuiAPI is a New API family panel: e-mail + password registration behind a Cloudflare Turnstile, and **1,000,000 starting quota on every new account** (`quota_per_unit` 500,000; the first account's balance reads 1,000,000). The catalogue carries **17 model ids** across discount groups (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`), and the chat-verified default is `deepseek-v4.1-flash`. The quota lives on the account — that is why this tool creates **one account per key**.

## 🎁 Free tier — how much you get

**Every new DuiAPI account starts with 1,000,000 quota units.**

| What | Value |
| --- | --- |
| Starting quota | **1,000,000** per account (measured on a fresh account, `quotaStart`) |
| Registration | e-mail + password + Cloudflare Turnstile |
| E-mail verification | 6-char code from the inbox (18 s measured round-trip) |
| Model catalogue | **17 ids** (panel pricing); zero-ratio free text models: none — quota is the free tier |
| Rate limits | register endpoint answered `429` under bursts (24 attempts: keys, 4× register-429, 5× mail-code misses) — pace the run, retries are safe |
| The unit that scales | **one account = one key** (quota is per account) |

> 💡 The free tier is set by DuiAPI and can change. Check the live numbers at [duiapi.com](https://duiapi.com) — and let the tool's own gate decide:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Models — all 17 of them

`GET /v1/models` serves the panel catalogue — **17 ids** (deepseek / glm / qwen / kimi discount groups). The chat-verified default:

| Model ID | Note |
| --- | --- |
| `deepseek-v4.1-flash` | the verification default (`default` group, ratio 0.5) — chat-200 with real content, quota delta ~180 units per tiny call |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 How to use the API

Base URL **`https://duiapi.com/v1`** — OpenAI-compatible.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Balance check (what the tool gates on): `GET /api/user/self` with the account token, or a 1-token chat call.

## 🛠 Quick start

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings
# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI keeps its register/sendCode/login calls behind a Cloudflare Turnstile, so one fresh token is spent per call (three per account): solve the widget on <https://duiapi.com> in any browser and paste the `cf-turnstile-response` values, or point `--token-cmd` at a solver that prints a token on stdout. `keys.txt` is written **only** after a live chat completion; `accounts.json` keeps the username / e-mail / key id.

### 🖱️ The launcher

The launcher asks for the proxy list (optional — without one the CLI runs direct) and count, remembers them, and runs the CLI — mint, verify, screen exits, settings, help, Connect to 9Router.

## ✅ Verified working — every key is tested

| Check | Result |
| --- | --- |
| `GET /api/status` | **200**, register+e-mail verification+turnstile all true, sitekey present |
| `POST /api/user/register` (Turnstile token) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 6-char code from the inbox (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — for **every** key before it is written | **200** with real `choices[0].message.content` and a quota delta |
| a key that fails its chat check | never written — account stays in `accounts.json` |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **Quality gates.** Every key here passed a live `chat-200` before it was written to `keys.txt`
> (dead keys are never shipped). Router combos follow the same rule at the model level: an entry is
> added **only if the model answers `chat-200` and can call tools** — a model without tool support,
> a dead key or a dead provider is dropped instead of added. Re-run the verify step any time;
> failures are reported, never hidden.

## 🧭 How it works

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Step | Call | Notes |
| --- | --- | --- |
| status | `GET /api/status` | gates: register on, e-mail verification on, turnstile on (sitekey served by the panel) |
| signup | panel register page → Turnstile (solved in-browser) → `POST /api/user/register` | token is single-use; a fresh one per register/sendCode |
| verify | `GET /api/verification?email=…` | 6-char code arrives in the inbox; brand mail is filtered out |
| login | `POST /api/user/login` | plain password (encryption off); session token in hand |
| key | `POST /api/token/` (+ reveal) | key returned once, written to disk immediately |
| proof | `POST /v1/chat/completions` | chat-200 with content = the key is live |

## ⚙️ CLI options

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Default | Meaning |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | exit pool, one line per proxy — optional: without a pool the CLI connects directly |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | model for the live verification |

## 🛰️ Behind a proxy

**Optional — recommended at scale (per-IP limits, bans).** The tool works without a proxy too.

- **With a pool:** pass `--pool <file>` (or set `DUIAPI_PROXIES`) and every request goes through an exit from that list — a dead line fails that attempt and the tool moves on. Register is the lossy step (`429` under burst) — the tool retries with the next exit and paces by design.
- **Without a pool:** the CLI prints `running WITHOUT a proxy (direct)` and connects directly. Nothing about sign-up, key minting or verification changes — only where the packets leave from.
- Run `--screen` against either mode: it tests the exits you have (or the direct route) against DuiAPI before a real run.

## 📄 Files

| Path | Contents |
| --- | --- |
| `keys.txt` | one live-verified key per line — git-ignored |
| `accounts.json` | username / e-mail / key id — recovery data |
| `proxies.txt` | your exits — git-ignored |
| `outputs/` | run logs — git-ignored |

## 🔌 Connect to 9Router

[9Router](https://9router.com) is a self-hosted, OpenAI-compatible router. DuiAPI speaks OpenAI, so register once and every key becomes a connection:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Defaults: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Models are served as `duiapi/<model id>`.

## 🔌 Connect to OmniRoute

[OmniRoute](https://omniroute.online) is a self-hosted, OpenAI-compatible router. DuiAPI speaks OpenAI, so register once and every key becomes a connection:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Defaults: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Models are served as `duiapi/<model id>`.

## 🔑 What one key gets you

| | |
| --- | --- |
| 🏢 Provider | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 What the provider gives | 1,000,000 starting quota per account |
| 🔑 What one key gets you | one key scoped to its own account: the 17 catalogue ids on `https://duiapi.com/v1`, at your account's quota |
| 🌐 API shape | OpenAI-compatible: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Your data | only `duiapi.com` and your inbox backend are contacted |

## 🐞 Issues

If DuiAPI changes the register shape, the Turnstile sitekey or the quota step, the tool stops there and says so. **[Open an issue](../../issues)** with the command and the exact output.

## 📄 License

MIT © 2026 — see [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>
