# Open Notebook v1 AI Research Assistant: Railway template spec

## Listing

| Field | Value |
|---|---|
| Name | **Open Notebook v1 AI Research Assistant** (LISTINGS_ROUND2 base name; pinned release 1.15.0, project tags use `v`) |
| Description | Private NotebookLM alternative: chat with your sources, generate podcasts. (74 chars, unchanged) |
| Category | AI/ML |
| Tags | open-notebook, notebooklm, research, rag, podcast, surrealdb, llm, knowledge-base |
| Icon | `https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/open-notebook.svg` (200 OK) |

The description is delivered: sources (PDF, web, YouTube, audio, text), chat with citations, search, notes, transformations and multi-speaker podcast generation all work once the user adds an AI provider in the UI (podcasts additionally need a text-to-speech provider).

## Base market template

- **Base: `open-notebook` (monotykamary, 11 active / 42 total)**, the strongest of the three: pinned app (1.14.0) and SurrealDB (2.6.5), generated SurrealDB credentials, encryption key, CORS limited to the public origin, two volumes. Problems fixed: an extra Caddy proxy service that is no longer needed (since the 1.x frontend proxies `/api/*` to the backend itself, see `frontend/next.config.ts` rewrites, one port covers UI and API), password protection off by default (README says to set it later), `SURREAL_CAPS_ALLOW_ALL=true` (wider SurrealDB capabilities than upstream uses), no healthcheck on the app or SurrealDB (only on the proxy).
- `open-notebook-1` (12 active): `v1-latest` floating tag, no healthcheck, password variable mislabelled, `OPENAI_API_KEY` as env (deprecated upstream). Nothing taken.
- `opennotebook` (2 active): single public port 8502 with `API_URL` and `CORS_ORIGINS` on the own domain, generated password and encryption key, SurrealDB healthcheck. Same topology as chosen here; its README text was not reused.
- `_upstream/monotykamary__railway-open-notebook-caddy-proxy` (MIT, Copyright (c) 2024 Brody): reviewed; not used because the proxy is unnecessary.

## Pinned versions

| Service | Image | Version | Released | Checked |
|---|---|---|---|---|
| Open Notebook | `lfnovo/open_notebook:1.15.0` (index digest `sha256:54b2025a4a48…`) | 1.15.0 (latest release) | 2026-10-04 | 2026-10-11 |
| SurrealDB | `surrealdb/surrealdb:v2.7.0` | 2.7.0 (latest v2; upstream compose pins the `v2` major) | 2026-09-23 | 2026-10-11 |

Not used: `v1-latest` (floating), `1.15.0-single` (bundles SurrealDB in the app container), SurrealDB v3 (upstream still targets v2).

Upgrade: change the image tags, redeploy. Open Notebook runs its SurrealDB migrations at API start (tested: 25 migrations applied on an empty database). Back up the SurrealDB volume before major upgrades.

## Architecture

```
 Browser ── HTTPS ──► Open Notebook :8502 (public)          /api/config = healthcheck
                      ┌───────────────────────────────────────────────────────────────┐
                      │ lfnovo/open_notebook:1.15.0 (supervisord, upstream image)       │
                      │  ├─ frontend: Next.js :8502, rewrites /api/* → localhost:5055   │
                      │  ├─ api: FastAPI :5055 (password auth, migrations on start)     │
                      │  └─ worker: surreal-commands (ingestion, embeddings, podcasts)  │
                      │ volume /app/data: uploads, podcasts, caches                     │
                      └───────────────┬───────────────────────────────────────────────┘
                                      │ ws://surrealdb.railway.internal:8000/rpc
                                      ▼
                      SurrealDB v2.7.0 (RocksDB, volume /data, root user generated)
```

Why the worker stays in the app container: the API saves uploads to `/app/data` and the worker reads them, and podcasts the worker renders are served by the API from the same directory. Railway volumes cannot be shared between services, so splitting the worker into its own service would break uploads and podcasts. Upstream's image runs all three processes under supervisord for this reason.

Why no Caddy: upstream 1.x serves the API through the frontend's `/api/*` rewrite and SSE proxy routes, documented in `docs/5-CONFIGURATION/reverse-proxy.md` (since v1.1 "You only need to proxy to one port"). One public port is enough.

## Services

### Open Notebook

| Setting | Value |
|---|---|
| Source | image `lfnovo/open_notebook:1.15.0` (entrypoint and supervisord unchanged) |
| Healthcheck | `/api/config` via port 8502: answers only after the frontend is up and proxies to a running API (frontend starts after the API's `/health` passes); unauthenticated by upstream design. Timeout 600 s (API waits for SurrealDB with retries, then runs migrations) |
| Public domain | yes, port 8502 |
| Volume | `/app/data` |
| Restart | ON_FAILURE, 10. Inside the container supervisord restarts the API if it gives up waiting for the database. |

Key variables: `API_URL` and `CORS_ORIGINS` = `https://${{RAILWAY_PUBLIC_DOMAIN}}` (without `API_URL` the frontend's auto-detection would point the browser at `https://{domain}:5055`, which Railway does not expose), `OPEN_NOTEBOOK_PASSWORD` generated, `OPEN_NOTEBOOK_ENCRYPTION_KEY` generated, `SURREAL_*` referenced from the SurrealDB service. No AI provider variables: upstream marks provider env vars as deprecated; keys are added and encrypted in the UI.

### SurrealDB

| Setting | Value |
|---|---|
| Source | image `surrealdb/surrealdb:v2.7.0`, start command `/surreal start` (flags from `SURREAL_*` variables) |
| Healthcheck | `/health` on `PORT=8000`, timeout 300 s |
| Public domain | none (private network only) |
| Volume | `/data` (`rocksdb:/data/open-notebook.db`) |
| User | `RAILWAY_RUN_UID=0` (image user 65532 cannot write the root-owned volume) |

Bound to `[::]:8000` (Rust listener, dual-stack: verified that IPv4 clients connect). Capabilities left at SurrealDB defaults (upstream does not widen them).

## First login

1. Open the public URL, enter `OPEN_NOTEBOOK_PASSWORD` from the Open Notebook service variables.
2. Manage → Models: add a credential for your provider (OpenAI, Anthropic, Google, OpenRouter, Ollama on another host, any OpenAI-compatible endpoint, ...), test it, register models, and set default chat, embedding and (for podcasts) text-to-speech models.
3. Create a notebook, add a source (URL or PDF), wait for processing (worker), then chat.

## Improvements over market templates

1. **Password protection on from the first request** (generated), encryption key generated, CORS limited to the own origin.
2. **One service fewer than the market leader** with the same single-domain result (no Caddy), and **healthchecks** on both services with Railway `PORT` set.
3. **Pinned current release** (1.15.0, SurrealDB 2.7.0) instead of `v1-latest` / `v2`.
4. SurrealDB with upstream capabilities (no `allow-all`), RocksDB as upstream, generated root user and password, clean shutdown drain.
5. Provider keys handled the upstream way (encrypted in the database via the UI) instead of deprecated env vars, so the template boots with zero required inputs.

## Resource estimate

Railway prices: RAM $10/GB-month, vCPU $20/vCPU-month, volume $0.15/GB-month.

| Service | Idle RAM / CPU | Typical RAM / CPU | Notes |
|---|---|---|---|
| Open Notebook | 0.45 GB / 0.02 vCPU (measured 423 MB: Next.js + API + worker) | 1.2 GB / 0.3 vCPU | Source processing, embeddings and podcast audio (ffmpeg) in the worker; `OPEN_NOTEBOOK_WORKER_MAX_TASKS` bounds concurrency |
| SurrealDB | 0.2 GB / 0.01 vCPU (measured 212 MB) | 0.4 GB / 0.05 vCPU | RocksDB block cache grows with data |
| Volumes | app 0.2 GB, db 0.1 GB | app 3 GB, db 1 GB | uploads and podcasts dominate |

Estimated monthly cost: **idle ≈ $7** (0.65 GB × $10 + 0.03 vCPU × $20), **typical ≈ $23** (1.6 GB × $10 + 0.35 vCPU × $20 + 4 GB × $0.15). AI provider usage is billed by the provider.

Minimum plan: **Trial** can boot it (each service < 1 GB idle) but processing large PDFs or podcasts may exceed 1 GB; **Hobby** recommended.

## Verification checklist

Local (done 2026-10-11, `linux/amd64` images, `docker run --init`, SurrealDB as uid 0 on a volume with the template's `SURREAL_*` variables): SurrealDB `Started web server on [::]:8000`, `/health` 200 from another container; Open Notebook waited for the DB, applied migrations 1–25, API `/health` 200, frontend started; `/api/config` 200 (`dbStatus: online`), `/config` returns the configured `API_URL`; `/api/notebooks` 401 without and 200 with the password; created a notebook, restarted both containers, notebook still present.

On first Railway deploy:

- [ ] Both services healthy (`/health` on SurrealDB, `/api/config` on Open Notebook).
- [ ] Public URL asks for the password; the generated value logs in.
- [ ] Add a provider credential in Models and pass its connection test (proves `OPEN_NOTEBOOK_ENCRYPTION_KEY` works).
- [ ] Add a web source and a PDF; both finish processing (worker); chat answers with citations.
- [ ] Generate a short podcast (needs a TTS model); the audio plays.
- [ ] Redeploy both services; notebooks, sources and the podcast file are still there.

## Open questions

- Large uploads: upstream allows 100 MB through the frontend proxy and the API; Railway's edge limits were not tested with files of that size.
- Chat streaming goes through Next.js SSE proxy routes; Railway's 15-minute request limit is far above typical answers but very long podcast generations run as background jobs anyway.
- If a custom domain is added, `API_URL` and `CORS_ORIGINS` must be updated to it (documented in the README).
