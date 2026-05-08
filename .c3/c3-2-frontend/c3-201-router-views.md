---
id: c3-201
c3-seal: 36ee0373d211da144d5a3e0eafbcedc2fb7e7bb172373eaa85091d754435b9b5
title: router-views
type: component
category: foundation
parent: c3-2
goal: Translate URL paths into top-level Vue views and apply auth and onboarding guards before each navigation, so every page in the SPA has a deterministic data and identity precondition.
uses:
    - rule-strong-typing
---

## Goal

Translate URL paths into top-level Vue views and apply auth and onboarding guards before each navigation, so every page in the SPA has a deterministic data and identity precondition.

## Parent Fit

| Field | Value |
| --- | --- |
| Container goal contribution | The shell that turns URLs into views and enforces page-level auth |
| Position in container | Foundation: every feature view depends on it |
| Upstream parents | c3-2 |
| Membership reason | Routing is global to the SPA; views and stores plug into it |

## Purpose

Owns the routing layer of the SPA. Maps `/`, `/login`, `/setup`, `/scan`, `/platform/:slug`, `/rom/:id`, `/play/:id`, `/patch`, `/pair`, `/404` to the views in `frontend/src/views/`. Provides global navigation guards that bounce unauthenticated users to `/login` and unconfigured installs to `/setup`. Non-goals: data fetching (delegated to Pinia stores), styling, page-internal navigation.

## Foundational Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Precondition | Pinia stores (auth, heartbeat, config) are instantiated before guards run | c3-2 |
| Input | Browser URL and history events received by vue-router | c3-2 |
| State | Active route reflected in the navigation Pinia store | c3-2 |
| Shared dependency | vue-router v4 declared in frontend/package.json | c3-2 |

## Business Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Outcome | User lands on the correct view with required data preconditions met | c3-2 |
| Primary path | / resolves to Home, /platform/:slug resolves to Gallery filtered by platform | c3-2 |
| Alternate path | First-run install redirects to Setup, OIDC callback handled at /auth | c3-2 |
| Failure | Unmatched route renders frontend/src/views/404.vue | c3-2 |

## Governance

| Reference | Type | Governs | Precedence | Notes |
| --- | --- | --- | --- | --- |
| rule-strong-typing | rule | Routes must declare typed RouteRecordRaw entries | normative | TypeScript-checked |

## Contract

| Surface | Direction | Contract | Boundary | Evidence |
| --- | --- | --- | --- | --- |
| router instance | OUT | Mounted on the Vue app at frontend/src/main.ts | TS import | frontend/src/main.ts |
| beforeEach guard | IN | Reads auth store and pushes to /login if unauthenticated | TS function | frontend/src/plugins/ |

## Change Safety

| Risk | Trigger | Detection | Required Verification |
| --- | --- | --- | --- |
| Adding a route without auth guard exposes data to anonymous users | Forgetting meta: { requiresAuth: true } | Code review | npm run typecheck |
| Renaming a path breaks deep links from external apps (Playnite, Argosy) | Editing route path | Native client smoke test | grep -r "/platform/" frontend/ |

## Derived Materials

| Material | Must derive from | Allowed variance | Evidence |
| --- | --- | --- | --- |
| Native client deep links (Playnite, Argosy) | Contract router row | Path changes require client coordination | README.md |
