# NOTICE: Open Notebook v1 AI Research Assistant template

## Software used

| Component | Source | License | How it is used |
|---|---|---|---|
| Open Notebook | `lfnovo/open_notebook:1.15.0`, https://github.com/lfnovo/open-notebook | MIT, Copyright (c) 2024 Luis Novo | Official image, run unmodified |
| SurrealDB | `surrealdb/surrealdb:v2.7.0`, https://github.com/surrealdb/surrealdb | Business Source License 1.1 (converts to Apache-2.0 after the change date); self-hosting for your own application is permitted, offering SurrealDB as a commercial database service is not | Official image, run unmodified as the app's database |

Product names belong to their owners. This is a community template, not affiliated with the Open Notebook project or SurrealDB Ltd.

## Upstream files and docs checked (2026-10-11)

- `lfnovo/open-notebook` tag `v1.15.0` (latest release, 2026-10-04): `docker-compose.yml` (SurrealDB `v2`, RocksDB, `SURREAL_EXPERIMENTAL_GRAPHQL=true`, app ports 8502/5055, volume `/app/data`), `Dockerfile` (supervisord with api, worker, frontend; `API_HOST=0.0.0.0`), `supervisord.conf`, `scripts/docker-entrypoint.sh`, `scripts/wait-for-api.sh`, `.env.example`, `api/main.py` (DB wait + migrations, password middleware exclusions `/health`, `/api/auth/status`, `/api/config`), `api/auth.py`, `api/routers/config.py`, `frontend/next.config.ts` (`/api/*` rewrites), `frontend/src/app/config/route.ts` (API URL auto-detection to port 5055 when `API_URL` is unset), `docs/5-CONFIGURATION/{environment-reference,security,reverse-proxy}.md`, `docs/3-USER-GUIDE/api-configuration.md`.
- Docker Hub tags: `lfnovo/open_notebook` (`1.15.0`, `1.15.0-single`, `v1-latest`), `surrealdb/surrealdb` (`v2` = `v2.7.0`, v3 line not used by upstream); `surreal start --help` of v2.7.0 for the `SURREAL_*` environment variables.

## Market templates and repositories reviewed

| Template / repo | License | Use |
|---|---|---|
| `open-notebook` / `monotykamary/railway-open-notebook-caddy-proxy` | MIT, Copyright (c) 2024 Brody | Base for service variables (pinned images, generated SurrealDB credentials, encryption key, CORS). Proxy not used, no files adapted. |
| `open-notebook-1` (image-only) | n/a | Reviewed, nothing taken. |
| `opennotebook` (image-only) | n/a | Reviewed; same single-port topology. No text reused. |

All files in this package (docs and `template.json`) were written for this template. No README text from other templates was reused.
