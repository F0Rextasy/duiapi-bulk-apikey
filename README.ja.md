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
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 クイックスタート](#-quick-start) · [🎁 無料枠](#-free-tier--how-much-you-get) · [🤖 モデル](#-models--all-17-of-them) · [🧪 API の使い方](#-how-to-use-the-api) · [✅ 検証済み](#-verified-working--every-key-is-tested) · [🧭 仕組み](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [⚙️ オプション](#️-cli-options) · [🛰️ プロキシ](#️-behind-a-proxy) · [🐞 問題](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ これが存在する理由

DuiAPI は New API ファミリーのパネルです。Cloudflare Turnstile の背後でメールアドレス + パスワードによる登録が行われ、**新規アカウントごとに 1,000,000 の初期クォータ**が付与されます（`quota_per_unit` は 500,000、最初のアカウントの残高は 1,000,000 と表示されます）。カタログには割引グループ（`default`、`deepseek-sale`、`glm-sale`、`qwen-sale`、`kimi-discount`）にまたがる **17 個のモデル ID** が含まれ、チャットで検証済みのデフォルトは `deepseek-v4.1-flash` です。クォータはアカウントに紐づきます。だからこそこのツールは **キーごとに 1 アカウント**を作成します。

## 🎁 無料枠 — どれだけ得られるか

**新規の DuiAPI アカウントはすべて 1,000,000 クォータ単位で開始します。**

| 項目 | 値 |
| --- | --- |
| 初期クォータ | アカウントごとに **1,000,000**（新規アカウントで計測、`quotaStart`） |
| 登録 | メールアドレス + パスワード + Cloudflare Turnstile |
| メール認証 | 受信トレイに届く 6 文字のコード（往復 18 秒を計測） |
| モデルカタログ | **17 個の ID**（パネル価格）; ゼロレートの無料テキストモデル: なし — クォータが無料枠です |
| レート制限 | 登録エンドポイントはバースト時に `429` を返しました（24 回の試行: 7 キー、4× register-429、5× メールコードの失敗）— 実行ペースを調整してください、リトライは安全です |
| スケールする単位 | **1 アカウント = 1 キー**（クォータはアカウント単位） |

> 💡 無料枠は DuiAPI によって設定され、変更される可能性があります。最新の数値は [duiapi.com](https://duiapi.com) で確認してください — そしてツール自身のゲートに判断を委ねてください:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 モデル — その 17 個すべて

`GET /v1/models` はパネルのカタログを提供します — **17 個の ID**（deepseek / glm / qwen / kimi の割引グループ）。チャットで検証済みのデフォルト:

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

残高確認（ツールがゲートする対象）: アカウントトークンを使った `GET /api/user/self`、または 1 トークンのチャット呼び出し。

## 🛠 クイックスタート

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI は register/sendCode/login の呼び出しを Cloudflare Turnstile の背後に置いているため、呼び出しごとに新しいトークンを 1 つ消費します（アカウントごとに 3 つ）。任意のブラウザで <https://duiapi.com> のウィジェットを解き、`cf-turnstile-response` の値を貼り付けるか、stdout にトークンを出力するソルバーを `--token-cmd` に指定してください。`keys.txt` は**ライブのチャット補完の後にのみ**書き込まれます。`accounts.json` はユーザー名 / メールアドレス / キー ID を保持します。

### 🖱️ ランチャー

ランチャーはプロキシリスト（任意 — なしの場合 CLI は直接実行）と件数を尋ね、それらを記憶して CLI を実行します — mint、verify、screen exits、settings、help、Connect to 9Router。

## ✅ 動作検証済み — すべてのキーがテストされます

| チェック | 結果 |
| --- | --- |
| `GET /api/status` | **200**、register+メール認証+turnstile すべて true、sitekey あり |
| `POST /api/user/register`（Turnstile トークン） | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 受信トレイからの 6 文字コード（`DuiAPI email verification`） |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions`（`deepseek-v4.1-flash`）— 書き込まれる前の**すべての**キーに対して | 実際の `choices[0].message.content` とクォータ差分を伴う **200** |
| チャットチェックに失敗したキー | 書き込まれない — アカウントは `accounts.json` に残る |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

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
| status | `GET /api/status` | ゲート: register オン、メール認証 オン、turnstile オン（sitekey はパネルが提供） |
| signup | パネルの登録ページ → Turnstile（ブラウザで解決）→ `POST /api/user/register` | トークンは単回使用; register/sendCode ごとに新しいもの |
| verify | `GET /api/verification?email=…` | 6 文字のコードが受信トレイに届く; ブランドメールは除外される |
| login | `POST /api/user/login` | 平文パスワード（暗号化オフ）; セッショントークンを取得 |
| key | `POST /api/token/`（+ reveal） | キーは一度だけ返され、即座にディスクに書き込まれる |
| proof | `POST /v1/chat/completions` | コンテンツを伴う chat-200 = キーはライブ |

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
| `DUIAPI_PROXIES` | `proxies.txt` | 出口プール、1 行につき 1 プロキシ — 任意: プールなしの場合 CLI は直接接続 |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | ライブ検証に使うモデル |

## 🛰️ プロキシの背後で

**任意 — 大規模運用では推奨（IP ごとの制限、BAN）。** このツールはプロキシなしでも動作します。

- **プールあり:** `--pool <file>` を渡す（または `DUIAPI_PROXIES` を設定する）と、すべてのリクエストがそのリストの出口を経由します — 死んだ回線はその試行を失敗させ、ツールは次に進みます。登録が損失の出るステップです（バースト時に `429`）— ツールは次の出口でリトライし、設計上ペースを調整します。
- **プールなし:** CLI は `running WITHOUT a proxy (direct)` と表示し、直接接続します。サインアップ、キー生成、検証について何も変わりません — 変わるのはパケットが出ていく場所だけです。
- どちらのモードでも `--screen` を実行してください: 実際の実行前に、手持ちの出口（または直接ルート）を DuiAPI に対してテストします。

## 📄 ファイル

| パス | 内容 |
| --- | --- |
| `keys.txt` | ライブ検証済みのキーを 1 行に 1 つ — git 管理外 |
| `accounts.json` | ユーザー名 / メールアドレス / キー ID — 復旧データ |
| `proxies.txt` | あなたの出口 — git 管理外 |
| `outputs/` | 実行ログ — git 管理外 |

## 🔌 9Router に接続

[9Router](https://9router.com) はセルフホスト型の OpenAI 互換ルーターです。DuiAPI は OpenAI を話すので、一度登録すればすべてのキーが接続になります:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

デフォルト: `--prefix duiapi`、`--base-url https://duiapi.com/v1`、`--model deepseek-v4.1-flash`。モデルは `duiapi/<model id>` として提供されます。

## 🔑 1 つのキーで得られるもの

| | |
| --- | --- |
| 🏢 プロバイダー | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 プロバイダーが提供するもの | アカウントごとに 1,000,000 の初期クォータ |
| 🔑 1 つのキーで得られるもの | 自身のアカウントにスコープされた 1 つのキー: `https://duiapi.com/v1` 上の 17 個のカタログ ID を、あなたのアカウントのクォータで |
| 🌐 API の形式 | OpenAI 互換: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 あなたのデータ | 接続されるのは `duiapi.com` とあなたの受信トレイバックエンドのみ |

## 🐞 問題

DuiAPI が登録の形式、Turnstile の sitekey、またはクォータのステップを変更した場合、ツールはそこで停止し、その旨を報告します。コマンドと正確な出力を添えて **[issue を作成してください](../../issues)**。

## 📄 ライセンス

MIT © 2026 — [LICENSE](LICENSE) を参照。

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>