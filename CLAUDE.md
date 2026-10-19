# splitea-legal: context for Claude

Static privacy policy, terms and support pages for Splitea, served by a Workers static-assets worker (no `main`, no worker code) at `splitea.app/legal/*` and `dev.splitea.app/legal/*`. Hand-written HTML, no build step.

## Project status: live users, every deploy is production

Splitea is live on the App Store (App ID 6760237781). The rules in `/Users/raulie/Workspace/Splitea/CLAUDE.md` apply here. For this repo that means:

- **One deploy serves both hosts.** `wrangler.toml` routes prod and dev to the same worker and there is no `[env.dev]`, so there is no dev-first step. Every `wrangler deploy` is a production deploy.
- **This page is a legal promise.** Released apps link to it from Settings and the paywall, App Review reads it, and real users rely on it. It must describe what the shipped code does today, not what a branch will do.
- **Deploy order:** when a change in code changes a data practice, the policy goes live no later than that code. When marketing copy elsewhere cites the policy (the splitea-web landing), this repo deploys first.

## Owner rules

- Deploying needs Raúl's explicit go, every time.
- Commit only when asked. One-line commit messages, no body, no Co-Authored-By lines.
- No em dashes or en dashes in any copy, meta tags included. All three pages are clean as of 2026-10-18; keep them that way.
- No explanatory comments, HTML comments included.
- Claims must match the code and the live policy. Never "on-device" scanning, "free forever", "no rounding", ratings or user counts.
- Every change ships in all 22 language blocks of the page in the same commit. No English-only edits.

## Where things live

| Path | What |
|---|---|
| `dist/legal/privacy.html` | Privacy policy, served at `/legal/privacy` |
| `dist/legal/terms.html` | Terms of Service, `/legal/terms` |
| `dist/legal/support.html` | Support page: contact card plus FAQ (safety, account and privacy, using Splitea, subscriptions, troubleshooting), `/legal/support` |
| `dist/legal/style.css` | Shared stylesheet; also hides inactive languages |
| `dist/legal/logo.svg` | The mark as a file. No page links it; each page inlines its own SVG copy |
| `wrangler.toml` | Routes, `[assets] directory = "./dist"`, `not_found_handling = "none"`, `workers_dev = false` |

Everything under `dist/` is public. `.html` is stripped from URLs, and an unknown path returns a plain 404.

## How a page is built

- Each page holds all 22 languages: `en, es, de, fr, it, ja, ko, pt, zh, zh-Hant, nl, sv, da, nb, fi, pl, cs, ro, hu, el, ar, he` (nl, sv, da and nb added 2026-10-16, the last eight 2026-10-18).
- `ar` and `he` blocks carry `dir="rtl"` on their opening div; the header and footer stay left to right. Lists use `padding-inline-start` so they indent on the correct side.
- The header title is one `<p data-lang="xx">` per language. The body is one `<div data-lang="xx">` per language (`class="legal-content"` on privacy and terms).
- An inline script at the bottom reads `?lang=`, else `navigator.language`, keeps the first two letters (`zh` with Hant/TW/HK/MO maps to `zh-Hant`), maps `no` and `nn` to `nb` and `iw` to `he`, falls back to `en` for anything unsupported, then toggles `.active`. CSS shows only `[data-lang].active`.
- Privacy and terms carry `<p class="updated">` in every block. Support has no date.
- Contact uses role addresses only (support@, privacy@, legal@ on splitea.app); there is no postal address.

## URL contract

Released apps open `https://splitea.app/legal/<privacy|terms|support>?lang=<code>` (`Splitea/Views/Settings/SettingsView.swift`, `Splitea/Views/Paywall/PaywallView.swift`), and splitea-web links to `/legal/privacy` and `/legal/terms` relatively, which on dev resolves to this same worker. Keep those three paths and the `?lang=` parameter working. The app sends only the language code, so Traditional Chinese users get `?lang=zh` and see Simplified.

## Changing a fact in every language

A fact usually lives in several places. The Web Analytics disclosure touched five spots in the policy (The Short Version, Splitea on the Web, the Cloudflare bullet, legitimate interests, the CCPA categories) plus the landing. The 30-day share lifetime touched privacy, terms and support (`b1f2c80`). Grep all three pages for the old wording before editing.

1. Edit the English block first. It is the source.
2. Make the same edit in the other 21 blocks at the same position, using the page's own terminology and register:

| Block | Address | receipt | split |
|---|---|---|---|
| `es` | tú | recibo | división |
| `de` | du in privacy, Sie in terms and support | Beleg | Split |
| `fr` | vous | reçu | addition |
| `it` | tu | scontrino | divisione |
| `ja` | お客様 | レシート | 割り勘 |
| `ko` | 귀하 | 영수증 | 정산 |
| `pt` | você (Brazilian; pt-PT readers get it too) | recibo | divisão |
| `zh` | 您 (Simplified) | 收据 | 分账 |
| `zh-Hant` | 您 | 收據 | 分帳 |
| `nl` | je | bon | split |
| `sv` | du | kvitto | nota |
| `da` | du | kvittering | split |
| `nb` | du | kvittering | regning |
| `fi` | sinä | kuitti | jako |
| `pl` | ty (Twój, Ci capitalized) | paragon | podział |
| `cs` | vy | účtenka | rozdělení |
| `ro` | tu | bon | împărțire |
| `hu` | te | blokk | felosztás |
| `el` | εσείς | απόδειξη | μοιρασιά |
| `ar` | أنت (Apple's Arabic register) | إيصال | تقسيم |
| `he` | gender-neutral impersonal forms | קבלה | חלוקה |

   Words above are from `privacy.html`; reuse whatever the surrounding paragraphs already say. In-app labels (Settings, Delete Account, Report, Stop Sharing) must match the app's own translations in `/Users/raulie/Workspace/Splitea/Splitea/Localizable.xcstrings` (`pt` uses pt-BR, `zh` uses zh-Hans, `nb` is the app's Norwegian).
3. Reviewer pass: a separate read of each block against the English, by a reviewer that did not write the translation (for example a fresh subagent), for meaning drift, dropped clauses, register and terminology.
4. Mechanical checks:

```bash
cd /Users/raulie/Workspace/splitea-legal
awk 'function out(){if(l)print l, "h1="a, "h2="h, "p="p, "li="i, "faq="q} /<div[^>]*data-lang="/{out(); match($0,/data-lang="[^"]+"/); l=substr($0,RSTART+11,RLENGTH-12); a=h=p=i=q=0} /<script>/{out(); l=""} l{a+=gsub(/<h1>/,"&"); h+=gsub(/<h2>/,"&"); p+=gsub(/<p[ >]/,"&"); i+=gsub(/<li>/,"&"); q+=gsub(/faq-item/,"&")}' dist/legal/privacy.html
/usr/bin/grep -n -E $'\xe2\x80\x93|\xe2\x80\x94' dist/legal/privacy.html
```

   The first prints element counts per language; every row must match. Run it on each page you changed. The second must print nothing for lines you touched.
5. Preview by opening the file in a browser and adding `?lang=ja` (and so on) to switch blocks.

## The policy must describe what the code does

Before publishing a data claim, read the code that does it. Repo docs are wrong in places: the splitea-id README says the client hashes phones (it sends E.164 and the server applies a keyed HMAC), and its migration 0001 says phones are never stored (an AES-GCM copy is stored since 0006).

| Area | Check in |
|---|---|
| Sign-in, directory profile, phone hash and encryption, email, push tokens, 7-day purge of unfinished sign-ins, sign-up alert | `/Users/raulie/Workspace/splitea-id` (`src/index.ts`, `migrations/`) |
| Scanning, AI provider and location hint, scan records, App Attest, reports, subscription notices, scan and subscription alerts | `/Users/raulie/Workspace/splitea-api` (`src/index.js`, `src/appleNotifications.js`, `src/llm/`, `migrations/`) |
| Share contents and lifetime (`SHARE_TTL_MS` in `src/receiptSession.ts`), public snapshot, live peers | `/Users/raulie/Workspace/splitea-shares` |
| What stays on the device and in iCloud, permissions, account deletion (`SettingsView.deleteAccount()`) | `/Users/raulie/Workspace/Splitea` |
| Web share page behavior | `/Users/raulie/Workspace/splitea-web` |
| Things Cloudflare adds at the edge | The live site (below) |

**Cloudflare Web Analytics** is injected by Cloudflare into every splitea.app page for real browsers, this one included, and it is disclosed. It does not show for plain curl. To see it:

```bash
curl -s -H 'Accept: text/html' -A 'Mozilla/5.0' https://splitea.app/legal/privacy | grep cloudflareinsights
```

**AI provider.** splitea-api picks the model with `LLM_PROVIDER` in its `wrangler.toml` (`gemini` today); `src/llm/` also has grok and claude. The policy names Gemini and promises an update before any switch, so the policy changes first.

Facts verified on 2026-10-08 that are easy to get wrong (re-check before relying on them):

- Raw coordinates go into the Gemini prompt (iOS supplies a reduced-accuracy position).
- Scan alerts carry the user's name and the merchant.
- After in-app deletion, splitea-api still keeps scan rows, the `users(sub, full_name)` row and entitlement rows. The policy's deletion section lists this gap.
- The Apple token revoke is never called.
- Anyone with a share link can fetch its snapshot, participants' phones and the receipt photo included. Everyone connected to a live share sees the others' account IDs.

Open assumptions in the current text: the Gemini API is on the paid tier, Cloudflare keeps logs 7 days and D1 backups 30 days. Rewrite the "Some things aren't removed automatically yet" paragraph when the deletion fixes ship.

## Decisions to keep

- **Telegram alerts:** disclosed in one short general line under "Who Else Handles Your Data" ("basic account and scan details"), plus its retention line, "alerts" under legitimate interests and the processing-location mention. Agreed with Raúl on 2026-10-08 as the compromise: he keeps the alerts as a monitoring tool, and the policy does not hide them. Keep the line short and general; it can go only if the alerts are removed. The sources are splitea-id `sendTelegram`, splitea-api `sendScanAlert` and `src/appleNotifications.js`; the scan alert is the `scan_alerts_telegram` config flag.
- **Web Analytics:** kept and disclosed, as above.
- **Landing consistency:** the privacy section of the splitea.app landing (`privacy` block in `/Users/raulie/Workspace/splitea-web/src/views/landing/copy/*.ts`, one file per language, plus the FAQ "Where is my data?") summarizes this policy: Gemini named, scan records, share contents, 30-day links, no ads or tracking SDKs, Web Analytics without cookies, the profile kept on servers until deletion, deletion in Settings. It leaves out Telegram on purpose. When a fact changes here, update that copy in all its languages too, and deploy this repo before splitea-web (dev first there).

## Last updated date

- Bump `<p class="updated">` in all 22 blocks whenever the substance of privacy or terms changes, in each block's existing date format: `grep -n 'class="updated"' dist/legal/privacy.html`.
- The policy promises an in-app notice for important changes. Flag any material change to Raúl so he can decide on the notice.

## Deploy

1. Get Raúl's go.
2. Check what will ship. The deploy uploads `dist/` as it is on disk, uncommitted edits included, and earlier commits were once pushed but never deployed (August 2026). See what differs from live:

```bash
cd /Users/raulie/Workspace/splitea-legal
git status --short
for p in privacy terms support; do curl -s https://splitea.app/legal/$p | diff -q - dist/legal/$p.html >/dev/null && echo "$p same" || echo "$p changes"; done
```

3. Deploy. There is no `node_modules`, so `npx` fetches a current wrangler, which needs Node 22; the default `node` is 20:

```bash
cd /Users/raulie/Workspace/splitea-legal
PATH=/opt/homebrew/opt/node@22/bin:$PATH npx wrangler deploy
```

4. Re-run the loop from step 2 (every page should say `same`), and open `https://splitea.app/legal/privacy?lang=ja` to spot-check a block.
5. If splitea-web copy depends on the change, deploy it next (dev first, then prod).

## Known issues and stale docs

- `privacy-policy-1.6.1` is a local branch already fast-forwarded into `main`. Work on `main`.
- Facts fixed in every language on 2026-10-16: the paid tier is Splitea Unlimited (never Pro), free scans are a lifetime allowance per account (the server counts per Apple sign-in), Apple's account is "Apple Account" in Apple's own wording (es cuenta de Apple, it and ja Apple Account), in-app deletion erases receipts from the device and iCloud, and the app labels are Share Split, Restore and Report User. Terms is dated October 16, 2026.
- Facts fixed in every language on 2026-10-18: a locked share lets others only view the split and record payments; Sign in with Apple sits at Settings → [your name] → Sign in with Apple; iOS 26 blocking is Messages → the person's icon → Info → Block Contact, or Contacts → Block Contact, with the list at Settings → Privacy & Security → Blocked Contacts; app permissions are under Settings → Apps → Splitea; in-app blocking is Block User on the edit-contact sheet, unblocked under Blocked Users; items are edited or deleted from their touch-and-hold menu and added with Add Item in edit mode; the photo-screening sentence opens the closing paragraph of User Profile Content; every terms block has the renewal-charge bullet. Privacy and terms are dated October 18, 2026.
