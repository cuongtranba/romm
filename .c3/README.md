---
id: c3-0
c3-seal: c9972372674bf8f1c650d6687070187318a3996dcad17b90049f43c2197c52c7
title: romm
goal: Self-hosted ROM manager that scans, enriches with metadata from external services, exposes a REST + WebSocket API, and serves a Vue web UI for browsing and playing emulated games in the browser.
---

## Summary

RomM packages a FastAPI backend (HTTP API + Socket.IO + RQ worker + scheduler + filesystem watchers) and a Vue 3 / Vuetify SPA into a single Docker image, backed by MariaDB/Postgres and Valkey/Redis, integrating with IGDB, ScreenScraper, MobyGames, RetroAchievements, SteamGridDB, LaunchBox, Hasheous, HLTB, Libretro, Flashpoint, and TGDB metadata sources.

## Abstract Constraints

- Self-hosted, single Docker image deployable: backend, worker, scheduler, watcher and frontend dev server all start from `entrypoint.sh`.
- Multi-database support via SQLAlchemy 2.0: MariaDB, MySQL and Postgres connectors are all installed and selected by env.
- Async-first Python (FastAPI + asyncio + httpx + aiohttp); long-running work is offloaded to RQ workers, never run inline in HTTP handlers.
- Authentication is hybrid: session cookies, basic auth, OIDC (Authentik or generic), and CSRF protection on state-mutating routes.
- Metadata adapters are pluggable: each external provider lives under `backend/adapters/services/` and is wrapped by a corresponding `backend/handler/metadata/<provider>_handler.py`.
- Filesystem layout is the source of truth for the library; scans are driven by `watchfiles` and reconciled into the database.
- AGPL-3.0 license — every contribution and derivative must remain license-compatible.
- AI-assisted contributions must be disclosed in pull requests (per `CONTRIBUTING.md`).
- Strong typing required: backend uses Python 3.13 with `mypy`; frontend uses TypeScript with `vue-tsc`.

## Containers

| ID | Name | Boundary | Goal |
| --- | --- | --- | --- |
| c3-1 | backend | Python process tree (FastAPI + RQ worker + scheduler + watchers) | Serve REST/WebSocket API and run background scans/syncs/jobs |
| c3-2 | frontend | Vue SPA served by Vite (dev) or static bundle (prod) | Browse, manage and play the ROM library in the browser |
