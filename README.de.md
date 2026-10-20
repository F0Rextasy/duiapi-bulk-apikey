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

**DuiAPI-Konten im Bulk erstellen mit automatisch generierten API-Schlüsseln — OpenAI-kompatible Inferenz + 9Router-Connector.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Schnellstart](#-quick-start) · [🎁 Free-Tier](#-free-tier--how-much-you-get) · [🤖 Modelle](#-models--all-17-of-them) · [🧪 Die API nutzen](#-how-to-use-the-api) · [✅ Verifiziert](#-verified-working--every-key-is-tested) · [🧭 So funktioniert es](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [⚙️ Optionen](#️-cli-options) · [🛰️ Proxies](#️-behind-a-proxy) · [🐞 Issues](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Warum es das gibt

DuiAPI ist ein Panel aus der New-API-Familie: Registrierung per E-Mail + Passwort hinter einem Cloudflare Turnstile, und **1.000.000 Startguthaben auf jedem neuen Konto** (`quota_per_unit` 500.000; das Guthaben des ersten Kontos wird als 1.000.000 angezeigt). Der Katalog umfasst **17 Modell-IDs** über Rabattgruppen (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`), und der im Chat verifizierte Standard ist `deepseek-v4.1-flash`. Das Guthaben liegt auf dem Konto — deshalb erstellt dieses Tool **ein Konto pro Schlüssel**.

## 🎁 Free-Tier — wie viel du bekommst

**Jedes neue DuiAPI-Konto startet mit 1.000.000 Guthaben-Einheiten.**

| Was | Wert |
| --- | --- |
| Startguthaben | **1.000.000** pro Konto (gemessen an einem frischen Konto, `quotaStart`) |
| Registrierung | E-Mail + Passwort + Cloudflare Turnstile |
| E-Mail-Verifizierung | 6-stelliger Code aus dem Postfach (18 s gemessene Round-Trip-Zeit) |
| Modellkatalog | **17 IDs** (Panel-Preise); Modelle mit Null-Ratio als Freitext: keine — das Guthaben ist der Free-Tier |
| Rate-Limits | Der Register-Endpunkt antwortete mit `429` bei Bursts (24 Versuche: 7 Schlüssel, 4× register-429, 5× fehlende Mail-Codes) — den Lauf drosseln, Wiederholungen sind sicher |
| Die skalierende Einheit | **ein Konto = ein Schlüssel** (Guthaben ist pro Konto) |

> 💡 Der Free-Tier wird von DuiAPI festgelegt und kann sich ändern. Prüfe die aktuellen Zahlen auf [duiapi.com](https://duiapi.com) — und lass das Gate des Tools selbst entscheiden:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modelle — alle 17

`GET /v1/models` liefert den Panel-Katalog — **17 IDs** (deepseek / glm / qwen / kimi Rabattgruppen). Der im Chat verifizierte Standard:

| Modell-ID | Hinweis |
| --- | --- |
| `deepseek-v4.1-flash` | der Verifizierungs-Standard (`default`-Gruppe, Ratio 0.5) — Chat-200 mit echtem Inhalt, Guthaben-Delta ~180 Einheiten pro winzigem Aufruf |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 So nutzt du die API

Basis-URL **`https://duiapi.com/v1`** — OpenAI-kompatibel.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Guthaben prüfen (worauf das Tool prüft): `GET /api/user/self` mit dem Konto-Token, oder ein Chat-Aufruf mit 1 Token.

## 🛠 Schnellstart

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings
# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI hält seine register/sendCode/login-Aufrufe hinter einem Cloudflare Turnstile, daher wird pro Aufruf ein frisches Token verbraucht (drei pro Konto): Löse das Widget auf <https://duiapi.com> in einem beliebigen Browser und füge die `cf-turnstile-response`-Werte ein, oder richte `--token-cmd` auf einen Solver, der ein Token auf stdout ausgibt. `keys.txt` wird **nur** nach einer echten Chat-Completion geschrieben; `accounts.json` speichert Benutzername / E-Mail / Schlüssel-ID.

### 🖱️ Der Launcher

Der Launcher fragt nach der Proxy-Liste (optional — ohne eine läuft die CLI direkt) und der Anzahl, merkt sie sich und führt die CLI aus — mint, verify, screen exits, settings, help, Connect to 9Router.

## ✅ Verifiziert funktionierend — jeder Schlüssel wird getestet

| Prüfung | Ergebnis |
| --- | --- |
| `GET /api/status` | **200**, register+E-Mail-Verifizierung+turnstile alle true, sitekey vorhanden |
| `POST /api/user/register` (Turnstile-Token) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 6-stelliger Code aus dem Postfach (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — für **jeden** Schlüssel, bevor er geschrieben wird | **200** mit echtem `choices[0].message.content` und einem Guthaben-Delta |
| ein Schlüssel, der seine Chat-Prüfung nicht besteht | wird nie geschrieben — das Konto bleibt in `accounts.json` |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

## 🧭 So funktioniert es

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Schritt | Aufruf | Hinweise |
| --- | --- | --- |
| status | `GET /api/status` | Gates: register an, E-Mail-Verifizierung an, turnstile an (sitekey vom Panel geliefert) |
| signup | Panel-Registrierungsseite → Turnstile (im Browser gelöst) → `POST /api/user/register` | Token ist einmalig verwendbar; ein frisches pro register/sendCode |
| verify | `GET /api/verification?email=…` | 6-stelliger Code kommt im Postfach an; Marken-Mails werden herausgefiltert |
| login | `POST /api/user/login` | Klartext-Passwort (Verschlüsselung aus); Session-Token in der Hand |
| key | `POST /api/token/` (+ reveal) | Schlüssel wird einmal zurückgegeben, sofort auf die Festplatte geschrieben |
| proof | `POST /v1/chat/completions` | Chat-200 mit Inhalt = der Schlüssel ist live |

## ⚙️ CLI-Optionen

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Standard | Bedeutung |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | Exit-Pool, eine Zeile pro Proxy — optional: ohne Pool verbindet sich die CLI direkt |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | Modell für die Live-Verifizierung |

## 🛰️ Hinter einem Proxy

**Optional — bei Skalierung empfohlen (Limits pro IP, Bans).** Das Tool funktioniert auch ohne Proxy.

- **Mit einem Pool:** übergib `--pool <file>` (oder setze `DUIAPI_PROXIES`), und jede Anfrage geht durch einen Exit aus dieser Liste — eine tote Leitung lässt diesen Versuch fehlschlagen und das Tool macht weiter. Register ist der verlustbehaftete Schritt (`429` bei Burst) — das Tool wiederholt mit dem nächsten Exit und drosselt von Natur aus.
- **Ohne einen Pool:** Die CLI gibt `running WITHOUT a proxy (direct)` aus und verbindet sich direkt. An Registrierung, Schlüssel-Erstellung oder Verifizierung ändert sich nichts — nur daran, wo die Pakete rausgehen.
- Führe `--screen` in beiden Modi aus: Es testet die Exits, die du hast (oder die direkte Route), gegen DuiAPI vor einem echten Lauf.

## 📄 Dateien

| Pfad | Inhalt |
| --- | --- |
| `keys.txt` | ein live-verifizierter Schlüssel pro Zeile — von Git ignoriert |
| `accounts.json` | Benutzername / E-Mail / Schlüssel-ID — Wiederherstellungsdaten |
| `proxies.txt` | deine Exits — von Git ignoriert |
| `outputs/` | Lauf-Logs — von Git ignoriert |

## 🔌 Mit 9Router verbinden

[9Router](https://9router.com) ist ein selbst gehosteter, OpenAI-kompatibler Router. DuiAPI spricht OpenAI, also einmal registrieren und jeder Schlüssel wird zu einer Verbindung:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Standardwerte: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Modelle werden als `duiapi/<model id>` bereitgestellt.

## 🔑 Was dir ein Schlüssel bringt

| | |
| --- | --- |
| 🏢 Anbieter | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 Was der Anbieter gibt | 1.000.000 Startguthaben pro Konto |
| 🔑 Was dir ein Schlüssel bringt | ein Schlüssel, beschränkt auf sein eigenes Konto: die 17 Katalog-IDs auf `https://duiapi.com/v1`, zu deinem Konto-Guthaben |
| 🌐 API-Form | OpenAI-kompatibel: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Deine Daten | nur `duiapi.com` und dein Postfach-Backend werden kontaktiert |

## 🐞 Issues

Wenn DuiAPI die Register-Form, den Turnstile-Sitekey oder den Guthaben-Schritt ändert, stoppt das Tool dort und sagt es dir. **[Öffne ein Issue](../../issues)** mit dem Befehl und der exakten Ausgabe.

## 📄 Lizenz

MIT © 2026 — siehe [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>