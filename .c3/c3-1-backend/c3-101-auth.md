---
id: c3-101
c3-seal: 21c75b009684009ab2fd9f22ca2ec0b19df2bae711ced470e112ac5fc934aedc
title: auth
type: component
category: foundation
parent: c3-1
goal: Authenticate every HTTP and WebSocket request through a hybrid backend that supports basic auth, session cookies, client tokens and OIDC, and protect state-mutating routes with CSRF.
uses:
    - ref-fastapi-router
    - rule-strong-typing
---

## Goal

Authenticate every HTTP and WebSocket request through a hybrid backend that supports basic auth, session cookies, client tokens and OIDC, and protect state-mutating routes with CSRF.

## Parent Fit

| Field | Value |
| --- | --- |
| Container goal contribution | Provides the Authorization and CSRF gate that all API endpoints depend on |
| Position in container | Foundation: middleware mounted in main.py before any router |
| Upstream parents | c3-1 |
| Membership reason | Auth is cross-cutting; every endpoint and socket runs through it |

## Purpose

Owns request-level identity. Provides `HybridAuthBackend` for Starlette's `AuthenticationMiddleware`, the `RedisSessionMiddleware` for cookie-based sessions, and the `CSRFMiddleware` for double-submit token enforcement. Hosts `auth.py` decorators used by endpoints to assert role, scope and ownership. Issues and validates client tokens for native clients (Argosy, Playnite, Grout). Non-goals: business authorization rules of specific resources, password reset UX, OIDC IdP configuration.

## Foundational Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Precondition | ROMM_AUTH_SECRET_KEY set and Redis reachable before middleware initializes | c3-1 |
| Input | Cookies, Authorization headers, and OIDC redirect parameters arriving on the HTTP layer | c3-1 |
| State | Sessions stored in Redis keyed by cookie; users persisted in the relational DB | c3-1 |
| Shared dependency | authlib, passlib[bcrypt], joserfc, itsdangerous declared in pyproject.toml | c3-1 |

## Business Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Outcome | A request.user is populated for every protected route, or 401 returned | c3-1 |
| Primary path | Cookie session resolves through Redis lookup and loads the user from the DB | c3-1 |
| Alternate path | OIDC sign-in via Authentik or generic provider, basic auth, or client-token exchange | c3-1 |
| Failure | Invalid token returns 401; CSRF mismatch on POST/PUT/DELETE returns 403 | c3-1 |

## Governance

| Reference | Type | Governs | Precedence | Notes |
| --- | --- | --- | --- | --- |
| ref-fastapi-router | ref | Endpoint-level auth decorators must wrap APIRouter routes | normative | All endpoints/*.py |
| rule-strong-typing | rule | Auth helpers must avoid Any/untyped dicts | normative | mypy enforced |

## Contract

| Surface | Direction | Contract | Boundary | Evidence |
| --- | --- | --- | --- | --- |
| HybridAuthBackend | IN | Resolves a Starlette AuthCredentials + BaseUser from the incoming request | HTTP/WebSocket | backend/main.py:113 |
| CSRFMiddleware | IN | Rejects state-mutating requests without a matching romm_csrftoken cookie/header | HTTP | backend/main.py:97-108 |
| decorators/auth.py | IN/OUT | Per-endpoint role/scope assertions raise AuthExceptions on failure | Python | backend/decorators/auth.py |

## Change Safety

| Risk | Trigger | Detection | Required Verification |
| --- | --- | --- | --- |
| Bypass: removing a decorator silently exposes a sensitive endpoint | Editing endpoints/*.py without re-applying @protected_route | Code review and integration tests | pytest backend/tests/endpoints/ |
| CSRF cookie / header mismatch breaks all writes | Changing cookie_name or rotating secret | Smoke test against /api/roms POST | pytest backend/tests/handler/auth/ |

## Derived Materials

| Material | Must derive from | Allowed variance | Evidence |
| --- | --- | --- | --- |
| Auth env vars in env.template | Foundational Flow Precondition row | New auth flag must appear in template before being honored | env.template |
