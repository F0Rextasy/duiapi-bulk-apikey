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

**Créez en masse des comptes DuiAPI avec des clés API générées automatiquement — inférence compatible OpenAI + connecteur 9Router.**

[![Node](https://img.shields.io/badge/Node-%E2%89%A518-3DA639?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-blue?style=for-the-badge)](#)
[![License](https://img.shields.io/badge/license-MIT-6ea8fe?style=for-the-badge)](LICENSE)
[![Models](https://img.shields.io/badge/models-17%20served-412991?style=for-the-badge)](https://duiapi.com)
[![9Router](https://img.shields.io/badge/9Router-connector-8B5CF6?style=for-the-badge)](#-connect-to-9router)
[![OmniRoute](https://img.shields.io/badge/OmniRoute-connector-8B5CF6?style=for-the-badge)](#-connect-to-omniroute)
[![X](https://img.shields.io/badge/X-%40bigbullapp-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/bigbullapp)

[🛠 Démarrage rapide](#-quick-start) · [🎁 Offre gratuite](#-free-tier--how-much-you-get) · [🤖 Modèles](#-models--all-17-of-them) · [🧪 Utiliser l'API](#-how-to-use-the-api) · [✅ Vérifié](#-verified-working--every-key-is-tested) · [🧭 Fonctionnement](#-how-it-works) · [🔌 9Router](#-connect-to-9router) · [🔌 OmniRoute](#-connect-to-omniroute) · [⚙️ Options](#️-cli-options) · [🛰️ Proxys](#️-behind-a-proxy) · [🐞 Problèmes](#-issues)

</div>

<div align="center">

**English** · [Türkçe](README.tr.md) · [中文](README.cn.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Français](README.fr.md) · [Deutsch](README.de.md) · [Русский](README.ru.md) · [Português](README.pt.md)

</div>

---

## ✨ Pourquoi cet outil existe

DuiAPI est un panneau de la famille New API : inscription par e-mail + mot de passe derrière un Cloudflare Turnstile, et **1 000 000 de quota de départ sur chaque nouveau compte** (`quota_per_unit` 500 000 ; le solde du premier compte affiche 1 000 000). Le catalogue contient **17 identifiants de modèles** répartis dans des groupes à tarif réduit (`default`, `deepseek-sale`, `glm-sale`, `qwen-sale`, `kimi-discount`), et le modèle par défaut vérifié par chat est `deepseek-v4.1-flash`. Le quota est rattaché au compte — c'est pourquoi cet outil crée **un compte par clé**.

## 🎁 Offre gratuite — ce que vous obtenez

**Chaque nouveau compte DuiAPI démarre avec 1 000 000 d'unités de quota.**

| Élément | Valeur |
| --- | --- |
| Quota de départ | **1 000 000** par compte (mesuré sur un compte neuf, `quotaStart`) |
| Inscription | e-mail + mot de passe + Cloudflare Turnstile |
| Vérification e-mail | code à 6 caractères depuis la boîte de réception (aller-retour mesuré à 18 s) |
| Catalogue de modèles | **17 identifiants** (tarification du panneau) ; modèles texte gratuits à ratio zéro : aucun — le quota constitue l'offre gratuite |
| Limites de débit | l'endpoint d'inscription a répondu `429` en rafale (24 tentatives : clés, 4× register-429, 5× codes mail manqués) — cadencez l'exécution, les nouvelles tentatives sont sûres |
| L'unité qui évolue | **un compte = une clé** (le quota est par compte) |

> 💡 L'offre gratuite est définie par DuiAPI et peut changer. Consultez les chiffres en direct sur [duiapi.com](https://duiapi.com) — et laissez le contrôle de l'outil décider :

```bash
node src/cli.js --verify-only     # every key in keys.txt is re-tested with a real chat call
```

## 🤖 Modèles — les 17 au complet

`GET /v1/models` sert le catalogue du panneau — **17 identifiants** (groupes à tarif réduit deepseek / glm / qwen / kimi). Le modèle par défaut vérifié par chat :

| ID du modèle | Remarque |
| --- | --- |
| `deepseek-v4.1-flash` | le modèle de vérification par défaut (groupe `default`, ratio 0,5) — chat-200 avec du contenu réel, delta de quota ~180 unités par petit appel |

```bash
curl https://duiapi.com/v1/chat/completions \
  -H "Authorization: Bearer t9jXE5…" -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4.1-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":8}'
```

## 🧪 Comment utiliser l'API

URL de base **`https://duiapi.com/v1`** — compatible OpenAI.

```python
from openai import OpenAI
c = OpenAI(base_url="https://duiapi.com/v1", api_key="t9jXE5…")
print(c.chat.completions.create(model="deepseek-v4.1-flash",
      messages=[{"role":"user","content":"hi"}]).choices[0].message.content)
```

Vérification du solde (ce sur quoi l'outil se base) : `GET /api/user/self` avec le jeton du compte, ou un appel chat de 1 jeton.

## 🛠 Démarrage rapide

```bash
cd duiapi-bulk-apikey
printf 'socks5://host:port\n' > proxies.txt   # optional — without it the tool runs direct
node src/launcher.js        # interactive: mint / verify / screen / settings

# or directly — three Turnstile tokens per account (see the note below):
node src/cli.js --count 5 --token "<tok1>,<tok2>,<tok3>"
# or wire a solver:  node src/cli.js --count 5 --token-cmd "your-solver-command"
```

DuiAPI garde ses appels register/sendCode/login derrière un Cloudflare Turnstile, donc un jeton frais est consommé par appel (trois par compte) : résolvez le widget sur <https://duiapi.com> dans n'importe quel navigateur et collez les valeurs `cf-turnstile-response`, ou pointez `--token-cmd` vers un solveur qui affiche un jeton sur stdout. `keys.txt` n'est écrit **qu'**après une complétion de chat en direct ; `accounts.json` conserve le nom d'utilisateur / l'e-mail / l'id de clé.

### 🖱️ Le lanceur

Le lanceur demande la liste de proxys (facultative — sans elle, le CLI s'exécute en direct) et le nombre, les mémorise, et exécute le CLI — mint, verify, screen exits, settings, help, Connect to 9Router.

## ✅ Vérifié et fonctionnel — chaque clé est testée

| Vérification | Résultat |
| --- | --- |
| `GET /api/status` | **200**, register+e-mail verification+turnstile tous à true, sitekey présent |
| `POST /api/user/register` (jeton Turnstile) | **200** `{"message":"","success":true}` |
| `GET /api/verification` | **200** + code à 6 caractères depuis la boîte de réception (`DuiAPI email verification`) |
| `POST /api/user/login` | **200** · `quota 1000000` |
| `POST /v1/chat/completions` (`deepseek-v4.1-flash`) — pour **chaque** clé avant son écriture | **200** avec un vrai `choices[0].message.content` et un delta de quota |
| une clé qui échoue à sa vérification de chat | jamais écrite — le compte reste dans `accounts.json` |

```bash
node src/cli.js --verify-only
node src/cli.js --screen
```

> **Contrôles qualité.** Chaque clé ici a passé un `chat-200` en direct avant d'être écrite dans `keys.txt`
> (les clés mortes ne sont jamais livrées). Les combos de routeur suivent la même règle au niveau du modèle : une entrée n'est
> ajoutée **que si le modèle répond `chat-200` et peut appeler des outils** — un modèle sans prise en charge des outils,
> une clé morte ou un fournisseur mort est écarté au lieu d'être ajouté. Relancez l'étape de vérification à tout moment ;
> les échecs sont signalés, jamais masqués.

## 🧭 Fonctionnement

```
inbox (one address per account)                                  a fresh address per account
   │
   ├─▶ sign-up: DuiAPI page → e-mail address → code / link from the inbox
   ├─▶ session: the account's cookies or access token in hand
   ├─▶ key mint: the account's key endpoint → plaintext key returned once (written to disk immediately)
   └─▶ live proof: POST https://duiapi.com/v1/chat/completions → real content = the key is live
```

| Étape | Appel | Remarques |
| --- | --- | --- |
| status | `GET /api/status` | contrôles : register activé, vérification e-mail activée, turnstile activé (sitekey servi par le panneau) |
| inscription | page register du panneau → Turnstile (résolu dans le navigateur) → `POST /api/user/register` | le jeton est à usage unique ; un nouveau par register/sendCode |
| vérification | `GET /api/verification?email=…` | le code à 6 caractères arrive dans la boîte de réception ; les mails de marque sont filtrés |
| connexion | `POST /api/user/login` | mot de passe en clair (chiffrement désactivé) ; jeton de session en main |
| clé | `POST /api/token/` (+ reveal) | clé renvoyée une fois, écrite immédiatement sur disque |
| preuve | `POST /v1/chat/completions` | chat-200 avec contenu = la clé est active |

## ⚙️ Options du CLI

```bash
node src/cli.js --count N        # mint N keys (screen exits, register, verify, chat-gate)
node src/cli.js --verify-only    # re-verify every key in keys.txt
node src/cli.js --screen         # exit pool self-test
node src/cli.js --connect-9router
node src/cli.js --help
```

| Env | Défaut | Signification |
| --- | --- | --- |
| `DUIAPI_PROXIES` | `proxies.txt` | pool de sorties, une ligne par proxy — facultatif : sans pool, le CLI se connecte directement |
| `DUIAPI_MODEL` | `deepseek-v4.1-flash` | modèle pour la vérification en direct |

## 🛰️ Derrière un proxy

**Facultatif — recommandé à grande échelle (limites par IP, bannissements).** L'outil fonctionne aussi sans proxy.

- **Avec un pool :** passez `--pool <file>` (ou définissez `DUIAPI_PROXIES`) et chaque requête passe par une sortie de cette liste — une ligne morte fait échouer cette tentative et l'outil passe à la suivante. L'inscription est l'étape la plus fragile (`429` en rafale) — l'outil réessaie avec la sortie suivante et cadence par conception.
- **Sans pool :** le CLI affiche `running WITHOUT a proxy (direct)` et se connecte directement. Rien ne change pour l'inscription, la création de clés ou la vérification — seul le point de départ des paquets change.
- Lancez `--screen` dans l'un ou l'autre mode : il teste les sorties dont vous disposez (ou la route directe) contre DuiAPI avant une exécution réelle.

## 📄 Fichiers

| Chemin | Contenu |
| --- | --- |
| `keys.txt` | une clé vérifiée en direct par ligne — ignoré par git |
| `accounts.json` | nom d'utilisateur / e-mail / id de clé — données de récupération |
| `proxies.txt` | vos sorties — ignoré par git |
| `outputs/` | journaux d'exécution — ignorés par git |

## 🔌 Connexion à 9Router

[9Router](https://9router.com) est un routeur auto-hébergé compatible OpenAI. DuiAPI parle OpenAI, donc inscrivez-vous une fois et chaque clé devient une connexion :

```bash
node src/connect-9router.js --base http://localhost:20128 --password 123456
```

Valeurs par défaut : `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Les modèles sont servis sous `duiapi/<model id>`.

## 🔌 Connexion à OmniRoute

[OmniRoute](https://omniroute.online) est un routeur auto-hébergé compatible OpenAI. DuiAPI parle OpenAI, donc inscrivez-vous une fois et chaque clé devient une connexion :

```bash
node src/connect-omniroute.js --base http://localhost:20128 --password 123456
```

Valeurs par défaut : `--prefix duiapi`, `--base-url https://duiapi.com/v1`, `--model deepseek-v4.1-flash`. Les modèles sont servis sous `duiapi/<model id>`.

## 🔑 Ce qu'une clé vous apporte

| | |
| --- | --- |
| 🏢 Fournisseur | **DuiAPI** — [duiapi.com](https://duiapi.com) |
| 🎁 Ce que le fournisseur offre | 1 000 000 de quota de départ par compte |
| 🔑 Ce qu'une clé vous apporte | une clé limitée à son propre compte : les 17 identifiants du catalogue sur `https://duiapi.com/v1`, selon le quota de votre compte |
| 🌐 Forme de l'API | compatible OpenAI : `POST https://duiapi.com/v1/chat/completions` |
| 🔒 Vos données | seuls `duiapi.com` et votre backend de boîte de réception sont contactés |

## 🐞 Problèmes

Si DuiAPI modifie la structure d'inscription, la sitekey Turnstile ou le palier de quota, l'outil s'arrête là et le signale. **[Ouvrez un ticket](../../issues)** avec la commande et la sortie exacte.

## 📄 Licence

MIT © 2026 — voir [LICENSE](LICENSE).

<div align="center">

**[MIT](LICENSE)** © 2026 duiapi-bulk-apikey contributors

</div>