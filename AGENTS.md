# Working in this repo (Base44 sandbox)

Interstellar is a single Node/Express process that serves a static UI, hosts the
Mercury Workshop transports (bare-mux `/bm`, epoxy `/ep`, bare `/ca/`, Wisp on the
same port) and rewrites proxied traffic through a service worker. There is no
build step, no database and no external service to configure.

## Run it

```bash
docker compose -f docker-compose.base44.yml up -d --build
docker compose -f docker-compose.base44.yml logs -f web
```

- Serves on host port **3000**; healthcheck polls `/` and one static asset.
- The app runs from the bind-mounted source with `node --watch index.js`, so backend
  edits restart the process and static assets (`static/**`) are re-read per request.
  Only call `reload_preview` after a restart that the user must see (e.g. compose or
  dependency changes).
- `index.js` sets `const __dirname = process.cwd()`, so the process must run from the
  repo root (`working_dir: /app`) or the static routes 404.

## Non-obvious details

- Dependencies are installed at container start from `pnpm-lock.yaml`
  (`pnpm install --frozen-lockfile`), using pnpm 10.33.0 via corepack. `node_modules`
  and the pnpm store are named volumes, so `docker compose down -v` wipes deps but not
  app data (there is none).
- `/bm` and `/ep` are served straight out of `node_modules`
  (`@mercuryworkshop/bare-mux`, `@mercuryworkshop/epoxy-transport`) — if those assets
  404, the dependency install did not finish.
- The Scramjet/UV bundles under `static/assets/{languagearts,mathematics,history}` are
  committed prebuilt files served by `static/sw.js`; they are unrelated to the npm
  `@mercuryworkshop/scramjet` package.
- The client derives its Wisp URL from `location.host` (`wss://<host>/wisp/`), so no
  public URL or CORS configuration is needed here.
- The Google AdSense script and the ad-injection block in `static/assets/js/m1.js` are
  part of upstream; the `Ad-Free` branch strips them.

## Credentials

None required. `config.js` has `challenge: false`; setting it to `true` requires the
`config=true` env var plus the users/passwords in `config.js`. The optional Masqr
license check (`MASQR=true`) is commented out and would reach an external license
server — ask before enabling it.
