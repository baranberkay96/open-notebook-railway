# Deploy and Host Open Notebook with Railway

Open Notebook is an open-source, privacy-focused alternative to Google NotebookLM. Collect PDFs, web pages, YouTube videos, audio and notes into notebooks, chat with them with citations, search across everything and turn your material into multi-speaker podcasts, using the AI providers you choose. This template runs Open Notebook with SurrealDB, password protection and persistent storage.

## About Hosting Open Notebook

Open Notebook's official image runs three processes: a Next.js web app, a FastAPI backend and a background worker that ingests sources, creates embeddings and renders podcasts. They share one data directory for uploads and audio, and store everything else in SurrealDB. Hosting it means keeping those processes together, giving them persistent storage, protecting the instance with a password and an encryption key for stored API keys, and pointing the browser at the right API URL. This community template pins Open Notebook 1.15.0 and SurrealDB 2.7.0, generates every secret, serves the UI and API from one HTTPS domain, and adds healthchecks so a deploy only goes live once the database is migrated and the app answers.

## Common Use Cases

- Research notebooks that combine papers, articles and videos, with answers that cite their sources
- A private knowledge base for a team or client project, kept in your own database
- Generating podcast-style audio summaries of reading material
- Summaries, insights and custom transformations over large sets of documents
- Self-hosted NotebookLM-style workflows with local or alternative models, such as Ollama on another server or any OpenAI-compatible endpoint

## Dependencies for Open Notebook Hosting

- SurrealDB (included)
- At least one AI provider account, added after deployment in the app (OpenAI, Anthropic, Google, OpenRouter, Groq, Mistral, Ollama and others)
- Optional: a text-to-speech provider for podcasts (for example OpenAI, ElevenLabs or Google)

### Deployment Dependencies

- Open Notebook repository: https://github.com/lfnovo/open-notebook
- Documentation: https://www.open-notebook.ai
- Environment reference (1.15.0): https://github.com/lfnovo/open-notebook/blob/v1.15.0/docs/5-CONFIGURATION/environment-reference.md
- SurrealDB documentation: https://surrealdb.com/docs

### Implementation Details

| Service | Source | Purpose |
|---|---|---|
| Open Notebook | `lfnovo/open_notebook:1.15.0` | Web UI on port 8502 (also proxies `/api/*`), API and background worker; volume `/app/data`; healthcheck `/api/config` |
| SurrealDB | `surrealdb/surrealdb:v2.7.0` | Notebooks, sources, notes, embeddings, job queue and encrypted credentials; RocksDB on volume `/data`; private only |

**First login**

1. Open the service URL and enter `OPEN_NOTEBOOK_PASSWORD` from the Open Notebook service's Variables tab. The same value works as a Bearer token for the REST API.
2. Go to Manage → Models, add a credential for your AI provider, test it and choose default chat and embedding models (plus a text-to-speech model for podcasts). Keys are stored encrypted with `OPEN_NOTEBOOK_ENCRYPTION_KEY`; keep that variable unchanged.
3. Create a notebook, add a source and start asking questions once processing finishes.

**Custom domain**: after adding one in Railway, set `API_URL` and `CORS_ORIGINS` on the Open Notebook service to `https://{your-domain}` and redeploy.

**Scaling**: the web app, API and worker share the `/app/data` volume, so the service runs as one replica. Give it more RAM and CPU for large documents or parallel podcast jobs, and tune `OPEN_NOTEBOOK_WORKER_MAX_TASKS` to your provider's rate limits.

**Pinning and upgrades**: both images use exact version tags. Change the tags and redeploy; Open Notebook applies its database migrations on start. Back up the SurrealDB volume before major upgrades.

### Why Deploy Open Notebook on Railway?

Railway keeps the app and its database on a private network with persistent volumes and HTTPS, so your research workspace is online in minutes and your sources stay on infrastructure you control. You pay for the resources it uses, and AI usage goes directly to the providers you pick.
