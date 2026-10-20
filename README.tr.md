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

**Otomatik oluşturulan API anahtarlarıyla toplu DuiAPI hesabı oluşturun — OpenAI uyumlu çıkarım + 9Router bağlayıcısı.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Hızlı başlangıç](#-quick-start) · [🎁 Ücretsiz katman](#-free-tier--how-much-you-get) · [🤖 Modeller](#-models--all-17-of-them) · [🧪 API'yi kullanma](#-how-to-use-the-api) · [✅ Doğrulandı](#-verified-working--every-key-is-tested) · [🧭 Nasıl çalışır](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [⚙️ Seçenekler](#️-cli-options) · [🛰️ Proxy'ler](#️-behind-a-proxy) · [🐞 Sorunlar](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Bu neden var

DuiAPI, bir New API ailesi panelidir: Cloudflare Turnstile arkasında e-posta + parola kaydı ve **her yeni hesapta 1.000.000 başlangıç kotası** (`quota_per_unit` 500.000; ilk hesabın bakiyesi 1.000.000 okunur). Katalog, indirim grupları (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`) genelinde **17 model kimliği** taşır ve sohbetle doğrulanan varsayılan `deepseek-v4.1-flash`'tır. Kota hesapta yaşar — bu aracın **anahtar başına bir hesap** oluşturmasının nedeni budur.

## 🎁 Ücretsiz katman — ne kadar alıyorsunuz

**Her yeni DuiAPI hesabı 1.000.000 kota birimiyle başlar.**

| Ne | Değer |
| --- | --- |
| Başlangıç kotası | Hesap başına **1.000.000** (yeni bir hesapta ölçüldü, `quotaStart`) |
| Kayıt | e-posta + parola + Cloudflare Turnstile |
| E-posta doğrulama | Gelen kutusundan 6 karakterli kod (18 sn ölçülen gidiş-dönüş) |
| Model kataloğu | **17 kimlik** (panel fiyatlandırması); sıfır oranlı ücretsiz metin modelleri: yok — ücretsiz katman kotadır |
| Hız sınırları | Kayıt uç noktası ani yüklenmelerde `429` yanıtladı (24 deneme: 7 anahtar, 4× kayıt-429, 5× posta kodu ıskası) — çalışmayı temposuna bırakın, yeniden denemeler güvenlidir |
| Ölçeklenen birim | **bir hesap = bir anahtar** (kota hesap başınadır) |

> 💡 Ücretsiz katman DuiAPI tarafından belirlenir ve değişebilir. Güncel sayıları [duiapi.com](https://duiapi.com) adresinden kontrol edin — ve kararı aracın kendi kapısına bırakın:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modeller — 17'sinin tamamı

`GET /v1/models` panel kataloğunu sunar — **17 kimlik** (deepseek / glm / qwen / kimi indirim grupları). Sohbetle doğrulanan varsayılan:

| Model Kimliği | Not |
| --- | --- |
| `deepseek-v4.1-flash` | doğrulama varsayılanı (`default` grubu, oran 0.5) — gerçek içerikle sohbet-200, küçük bir çağrı başına kota deltası ~180 birim |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 API'yi nasıl kullanırım

Temel URL **`https://duiapi.com/v1`** — OpenAI uyumlu.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Bakiye kontrolü (aracın kapı olarak kullandığı şey): hesap tokenıyla `GET /api/user/self` veya 1 tokenlık bir sohbet çağrısı.

## 🛠 Hızlı başlangıç

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings
# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI, kayıt/sendCode/login çağrılarını bir Cloudflare Turnstile arkasında tutar, bu yüzden çağrı başına bir taze token harcanır (hesap başına üç): herhangi bir tarayıcıda <https://duiapi.com> üzerindeki widget'ı çözün ve `cf-turnstile-response` değerlerini yapıştırın ya da `--token-cmd`'i stdout'a bir token yazdıran bir çözücüye yönlendirin. `keys.txt` **yalnızca** canlı bir sohbet tamamlamasından sonra yazılır; `accounts.json` kullanıcı adını / e-postayı / anahtar kimliğini saklar.

### 🖱️ Başlatıcı

Başlatıcı proxy listesini (isteğe bağlı — olmadan CLI doğrudan çalışır) ve sayıyı sorar, bunları hatırlar ve CLI'yi çalıştırır — oluştur, doğrula, çıkışları tara, ayarlar, yardım, 9Router'a Bağlan.

## ✅ Çalıştığı doğrulandı — her anahtar test edilir

| Kontrol | Sonuç |
| --- | --- |
| `GET /api/status` | **200**, register+e-mail verification+turnstile hepsi true, sitekey mevcut |
| `POST /api/user/register` (Turnstile token) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + gelen kutusundan 6 karakterli kod (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — yazılmadan önce **her** anahtar için | gerçek `choices[0].message.content` ve bir kota deltasıyla **200** |
| sohbet kontrolünde başarısız olan bir anahtar | asla yazılmaz — hesap `accounts.json` içinde kalır |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

## 🧭 Nasıl çalışır

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Adım | Çağrı | Notlar |
| --- | --- | --- |
| durum | `GET /api/status` | kapılar: kayıt açık, e-posta doğrulama açık, turnstile açık (sitekey panel tarafından sunulur) |
| kayıt | panel kayıt sayfası → Turnstile (tarayıcıda çözülür) → `POST /api/user/register` | token tek kullanımlıktır; her register/sendCode için taze bir tane |
| doğrulama | `GET /api/verification?email=…` | gelen kutusuna 6 karakterli kod gelir; marka postaları filtrelenir |
| giriş | `POST /api/user/login` | düz parola (şifreleme kapalı); oturum tokenı elde |
| anahtar | `POST /api/token/` (+ reveal) | anahtar bir kez döndürülür, hemen diske yazılır |
| kanıt | `POST /v1/chat/completions` | içerikle sohbet-200 = anahtar canlı |

## ⚙️ CLI seçenekleri

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Ortam | Varsayılan | Anlam |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | çıkış havuzu, proxy başına bir satır — isteğe bağlı: havuz olmadan CLI doğrudan bağlanır |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | canlı doğrulama için model |

## 🛰️ Proxy arkasında

**İsteğe bağlı — ölçekte önerilir (IP başına sınırlar, yasaklar).** Araç proxy olmadan da çalışır.

- **Havuzla:** `--pool <file>` geçin (veya `DUIAPI_PROXIES` ayarlayın) ve her istek o listeden bir çıkıştan geçer — ölü bir hat o denemeyi başarısız kılar ve araç devam eder. Kayıt, kayıplı adımdır (ani yüklenmede `429`) — araç bir sonraki çıkışla yeniden dener ve tasarımı gereği tempo tutar.
- **Havuz olmadan:** CLI `running WITHOUT a proxy (direct)` yazdırır ve doğrudan bağlanır. Kayıt, anahtar oluşturma veya doğrulamayla ilgili hiçbir şey değişmez — yalnızca paketlerin nereden çıktığı değişir.
- Her iki moda karşı `--screen` çalıştırın: gerçek bir çalıştırmadan önce sahip olduğunuz çıkışları (veya doğrudan rotayı) DuiAPI'ye karşı test eder.

## 📄 Dosyalar

| Yol | İçerik |
| --- | --- |
| `keys.txt` | satır başına bir canlı doğrulanmış anahtar — git tarafından yok sayılır |
| `accounts.json` | kullanıcı adı / e-posta / anahtar kimliği — kurtarma verisi |
| `proxies.txt` | çıkışlarınız — git tarafından yok sayılır |
| `outputs/` | çalıştırma günlükleri — git tarafından yok sayılır |

## 🔌 9Router'a bağlan

[9Router](https://9router.com) kendi kendine barındırılan, OpenAI uyumlu bir yönlendiricidir. DuiAPI OpenAI konuşur, bu yüzden bir kez kaydolun ve her anahtar bir bağlantıya dönüşür:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Varsayılanlar: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Modeller `duiapi/<model id>` olarak sunulur.

## 🔑 Bir anahtar size ne kazandırır

| | |
| --- | --- |
| 🏢 Sağlayıcı | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 Sağlayıcının verdikleri | Hesap başına 1.000.000 başlangıç kotası |
| 🔑 Bir anahtarın size kazandırdıkları | kendi hesabına kapsanmış bir anahtar: `https://duiapi.com/v1` üzerinde 17 katalog kimliği, hesabınızın kotasında |
| 🌐 API şekli | OpenAI uyumlu: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Verileriniz | yalnızca `duiapi.com` ve gelen kutusu arka ucunuzla iletişim kurulur |

## 🐞 Sorunlar

DuiAPI kayıt şeklini, Turnstile sitekey'ini veya kota adımını değiştirirse, araç orada durur ve bunu söyler. Komut ve tam çıktıyla birlikte **[bir sorun açın](../../issues)**.

## 📄 Lisans

MIT © 2026 — bkz. [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey katkıda bulunanlar

</div>