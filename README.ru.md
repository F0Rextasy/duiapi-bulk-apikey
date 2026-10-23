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

**Массовое создание аккаунтов DuiAPI с автоматически сгенерированными API-ключами — OpenAI-совместимый инференс + коннектор 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Быстрый старт](#-quick-start) · [🎁 Бесплатный тариф](#-free-tier--how-much-you-get) · [🤖 Модели](#-models--all-17-of-them) · [🧪 Использование API](#-how-to-use-the-api) · [✅ Проверено](#-verified-working--every-key-is-tested) · [🧭 Как это работает](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Опции](#️-cli-options) · [🛰️ Прокси](#️-behind-a-proxy) · [🐞 Проблемы](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Зачем это нужно

DuiAPI — это панель семейства New API: регистрация по e-mail + паролю за Cloudflare Turnstile и **1 000 000 стартовой квоты на каждом новом аккаунте** (`quota_per_unit` 500 000; баланс первого аккаунта отображается как 1 000 000). В каталоге представлено **17 идентификаторов моделей** по скидочным группам (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`), а проверенная в чате модель по умолчанию — `deepseek-v4.1-flash`. Квота привязана к аккаунту — именно поэтому этот инструмент создаёт **один аккаунт на ключ**.

## 🎁 Бесплатный тариф — сколько вы получаете

**Каждый новый аккаунт DuiAPI начинается с 1 000 000 единиц квоты.**

| Что | Значение |
| --- | --- |
| Стартовая квота | **1 000 000** на аккаунт (измерено на свежем аккаунте, `quotaStart`) |
| Регистрация | e-mail + пароль + Cloudflare Turnstile |
| Подтверждение e-mail | 6-символьный код из почтового ящика (измеренный круговой рейс — 18 с) |
| Каталог моделей | **17 идентификаторов** (цены панели); бесплатных текстовых моделей с нулевым коэффициентом нет — бесплатный тариф — это квота |
| Ограничения скорости | эндпоинт регистрации отвечал `429` при всплесках (24 попытки: ключи, 4× register-429, 5× промахов по почтовому коду) — задавайте темп, повторные попытки безопасны |
| Единица масштабирования | **один аккаунт = один ключ** (квота на аккаунт) |

> 💡 Бесплатный тариф устанавливается DuiAPI и может меняться. Проверяйте актуальные цифры на [duiapi.com](https://duiapi.com) — и пусть решение принимает собственный гейт инструмента:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Модели — все 17

`GET /v1/models` отдаёт каталог панели — **17 идентификаторов** (скидочные группы deepseek / glm / qwen / kimi). Проверенная в чате модель по умолчанию:

| ID модели | Примечание |
| --- | --- |
| `deepseek-v4.1-flash` | модель проверки по умолчанию (группа `default`, коэффициент 0.5) — chat-200 с реальным содержимым, дельта квоты ~180 единиц на крошечный вызов |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 Как использовать API

Базовый URL **`https://duiapi.com/v1`** — совместим с OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Проверка баланса (то, по чему инструмент принимает решение): `GET /api/user/self` с токеном аккаунта или вызов чата на 1 токен.

## 🛠 Быстрый старт

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI держит свои вызовы register/sendCode/login за Cloudflare Turnstile, поэтому на каждый вызов расходуется свежий токен (три на аккаунт): решите виджет на <https://duiapi.com> в любом браузере и вставьте значения `cf-turnstile-response`, либо укажите `--token-cmd` на солвер, который выводит токен в stdout. `keys.txt` записывается **только** после живого завершения чата; `accounts.json` хранит имя пользователя / e-mail / id ключа.

### 🖱️ Лаунчер

Лаунчер запрашивает список прокси (необязательно — без него CLI работает напрямую) и количество, запоминает их и запускает CLI — mint, verify, screen exits, settings, help, Connect to 9Router.

## ✅ Проверено в работе — каждый ключ протестирован

| Проверка | Результат |
| --- | --- |
| `GET /api/status` | **200**, register+e-mail verification+turnstile все true, sitekey присутствует |
| `POST /api/user/register` (Turnstile token) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 6-символьный код из почтового ящика (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — для **каждого** ключа перед записью | **200** с реальным `choices[0].message.content` и дельтой квоты |
| ключ, не прошедший проверку чата | никогда не записывается — аккаунт остаётся в `accounts.json` |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **Гейты качества.** Каждый ключ здесь прошёл живой `chat-200` до того, как был записан в `keys.txt`
> (мёртвые ключи никогда не поставляются). Комбинации роутера следуют тому же правилу на уровне модели: запись
> добавляется **только если модель отвечает `chat-200` и умеет вызывать инструменты** — модель без поддержки инструментов,
> мёртвый ключ или мёртвый провайдер отбрасывается, а не добавляется. Перезапускайте шаг проверки в любое время;
> сбои сообщаются, никогда не скрываются.

## 🧭 Как это работает

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Шаг | Вызов | Примечания |
| --- | --- | --- |
| status | `GET /api/status` | гейты: register on, e-mail verification on, turnstile on (sitekey отдаётся панелью) |
| signup | страница регистрации панели → Turnstile (решается в браузере) → `POST /api/user/register` | токен одноразовый; свежий на каждый register/sendCode |
| verify | `GET /api/verification?email=…` | 6-символьный код приходит в почтовый ящик; брендовые письма отфильтровываются |
| login | `POST /api/user/login` | пароль в открытом виде (шифрование выключено); токен сессии на руках |
| key | `POST /api/token/` (+ reveal) | ключ возвращается один раз, сразу записывается на диск |
| proof | `POST /v1/chat/completions` | chat-200 с содержимым = ключ живой |

## ⚙️ Опции CLI

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Переменная | По умолчанию | Значение |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | пул выходов, по одной строке на прокси — необязательно: без пула CLI подключается напрямую |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | модель для живой проверки |

## 🛰️ За прокси

**Необязательно — рекомендуется при масштабе (лимиты на IP, баны).** Инструмент работает и без прокси.

- **С пулом:** передайте `--pool <file>` (или задайте `DUIAPI_PROXIES`), и каждый запрос идёт через выход из этого списка — мёртвая линия проваливает эту попытку, и инструмент двигается дальше. Регистрация — самый подверженный потерям шаг (`429` при всплесках) — инструмент повторяет с следующим выходом и задаёт темп по замыслу.
- **Без пула:** CLI печатает `running WITHOUT a proxy (direct)` и подключается напрямую. Ничего в регистрации, выпуске ключей или проверке не меняется — только то, откуда уходят пакеты.
- Запустите `--screen` в любом из режимов: он тестирует имеющиеся у вас выходы (или прямой маршрут) против DuiAPI перед реальным запуском.

## 📄 Файлы

| Путь | Содержимое |
| --- | --- |
| `keys.txt` | по одному проверенному вживую ключу на строку — игнорируется git |
| `accounts.json` | имя пользователя / e-mail / id ключа — данные для восстановления |
| `proxies.txt` | ваши выходы — игнорируется git |
| `outputs/` | логи запусков — игнорируется git |

## 🔌 Подключение к 9Router

[9Router](https://9router.com) — это самостоятельно размещаемый, OpenAI-совместимый роутер. DuiAPI говорит на OpenAI, поэтому зарегистрируйтесь один раз, и каждый ключ становится подключением:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

По умолчанию: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Модели отдаются как `duiapi/<model id>`.

## 🔌 Подключение к OmniRoute

[OmniRoute](https://omniroute.online) — это самостоятельно размещаемый, OpenAI-совместимый роутер. DuiAPI говорит на OpenAI, поэтому зарегистрируйтесь один раз, и каждый ключ становится подключением:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

По умолчанию: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Модели отдаются как `duiapi/<model id>`.

## 🔑 Что даёт один ключ

| | |
| --- | --- |
| 🏢 Провайдер | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 Что даёт провайдер | 1 000 000 стартовой квоты на аккаунт |
| 🔑 Что даёт один ключ | один ключ, привязанный к своему аккаунту: 17 идентификаторов каталога на `https://duiapi.com/v1`, в пределах квоты вашего аккаунта |
| 🌐 Форма API | Совместим с OpenAI: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Ваши данные | контакт только с `duiapi.com` и вашим почтовым бэкендом |

## 🐞 Проблемы

Если DuiAPI изменит форму регистрации, sitekey Turnstile или шаг квоты, инструмент остановится на этом месте и сообщит об этом. **[Откройте issue](../../issues)** с командой и точным выводом.

## 📄 Лицензия

MIT © 2026 — см. [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>