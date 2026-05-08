---
id: c3-1
c3-seal: 8800bceeb99514d6c848026412908b7fbfb26837e74452298af032f551e9c003
title: backend
type: container
boundary: service
parent: c3-0
goal: Serve the RomM HTTP/WebSocket API, run background jobs that scan and enrich the library, persist data to the relational database, and integrate with external metadata providers.
---

## Goal

Serve the RomM HTTP/WebSocket API, run background jobs that scan and enrich the library, persist data to the relational database, and integrate with external metadata providers.

## Components

| ID | Name | Category | Status | Goal Contribution |
| --- | --- | --- | --- | --- |
| c3-102 | config | Foundation | active | Centralized environment + YAML config loader for every other component |
| c3-101 | auth | Foundation | active | Hybrid authentication (session, basic, OIDC) and CSRF middleware applied app-wide |
| c3-110 | rom-endpoints | Feature | active | REST surface for ROM browsing, upload, manual, notes, files, hashing |

## Responsibilities

- Boot the FastAPI app with CORS, CSRF, AuthenticationMiddleware, RedisSessionMiddleware and request-scoped context (`backend/main.py`).
- Mount Socket.IO namespaces for scan, sync and netplay (`backend/handler/socket_handler.py`).
- Run Alembic migrations on startup before serving traffic (`backend/main.py`).
- Schedule recurring jobs through `rq-scheduler` and dispatch ad-hoc jobs through `rq` queues `high`, `default`, `low` (`entrypoint.sh`).
- Watch the library and sync directories with `watchfiles` and trigger re-scans (`backend/watcher.py`, `backend/sync_watcher.py`).
- Fan out to metadata providers via `backend/adapters/services/` and cache responses in Redis/Valkey.
- Surface OpenAPI docs at `/api/docs` and `/api/redoc`.

## Complexity Assessment

Critical complexity. The container hosts five concurrent processes (API, scheduler, worker, library watcher, sync watcher) sharing the same code tree, talks to two databases (MariaDB/Postgres) and a cache (Valkey/Redis), and integrates with at least 12 external metadata or asset providers. State drift between filesystem and database is the dominant failure mode and is mitigated by scan reconciliation.
