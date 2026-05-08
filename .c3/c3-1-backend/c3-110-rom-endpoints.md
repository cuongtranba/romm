---
id: c3-110
c3-seal: cc36e07647bf5fa90876e4940b0fbfdefa7e7fe8ec0353a7c509be5ac3a35926
title: rom-endpoints
type: component
category: feature
parent: c3-1
goal: 'Expose the ROM resource over REST: list, retrieve, upload, download, patch metadata, manage notes, manuals and files, all backed by SQLAlchemy and the filesystem handler.'
uses:
    - ref-fastapi-router
    - rule-strong-typing
---

## Goal

Expose the ROM resource over REST: list, retrieve, upload, download, patch metadata, manage notes, manuals and files, all backed by SQLAlchemy and the filesystem handler.

## Parent Fit

| Field | Value |
| --- | --- |
| Container goal contribution | Primary read/write surface for ROMs, the central resource of the app |
| Position in container | Feature: depends on the config and auth foundation components |
| Upstream parents | c3-1 |
| Membership reason | ROM endpoints are HTTP request handlers, hence belong to the backend container |

## Purpose

Owns the `/api/roms` HTTP surface. Routes live under `backend/endpoints/roms/` (`__init__.py`, `files.py`, `manual.py`, `notes.py`, `upload.py`). Each route delegates to `db_rom_handler` for persistence and `fs_rom_handler` for filesystem access; uploads stream through `streaming-form-data` to avoid buffering large ROMs in memory. Non-goals: scanning the library (handled by the worker scan task), enriching metadata (handled by metadata handlers), serving the player (frontend).

## Foundational Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Precondition | Authenticated user with appropriate scope before the route body runs | c3-1 |
| Input | Path/query params, JSON bodies, and multipart uploads accepted at /api/roms | c3-1 |
| State | Rom rows in the DB plus files under ${ROMM_BASE_PATH}/library/roms/<platform> | c3-1 |
| Shared dependency | db_rom_handler, fs_rom_handler, meta_*_handler instantiated by handler packages | c3-1 |

## Business Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Outcome | Client can list/filter ROMs, fetch details, upload new ones, attach manuals/notes | c3-1 |
| Primary path | GET /api/roms resolves through db_rom_handler.get_roms() paginated via fastapi-pagination | c3-1 |
| Alternate path | POST /api/roms upload streams the ROM to disk and schedules hash + metadata refresh | c3-1 |
| Failure | Invalid platform or missing file raises typed endpoint_exceptions mapped to HTTP errors | c3-1 |

## Governance

| Reference | Type | Governs | Precedence | Notes |
| --- | --- | --- | --- | --- |
| ref-fastapi-router | ref | Routes must use APIRouter mounted from main.py | normative | All endpoints/* |
| rule-strong-typing | rule | Request/response models use Pydantic, no untyped dicts | normative | backend/endpoints/responses/ |

## Contract

| Surface | Direction | Contract | Boundary | Evidence |
| --- | --- | --- | --- | --- |
| GET /api/roms | IN | Returns paginated SimpleRomSchema list | HTTP | backend/endpoints/roms/__init__.py |
| POST /api/roms (multipart) | IN | Accepts streaming upload, returns the created RomSchema | HTTP | backend/endpoints/roms/upload.py |
| rom_router | OUT | Mounted under /api in main.py:134 | Python | backend/main.py |

## Change Safety

| Risk | Trigger | Detection | Required Verification |
| --- | --- | --- | --- |
| Schema break: changing a field on RomSchema ripples to the generated TS client | Editing backend/endpoints/responses/rom.py | Frontend vue-tsc failure after npm run generate | npm run generate && npm run typecheck |
| Upload stream not closed on early abort | Refactoring upload.py without reusing streaming-form-data parser | Manual smoke and tmp dir cleanup task | pytest backend/tests/endpoints/roms/ |

## Derived Materials

| Material | Must derive from | Allowed variance | Evidence |
| --- | --- | --- | --- |
| frontend/src/__generated__/services/RomsService.ts | Contract rom_router row | Field renames need npm run generate to regenerate | frontend/package.json |
