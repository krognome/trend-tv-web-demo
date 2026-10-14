# Trend TV — Web Demo (static snapshot)

This repo hosts a pre-built, static production build of the Trend TV
webOS/Tizen/VIDAA web client (`apps/tv-web` in the source monorepo,
`wpconcierges/trend-tv`), published via GitHub Pages.

**This is not the source code** — it's a build artifact, committed here
specifically because the source repo is private/shared (not owned by
this account) and can't host GitHub Pages for this account. There is no
`yarn install`/build step in this repo's own CI; the workflow just
republishes whatever static files already exist here.

## Live demo

Once GitHub Pages is enabled for this repo (Settings → Pages → Source →
"GitHub Actions" — one-time, manual, repo-admin action), this will be
live at whatever URL GitHub assigns (shown on that same Settings → Pages
screen, and in the Actions tab's deployment output).

**How to test:** open the URL in Chrome, press F12 → toggle device
toolbar → set the viewport to 1920×1080, then navigate using **only the
keyboard arrow keys and Enter** (not the mouse) — that's what exercises
the actual TV Focus Engine.

## Login gate (new)

Every route is now behind a login screen — nothing else in the app
renders until a session exists, no more silent access on top of
fixture/demo data regardless of auth state.

**This build runs in simulated-login mode (`VITE_TREND_AUTH_DEMO_MODE=true`),
clearly marked with a "DEMO MODE" badge in the corner of the login
screen.** Any non-empty email/password "succeeds" locally with
obviously-fake values — nothing here is a real `auth.trend.app` call.
That's deliberate, not a shortcut taken quietly: `auth.trend.app` sends
no `Access-Control-Allow-Origin` header at all (confirmed directly), so
a browser blocks reading its response from any origin that isn't the
API's own server, and this static GitHub Pages site has no server of
its own to route around that. Real login against the live backend is
verified and working — just not reachable from *this* static build.
See the "Real login" section below for where that actually runs.

### Real login

`yarn dev` in the source repo (`wpconcierges/trend-tv`, `apps/tv-web`)
talks to the real `auth.trend.app`, proxied around the CORS gap above
by Vite's own dev server (see `vite.config.ts`'s `server.proxy`
comment) — that's where to see an actual account log in, not this
static snapshot. Fixing this here for good needs either CORS headers
added to `auth.trend.app` for this demo's origin, or a small standalone
proxy — neither exists yet.

## For You feed — real, not fixture data (new)

Even though login above is simulated, the **For You** tab on this demo
pulls real data from `https://api-dev.trend.app/v1/entity/feed/` — a
different host than `auth.trend.app`, and (confirmed live, by direct
`curl`) one that *does* send a proper `Access-Control-Allow-Origin: *`
header. The CORS gap only ever blocked login, never this endpoint.

This build has one real, pre-minted session (`access_token`,
`tavuid`, `tavsid`, `trend_api_token` for one specific test account)
baked in via `VITE_TREND_DEMO_*` build-time env vars (see
`env.ts`/`trendAuthClient.ts` in the source repo for exactly how).
Submitting any email/password on the login screen above returns that
real session instead of a fully-fake one, which is enough for
`HybridDataSource` to route the For You feed to the real API.

**Known limits, on purpose, not hidden:**
- That baked-in `trend_api_token` has a real expiry (~12 hours,
  decoded from its own JWT on 2026-10-14) — after that, For You will
  stop returning real data until someone re-mints a token and rebuilds
  this snapshot per the refresh instructions below.
- The token is a real, usable bearer credential for one specific test
  account, shipped in this public repo's JS bundle — extractable by
  anyone who opens it. Acceptable only because it's a disposable test
  account, not a real user's login. Don't reuse this pattern for a real
  account's credentials.

## How to refresh this snapshot

From the `wpconcierges/trend-tv` repo, with this repo's name substituted
for `<repo-name>` if you renamed it from `trend-tv-web-demo`:

```bash
cd apps/tv-web
VITE_TREND_AUTH_DEMO_MODE=true \
VITE_TREND_API_URL=https://api-dev.trend.app/v1 \
VITE_TREND_DEMO_ACCESS_TOKEN=<fresh access_token> \
VITE_TREND_DEMO_TAVUID=<fresh tavuid> \
VITE_TREND_DEMO_TAVSID=<fresh tavsid> \
VITE_TREND_DEMO_TREND_API_TOKEN=<fresh trend_api_token> \
yarn vite build --base=/<repo-name>/ --outDir /path/to/this/repo --emptyOutDir
```

`VITE_TREND_AUTH_DEMO_MODE=true` is what's behind the simulated-login
behavior above — omitting it would point this static build at the real
`auth.trend.app` again, which (per the CORS gap above) would make every
login attempt here fail with a network error instead.

The four `VITE_TREND_DEMO_*` vars are optional — omit all four to go
back to the fully-fake session (For You falls back to fixture data,
same as before this feature existed). Include all four, freshly minted
(see the source repo's `realForYouFeedAdapter.ts`/`trendAuthClient.ts`
for the real `token/app` → `token/create` → `users/login` sequence), to
get a real For You feed again for another ~12 hours.

Then commit and push the changed files in this repo. GitHub Pages
redeploys automatically on every push to `main`.
