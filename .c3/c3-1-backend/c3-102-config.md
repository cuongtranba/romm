---
id: c3-102
c3-seal: 230676e9dcd048e24a67a819103ff1760bd463793a38abaa184db7098f64c2cb
title: config
type: component
category: foundation
parent: c3-1
goal: Centralize environment-variable parsing and YAML configuration loading so every other backend component reads settings from a single typed module instead of re-parsing env on demand.
uses:
    - rule-strong-typing
---

## Goal

Centralize environment-variable parsing and YAML configuration loading so every other backend component reads settings from a single typed module instead of re-parsing env on demand.

## Parent Fit

| Field | Value |
| --- | --- |
| Container goal contribution | Provides the typed config every other component imports from config and config.config_manager |
| Position in container | Foundation: imported by main.py, startup.py, every handler and task |
| Upstream parents | c3-1 |
| Membership reason | Configuration is shared infrastructure for the entire backend container |

## Purpose

Owns runtime configuration. Reads `.env` via `python-dotenv`, exposes constants such as `OIDC_ENABLED`, `IS_PYTEST_RUN`, `SENTRY_DSN`, `ROMM_AUTH_SECRET_KEY`, `DEV_HOST`, `DEV_PORT`, `DISABLE_CSRF_PROTECTION`, scheduled-task feature flags, and database/Redis URLs. Loads user-provided YAML through `config_manager`. Non-goals: validating business data, persisting state, talking to the network — it is read-only configuration.

## Foundational Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Precondition | .env file present or env vars exported in the process before import | c3-1 |
| Input | Environment variables read by backend/config/__init__.py and YAML at config.yml | c3-1 |
| State | Module-level constants and singleton config_manager exposed from backend/config/ | c3-1 |
| Shared dependency | python-dotenv and pyyaml declared in pyproject.toml | c3-1 |

## Business Flow

| Aspect | Detail | Reference |
| --- | --- | --- |
| Outcome | Backend boots with a coherent set of feature flags and credentials available to every importer | c3-1 |
| Primary path | Module import triggers env parse, exposing typed constants for the rest of the app | c3-1 |
| Alternate path | YAML overrides applied through config_manager for user-tunable settings such as platform aliases | c3-1 |
| Failure | Missing required env (e.g. DB credentials) raises at first use, not at import | c3-1 |

## Governance

| Reference | Type | Governs | Precedence | Notes |
| --- | --- | --- | --- | --- |
| rule-strong-typing | rule | All exported config values must have explicit types | normative | Enforced by mypy |

## Contract

| Surface | Direction | Contract | Boundary | Evidence |
| --- | --- | --- | --- | --- |
| from config import ... | OUT | Module exposes typed constants for env-driven settings | Python import | backend/main.py:19-27 |
| config_manager singleton | OUT | Returns parsed YAML config object | Python import | backend/config/config_manager.py |

## Change Safety

| Risk | Trigger | Detection | Required Verification |
| --- | --- | --- | --- |
| Renaming a constant breaks every importer | Removing or renaming an exported config name | Backend fails to import on boot | mypy backend/config/ |
| Default value drift between code and env.template | Changing a default without updating the template | Manual review of env.template | diff env.template backend/config/__init__.py |

## Derived Materials

| Material | Must derive from | Allowed variance | Evidence |
| --- | --- | --- | --- |
| env.template | Foundational Flow Input row | New var must appear in template before code reads it | env.template |
