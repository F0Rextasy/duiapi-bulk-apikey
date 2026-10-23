# Advanced usage (escape hatches)

Everything below is **optional**. The normal path is one command:

```bash
node src/cli.js --count 5      # or: npm start → "1) Mint keys"
```

The CLI solves the panel's captcha by itself (`--auto`, the default) through the
bundled `src/solver.js`, pinned to the same exit that spends the token.

## Run your own solver instead

`--auto` is skipped the moment you supply your own token source, so an existing
solver keeps working unchanged:

| Flag | What it does |
| --- | --- |
| `--token <tok>[,<tok>…]` | use captured token(s) verbatim (one is spent per panel call) |
| `--token-cmd "<cmd>"` | run a command that prints one fresh token on stdout (once per call) |
| `--manual` | print the widget URL and paste the token by hand (attended runs only) |

Environment variables mirror the flags (for example `DUIAPI_TOKEN_CMD`), which is
handy on a headless/server box where no desktop browser can open.

## The embedded solver

- `src/solver.js` drives a real, **headed** Chrome (headless is refused by the
  Turnstile widget: measured 0/N).
- Chrome is auto-detected; set `CHROME_PATH` (or `PUPPETEER_EXECUTABLE_PATH`) if
  it lives somewhere unusual, or install it once:
  `winget install --id Google.Chrome`.
- Every account's pinned exit (`--pool`) is handed to the solver as `--proxy`, so
  the token is solved and spent through the same IP. Without a pool it runs DIRECT.

## Terms

Automating account creation may violate the provider's terms of service. You are
responsible for how you use this tool; keep the cadence human and the per-IP
account count low.
