---
id: c3-2
c3-seal: 944dd955abf1cd153481b34b9ad9b9f52add5b4af9e80183d16fd18ec368c29c
title: frontend
type: container
boundary: service
parent: c3-0
goal: 'Render the RomM web UI: browse the library, edit metadata, upload ROMs, configure the server, and play games directly in the browser via EmulatorJS / Ruffle.'
---

## Goal

Render the RomM web UI: browse the library, edit metadata, upload ROMs, configure the server, and play games directly in the browser via EmulatorJS / Ruffle.

## Components

| ID | Name | Category | Status | Goal Contribution |
| --- | --- | --- | --- | --- |
| c3-201 | router-views | Foundation | active | Vue Router page layer mapping URLs to top-level views (Gallery, GameDetails, Player, Auth, Scan) |

## Responsibilities

- Boot the Vue 3 + Vuetify + Pinia + Tailwind app from `frontend/src/main.ts` and `frontend/src/RomM.vue`.
- Generate a typed API client from the backend OpenAPI schema into `frontend/src/__generated__/` (`npm run generate`).
- Talk to the backend over `axios` (`frontend/src/services/api/`) and `socket.io-client` (`frontend/src/services/socket.ts`).
- Serve static assets and the dev server via Vite (HTTPS, mkcert) — see `frontend/vite.config.js`.
- Provide internationalization through `vue-i18n` (`frontend/src/locales/`).
- Embed third-party players: `rom-patcher` for patching, EmulatorJS / Ruffle for in-browser play.

## Complexity Assessment

Moderate complexity. The SPA is mostly view-layer logic; the heavy lifting (scanning, metadata, persistence) is on the backend. Notable risk areas are auth state synchronization (cookies, OIDC redirects, CSRF) and large gallery rendering performance.
