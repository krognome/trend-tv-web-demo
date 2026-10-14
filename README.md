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

Every route is now behind a real login screen — nothing else in the app
renders until `auth.trend.app` returns a session, no more silent access
on top of fixture/demo data regardless of auth state.

**Known limitation on *this* static demo specifically:** the login
*screen* itself renders correctly here, but submitting it will fail
with a network error. `auth.trend.app` sends no
`Access-Control-Allow-Origin` header at all (confirmed directly), so a
browser blocks reading its response from any origin that isn't the
API's own server — this static GitHub Pages site has no server of its
own to route around that, unlike `yarn dev` (which proxies around it
locally — see `apps/tv-web/vite.config.ts`'s `server.proxy` comment in
the source repo). Real login only works today via `yarn dev` in the
source repo. Fixing this here needs either CORS headers added to
`auth.trend.app` for this demo's origin, or a small standalone proxy —
neither exists yet.

## How to refresh this snapshot

From the `wpconcierges/trend-tv` repo, with this repo's name substituted
for `<repo-name>` if you renamed it from `trend-tv-web-demo`:

```bash
cd apps/tv-web
yarn vite build --base=/<repo-name>/ --outDir /path/to/this/repo --emptyOutDir
```

Then commit and push the changed files in this repo. GitHub Pages
redeploys automatically on every push to `main`.
