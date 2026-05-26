# splitea-legal

Cloudflare Worker that serves [Splitea](https://github.com/Raulie/Splitea)'s legal + support pages as static HTML at `splitea.app/legal/*`.

Three pages, one stylesheet, one logo. No JavaScript, no server-side rendering — Cloudflare serves the bytes from edge cache.

| URL | File |
|---|---|
| `https://splitea.app/legal/privacy` | `dist/legal/privacy.html` |
| `https://splitea.app/legal/terms` | `dist/legal/terms.html` |
| `https://splitea.app/legal/support` | `dist/legal/support.html` |

Plus shared assets (`style.css`, `logo.svg`) served from the same path.

## Why a separate worker

Split out of `splitea-shares` and `splitea-web` so that:

- The bundle stays static. No JS execution, no cold-start tax, no SPA hydration to wait on.
- Updates ship independently. Fixing a typo in the Privacy Policy doesn't require redeploying `splitea-web`'s SPA bundle.
- Apple's review process reaches a fast, JS-free page even if the reviewer's browser blocks scripts.

Pairs with the other workers under `splitea.app` via path-precedence routing:

| Worker | Path | What it serves |
|---|---|---|
| `splitea-shares` | `/r/*`, `/p/*`, `/live/*`, `/.well-known/*`, favicons | Share-link HTML, ATH Móvil pay landing, WebSocket relay, AASA |
| **`splitea-legal`** | `/legal/*` | This worker |
| `splitea-id` | `/id/*` | Profile + identity API |
| `splitea-web` | everything else | SPA + future marketing surface |

## Develop

The pages are hand-written HTML. Edit `dist/legal/*.html` directly; preview by opening the file in a browser.

## Deploy

```bash
npx wrangler deploy
```

Routes are registered for both prod and `dev.splitea.app` so the iOS in-app About screen can deep-link to `dev.splitea.app/legal/*` from pre-release builds without falling through to `splitea-web`'s 404.

## Related repos

- [Splitea](https://github.com/Raulie/Splitea) — iOS client (links here from the About screen).
- [splitea-web](https://github.com/Raulie/splitea-web) — apex SPA. Co-routed on `splitea.app`.
