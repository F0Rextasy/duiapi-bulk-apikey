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

**Crie contas DuiAPI em massa com chaves de API geradas automaticamente — inferência compatível com OpenAI + conector 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Início rápido](#-quick-start) · [🎁 Camada gratuita](#-free-tier--how-much-you-get) · [🤖 Modelos](#-models--all-17-of-them) · [🧪 Use a API](#-how-to-use-the-api) · [✅ Verificado](#-verified-working--every-key-is-tested) · [🧭 Como funciona](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Opções](#️-cli-options) · [🛰️ Proxies](#️-behind-a-proxy) · [🐞 Problemas](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Por que isto existe

DuiAPI é um painel da família New API: registro com e-mail + senha atrás de um Cloudflare Turnstile, e **cota inicial de 1.000.000 em cada nova conta** (`quota_per_unit` 500.000; o saldo da primeira conta aparece como 1.000.000). O catálogo traz **17 ids de modelos** em grupos de desconto (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`), e o padrão verificado por chat é `deepseek-v4.1-flash`. A cota fica na conta — por isso esta ferramenta cria **uma conta por chave**.

## 🎁 Camada gratuita — quanto você recebe

**Toda nova conta DuiAPI começa com 1.000.000 de unidades de cota.**

| O quê | Valor |
| --- | --- |
| Cota inicial | **1.000.000** por conta (medido em uma conta nova, `quotaStart`) |
| Registro | e-mail + senha + Cloudflare Turnstile |
| Verificação de e-mail | código de 6 caracteres da caixa de entrada (ida e volta medida em 18 s) |
| Catálogo de modelos | **17 ids** (preços do painel); modelos de texto gratuitos com proporção zero: nenhum — a cota é a camada gratuita |
| Limites de taxa | o endpoint de registro respondeu `429` sob rajadas (24 tentativas: chaves, 4× registro-429, 5× falhas de código de e-mail) — controle o ritmo da execução, as tentativas repetidas são seguras |
| A unidade que escala | **uma conta = uma chave** (a cota é por conta) |

> 💡 A camada gratuita é definida pela DuiAPI e pode mudar. Confira os números ao vivo em [duiapi.com](https://duiapi.com) — e deixe o próprio gate da ferramenta decidir:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modelos — todos os 17

`GET /v1/models` serve o catálogo do painel — **17 ids** (grupos de desconto deepseek / glm / qwen / kimi). O padrão verificado por chat:

| ID do modelo | Observação |
| --- | --- |
| `deepseek-v4.1-flash` | o padrão de verificação (grupo `default`, proporção 0.5) — chat-200 com conteúdo real, delta de cota ~180 unidades por chamada pequena |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 Como usar a API

URL base **`https://duiapi.com/v1`** — compatível com OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Verificação de saldo (o que a ferramenta usa como gate): `GET /api/user/self` com o token da conta, ou uma chamada de chat com 1 token.

## 🛠 Início rápido

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

A DuiAPI mantém suas chamadas de registro/sendCode/login atrás de um Cloudflare Turnstile, então um token novo é gasto por chamada (três por conta): resolva o widget em <https://duiapi.com> em qualquer navegador e cole os valores de `cf-turnstile-response`, ou aponte `--token-cmd` para um solver que imprima um token no stdout. `keys.txt` é escrito **somente** após uma conclusão de chat ao vivo; `accounts.json` guarda o nome de usuário / e-mail / id da chave.

### 🖱️ O launcher

O launcher pede a lista de proxies (opcional — sem uma, o CLI roda direto) e a quantidade, lembra delas e executa o CLI — mint, verify, screen exits, settings, help, Connect to 9Router.

## 🚀 Um comando

O captcha é resolvido para você — um comando cria N contas e mantém apenas as chaves que uma chamada de chat ao vivo verificou:

```bash
npm install                  # once: pulls puppeteer-core
npm start                    # menu → "1) Mint keys" → "How many keys?" (Enter = 10)
# or non-interactive:
node src/cli.js --count N
```

`--auto` é o padrão e nada precisa ser passado: o `src/solver.js` embutido abre um Chrome com interface sobre a própria saída da conta e retorna o token Turnstile do painel automaticamente. `--dry` imprime o plano resolvido e sai **sem arquivo escrito e sem chamada de rede**.

> ⚠️ **Termos e risco de conta.** Esta é uma ferramenta de automação, não um vendedor de contas. Cadastros em massa e criação de chaves são inteiramente sua responsabilidade: podem violar os Termos de Serviço do provedor e levar ao bloqueio de endereços, saídas ou contas. Use apenas em infraestrutura que você tem permissão para automatizar e mantenha a taxa de requisições baixa.

## 🔧 Avançado (opcional)

Conectar seu próprio solver, colar um token manualmente ou apontar a ferramenta para um Chrome específico estão documentados em **[ADVANCED.md](ADVANCED.md)**. O caminho comum não precisa de nada disso.

## ✅ Verificado e funcionando — toda chave é testada

| Verificação | Resultado |
| --- | --- |
| `GET /api/status` | **200**, registro+verificação de e-mail+turnstile todos true, sitekey presente |
| `POST /api/user/register` (token Turnstile) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + código de 6 caracteres da caixa de entrada (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — para **toda** chave antes de ser escrita | **200** com `choices[0].message.content` real e um delta de cota |
| uma chave que falha na verificação de chat | nunca é escrita — a conta permanece em `accounts.json` |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **Gates de qualidade.** Toda chave aqui passou por um `chat-200` ao vivo antes de ser escrita em `keys.txt`
> (chaves mortas nunca são entregues). As combinações de roteador seguem a mesma regra no nível do modelo: uma entrada é
> adicionada **somente se o modelo responder `chat-200` e puder chamar ferramentas** — um modelo sem suporte a ferramentas,
> uma chave morta ou um provedor morto é descartado em vez de adicionado. Reexecute a etapa de verificação a qualquer momento;
> as falhas são relatadas, nunca escondidas.

## 🧭 Como funciona

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Etapa | Chamada | Observações |
| --- | --- | --- |
| status | `GET /api/status` | gates: registro ativo, verificação de e-mail ativa, turnstile ativo (sitekey servida pelo painel) |
| cadastro | página de registro do painel → Turnstile (resolvido no navegador) → `POST /api/user/register` | o token é de uso único; um novo por registro/sendCode |
| verificação | `GET /api/verification?email=…` | código de 6 caracteres chega na caixa de entrada; e-mails de marca são filtrados |
| login | `POST /api/user/login` | senha em texto simples (criptografia desativada); token de sessão em mãos |
| chave | `POST /api/token/` (+ reveal) | chave retornada uma vez, escrita em disco imediatamente |
| prova | `POST /v1/chat/completions` | chat-200 com conteúdo = a chave está ativa |

## ⚙️ Opções do CLI

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Padrão | Significado |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | pool de saídas, uma linha por proxy — opcional: sem um pool o CLI se conecta diretamente |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | modelo para a verificação ao vivo |

## 🛰️ Atrás de um proxy

**Opcional — recomendado em escala (limites por IP, banimentos).** A ferramenta funciona sem proxy também.

- **Com um pool:** passe `--pool <file>` (ou defina `DUIAPI_PROXIES`) e toda requisição passa por uma saída dessa lista — uma linha morta faz aquela tentativa falhar e a ferramenta segue em frente. O registro é a etapa com perdas (`429` sob rajada) — a ferramenta tenta novamente com a próxima saída e controla o ritmo por design.
- **Sem um pool:** o CLI imprime `running WITHOUT a proxy (direct)` e se conecta diretamente. Nada sobre cadastro, criação de chaves ou verificação muda — apenas de onde os pacotes saem.
- Execute `--screen` em qualquer um dos modos: ele testa as saídas que você tem (ou a rota direta) contra a DuiAPI antes de uma execução real.

## 📄 Arquivos

| Caminho | Conteúdo |
| --- | --- |
| `keys.txt` | uma chave verificada ao vivo por linha — ignorada pelo git |
| `accounts.json` | nome de usuário / e-mail / id da chave — dados de recuperação |
| `proxies.txt` | suas saídas — ignorado pelo git |
| `outputs/` | logs de execução — ignorados pelo git |

## 🔌 Conectar ao 9Router

[9Router](https://9router.com) é um roteador auto-hospedado e compatível com OpenAI. A DuiAPI fala OpenAI, então registre uma vez e toda chave se torna uma conexão:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Padrões: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Os modelos são servidos como `duiapi/<model id>`.

## 🔌 Conectar ao OmniRoute

[OmniRoute](https://omniroute.online) é um roteador auto-hospedado e compatível com OpenAI. A DuiAPI fala OpenAI, então registre uma vez e toda chave se torna uma conexão:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Padrões: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Os modelos são servidos como `duiapi/<model id>`.

## 🔑 O que uma chave lhe dá

| | |
| --- | --- |
| 🏢 Provedor | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 O que o provedor dá | 1.000.000 de cota inicial por conta |
| 🔑 O que uma chave lhe dá | uma chave vinculada à sua própria conta: os 17 ids do catálogo em `https://duiapi.com/v1`, na cota da sua conta |
| 🌐 Formato da API | compatível com OpenAI: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Seus dados | apenas `duiapi.com` e o backend da sua caixa de entrada são contatados |

## 🐞 Problemas

Se a DuiAPI mudar o formato de registro, a sitekey do Turnstile ou a etapa de cota, a ferramenta para ali e avisa. **[Abra uma issue](../../issues)** com o comando e a saída exata.

## 📄 Licença

MIT © 2026 — veja [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>