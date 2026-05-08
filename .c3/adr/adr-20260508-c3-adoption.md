---
id: adr-00000000-c3-adoption
c3-seal: f188e8c5698698fe7aad8e668613d993b33362a6945a5412400371804015c1ae
title: C3 Architecture Documentation Adoption
type: adr
goal: Adopt C3 architecture documentation under `.c3/` for the RomM repository so that future changes to the backend, frontend, metadata adapters and supporting infrastructure can be tracked top-down (system → containers → components) with explicit refs and rules instead of relying on tribal knowledge.
status: implemented
date: "2026-05-08"
affects:
    - c3-0
---

## Goal

Adopt C3 architecture documentation under `.c3/` for the RomM repository so that future changes to the backend, frontend, metadata adapters and supporting infrastructure can be tracked top-down (system → containers → components) with explicit refs and rules instead of relying on tribal knowledge.

## Context

RomM is a self-hosted ROM manager with a FastAPI backend (HTTP API + Socket.IO + RQ worker + scheduler + filesystem watchers), a Vue 3 / Vuetify SPA frontend, MariaDB/Postgres persistence, Valkey/Redis cache, and integrations with at least 12 external metadata or asset providers (IGDB, ScreenScraper, MobyGames, RetroAchievements, SteamGridDB, LaunchBox, Hasheous, HLTB, Libretro, Flashpoint, TGDB, Playmatch). The codebase has grown beyond what `README.md` and `DEVELOPER_SETUP.md` describe; there is no formal architecture documentation, and the project explicitly requires AI-assisted contributions to be disclosed in pull requests (`CONTRIBUTING.md`). Without C3 docs, every contributor — human or AI — has to rediscover the topology by reading source.

## Decision

Scaffold `.c3/` with two top-level containers — `c3-1 backend` and `c3-2 frontend` — and seed it with an initial set of foundation/feature components, one ref (`ref-fastapi-router`), and one rule (`rule-strong-typing`). Subsequent changes (new endpoints, new metadata adapters, new views) extend this skeleton through normal C3 ADRs rather than re-onboarding.

## Affected Topology

| Entity | Type | Why affected | Governance review |
| --- | --- | --- | --- |
| c3-0 | system | The system entity is created and given a goal/summary/abstract constraints | Reviewed during onboard |
| c3-1 | container | Backend container created with three seed components (config, auth, rom-endpoints) | Reviewed during onboard |
| c3-2 | container | Frontend container created with one seed component (router-views) | Reviewed during onboard |
| c3-101 | component | New component for hybrid auth and CSRF middleware | Reviewed during onboard |
| c3-102 | component | New component for env/YAML configuration | Reviewed during onboard |
| c3-110 | component | New feature component for the /api/roms HTTP surface | Reviewed during onboard |
| c3-201 | component | New component for the Vue Router views layer | Reviewed during onboard |

## Compliance Refs

| Ref | Why required | Action |
| --- | --- | --- |
| ref-fastapi-router | Backend endpoints follow a single router-mounting convention that the seed components must respect | create-ref |

## Compliance Rules

| Rule | Why required | Action |
| --- | --- | --- |
| rule-strong-typing | Backend (mypy) and frontend (vue-tsc) strong typing is a project-wide expectation that the seed components must comply with | create-rule |

## Work Breakdown

| Area | Detail | Evidence |
| --- | --- | --- |
| Scaffold | Run c3x init to create .c3/ and the bootstrap ADR | .c3/c3.db |
| System | Set c3-0 goal, summary and abstract constraints | c3x read c3-0 --full |
| Containers | Create c3-1 backend and c3-2 frontend with Goal/Components/Responsibilities | c3x read c3-1, c3x read c3-2 |
| Components | Create c3-101 auth, c3-102 config, c3-110 rom-endpoints, c3-201 router-views | c3x list |
| Refs/Rules | Create ref-fastapi-router and rule-strong-typing and wire them to citing components | c3x graph ref-fastapi-router |
| Code-map | Set codemap globs on every component, ref and rule and verify with c3x lookup | c3x lookup backend/endpoints/roms/upload.py |

## Underlay C3 Changes

| Underlay area | Exact C3 change | Verification evidence |
| --- | --- | --- |
| Commands | N.A - onboard does not change the c3x CLI; only data is added to .c3/c3.db | git status .c3/ |
| Validators | N.A - existing validators (c3x check, c3x check --include-adr) are exercised, not modified | c3x check |
| Hints/help | N.A - no help text changed | c3x --help |
| Templates | N.A - no template changes | Repository diff |

## Enforcement Surfaces

| Surface | Behavior | Evidence |
| --- | --- | --- |
| c3x check | Reports orphaned references, missing components, broken citations | c3x check |
| c3x check --include-adr | Reports ADR section gaps and ungrounded compliance rows | c3x check --include-adr |
| c3x lookup <file> | Maps files to their owning component plus any cited refs/rules | c3x lookup backend/endpoints/roms/upload.py |
| CONTRIBUTING.md | Project rule that AI assistance be disclosed; complements C3 review trails | CONTRIBUTING.md |

## Alternatives Considered

| Alternative | Rejected because |
| --- | --- |
| No formal architecture docs (status quo) | Topology was already opaque; future contributors keep paying the rediscovery tax |
| Free-form docs/architecture.md | No machine-checkable structure; drift goes unnoticed by c3x check |
| Generate docs from code only | Would describe what exists, not why; refs/rules need rationale that source code does not encode |
| One mega-container instead of backend + frontend | Hides the deployment boundary between the Python process tree and the Vue SPA, and the OpenAPI contract that crosses it |

## Risks

| Risk | Mitigation | Verification |
| --- | --- | --- |
| Seed inventory under-represents the codebase (only 4 components for ~30 endpoints + dozens of handlers) | Treat onboard as iterative; subsequent ADRs add more components and refs | c3x list shows the current coverage |
| Codemap globs go stale as files move | c3x lookup 'backend/**' and c3x lookup 'frontend/**' re-checked after large refactors | c3x lookup backend/endpoints/roms/upload.py |
| Refs and rules are too generic and become noise | Each ref must answer "why this over alternatives"; rules must cite real golden code | c3x read ref-fastapi-router --full, c3x read rule-strong-typing --full |

## Verification

| Check | Result |
| --- | --- |
| C3X_MODE=agent bash <c3x> list | 9 entities listed (1 system, 2 containers, 4 components, 1 ref, 1 rule) |
| C3X_MODE=agent bash <c3x> check | No issues |
| C3X_MODE=agent bash <c3x> lookup backend/endpoints/roms/upload.py | Resolves to c3-110 rom-endpoints plus ref-fastapi-router and rule-strong-typing |
| C3X_MODE=agent bash <c3x> lookup frontend/src/views/Home.vue | Resolves to c3-201 router-views plus rule-strong-typing |
