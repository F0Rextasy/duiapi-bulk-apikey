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

**Crea cuentas de DuiAPI en masa con claves API autogeneradas — inferencia compatible con OpenAI + conector 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Inicio rápido](#-quick-start) · [🎁 Nivel gratuito](#-free-tier--how-much-you-get) · [🤖 Modelos](#-models--all-17-of-them) · [🧪 Usa la API](#-how-to-use-the-api) · [✅ Verificado](#-verified-working--every-key-is-tested) · [🧭 Cómo funciona](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Opciones](#️-cli-options) · [🛰️ Proxies](#️-behind-a-proxy) · [🐞 Problemas](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Por qué existe esto

DuiAPI es un panel de la familia New API: registro con correo electrónico + contraseña detrás de un Cloudflare Turnstile, y **1.000.000 de cuota inicial en cada cuenta nueva** (`quota_per_unit` 500.000; el saldo de la primera cuenta marca 1.000.000). El catálogo incluye **17 ids de modelo** repartidos en grupos de descuento (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`), y el predeterminado verificado por chat es `deepseek-v4.1-flash`. La cuota vive en la cuenta — por eso esta herramienta crea **una cuenta por clave**.

## 🎁 Nivel gratuito — cuánto obtienes

**Cada cuenta nueva de DuiAPI comienza con 1.000.000 de unidades de cuota.**

| Qué | Valor |
| --- | --- |
| Cuota inicial | **1.000.000** por cuenta (medida en una cuenta nueva, `quotaStart`) |
| Registro | correo electrónico + contraseña + Cloudflare Turnstile |
| Verificación por correo | código de 6 caracteres desde la bandeja de entrada (18 s de ida y vuelta medidos) |
| Catálogo de modelos | **17 ids** (precios del panel); modelos de texto gratuitos con ratio cero: ninguno — la cuota es el nivel gratuito |
| Límites de velocidad | el endpoint de registro respondió `429` bajo ráfagas (24 intentos: claves, 4× registro-429, 5× fallos de código de correo) — modera el ritmo, los reintentos son seguros |
| La unidad que escala | **una cuenta = una clave** (la cuota es por cuenta) |

> 💡 El nivel gratuito lo define DuiAPI y puede cambiar. Consulta los números en vivo en [duiapi.com](https://duiapi.com) — y deja que la propia puerta de la herramienta decida:

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modelos — los 17

`GET /v1/models` sirve el catálogo del panel — **17 ids** (grupos de descuento deepseek / glm / qwen / kimi). El predeterminado verificado por chat:

| ID de modelo | Nota |
| --- | --- |
| `deepseek-v4.1-flash` | el predeterminado de verificación (grupo `default`, ratio 0.5) — chat-200 con contenido real, delta de cuota ~180 unidades por llamada pequeña |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 Cómo usar la API

URL base **`https://duiapi.com/v1`** — compatible con OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Consulta de saldo (lo que la herramienta verifica): `GET /api/user/self` con el token de la cuenta, o una llamada de chat de 1 token.

## 🛠 Inicio rápido

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI mantiene sus llamadas de registro/envío de código/inicio de sesión detrás de un Cloudflare Turnstile, por lo que se consume un token nuevo por llamada (tres por cuenta): resuelve el widget en <https://duiapi.com> en cualquier navegador y pega los valores de `cf-turnstile-response`, o apunta `--token-cmd` a un solucionador que imprima un token en stdout. `keys.txt` se escribe **solo** tras una finalización de chat en vivo; `accounts.json` conserva el nombre de usuario / correo electrónico / id de clave.

### 🖱️ El lanzador

El lanzador pide la lista de proxies (opcional — sin una, el CLI se ejecuta directo) y la cantidad, las recuerda y ejecuta el CLI — crear, verificar, probar salidas, ajustes, ayuda, Conectar a 9Router.

## ✅ Verificado y funcionando — cada clave se prueba

| Comprobación | Resultado |
| --- | --- |
| `GET /api/status` | **200**, registro + verificación por correo + turnstile todos en true, sitekey presente |
| `POST /api/user/register` (token de Turnstile) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + código de 6 caracteres desde la bandeja de entrada (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — para **cada** clave antes de escribirse | **200** con `choices[0].message.content` real y un delta de cuota |
| una clave que falla su comprobación de chat | nunca se escribe — la cuenta permanece en `accounts.json` |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **Puertas de calidad.** Cada clave aquí pasó un `chat-200` en vivo antes de escribirse en `keys.txt`
> (las claves muertas nunca se entregan). Las combinaciones del router siguen la misma regla a nivel de modelo: una entrada se
> añade **solo si el modelo responde `chat-200` y puede llamar a herramientas** — un modelo sin soporte de herramientas,
> una clave muerta o un proveedor muerto se descarta en lugar de añadirse. Vuelve a ejecutar el paso de verificación cuando quieras;
> los fallos se reportan, nunca se ocultan.

## 🧭 Cómo funciona

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Paso | Llamada | Notas |
| --- | --- | --- |
| estado | `GET /api/status` | puertas: registro activado, verificación por correo activada, turnstile activado (sitekey servida por el panel) |
| registro | página de registro del panel → Turnstile (resuelto en el navegador) → `POST /api/user/register` | el token es de un solo uso; uno nuevo por registro/envío de código |
| verificación | `GET /api/verification?email=…` | llega un código de 6 caracteres a la bandeja de entrada; el correo de marca se filtra |
| inicio de sesión | `POST /api/user/login` | contraseña en texto plano (cifrado desactivado); token de sesión en mano |
| clave | `POST /api/token/` (+ revelar) | la clave se devuelve una vez, se escribe en disco inmediatamente |
| prueba | `POST /v1/chat/completions` | chat-200 con contenido = la clave está viva |

## ⚙️ Opciones del CLI

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Variable de entorno | Predeterminado | Significado |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | grupo de salidas, una línea por proxy — opcional: sin un grupo, el CLI se conecta directamente |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | modelo para la verificación en vivo |

## 🛰️ Detrás de un proxy

**Opcional — recomendado a escala (límites por IP, bloqueos).** La herramienta también funciona sin proxy.

- **Con un grupo:** pasa `--pool <file>` (o define `DUIAPI_PROXIES`) y cada solicitud pasa por una salida de esa lista — una línea muerta hace fallar ese intento y la herramienta continúa. El registro es el paso con pérdidas (`429` bajo ráfaga) — la herramienta reintenta con la siguiente salida y regula el ritmo por diseño.
- **Sin un grupo:** el CLI imprime `running WITHOUT a proxy (direct)` y se conecta directamente. Nada del registro, la creación de claves o la verificación cambia — solo desde dónde salen los paquetes.
- Ejecuta `--screen` con cualquiera de los dos modos: prueba las salidas que tengas (o la ruta directa) contra DuiAPI antes de una ejecución real.

## 📄 Archivos

| Ruta | Contenido |
| --- | --- |
| `keys.txt` | una clave verificada en vivo por línea — ignorada por git |
| `accounts.json` | nombre de usuario / correo electrónico / id de clave — datos de recuperación |
| `proxies.txt` | tus salidas — ignorado por git |
| `outputs/` | registros de ejecución — ignorados por git |

## 🔌 Conectar a 9Router

[9Router](https://9router.com) es un router autoalojado compatible con OpenAI. DuiAPI habla OpenAI, así que regístrate una vez y cada clave se convierte en una conexión:

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Predeterminados: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Los modelos se sirven como `duiapi/<model id>`.

## 🔌 Conectar a OmniRoute

[OmniRoute](https://omniroute.online) es un router autoalojado compatible con OpenAI. DuiAPI habla OpenAI, así que regístrate una vez y cada clave se convierte en una conexión:

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Predeterminados: `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Los modelos se sirven como `duiapi/<model id>`.

## 🔑 Qué te da una clave

| | |
| --- | --- |
| 🏢 Proveedor | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 Qué da el proveedor | 1.000.000 de cuota inicial por cuenta |
| 🔑 Qué te da una clave | una clave limitada a su propia cuenta: los 17 ids del catálogo en `https://duiapi.com/v1`, con la cuota de tu cuenta |
| 🌐 Forma de la API | compatible con OpenAI: `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Tus datos | solo se contacta con `duiapi.com` y tu backend de bandeja de entrada |

## 🐞 Problemas

Si DuiAPI cambia la forma del registro, la sitekey de Turnstile o el paso de cuota, la herramienta se detiene ahí y lo indica. **[Abre un issue](../../issues)** con el comando y la salida exacta.

## 📄 Licencia

MIT © 2026 — consulta [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>