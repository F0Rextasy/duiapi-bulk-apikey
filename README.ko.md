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

**자동 생성된 API 키로 DuiAPI 계정을 대량 생성 — OpenAI 호환 추론 + 9Router 커넥터.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 빠른 시작](#-quick-start) · [🎁 무료 티어](#-free-tier--how-much-you-get) · [🤖 모델](#-models--all-17-of-them) · [🧪 API 사용하기](#-how-to-use-the-api) · [✅ 검증됨](#-verified-working--every-key-is-tested) · [🧭 작동 방식](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ 옵션](#️-cli-options) · [🛰️ 프록시](#️-behind-a-proxy) · [🐞 이슈](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ 왜 이 도구가 존재하는가

DuiAPI는 New API 계열 패널입니다: Cloudflare Turnstile 뒤에서 이메일 + 비밀번호로 가입하며, **모든 신규 계정에 1,000,000 시작 쿼터**가 주어집니다 (`quota_per_unit` 500,000; 첫 계정의 잔액은 1,000,000으로 표시됨). 카탈로그에는 할인 그룹(`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`)에 걸쳐 **17개의 모델 id**가 있으며, 채팅으로 검증된 기본값은 `deepseek-v4.1-flash`입니다. 쿼터는 계정에 귀속됩니다 — 그래서 이 도구는 **키 하나당 계정 하나**를 생성합니다.

## 🎁 무료 티어 — 얼마나 받을 수 있는가

**모든 신규 DuiAPI 계정은 1,000,000 쿼터 단위로 시작합니다.**

| 항목 | 값 |
| --- | --- |
| 시작 쿼터 | 계정당 **1,000,000** (신규 계정에서 측정, `quotaStart`) |
| 가입 | 이메일 + 비밀번호 + Cloudflare Turnstile |
| 이메일 인증 | 받은편지함에서 온 6자리 코드 (왕복 18초 측정) |
| 모델 카탈로그 | **17개 id** (패널 가격 기준); 비율 0인 무료 텍스트 모델: 없음 — 쿼터가 곧 무료 티어 |
| 속도 제한 | 버스트 상황에서 register 엔드포인트가 `429`를 반환함 (24회 시도: 키, 4× register-429, 5× 메일 코드 실패) — 실행 속도를 조절하세요, 재시도는 안전합니다 |
| 확장되는 단위 | **계정 하나 = 키 하나** (쿼터는 계정별) |

> 💡 무료 티어는 DuiAPI가 정하며 변경될 수 있습니다. 실시간 수치는 [duiapi.com](https://duiapi.com)에서 확인하세요 — 그리고 도구 자체의 게이트가 판단하도록 하세요:

```bash
node src/cli.js --verify-only     # keys.txt의 모든 키가 실제 채팅 호출로 재검증됨
```

## 🤖 모델 — 17개 전부

`GET /v1/models`는 패널 카탈로그를 제공합니다 — **17개 id** (deepseek / glm / qwen / kimi 할인 그룹). 채팅으로 검증된 기본값:

| 모델 ID | 비고 |
| --- | --- |
| `deepseek-v4.1-flash` | 검증 기본값 (`default` 그룹, 비율 0.5) — 실제 콘텐츠와 함께 chat-200, 소규모 호출당 쿼터 변화량 약 180 단위 |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 API 사용 방법

Base URL **`https://duiapi.com/v1`** — OpenAI 호환.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

잔액 확인 (도구가 게이트로 사용하는 것): 계정 토큰으로 `GET /api/user/self`, 또는 1토큰 채팅 호출.

## 🛠 빠른 시작

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # 선택 사항 — 없으면 도구는 직접 연결로 실행됨
node src/launcher.js        # 대화형: 발급 / 검증 / 스크린 / 설정

# 또는 직접 — 계정당 Turnstile 토큰 3개 (아래 참고):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# 또는 솔버 연결:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI는 register/sendCode/login 호출을 Cloudflare Turnstile 뒤에 두므로, 호출당 새 토큰 하나가 소모됩니다 (계정당 3개): 아무 브라우저에서 <https://duiapi.com>의 위젯을 풀고 `cf-turnstile-response` 값을 붙여넣거나, stdout에 토큰을 출력하는 솔버를 `--token-cmd`로 지정하세요. `keys.txt`는 실제 채팅 완료 이후에**만** 기록됩니다; `accounts.json`은 사용자 이름 / 이메일 / 키 id를 보관합니다.

### 🖱️ 런처

런처는 프록시 목록(선택 사항 — 없으면 CLI는 직접 실행됨)과 개수를 묻고, 이를 기억한 뒤 CLI를 실행합니다 — 발급, 검증, 스크린 종료, 설정, 도움말, 9Router 연결.

## 🚀 명령 하나

캡차는 자동으로 풀립니다 — 명령 하나로 N개 계정을 발급하고 실제 채팅 호출로 검증된 키만 보관합니다:

```bash
npm install                  # 최초 1회: puppeteer-core 설치
npm start                    # 메뉴 → "1) Mint keys" → "How many keys?" (Enter = 10)
# 또는 비대화형:
node src/cli.js --count N
```

`--auto`가 기본값이며 아무것도 전달할 필요가 없습니다: 내장된 `src/solver.js`가 계정 자체의 출구를 통해 헤드리스가 아닌 Chrome을 열고 패널의 Turnstile 토큰을 자동으로 반환합니다. `--dry`는 결정된 계획을 출력하고 **파일 기록도 네트워크 호출도 없이** 종료합니다.

> ⚠️ **약관 및 계정 위험.** 이것은 자동화 도구이지 계정 판매자가 아닙니다. 대량 가입과 키 발급은 전적으로 귀하의 책임입니다: 제공자의 서비스 약관을 위반할 수 있으며 주소, 출구 또는 계정이 차단될 수 있습니다. 자동화가 허용된 인프라에서만 사용하고 요청 속도를 낮게 유지하세요.

## 🔧 고급 (선택 사항)

자체 솔버 연결, 토큰 수동 붙여넣기, 특정 Chrome 지정은 **[ADVANCED.md](ADVANCED.md)**에 문서화되어 있습니다. 일반적인 경로에는 이 중 어느 것도 필요하지 않습니다.

## ✅ 검증됨 — 모든 키가 테스트됨

| 검사 | 결과 |
| --- | --- |
| `GET /api/status` | **200**, register+이메일 인증+turnstile 모두 true, sitekey 존재 |
| `POST /api/user/register` (Turnstile 토큰) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + 받은편지함에서 온 6자리 코드 (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — 기록되기 **전** 모든 키에 대해 | 실제 `choices[0].message.content`와 쿼터 변화량과 함께 **200** |
| 채팅 검사에 실패한 키 | 기록되지 않음 — 계정은 `accounts.json`에 남음 |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **품질 게이트.** 여기 있는 모든 키는 `keys.txt`에 기록되기 전에 실제 `chat-200`을 통과했습니다
> (죽은 키는 절대 배포되지 않습니다). 라우터 콤보도 모델 수준에서 같은 규칙을 따릅니다: 항목은
> **모델이 `chat-200`으로 응답하고 도구 호출이 가능한 경우에만** 추가됩니다 — 도구 지원이 없는 모델,
> 죽은 키, 죽은 제공자는 추가되지 않고 제거됩니다. 검증 단계는 언제든 다시 실행하세요;
> 실패는 보고되며 절대 숨겨지지 않습니다.

## 🧭 작동 방식

```
받은편지함 (계정당 주소 하나)                                  계정마다 새 주소
   │
   ├─▶ 가입: DuiAPI 페이지 → 이메일 주소 → 받은편지함에서 온 코드 / 링크
   ├─▶ 세션: 계정의 쿠키 또는 액세스 토큰 확보
   ├─▶ 키 발급: 계정의 키 엔드포인트 → 평문 키가 한 번 반환됨 (즉시 디스크에 기록)
   └─▶ 실시간 증명: POST https://duiapi.com/v1/chat/completions → 실제 콘텐츠 = 키가 살아 있음
```

| 단계 | 호출 | 비고 |
| --- | --- | --- |
| 상태 | `GET /api/status` | 게이트: register 켜짐, 이메일 인증 켜짐, turnstile 켜짐 (sitekey는 패널이 제공) |
| 가입 | 패널 register 페이지 → Turnstile (브라우저에서 해결) → `POST /api/user/register` | 토큰은 일회용; register/sendCode마다 새 토큰 |
| 인증 | `GET /api/verification?email=…` | 받은편지함에 6자리 코드 도착; 브랜드 메일은 필터링됨 |
| 로그인 | `POST /api/user/login` | 평문 비밀번호 (암호화 꺼짐); 세션 토큰 확보 |
| 키 | `POST /api/token/` (+ 공개) | 키는 한 번 반환되며 즉시 디스크에 기록됨 |
| 증명 | `POST /v1/chat/completions` | 콘텐츠와 함께 chat-200 = 키가 살아 있음 |

## ⚙️ CLI 옵션

```bash
node src/cli.js --count N        # N개 키 발급 (스크린 종료, 가입, 인증, 채팅 게이트)
node src/cli.js --verify-only    # keys.txt의 모든 키 재검증
node src/cli.js --screen         # 출구 풀 자체 테스트
node src/cli.js --connect-9router
node src/cli.js --help
```

| 환경 변수 | 기본값 | 의미 |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | 출구 풀, 프록시당 한 줄 — 선택 사항: 풀이 없으면 CLI는 직접 연결 |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | 실시간 검증에 사용할 모델 |

## 🛰️ 프록시 뒤에서

**선택 사항 — 대규모에서는 권장됨 (IP별 제한, 차단).** 이 도구는 프록시 없이도 작동합니다.

- **풀 사용 시:** `--pool <file>`을 전달하거나 `DUIAPI_PROXIES`를 설정하면 모든 요청이 해당 목록의 출구를 통해 나갑니다 — 죽은 회선은 그 시도를 실패시키고 도구는 다음으로 넘어갑니다. Register가 손실이 큰 단계입니다 (버스트 시 `429`) — 도구는 다음 출구로 재시도하며 설계상 속도를 조절합니다.
- **풀 없이:** CLI는 `running WITHOUT a proxy (direct)`를 출력하고 직접 연결합니다. 가입, 키 발급, 검증에는 아무 변화가 없습니다 — 패킷이 나가는 위치만 다를 뿐입니다.
- 어느 모드에서든 `--screen`을 실행하세요: 실제 실행 전에 보유한 출구(또는 직접 경로)를 DuiAPI에 대해 테스트합니다.

## 📄 파일

| 경로 | 내용 |
| --- | --- |
| `keys.txt` | 한 줄에 하나씩 실제 검증된 키 — git 무시됨 |
| `accounts.json` | 사용자 이름 / 이메일 / 키 id — 복구 데이터 |
| `proxies.txt` | 귀하의 출구 — git 무시됨 |
| `outputs/` | 실행 로그 — git 무시됨 |

## 🔌 9Router에 연결

[9Router](https://9router.com)는 자체 호스팅되는 OpenAI 호환 라우터입니다. DuiAPI는 OpenAI를 사용하므로 한 번 등록하면 모든 키가 연결이 됩니다:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

기본값: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. 모델은 `duiapi/<model id>`로 제공됩니다.

## 🔌 OmniRoute에 연결

[OmniRoute](https://omniroute.online)는 자체 호스팅되는 OpenAI 호환 라우터입니다. DuiAPI는 OpenAI를 사용하므로 한 번 등록하면 모든 키가 연결이 됩니다:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

기본값: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. 모델은 `duiapi/<model id>`로 제공됩니다.

## 🔑 키 하나로 얻는 것

| | |
| --- | --- |
| 🏢 제공자 | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 제공자가 주는 것 | 계정당 1,000,000 시작 쿼터 |
| 🔑 키 하나로 얻는 것 | 자체 계정에 한정된 키 하나: `https://duiapi.com/v1`에서 17개 카탈로그 id를, 귀하의 계정 쿼터로 |
| 🌐 API 형태 | OpenAI 호환: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 귀하의 데이터 | `duiapi.com`과 귀하의 받은편지함 백엔드만 접촉됨 |

## 🐞 이슈

DuiAPI가 register 형태, Turnstile sitekey 또는 쿼터 단계를 변경하면, 도구는 그 지점에서 멈추고 그렇게 알립니다. 명령과 정확한 출력을 포함하여 **[이슈를 열어주세요](../../issues)**.

## 📄 라이선스

MIT © 2026 — [LICENSE](LICENSE) 참조.

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>