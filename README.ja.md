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

**自動生成された API キーで DuiAPI アカウントを一括作成 — OpenAI 互換の推論 + 9Router コネクタ。**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 クイックスタート](#-quick-start) · [🎁 無料枠](#-free-tier--how-much-you-get) · [🤖 モデル](#-models--all-17-of-them) · [🧪 API の使い方](#-how-to-use-the-api) · [✅ 検証済み](#-verified-working--every-key-is-tested) · [🧭 仕組み](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ オプション](#️-cli-options) · [🛰️ プロキシ](#️-behind-a-proxy) · [🐞 問題報告](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ なぜこれが存在するのか

DuiAPI は New API ファミリーのパネルです。Cloudflare Turnstile の背後でメールアドレス + パスワードによる登録ができ、**新規アカウントごとに 1,000,000 の初期クォータ**が付与されます（`quota_per_unit` は 500,000、最初のアカウントの残高は 1,000,000 と表示されます）。カタログには割引グループ（`default`、`deepseek-sale`、`glm-sale`、`qwen-sale`、`kimi-discount`）にまたがる **17 個のモデル ID** があり、チャットで検証済みのデフォルトは `deepseek-v4.1-flash` です。クォータはアカウントに紐づきます — だからこのツールは**キーごとに 1 アカウント**を作成します。

## 🎁 無料枠 — どれだけ得られるか

**新規の DuiAPI アカウントはすべて 1,000,000 クォータ単位から始まります。**

| 項目 | 値 |
| --- | --- |
| 初期クォータ | アカウントごとに **1,000,000**（新規アカウントで実測、`quotaStart`） |
| 登録 | メールアドレス + パスワード + Cloudflare Turnstile |
| メール認証 | 受信トレイに届く 6 文字のコード（実測ラウンドトリップ 18 秒） |
| モデルカタログ | **17 個の ID**（パネル価格）；ゼロレートの無料テキストモデル：なし — クォータが無料枠です |
| レート制限 | 登録エンドポイントはバースト時に `429` を返しました（24 回の試行：キー、4× 登録 429、5× メールコードの失敗）— 実行ペースを調整してください、リトライは安全です |
| スケールする単位 | **1 アカウント = 1 キー**（クォータはアカウント単位） |

> 💡 無料枠は DuiAPI が設定するもので、変更される可能性があります。最新の数値は [duiapi.com](https://duiapi.com) で確認してください — そしてツール自身のゲートに判断させてください：

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 モデル — 全 17 個

`GET /v1/models` はパネルのカタログを提供します — **17 個の ID**（deepseek / glm / qwen / kimi の割引グループ）。チャットで検証済みのデフォルト：

| モデル ID | 備考 |
| --- | --- |
| `deepseek-v4.1-flash` | 検証のデフォルト（`default` グループ、レート 0.5）— 実際のコンテンツで chat-200、小さな呼び出しごとにクォータ差分は約 180 単位 |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 API の使い方

ベース URL は **`https://duiapi.com/v1`** — OpenAI 互換です。

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

残高確認（ツールがゲートする対象）：アカウントトークンを使った `GET /api/user/self`、または 1 トークンのチャット呼び出し。

## 🛠 クイックスタート

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI は register/sendCode/login の呼び出しを Cloudflare Turnstile の背後に置いているため、呼び出しごとに新しいトークンを 1 つ消費します（アカウントごとに 3 つ）：任意のブラウザで <https://duiapi.com> のウィジェットを解き、`cf-turnstile-response` の値を貼り付けるか、stdout にトークンを出力するソルバーを `--token-cmd` に指定してください。`keys.txt` は**実際のチャット完了の後でのみ**書き込まれます；`accounts.json` はユーザー名 / メールアドレス / キー ID を保持します。

### 🖱️ ランチャー

ランチャーはプロキシリスト（任意 — なしの場合は CLI は直接実行されます）と件数を尋ね、それらを記憶して CLI を実行します — mint、verify、screen exits、settings、help、Connect to 9Router。

## 🚀 コマンド 1 つ

キャプチャは自動で解決されます — コマンド 1 つで N 個のアカウントを作成し、実際のチャット呼び出しで検証されたキーだけを保持します：

```bash
npm install                  # once: pulls puppeteer-core
npm start                    # menu → "1) Mint keys" → "How many keys?" (Enter = 10)
# or non-interactive:
node src/cli.js --count N
```

`--auto` がデフォルトで、何も渡す必要はありません：組み込みの `src/solver.js` がアカウント自身の出口を経由してヘッド付き Chrome を開き、パネルの Turnstile トークンを自動的に返します。`--dry` は解決されたプランを表示し、**ファイルを書き込まず、ネットワーク呼び出しも行わずに**終了します。

> ⚠️ **利用規約とアカウントのリスク。** これは自動化ツールであり、アカウント販売業者ではありません。一括サインアップとキーの発行は完全にあなたの責任です：プロバイダーの利用規約に違反する可能性があり、アドレス、出口、アカウントがブロックされることがあります。自動化が許可されているインフラでのみ使用し、リクエストレートを低く保ってください。

## 🔧 上級（任意）

独自のソルバーの接続、トークンの手動貼り付け、特定の Chrome を指定する方法は **[ADVANCED.md](ADVANCED.md)** に記載されています。一般的な使い方ではこれらは不要です。

## ✅ 動作検証済み — すべてのキーがテストされています

| チェック | 結果 |
| --- | --- |
| `GET /api/status` | **200**、register+メール認証+turnstile すべて true、sitekey あり |
| `POST /api/user/register`（Turnstile トークン） | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 受信トレイに届く 6 文字のコード（`DuiAPI email verification`） |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions`（`deepseek-v4.1-flash`）— 書き込まれる前の**すべての**キーに対して | 実際の `choices[0].message.content` とクォータ差分を伴う **200** |
| チャットチェックに失敗したキー | 書き込まれない — アカウントは `accounts.json` に残る |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **品質ゲート。** ここにあるすべてのキーは、`keys.txt` に書き込まれる前に実際の `chat-200` を通過しています
> （死んだキーは決して出荷されません）。ルーターのコンボもモデルレベルで同じルールに従います：エントリは
> **モデルが `chat-200` を返し、ツールを呼び出せる場合にのみ**追加されます — ツールサポートのないモデル、
> 死んだキー、死んだプロバイダーは追加されずに除外されます。検証ステップはいつでも再実行できます；
> 失敗は報告され、決して隠されません。

## 🧭 仕組み

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| ステップ | 呼び出し | 備考 |
| --- | --- | --- |
| status | `GET /api/status` | ゲート：register オン、メール認証オン、turnstile オン（sitekey はパネルが提供） |
| signup | パネル登録ページ → Turnstile（ブラウザ内で解決）→ `POST /api/user/register` | トークンは使い捨て；register/sendCode ごとに新しいもの |
| verify | `GET /api/verification?email=…` | 6 文字のコードが受信トレイに届く；ブランドメールは除外される |
| login | `POST /api/user/login` | 平文パスワード（暗号化オフ）；セッショントークンを取得 |
| key | `POST /api/token/`（+ reveal） | キーは一度だけ返され、即座にディスクに書き込まれる |
| proof | `POST /v1/chat/completions` | コンテンツを伴う chat-200 = キーは有効 |

## ⚙️ CLI オプション

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| 環境変数 | デフォルト | 意味 |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | 出口プール、1 行につき 1 プロキシ — 任意：プールなしの場合 CLI は直接接続します |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | 実際の検証に使うモデル |

## 🛰️ プロキシの背後で

**任意 — 大規模運用では推奨（IP ごとの制限、BAN）。** このツールはプロキシなしでも動作します。

- **プールあり：** `--pool <file>` を渡す（または `DUIAPI_PROXIES` を設定する）と、すべてのリクエストがそのリストの出口を経由します — 死んだ回線はその試行を失敗させ、ツールは次に進みます。登録が最も失敗しやすいステップです（バースト時に `429`）— ツールは次の出口でリトライし、設計上ペースを調整します。
- **プールなし：** CLI は `running WITHOUT a proxy (direct)` と表示し、直接接続します。サインアップ、キー発行、検証の内容は何も変わりません — パケットが出ていく場所だけが変わります。
- どちらのモードでも `--screen` を実行してください：実際の実行前に、利用可能な出口（または直接ルート）を DuiAPI に対してテストします。

## 📄 ファイル

| パス | 内容 |
| --- | --- |
| `keys.txt` | 1 行につき 1 つの実際に検証済みのキー — git で無視 |
| `accounts.json` | ユーザー名 / メールアドレス / キー ID — 復旧データ |
| `proxies.txt` | あなたの出口 — git で無視 |
| `outputs/` | 実行ログ — git で無視 |

## 🔌 9Router に接続

[9Router](https://9router.com) はセルフホスト型の OpenAI 互換ルーターです。DuiAPI は OpenAI を話すので、一度登録すればすべてのキーが接続になります：

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

デフォルト：`--prefix duiapi`、`--base-url https://duiapi.com/v1`、`--model deepseek-v4.1-flash`。モデルは `duiapi/<model id>` として提供されます。

## 🔌 OmniRoute に接続

[OmniRoute](https://omniroute.online) はセルフホスト型の OpenAI 互換ルーターです。DuiAPI は OpenAI を話すので、一度登録すればすべてのキーが接続になります：

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

デフォルト：`--prefix duiapi`、`--base-url https://duiapi.com/v1`、`--model deepseek-v4.1-flash`。モデルは `duiapi/<model id>` として提供されます。

## 🔑 1 つのキーで得られるもの

| | |
| --- | --- |
| 🏢 プロバイダー | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 プロバイダーが提供するもの | アカウントごとに 1,000,000 の初期クォータ |
| 🔑 1 つのキーで得られるもの | 自身のアカウントにスコープされた 1 つのキー：`https://duiapi.com/v1` で 17 個のカタログ ID を、あなたのアカウントのクォータで |
| 🌐 API の形式 | OpenAI 互換：`POST https://duiapi.com/v1/chat/completions` |
| 🔒 あなたのデータ | `duiapi.com` とあなたの受信トレイバックエンドのみが接続されます |

## 🐞 問題報告

DuiAPI が登録の形式、Turnstile の sitekey、またはクォータのステップを変更した場合、ツールはそこで停止し、その旨を報告します。コマンドと正確な出力を添えて **[issue を作成してください](../../issues)**。

## 📄 ライセンス

MIT © 2026 — [LICENSE](LICENSE) を参照。

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>