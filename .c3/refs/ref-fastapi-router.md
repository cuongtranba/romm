---
id: ref-fastapi-router
c3-seal: 71adbafce02783946740317fb2012d60a84b038e8b30243e73d571532bc976b4
title: fastapi-router
type: ref
goal: Standardize how HTTP and Socket.IO endpoints are organized in the backend so that every feature exposes its routes through a single, mountable `APIRouter` (or Socket.IO namespace), letting `main.py` compose the application without importing endpoint internals.
---

## Goal

Standardize how HTTP and Socket.IO endpoints are organized in the backend so that every feature exposes its routes through a single, mountable `APIRouter` (or Socket.IO namespace), letting `main.py` compose the application without importing endpoint internals.

## Choice

Each `backend/endpoints/<feature>.py` (or `<feature>/__init__.py`) defines `router = APIRouter(prefix="/<feature>", tags=["<feature>"])`, decorates handlers with `@router.<method>`, and is mounted from `backend/main.py` under the global `/api` prefix via `app.include_router(<feature>_router, prefix="/api")`.

## Why

- Lets `main.py` enumerate every feature exactly once — adding a new endpoint module is a single import + `include_router` line, which makes the surface auditable and reviewable in one file.
- Lets FastAPI generate one consistent OpenAPI schema, which the frontend consumes via `npm run generate` to produce `frontend/src/__generated__/`. Decentralized routing would split the schema and break the typed client.
- Lets `decorators/auth.py` wrap routes per-feature instead of globally, so feature owners control the auth scope of their endpoints without touching middleware.
- Alternatives — registering routes via decorators imported into `main.py`, or using `fastapi.APIRoute` factories — were rejected because they hide the routing graph and break the OpenAPI generator's tag boundaries.

## How

Real example (`backend/endpoints/heartbeat.py` style, used throughout `backend/endpoints/`):

```python
# REQUIRED: feature-local router with prefix and tag
from fastapi import APIRouter

router = APIRouter(prefix="/heartbeat", tags=["heartbeat"])


# REQUIRED: route decorator on the local router
@router.get("")
async def heartbeat() -> dict[str, str]:
    return {"status": "ok"}
```

Mounted in `backend/main.py:127`:

```python
# REQUIRED: import as <feature>_router and include with /api prefix
from endpoints.heartbeat import router as heartbeat_router

app.include_router(heartbeat_router, prefix="/api")
```

OPTIONAL elements: dependency injection via `Depends`, response model declarations, per-route auth decorators from `backend/decorators/auth.py`.
